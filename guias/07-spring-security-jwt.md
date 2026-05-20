# Guía 07 — Seguridad: Spring Security 6 + JWT desde Cero

> **Materia:** Programación Avanzada | **Núcleo temático:** 3 - Construcción de Servicios de Negocio  
> **Proyecto base:** Sistema de Triage y Gestión de Solicitudes Académicas  
> **Stack:** Java 21 · Spring Boot 3.4+ · Spring Security 6 · jjwt 0.12+  
> **Prerrequisitos:** Guías 04, 05 y 06 completadas (Spring Boot, JPA y API REST funcionando)

---

## Tabla de Contenidos

1. [¿Por qué necesitas seguridad?](#1-por-qué-necesitas-seguridad)
2. [Autenticación vs Autorización](#2-autenticación-vs-autorización)
3. [Spring Security 6: La nueva arquitectura](#3-spring-security-6-la-nueva-arquitectura)
4. [Antes vs Ahora: SecurityFilterChain vs WebSecurityConfigurerAdapter](#4-antes-vs-ahora)
5. [Estructura de carpetas del módulo de seguridad](#5-estructura-de-carpetas)
6. [Dependencias necesarias](#6-dependencias-necesarias)
7. [Modelo de usuario: entidades Usuario y Rol](#7-modelo-de-usuario)
8. [UserDetails y UserDetailsService](#8-userdetails-y-userdetailsservice)
9. [Configurar Spring Security desde cero](#9-configurar-spring-security-desde-cero)
10. [Flujo de registro de usuario](#10-flujo-de-registro-de-usuario)
11. [JWT: qué es y cómo funciona](#11-jwt-qué-es-y-cómo-funciona)
12. [Implementar JwtService](#12-implementar-jwtservice)
13. [Implementar JwtAuthenticationFilter](#13-implementar-jwtauthenticationfilter)
14. [Endpoint de login](#14-endpoint-de-login)
15. [Autorización por roles con @PreAuthorize](#15-autorización-por-roles-con-preauthorize)
16. [Configuración CORS para Angular](#16-configuración-cors-para-angular)
17. [Refresh Tokens: concepto e implementación básica](#17-refresh-tokens)
18. [Flujo completo de prueba en Postman](#18-flujo-completo-en-postman)
19. [Errores comunes y Troubleshooting](#19-errores-comunes-y-troubleshooting)
20. [Resumen y Cheat Sheet](#20-resumen-y-cheat-sheet)
21. [Ejercicios prácticos](#21-ejercicios-prácticos)
22. [Referencias](#22-referencias)

---

## 1. ¿Por qué necesitas seguridad?

Imagina que construiste toda la API REST del Triage: registrar solicitudes, cambiar estados, asignar responsables. Está funcionando perfectamente. Luego lanzas la URL en producción y... cualquier persona del planeta puede hacer `GET /api/v1/solicitudes` y ver todas las solicitudes de los estudiantes, incluyendo datos personales. Cualquiera puede hacer `DELETE /api/v1/solicitudes/42` y borrar una solicitud. Cualquiera puede hacer `POST /api/v1/solicitudes/{id}/cerrar` y cerrar solicitudes sin ser funcionario.

Eso es exactamente el problema que resuelve la seguridad en una API.

En el mundo real, toda aplicación empresarial necesita responder estas dos preguntas antes de procesar cualquier petición:

1. **¿Quién eres?** → *Autenticación*
2. **¿Qué puedes hacer?** → *Autorización*

Sin seguridad, tu API es como un edificio sin puertas: cualquiera entra, toca lo que quiere y se va. Con Spring Security y JWT, cada puerta tiene llave y cada persona solo puede entrar a las habitaciones para las que tiene permiso.

---

## 2. Autenticación vs Autorización

Estos dos conceptos se confunden mucho. Vamos a dejarlos muy claros con un ejemplo de la vida real del campus universitario:

```
🏛️ ANALOGÍA: El sistema de la Universidad

AUTENTICACIÓN: Mostrar tu carné universitario en la entrada.
- El guardia verifica que eres quien dices ser.
- Verifica que el carné es válido y no está vencido.
- No sabe todavía qué puedes hacer, solo QUIÉN eres.

AUTORIZACIÓN: Una vez dentro, ¿a qué salones puedes entrar?
- Los estudiantes pueden entrar a las aulas pero no a la sala de profesores.
- Los profesores pueden entrar a su oficina y a las aulas.
- El rector puede entrar a todas partes.
```

Traducido al Sistema de Triage:

| Concepto        | Pregunta                        | Ejemplo en el Triage                                              |
|-----------------|---------------------------------|-------------------------------------------------------------------|
| Autenticación   | ¿Eres realmente tú?             | El usuario envía email + contraseña → el sistema verifica         |
| Autorización    | ¿Tienes permiso para esto?      | Solo un FUNCIONARIO puede clasificar una solicitud                |

### Los tres roles del Triage

En nuestro sistema tenemos tres roles definidos:

```
ESTUDIANTE   → Puede registrar solicitudes, consultar SUS solicitudes, ver el historial
FUNCIONARIO  → Puede clasificar, priorizar, asignar responsable, cambiar estados, cerrar
ADMIN        → Puede todo lo anterior + gestionar usuarios, ver reportes globales
```

---

## 3. Spring Security 6: La nueva arquitectura

Spring Security es el módulo de seguridad del ecosistema Spring. Lleva años existiendo pero sufrió una renovación muy importante en la versión 6 (que viene con Spring Boot 3.x).

**¿Cómo funciona Spring Security internamente?**

Cuando una petición HTTP llega a tu aplicación Spring Boot, NO llega directamente al `@RestController`. Primero pasa por una cadena de filtros (Filter Chain). Puedes imaginarlo como una fila de revisiones en el aeropuerto:

```
Petición HTTP entrante
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│                   FILTER CHAIN                          │
│                                                         │
│  ┌───────────────────────────────────┐                  │
│  │  SecurityContextPersistenceFilter  │  ← Carga contexto│
│  └──────────────────┬────────────────┘                  │
│                     │                                   │
│  ┌──────────────────▼────────────────┐                  │
│  │    JwtAuthenticationFilter        │  ← TÚ lo creates │
│  │  (extrae y valida el JWT)         │                  │
│  └──────────────────┬────────────────┘                  │
│                     │                                   │
│  ┌──────────────────▼────────────────┐                  │
│  │  ExceptionTranslationFilter        │  ← 401 / 403    │
│  └──────────────────┬────────────────┘                  │
│                     │                                   │
│  ┌──────────────────▼────────────────┐                  │
│  │  FilterSecurityInterceptor         │  ← Autorización │
│  └──────────────────┬────────────────┘                  │
└─────────────────────┼───────────────────────────────────┘
                      │
                      ▼
            DispatcherServlet → @RestController
```

**El SecurityContext:**  
Cuando un filtro valida exitosamente al usuario, guarda su información en el `SecurityContextHolder`. Ese contenedor está disponible durante toda la petición. Al terminar la petición, se limpia automáticamente.

```
SecurityContextHolder
    └── SecurityContext
            └── Authentication
                    ├── Principal  (el objeto UserDetails del usuario)
                    ├── Credentials (la contraseña, generalmente null después de auth)
                    └── Authorities (los roles/permisos)
```

---

## 4. Antes vs Ahora

Esta es una de las diferencias más importantes que debes conocer. Si buscas tutoriales viejos en internet (anteriores a 2022), verás el **enfoque antiguo** que ya está **eliminado** en Spring Security 6.

### ❌ Forma ANTIGUA — WebSecurityConfigurerAdapter (ELIMINADO en Spring Security 6)

```java
// ⚠️ ESTE CÓDIGO YA NO FUNCIONA EN SPRING BOOT 3.x — NO LO USES

@Configuration
@EnableWebSecurity
public class SecurityConfigAntigua extends WebSecurityConfigurerAdapter {  // ← ELIMINADO

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeRequests()                    // ← también deprecado
                .antMatchers("/api/auth/**").permitAll()
                .antMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            .and()
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS);
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService)
            .passwordEncoder(passwordEncoder());
    }
}
```

### ✅ Forma MODERNA — SecurityFilterChain (Spring Security 6 / Spring Boot 3.4+)

```java
// ✅ ESTE ES EL ENFOQUE CORRECTO PARA 2026

@Configuration
@EnableWebSecurity
@EnableMethodSecurity  // Habilita @PreAuthorize, @PostAuthorize
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(AbstractHttpConfigurer::disable)
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                .anyRequest().authenticated()
            )
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }
}
```

### Tabla comparativa completa

| Característica                | Antes (≤ Spring Boot 2.x)             | Ahora (Spring Boot 3.x / Spring Security 6) |
|-------------------------------|---------------------------------------|----------------------------------------------|
| Clase base                    | `extends WebSecurityConfigurerAdapter`| Bean `SecurityFilterChain` sin herencia      |
| Método de configuración       | `configure(HttpSecurity http)`        | Lambdas directas en `securityFilterChain()`  |
| Autorización de requests      | `.authorizeRequests().antMatchers()`  | `.authorizeHttpRequests().requestMatchers()`  |
| Patrones de URL               | `AntMatcher` con `antMatchers()`      | `requestMatchers()` (más flexible)           |
| CSRF                          | `.csrf().disable()`                   | `.csrf(AbstractHttpConfigurer::disable)`      |
| Sesiones                      | `.sessionManagement().sessionCreationPolicy()` | Idem pero con lambda                |
| Habilitación de @PreAuthorize | `@EnableGlobalMethodSecurity(prePostEnabled = true)` | `@EnableMethodSecurity`        |
| AuthenticationManager         | `configure(AuthenticationManagerBuilder)` | `@Bean AuthenticationManager`           |

---

## 5. Estructura de carpetas

Antes de escribir una sola línea de código, veamos dónde va todo. Partimos de la estructura que construimos en guías anteriores y agregamos el módulo de seguridad:

```
triage-backend/
└── src/
    └── main/
        └── java/
            └── co/
                └── uniquindio/
                    └── triage/
                        ├── TriageApplication.java
                        │
                        ├── config/
                        │   ├── SecurityConfig.java          ← Configuración central de seguridad
                        │   ├── ApplicationConfig.java       ← Beans auxiliares (PasswordEncoder, etc.)
                        │   └── OpenApiConfig.java           ← Swagger (de guía anterior)
                        │
                        ├── security/
                        │   ├── filter/
                        │   │   └── JwtAuthenticationFilter.java  ← Intercepta cada petición
                        │   ├── service/
                        │   │   ├── JwtService.java               ← Genera y valida JWT
                        │   │   └── UserDetailsServiceImpl.java   ← Carga usuario desde BD
                        │   └── dto/
                        │       ├── LoginRequest.java             ← Record para el login
                        │       ├── LoginResponse.java            ← Record con el token
                        │       ├── RegisterRequest.java          ← Record para el registro
                        │       └── TokenRefreshRequest.java      ← Para renovar tokens
                        │
                        ├── auth/
                        │   ├── AuthController.java          ← POST /auth/login, /auth/register
                        │   └── AuthService.java             ← Lógica de autenticación
                        │
                        ├── domain/
                        │   ├── model/
                        │   │   ├── Usuario.java             ← Entidad de usuario (nueva)
                        │   │   ├── Rol.java                 ← Enum de roles
                        │   │   ├── RefreshToken.java        ← Para renovación de tokens
                        │   │   ├── Solicitud.java           ← (ya existía)
                        │   │   └── ...
                        │   └── repository/
                        │       ├── UsuarioRepository.java
                        │       ├── RefreshTokenRepository.java
                        │       └── ...
                        │
                        └── solicitud/
                            ├── SolicitudController.java    ← (ya existía, ahora con @PreAuthorize)
                            └── ...
```

---

## 6. Dependencias necesarias

Abre tu `pom.xml` y agrega lo siguiente. Si ya tienes spring-boot-starter-web y spring-boot-starter-data-jpa de las guías anteriores, solo necesitas añadir lo marcado como nuevo:

```xml
<dependencies>
    <!-- Ya deberías tener estas de guías anteriores -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <!-- ✅ NUEVO: Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- ✅ NUEVO: JWT con jjwt (versión más reciente 2026) -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.12.6</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.12.6</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
        <version>0.12.6</version>
        <scope>runtime</scope>
    </dependency>

    <!-- PostgreSQL y Flyway (ya deberías tenerlos) -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.flywaydb</groupId>
        <artifactId>flyway-core</artifactId>
    </dependency>
    <dependency>
        <groupId>org.flywaydb</groupId>
        <artifactId>flyway-database-postgresql</artifactId>
    </dependency>

    <!-- Lombok para reducir boilerplate -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

> ⚠️ **Importante:** En cuanto agregas `spring-boot-starter-security` al proyecto, Spring Security protege **todos** los endpoints automáticamente con una pantalla de login HTML. Hasta que configures tu propio `SecurityFilterChain`, ningún endpoint va a funcionar. Eso es intencional: la política por defecto es "todo está bloqueado".

---

## 7. Modelo de usuario

### 7.1. El enum Rol

En el Triage tenemos tres roles. Lo modelamos como un `enum` de Java:

```java
// src/main/java/co/uniquindio/triage/domain/model/Rol.java

package co.uniquindio.triage.domain.model;

/**
 * Roles del sistema de Triage.
 * ESTUDIANTE: registra y consulta sus propias solicitudes.
 * FUNCIONARIO: atiende, clasifica y cierra solicitudes.
 * ADMIN: gestión global del sistema.
 */
public enum Rol {
    ESTUDIANTE,
    FUNCIONARIO,
    ADMIN
}
```

### 7.2. La entidad Usuario

Esta entidad implementa `UserDetails`, que es la interfaz que Spring Security necesita para entender qué es un "usuario". Piénsalo así: Spring Security no sabe nada de tu base de datos. Él solo entiende objetos `UserDetails`. Por eso tienes que decirle: "mi clase `Usuario` ES un `UserDetails`".

```java
// src/main/java/co/uniquindio/triage/domain/model/Usuario.java

package co.uniquindio.triage.domain.model;

import jakarta.persistence.*;
import lombok.*;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.time.LocalDateTime;
import java.util.Collection;
import java.util.List;

@Entity
@Table(
    name = "usuarios",
    uniqueConstraints = {
        @UniqueConstraint(columnNames = "email", name = "uk_usuarios_email"),
        @UniqueConstraint(columnNames = "codigo_estudiante", name = "uk_usuarios_codigo")
    }
)
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Usuario implements UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String nombre;

    @Column(nullable = false, length = 100)
    private String apellido;

    @Column(nullable = false, length = 150)
    private String email;

    @Column(nullable = false)
    private String password;   // almacena el hash BCrypt, nunca texto plano

    @Column(name = "codigo_estudiante", length = 20)
    private String codigoEstudiante;  // solo aplica para estudiantes

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private Rol rol;

    @Column(nullable = false)
    @Builder.Default
    private boolean activo = true;

    @Column(name = "fecha_registro", nullable = false, updatable = false)
    @Builder.Default
    private LocalDateTime fechaRegistro = LocalDateTime.now();

    // ─────────────────────────────────────────────────────────────
    // Métodos de UserDetails — Spring Security los usa internamente
    // ─────────────────────────────────────────────────────────────

    /**
     * Devuelve los "permisos" del usuario. Spring Security los llama GrantedAuthority.
     * Nosotros los mapeamos desde nuestro enum Rol.
     * El prefijo "ROLE_" es una convención de Spring Security para hasRole().
     */
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(new SimpleGrantedAuthority("ROLE_" + rol.name()));
    }

    /**
     * Spring Security usa getUsername() como identificador único.
     * Nosotros usamos el email como username.
     */
    @Override
    public String getUsername() {
        return email;
    }

    @Override
    public String getPassword() {
        return password;
    }

    /**
     * Los siguientes métodos controlan el estado de la cuenta.
     * Los delegamos a nuestro campo "activo".
     */
    @Override
    public boolean isAccountNonExpired() { return true; }

    @Override
    public boolean isAccountNonLocked() { return activo; }

    @Override
    public boolean isCredentialsNonExpired() { return true; }

    @Override
    public boolean isEnabled() { return activo; }
}
```

**Explicación línea por línea de las partes más importantes:**

- `implements UserDetails`: le dice a Spring Security que esta clase representa a un usuario del sistema.
- `getAuthorities()`: es el método más importante. Spring Security pregunta aquí qué puede hacer este usuario. Devolvemos `ROLE_ESTUDIANTE`, `ROLE_FUNCIONARIO` o `ROLE_ADMIN` según el rol.
- `getUsername()`: aunque se llama "username", nosotros usamos el email como identificador.
- `isAccountNonLocked()`: si un admin desactiva un usuario (`activo = false`), Spring Security rechaza su login automáticamente.

### 7.3. Migration de Flyway para la tabla usuarios

Siguiendo las buenas prácticas de la Guía 05, creamos el script de migración:

```sql
-- src/main/resources/db/migration/V3__create_tabla_usuarios.sql

CREATE TABLE usuarios (
    id                BIGSERIAL PRIMARY KEY,
    nombre            VARCHAR(100) NOT NULL,
    apellido          VARCHAR(100) NOT NULL,
    email             VARCHAR(150) NOT NULL,
    password          VARCHAR(255) NOT NULL,
    codigo_estudiante VARCHAR(20),
    rol               VARCHAR(20)  NOT NULL DEFAULT 'ESTUDIANTE',
    activo            BOOLEAN      NOT NULL DEFAULT TRUE,
    fecha_registro    TIMESTAMP    NOT NULL DEFAULT NOW(),

    CONSTRAINT uk_usuarios_email   UNIQUE (email),
    CONSTRAINT uk_usuarios_codigo  UNIQUE (codigo_estudiante),
    CONSTRAINT ck_usuarios_rol     CHECK  (rol IN ('ESTUDIANTE', 'FUNCIONARIO', 'ADMIN'))
);

-- Insertar un admin inicial para poder hacer login desde el primer momento
-- La contraseña 'Admin2026!' hasheada con BCrypt se ve así:
INSERT INTO usuarios (nombre, apellido, email, password, rol)
VALUES (
    'Admin',
    'Sistema',
    'admin@uniquindio.edu.co',
    '$2a$12$RlV3SiUGVtPxEPi8eTlAFuJc1qIhv7Yn4vu0AZ5MQKA0aqHFKUzMu',
    'ADMIN'
);
```

### 7.4. El repositorio de Usuario

```java
// src/main/java/co/uniquindio/triage/domain/repository/UsuarioRepository.java

package co.uniquindio.triage.domain.repository;

import co.uniquindio.triage.domain.model.Usuario;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface UsuarioRepository extends JpaRepository<Usuario, Long> {

    /**
     * Busca un usuario por email. Lo usaremos en UserDetailsService para el login.
     */
    Optional<Usuario> findByEmail(String email);

    /**
     * Verifica si ya existe un usuario con ese email (para el registro).
     */
    boolean existsByEmail(String email);

    /**
     * Verifica si ya existe un usuario con ese código estudiantil.
     */
    boolean existsByCodigoEstudiante(String codigoEstudiante);
}
```

---

## 8. UserDetails y UserDetailsService

Spring Security necesita saber cómo cargar un usuario desde tu fuente de datos (en nuestro caso, la base de datos). Para eso existe `UserDetailsService`. Es una interfaz con un solo método: `loadUserByUsername(String username)`.

```
ANALOGÍA: El portero del edificio

Cuando alguien llega al edificio y dice "soy Juan García",
el portero consulta su libreta de registros para verificar si Juan García
realmente vive ahí y cuáles son sus permisos.

UserDetailsService es esa "libreta de registros".
loadUserByUsername() es el acto de consultarla.
```

```java
// src/main/java/co/uniquindio/triage/security/service/UserDetailsServiceImpl.java

package co.uniquindio.triage.security.service;

import co.uniquindio.triage.domain.repository.UsuarioRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
public class UserDetailsServiceImpl implements UserDetailsService {

    private final UsuarioRepository usuarioRepository;

    /**
     * Spring Security llama a este método con el email del usuario
     * cuando necesita autenticarlo. Buscamos el usuario en la BD.
     * Si no lo encontramos, lanzamos UsernameNotFoundException
     * (Spring Security la captura y devuelve 401 automáticamente).
     */
    @Override
    @Transactional(readOnly = true)
    public UserDetails loadUserByUsername(String email)
            throws UsernameNotFoundException {

        return usuarioRepository.findByEmail(email)
            .orElseThrow(() ->
                new UsernameNotFoundException(
                    "Usuario no encontrado con email: " + email
                )
            );
        // Nuestro Usuario ya implementa UserDetails,
        // así que lo podemos devolver directamente sin conversión.
    }
}
```

---

## 9. Configurar Spring Security desde cero

Ahora vamos a crear la configuración central. La dividimos en dos archivos para mantener el código organizado:

### 9.1. ApplicationConfig — Los Beans auxiliares

```java
// src/main/java/co/uniquindio/triage/config/ApplicationConfig.java

package co.uniquindio.triage.config;

import co.uniquindio.triage.security.service.UserDetailsServiceImpl;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
@RequiredArgsConstructor
public class ApplicationConfig {

    private final UserDetailsServiceImpl userDetailsService;

    /**
     * PasswordEncoder: responsable de hashear y verificar contraseñas.
     * BCrypt es el estándar recomendado en 2026. El número 12 es el "strength"
     * (cuántas rondas de hashing aplica). Más alto = más seguro pero más lento.
     * 10-12 es el rango recomendado para producción.
     *
     * NUNCA guardes contraseñas en texto plano. BCrypt las hashea así:
     * "mi-contraseña" → "$2a$12$abc123..." (siempre diferente por el salt)
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }

    /**
     * AuthenticationProvider: el componente que realiza la autenticación.
     * DaoAuthenticationProvider usa tu UserDetailsService para cargar el usuario
     * y tu PasswordEncoder para verificar la contraseña.
     */
    @Bean
    public AuthenticationProvider authenticationProvider() {
        var provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

    /**
     * AuthenticationManager: el orquestador de la autenticación.
     * Lo exponemos como Bean para poder inyectarlo en AuthService
     * y usarlo al autenticar en el endpoint de login.
     */
    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
}
```

### 9.2. SecurityConfig — La configuración principal

```java
// src/main/java/co/uniquindio/triage/config/SecurityConfig.java

package co.uniquindio.triage.config;

import co.uniquindio.triage.security.filter.JwtAuthenticationFilter;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import java.util.List;

@Configuration
@EnableWebSecurity          // Activa Spring Security
@EnableMethodSecurity       // Activa @PreAuthorize en los @RestControllers
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtAuthFilter;
    private final AuthenticationProvider authenticationProvider;

    /**
     * Lista blanca de URLs que NO requieren autenticación.
     * Cualquier URL que no esté aquí requiere un JWT válido.
     */
    private static final String[] PUBLIC_URLS = {
        "/api/v1/auth/**",          // login y registro
        "/swagger-ui/**",           // documentación API
        "/swagger-ui.html",
        "/v3/api-docs/**",
        "/actuator/health"          // health check para monitoring
    };

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
            // 1. Desactivar CSRF: no lo necesitamos con JWT (API stateless)
            .csrf(AbstractHttpConfigurer::disable)

            // 2. Configurar CORS (para que Angular pueda consumir la API)
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))

            // 3. Reglas de autorización por URL
            .authorizeHttpRequests(auth -> auth
                .requestMatchers(PUBLIC_URLS).permitAll()   // estas URLs son libres
                .anyRequest().authenticated()               // todo lo demás requiere JWT
            )

            // 4. Sesiones STATELESS: Spring Security NO guardará sesiones en el servidor.
            //    Cada petición debe traer su JWT. Esto es fundamental para APIs REST.
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )

            // 5. Registrar nuestro proveedor de autenticación
            .authenticationProvider(authenticationProvider)

            // 6. Agregar nuestro filtro JWT ANTES del filtro de autenticación estándar
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)

            .build();
    }

    /**
     * Configuración CORS: qué orígenes pueden hacer peticiones a nuestra API.
     * En desarrollo, Angular corre en http://localhost:4200.
     * En producción, cambia esto por la URL real del frontend.
     *
     * Profundizamos en CORS en la sección 16.
     */
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        var config = new CorsConfiguration();
        config.setAllowedOrigins(List.of(
            "http://localhost:4200",    // Angular en desarrollo
            "https://triage.uniquindio.edu.co"  // Frontend en producción
        ));
        config.setAllowedMethods(List.of("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"));
        config.setAllowedHeaders(List.of("Authorization", "Content-Type", "Accept"));
        config.setExposedHeaders(List.of("Authorization"));
        config.setAllowCredentials(true);
        config.setMaxAge(3600L);

        var source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", config);
        return source;
    }
}
```

---

## 10. Flujo de registro de usuario

Antes de que alguien pueda hacer login, debe estar registrado. El flujo es:

```
1. Cliente envía: { nombre, apellido, email, password, codigoEstudiante? }
2. Validar que el email no exista ya
3. Hashear la contraseña con BCrypt
4. Guardar el usuario en la BD
5. Generar un JWT (opcional: algunos sistemas devuelven el token en el registro)
6. Responder con el token
```

### 10.1. DTOs con Records de Java 21

```java
// src/main/java/co/uniquindio/triage/security/dto/RegisterRequest.java

package co.uniquindio.triage.security.dto;

import jakarta.validation.constraints.*;

/**
 * DTO de entrada para el registro. Usamos Record de Java 21:
 * inmutable, conciso, con equals/hashCode/toString automáticos.
 */
public record RegisterRequest(
    @NotBlank(message = "El nombre es obligatorio")
    @Size(min = 2, max = 100, message = "El nombre debe tener entre 2 y 100 caracteres")
    String nombre,

    @NotBlank(message = "El apellido es obligatorio")
    @Size(min = 2, max = 100)
    String apellido,

    @NotBlank(message = "El email es obligatorio")
    @Email(message = "El email no tiene formato válido")
    String email,

    @NotBlank(message = "La contraseña es obligatoria")
    @Size(min = 8, message = "La contraseña debe tener al menos 8 caracteres")
    @Pattern(
        regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d).+$",
        message = "La contraseña debe tener al menos una mayúscula, una minúscula y un número"
    )
    String password,

    // El código estudiantil es opcional (los funcionarios no lo tienen)
    String codigoEstudiante
) {}
```

```java
// src/main/java/co/uniquindio/triage/security/dto/LoginRequest.java

package co.uniquindio.triage.security.dto;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;

public record LoginRequest(
    @NotBlank(message = "El email es obligatorio")
    @Email(message = "El email no tiene formato válido")
    String email,

    @NotBlank(message = "La contraseña es obligatoria")
    String password
) {}
```

```java
// src/main/java/co/uniquindio/triage/security/dto/LoginResponse.java

package co.uniquindio.triage.security.dto;

/**
 * Lo que devolvemos al cliente cuando hace login exitosamente.
 * El cliente Angular guarda estos tokens y los envía en cada petición.
 */
public record LoginResponse(
    String accessToken,
    String refreshToken,
    String tipo,           // siempre "Bearer"
    long expiresIn,        // milisegundos hasta que expira
    String email,
    String nombre,
    String rol
) {}
```

---

## 11. JWT: qué es y cómo funciona

JWT (JSON Web Token) es el mecanismo que usamos para que un usuario "demuestre" que ya se autenticó sin necesidad de que el servidor guarde información de sesión.

```
ANALOGÍA: El brazalete del festival

Cuando llegas a un festival de música, muestras tu entrada (contraseña).
El guardia la verifica y te pone un brazalete (JWT).
A partir de ahí, cada vez que quieres entrar a algún área del festival,
solo muestras tu brazalete. Los guardias de cada área ven el brazalete
y saben que ya fuiste verificado y qué áreas puedes acceder.
El guardia de la zona VIP no necesita llamar a la entrada principal
para preguntar "¿este señor pagó?". El brazalete lo dice todo.

El JWT es ese brazalete digital.
```

### Estructura de un JWT

Un JWT tiene tres partes separadas por puntos (`.`):

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJqdWFuQHVuaXF1aW5kaW8uZWR1LmNvIiwicm9sIjoiRVNUVURJQU5URSIsImlhdCI6MTcxNTAwMDAwMCwiZXhwIjoxNzE1MDg2NDAwfQ.xK9nHq3mCjWvUOqErtYJdBjKw2nsFpN_LkPMQRvFcDo

     HEADER                        PAYLOAD                            SIGNATURE
eyJhbGciOiJIUzI1NiJ9  .  eyJzdWIiOiJ...  .  xK9nHq3mCjWvUOqE...
```

Cada parte está codificada en **Base64URL** (no encriptada, solo codificada). Si decodificas el payload del ejemplo de arriba, obtienes:

```json
{
  "sub": "juan@uniquindio.edu.co",
  "rol": "ESTUDIANTE",
  "iat": 1715000000,
  "exp": 1715086400
}
```

Estos datos dentro del JWT se llaman **claims** (reclamaciones). Los más importantes son:

| Claim | Nombre          | Qué es                                          |
|-------|-----------------|-------------------------------------------------|
| `sub` | Subject         | El identificador del usuario (email)            |
| `iat` | Issued At       | Cuándo fue emitido (timestamp Unix)             |
| `exp` | Expiration      | Cuándo vence (timestamp Unix)                   |
| `rol` | Custom claim    | El rol del usuario (lo añadimos nosotros)       |

> ⚠️ **El JWT no está encriptado.** Cualquiera puede decodificar el payload. Por eso NUNCA metas contraseñas u información sensible en el JWT. Lo que sí es seguro es la **firma**: si alguien modifica el payload, la firma no coincide y el servidor rechaza el token.

**¿Cómo garantiza la firma la autenticidad?**

```
Servidor genera:   Firma = HMAC-SHA256(header + "." + payload, secreto)
Cliente recibe:    header.payload.firma

En la próxima petición:
Servidor recalcula: firma_esperada = HMAC-SHA256(header + "." + payload, secreto)
Compara:            ¿firma_recibida == firma_esperada?
  Sí → Token válido
  No → Token fue manipulado → 401 Unauthorized
```

---

## 12. Implementar JwtService

Este servicio es el responsable de dos cosas: **generar** tokens JWT y **validarlos**.

```java
// src/main/java/co/uniquindio/triage/security/service/JwtService.java

package co.uniquindio.triage.security.service;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.JwtException;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.security.Keys;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Service;

import javax.crypto.SecretKey;
import java.nio.charset.StandardCharsets;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

@Service
@Slf4j
public class JwtService {

    // Leemos la clave secreta y los tiempos de expiración desde application.yml
    @Value("${triage.jwt.secret-key}")
    private String secretKey;

    @Value("${triage.jwt.expiration}")
    private long jwtExpiration;           // 86400000 = 24 horas en ms

    @Value("${triage.jwt.refresh-expiration}")
    private long refreshExpiration;       // 604800000 = 7 días en ms

    // ─────────────────────────────────────────────────────────────
    // MÉTODOS PÚBLICOS
    // ─────────────────────────────────────────────────────────────

    /**
     * Genera un access token para el usuario.
     * Le agregamos el rol como claim personalizado.
     */
    public String generarToken(UserDetails userDetails) {
        Map<String, Object> extraClaims = new HashMap<>();
        // Si el usuario tiene roles, agregamos el primero como claim "rol"
        userDetails.getAuthorities().stream()
            .findFirst()
            .ifPresent(auth -> extraClaims.put("rol",
                auth.getAuthority().replace("ROLE_", "")));

        return buildToken(extraClaims, userDetails, jwtExpiration);
    }

    /**
     * Genera un refresh token (sin claims extra, solo para renovar el access token).
     */
    public String generarRefreshToken(UserDetails userDetails) {
        return buildToken(new HashMap<>(), userDetails, refreshExpiration);
    }

    /**
     * Extrae el email (subject) del token.
     */
    public String extraerEmail(String token) {
        return extraerClaim(token, Claims::getSubject);
    }

    /**
     * Verifica si el token es válido:
     * - El email coincide con el del usuario en BD
     * - El token no ha expirado
     */
    public boolean esTokenValido(String token, UserDetails userDetails) {
        try {
            final String email = extraerEmail(token);
            return email.equals(userDetails.getUsername())
                && !estaExpirado(token);
        } catch (JwtException e) {
            log.warn("Token JWT inválido: {}", e.getMessage());
            return false;
        }
    }

    /**
     * Devuelve el tiempo de expiración configurado (para incluirlo en la respuesta).
     */
    public long getExpiration() {
        return jwtExpiration;
    }

    // ─────────────────────────────────────────────────────────────
    // MÉTODOS PRIVADOS
    // ─────────────────────────────────────────────────────────────

    /**
     * Construye el token JWT con la API fluida de jjwt 0.12+.
     */
    private String buildToken(
            Map<String, Object> extraClaims,
            UserDetails userDetails,
            long expiration) {

        return Jwts.builder()
            .claims(extraClaims)                           // claims personalizados
            .subject(userDetails.getUsername())            // el email como subject
            .issuedAt(new Date(System.currentTimeMillis())) // cuándo se emitió
            .expiration(new Date(System.currentTimeMillis() + expiration)) // cuándo vence
            .signWith(getSigningKey())                     // firma con nuestra clave secreta
            .compact();                                    // genera el string final
    }

    /**
     * Extrae un claim específico del token usando una función.
     * Ejemplo: extraerClaim(token, Claims::getSubject) → el email
     *          extraerClaim(token, Claims::getExpiration) → la fecha de expiración
     */
    private <T> T extraerClaim(String token, Function<Claims, T> resolver) {
        return resolver.apply(extraerTodosLosClaims(token));
    }

    /**
     * Parsea el token y extrae todos los claims.
     * Si el token es inválido o expirado, jjwt lanza una excepción.
     */
    private Claims extraerTodosLosClaims(String token) {
        return Jwts.parser()
            .verifyWith(getSigningKey())    // verifica la firma con nuestra clave
            .build()
            .parseSignedClaims(token)       // parsea y valida
            .getPayload();                  // devuelve los claims
    }

    /**
     * Construye la clave de firma a partir de la cadena secreta.
     * Usamos HMAC-SHA256 (HS256).
     */
    private SecretKey getSigningKey() {
        byte[] keyBytes = secretKey.getBytes(StandardCharsets.UTF_8);
        return Keys.hmacShaKeyFor(keyBytes);
    }

    private boolean estaExpirado(String token) {
        return extraerClaim(token, Claims::getExpiration).before(new Date());
    }
}
```

### Configuración en application.yml

```yaml
# src/main/resources/application.yml

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/triage_db
    username: ${DB_USERNAME:triage_user}
    password: ${DB_PASSWORD:triage_pass}
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false

# ✅ Configuración de JWT
triage:
  jwt:
    # La clave secreta debe tener al menos 256 bits (32 caracteres).
    # En producción, cárgala desde una variable de entorno.
    secret-key: ${JWT_SECRET:triage-sistema-academico-uniquindio-2026-secret-key-256bits}
    expiration: 86400000          # 24 horas en milisegundos
    refresh-expiration: 604800000  # 7 días en milisegundos
```

> ⚠️ **Seguridad del secreto:** Nunca comitees el valor real de `JWT_SECRET` en Git. En desarrollo puedes usar el valor por defecto. En producción, siempre usa variables de entorno reales.

---

## 13. Implementar JwtAuthenticationFilter

Este es el componente más importante de toda la integración de seguridad. Se ejecuta en cada petición HTTP y hace lo siguiente:

```
Petición llega
      │
      ▼
¿Tiene header "Authorization: Bearer <token>"?
      │
  No ──┤──► Continúa sin autenticar (Spring Security rechazará si el endpoint lo requiere)
      │
  Sí  ▼
Extrae el token (quita el prefijo "Bearer ")
      │
      ▼
¿El token tiene un email válido?
      │
  No ──┤──► 401 Unauthorized
      │
  Sí  ▼
¿El usuario YA está autenticado en el contexto?
      │
  Sí ──┤──► Continúa (evita procesar dos veces)
      │
  No  ▼
Carga el usuario desde la BD (UserDetailsService)
      │
      ▼
¿El token es válido para ese usuario? (firma + expiración)
      │
  No ──┤──► 401 Unauthorized
      │
  Sí  ▼
Registra la autenticación en el SecurityContext
      │
      ▼
Continúa hacia el @RestController
```

```java
// src/main/java/co/uniquindio/triage/security/filter/JwtAuthenticationFilter.java

package co.uniquindio.triage.security.filter;

import co.uniquindio.triage.security.service.JwtService;
import co.uniquindio.triage.security.service.UserDetailsServiceImpl;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.lang.NonNull;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

/**
 * Filtro JWT que se ejecuta UNA SOLA VEZ por petición.
 * Extiende OncePerRequestFilter para garantizar eso.
 */
@Component
@RequiredArgsConstructor
@Slf4j
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsServiceImpl userDetailsService;

    private static final String AUTHORIZATION_HEADER = "Authorization";
    private static final String BEARER_PREFIX = "Bearer ";

    @Override
    protected void doFilterInternal(
            @NonNull HttpServletRequest request,
            @NonNull HttpServletResponse response,
            @NonNull FilterChain filterChain
    ) throws ServletException, IOException {

        // 1. Intentar extraer el token del header Authorization
        final String authHeader = request.getHeader(AUTHORIZATION_HEADER);

        // Si no hay header o no empieza con "Bearer ", no hay JWT.
        // Pasamos la petición al siguiente filtro sin autenticar.
        if (authHeader == null || !authHeader.startsWith(BEARER_PREFIX)) {
            filterChain.doFilter(request, response);
            return;
        }

        // 2. Extraer el token quitando el prefijo "Bearer "
        final String jwt = authHeader.substring(BEARER_PREFIX.length());

        // 3. Extraer el email del token
        final String email;
        try {
            email = jwtService.extraerEmail(jwt);
        } catch (Exception e) {
            log.warn("No se pudo extraer el email del JWT: {}", e.getMessage());
            filterChain.doFilter(request, response);
            return;
        }

        // 4. Si hay email y el usuario NO está ya autenticado en este contexto
        if (email != null &&
                SecurityContextHolder.getContext().getAuthentication() == null) {

            // 5. Cargar el usuario desde la base de datos
            var userDetails = userDetailsService.loadUserByUsername(email);

            // 6. Validar el token contra ese usuario
            if (jwtService.esTokenValido(jwt, userDetails)) {

                // 7. Crear el objeto de autenticación de Spring Security
                var authToken = new UsernamePasswordAuthenticationToken(
                    userDetails,    // el principal (quién es)
                    null,           // credentials (null porque ya autenticamos con JWT)
                    userDetails.getAuthorities()  // los permisos/roles
                );
                authToken.setDetails(
                    new WebAuthenticationDetailsSource().buildDetails(request)
                );

                // 8. Registrar la autenticación en el SecurityContext
                // A partir de este punto, Spring Security sabe quién es el usuario
                // y puede aplicar reglas de autorización (@PreAuthorize, etc.)
                SecurityContextHolder.getContext().setAuthentication(authToken);

                log.debug("Usuario autenticado: {} con rol: {}",
                    email, userDetails.getAuthorities());
            }
        }

        // 9. Continuar con el resto de la cadena de filtros
        filterChain.doFilter(request, response);
    }
}
```

---

## 14. Endpoint de login

Ahora implementamos el `AuthService` y `AuthController` que exponen el login y el registro.

### 14.1. AuthService

```java
// src/main/java/co/uniquindio/triage/auth/AuthService.java

package co.uniquindio.triage.auth;

import co.uniquindio.triage.domain.model.Rol;
import co.uniquindio.triage.domain.model.Usuario;
import co.uniquindio.triage.domain.repository.UsuarioRepository;
import co.uniquindio.triage.security.dto.LoginRequest;
import co.uniquindio.triage.security.dto.LoginResponse;
import co.uniquindio.triage.security.dto.RegisterRequest;
import co.uniquindio.triage.security.service.JwtService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
@Slf4j
public class AuthService {

    private final UsuarioRepository usuarioRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtService jwtService;
    private final AuthenticationManager authenticationManager;

    /**
     * Registra un nuevo usuario (por defecto con rol ESTUDIANTE).
     * Hashea la contraseña antes de guardar. Nunca texto plano.
     */
    @Transactional
    public LoginResponse registrar(RegisterRequest request) {

        // Verificar que el email no esté en uso
        if (usuarioRepository.existsByEmail(request.email())) {
            throw new IllegalArgumentException(
                "Ya existe un usuario registrado con el email: " + request.email()
            );
        }

        // Verificar código estudiantil si se proporcionó
        if (request.codigoEstudiante() != null &&
                usuarioRepository.existsByCodigoEstudiante(request.codigoEstudiante())) {
            throw new IllegalArgumentException(
                "El código estudiantil ya está en uso: " + request.codigoEstudiante()
            );
        }

        // Construir el usuario. El rol por defecto es ESTUDIANTE al registrarse.
        // Los administradores crean funcionarios manualmente o vía endpoint protegido.
        var usuario = Usuario.builder()
            .nombre(request.nombre())
            .apellido(request.apellido())
            .email(request.email())
            .password(passwordEncoder.encode(request.password()))  // ← hashear siempre
            .codigoEstudiante(request.codigoEstudiante())
            .rol(Rol.ESTUDIANTE)
            .build();

        usuarioRepository.save(usuario);
        log.info("Nuevo usuario registrado: {} ({})", usuario.getEmail(), usuario.getRol());

        // Generar tokens y responder (el usuario queda logueado inmediatamente)
        var accessToken = jwtService.generarToken(usuario);
        var refreshToken = jwtService.generarRefreshToken(usuario);

        return construirRespuesta(usuario, accessToken, refreshToken);
    }

    /**
     * Autentica un usuario existente.
     *
     * AuthenticationManager.authenticate() hace internamente:
     * 1. Llama a UserDetailsService.loadUserByUsername(email)
     * 2. Verifica la contraseña con PasswordEncoder.matches()
     * 3. Si todo está bien, devuelve el Authentication object
     * 4. Si algo falla, lanza BadCredentialsException o DisabledException
     */
    public LoginResponse login(LoginRequest request) {

        // Esta línea hace todo el trabajo de verificar email + contraseña
        authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                request.email(),
                request.password()
            )
        );

        // Si llegamos aquí, las credenciales son correctas.
        // Cargamos el usuario completo desde la BD.
        var usuario = usuarioRepository.findByEmail(request.email())
            .orElseThrow();

        var accessToken = jwtService.generarToken(usuario);
        var refreshToken = jwtService.generarRefreshToken(usuario);

        log.info("Login exitoso: {}", usuario.getEmail());
        return construirRespuesta(usuario, accessToken, refreshToken);
    }

    private LoginResponse construirRespuesta(
            Usuario usuario, String accessToken, String refreshToken) {
        return new LoginResponse(
            accessToken,
            refreshToken,
            "Bearer",
            jwtService.getExpiration(),
            usuario.getEmail(),
            usuario.getNombre() + " " + usuario.getApellido(),
            usuario.getRol().name()
        );
    }
}
```

### 14.2. AuthController

```java
// src/main/java/co/uniquindio/triage/auth/AuthController.java

package co.uniquindio.triage.auth;

import co.uniquindio.triage.security.dto.LoginRequest;
import co.uniquindio.triage.security.dto.LoginResponse;
import co.uniquindio.triage.security.dto.RegisterRequest;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/auth")
@RequiredArgsConstructor
@Tag(name = "Autenticación", description = "Endpoints de registro y login — públicos, no requieren JWT")
public class AuthController {

    private final AuthService authService;

    /**
     * POST /api/v1/auth/register
     * Cuerpo: { nombre, apellido, email, password, codigoEstudiante? }
     * Respuesta 201: { accessToken, refreshToken, tipo, expiresIn, email, nombre, rol }
     */
    @PostMapping("/register")
    @Operation(
        summary = "Registrar nuevo usuario",
        description = "Crea una cuenta con rol ESTUDIANTE y devuelve tokens JWT"
    )
    public ResponseEntity<LoginResponse> register(
            @Valid @RequestBody RegisterRequest request) {
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(authService.registrar(request));
    }

    /**
     * POST /api/v1/auth/login
     * Cuerpo: { email, password }
     * Respuesta 200: { accessToken, refreshToken, tipo, expiresIn, email, nombre, rol }
     */
    @PostMapping("/login")
    @Operation(
        summary = "Iniciar sesión",
        description = "Autentica con email y contraseña. Devuelve JWT de acceso y refresh token."
    )
    public ResponseEntity<LoginResponse> login(
            @Valid @RequestBody LoginRequest request) {
        return ResponseEntity.ok(authService.login(request));
    }
}
```

---

## 15. Autorización por roles con @PreAuthorize

Una vez que el usuario está autenticado (tiene JWT válido), necesitamos controlar qué operaciones puede hacer según su rol. `@PreAuthorize` evalúa la expresión ANTES de ejecutar el método.

Primero, recuerda que activamos esto con `@EnableMethodSecurity` en `SecurityConfig`.

```java
// src/main/java/co/uniquindio/triage/solicitud/SolicitudController.java
// Versión con seguridad por roles aplicada

package co.uniquindio.triage.solicitud;

import co.uniquindio.triage.solicitud.dto.*;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.security.SecurityRequirement;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/solicitudes")
@RequiredArgsConstructor
@Tag(name = "Solicitudes", description = "Gestión del ciclo de vida de solicitudes académicas")
@SecurityRequirement(name = "bearerAuth")  // Indica en Swagger que requiere JWT
public class SolicitudController {

    private final SolicitudService solicitudService;

    /**
     * RF-01: Cualquier usuario autenticado puede registrar una solicitud.
     * Usamos @AuthenticationPrincipal para obtener el usuario actual del JWT.
     */
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    @Operation(summary = "Registrar solicitud (RF-01)", description = "Cualquier usuario autenticado")
    public SolicitudResponseDto crear(
            @Valid @RequestBody CrearSolicitudDto dto,
            @AuthenticationPrincipal UserDetails usuarioActual) {

        return solicitudService.crear(dto, usuarioActual.getUsername());
    }

    /**
     * RF-02 y RF-03: Solo FUNCIONARIO o ADMIN pueden clasificar y priorizar.
     * Un ESTUDIANTE que intente esto recibe 403 Forbidden.
     */
    @PatchMapping("/{id}/clasificar")
    @PreAuthorize("hasAnyRole('FUNCIONARIO', 'ADMIN')")
    @Operation(summary = "Clasificar solicitud (RF-02, RF-03)")
    public SolicitudResponseDto clasificar(
            @PathVariable Long id,
            @Valid @RequestBody ClasificarSolicitudDto dto) {
        return solicitudService.clasificar(id, dto);
    }

    /**
     * RF-05: Asignar responsable — solo FUNCIONARIO o ADMIN.
     */
    @PatchMapping("/{id}/asignar-responsable")
    @PreAuthorize("hasAnyRole('FUNCIONARIO', 'ADMIN')")
    @Operation(summary = "Asignar responsable (RF-05)")
    public SolicitudResponseDto asignarResponsable(
            @PathVariable Long id,
            @Valid @RequestBody AsignarResponsableDto dto) {
        return solicitudService.asignarResponsable(id, dto);
    }

    /**
     * RF-08: Cerrar solicitud — solo FUNCIONARIO o ADMIN.
     */
    @PatchMapping("/{id}/cerrar")
    @PreAuthorize("hasAnyRole('FUNCIONARIO', 'ADMIN')")
    @Operation(summary = "Cerrar solicitud (RF-08)")
    public SolicitudResponseDto cerrar(
            @PathVariable Long id,
            @Valid @RequestBody CerrarSolicitudDto dto) {
        return solicitudService.cerrar(id, dto);
    }

    /**
     * RF-07: Consultar solicitudes.
     * ESTUDIANTE: solo ve las suyas (la lógica en el Service filtra por usuario).
     * FUNCIONARIO/ADMIN: ve todas.
     */
    @GetMapping
    @Operation(summary = "Consultar solicitudes (RF-07)")
    public Page<SolicitudResponseDto> listar(
            @RequestParam(required = false) String estado,
            @RequestParam(required = false) String tipo,
            @RequestParam(required = false) String prioridad,
            Pageable pageable,
            @AuthenticationPrincipal UserDetails usuarioActual) {
        return solicitudService.listar(estado, tipo, prioridad, pageable, usuarioActual);
    }

    /**
     * RF-06: Ver historial de una solicitud.
     * Cualquier autenticado puede ver, pero en el Service filtramos
     * que un ESTUDIANTE solo pueda ver el historial de SUS solicitudes.
     */
    @GetMapping("/{id}/historial")
    @Operation(summary = "Historial de solicitud (RF-06)")
    public Object verHistorial(
            @PathVariable Long id,
            @AuthenticationPrincipal UserDetails usuarioActual) {
        return solicitudService.obtenerHistorial(id, usuarioActual);
    }

    /**
     * Endpoint exclusivo para ADMIN: borrar una solicitud (operación peligrosa).
     */
    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    @Operation(summary = "Eliminar solicitud — solo ADMIN")
    public void eliminar(@PathVariable Long id) {
        solicitudService.eliminar(id);
    }
}
```

### Tabla de expresiones @PreAuthorize más usadas

| Expresión                              | Qué verifica                                          |
|----------------------------------------|-------------------------------------------------------|
| `hasRole('ADMIN')`                     | El usuario tiene exactamente el rol ADMIN             |
| `hasAnyRole('FUNCIONARIO', 'ADMIN')`   | El usuario tiene alguno de esos roles                 |
| `isAuthenticated()`                    | El usuario está autenticado (tiene JWT válido)        |
| `isAnonymous()`                        | El usuario NO está autenticado                        |
| `#id == principal.id`                  | El parámetro `id` coincide con el ID del usuario      |
| `@solicitudService.esElDueno(#id, authentication.name)` | Llama a un método para verificar |

### Cómo obtener el usuario actual en cualquier punto

```java
// Opción 1: Con @AuthenticationPrincipal en el método del Controller
public ResponseEntity<?> endpoint(@AuthenticationPrincipal UserDetails user) {
    String email = user.getUsername();
    // ...
}

// Opción 2: Desde cualquier lugar (Service, etc.) — acceso al SecurityContext
import org.springframework.security.core.context.SecurityContextHolder;

public String obtenerEmailActual() {
    return SecurityContextHolder.getContext()
        .getAuthentication()
        .getName();  // devuelve el username (email en nuestro caso)
}
```

---

## 16. Configuración CORS para Angular

CORS (Cross-Origin Resource Sharing) es un mecanismo de seguridad del navegador que bloquea las peticiones cuando el origen (dominio + puerto) del JavaScript que hace la petición no coincide con el origen del servidor.

```
PROBLEMA CORS (sin configurar):

Navegador (http://localhost:4200)
    │
    ├──► GET http://localhost:8080/api/v1/solicitudes
    │
    └──► El navegador ve: ¡el origen del JS (4200) ≠ el destino (8080)!
         BLOQUEA la respuesta. Error: "Access-Control-Allow-Origin header missing"

SOLUCIÓN: el servidor (Spring Boot) le dice al navegador:
    "Sí, el origen http://localhost:4200 tiene permiso para consumirme"
```

> ⚠️ Importante: CORS es una restricción del **navegador**, no del servidor. Si pruebas con Postman o curl, CORS no aplica porque no hay navegador.

La configuración CORS la pusimos en `SecurityConfig` (sección 9.2). Aquí profundizamos en los headers:

```java
// Dentro de corsConfigurationSource() en SecurityConfig:

config.setAllowedOrigins(List.of(
    "http://localhost:4200",    // Angular dev server
    "https://triage.uniquindio.edu.co"  // producción
));

// Métodos HTTP permitidos
config.setAllowedMethods(List.of("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"));
// ↑ OPTIONS es fundamental: los navegadores envían un "preflight" OPTIONS
//   antes de POST/PUT/DELETE para verificar si tienen permiso.

// Headers que el frontend puede enviar
config.setAllowedHeaders(List.of(
    "Authorization",  // para enviar el Bearer token
    "Content-Type",   // application/json
    "Accept"
));

// Headers que el frontend puede leer de la respuesta
config.setExposedHeaders(List.of("Authorization"));

// Permite enviar cookies y credenciales (necesario si usas cookies para refresh tokens)
config.setAllowCredentials(true);

// El navegador cachea el preflight por este tiempo (en segundos)
config.setMaxAge(3600L);
```

### Error CORS con Spring Security activo — cuidado

Cuando Spring Security está activo y configurado, el problema CORS típico es que el filtro de seguridad rechaza la petición **antes** de que el header CORS sea añadido. Por eso es fundamental que en `SecurityConfig` configures CORS antes de agregar el filtro JWT. El orden que mostramos en la sección 9.2 lo garantiza.

---

## 17. Refresh Tokens

### ¿Qué problema resuelven?

Los access tokens (JWT) tienen vida corta por seguridad (24 horas en nuestro caso). Si el usuario está usando la aplicación cuando el token expira, no queremos que tenga que hacer login de nuevo. Ahí entran los refresh tokens.

```
┌─────────────────────────────────────────────────────────────────┐
│                     FLUJO DE TOKENS                             │
│                                                                 │
│  Login exitoso                                                  │
│       │                                                         │
│       ├──► access_token  (vive 24h) → para cada petición API    │
│       └──► refresh_token (vive 7d)  → solo para renovar         │
│                                                                 │
│  Cada petición API → envía access_token en el header            │
│                                                                 │
│  access_token expira (error 401) → Angular detecta el 401       │
│       │                                                         │
│       └──► POST /auth/refresh con el refresh_token              │
│                 │                                               │
│                 └──► Servidor valida → devuelve nuevo           │
│                      access_token (y opcionalmente nuevo        │
│                      refresh_token = "rolling refresh")         │
└─────────────────────────────────────────────────────────────────┘
```

### Entidad RefreshToken (almacenada en BD)

A diferencia del access token (que es stateless y no se guarda en BD), el refresh token sí se almacena para poder invalidarlo si el usuario hace logout:

```java
// src/main/java/co/uniquindio/triage/domain/model/RefreshToken.java

package co.uniquindio.triage.domain.model;

import jakarta.persistence.*;
import lombok.*;

import java.time.Instant;

@Entity
@Table(name = "refresh_tokens")
@Getter @Setter @NoArgsConstructor @AllArgsConstructor @Builder
public class RefreshToken {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true, length = 500)
    private String token;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "usuario_id", nullable = false)
    private Usuario usuario;

    @Column(name = "fecha_expiracion", nullable = false)
    private Instant fechaExpiracion;

    @Column(nullable = false)
    @Builder.Default
    private boolean revocado = false;

    public boolean estaExpirado() {
        return Instant.now().isAfter(fechaExpiracion);
    }

    public boolean esValido() {
        return !revocado && !estaExpirado();
    }
}
```

```java
// src/main/java/co/uniquindio/triage/domain/repository/RefreshTokenRepository.java

package co.uniquindio.triage.domain.repository;

import co.uniquindio.triage.domain.model.RefreshToken;
import co.uniquindio.triage.domain.model.Usuario;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;

import java.util.Optional;

public interface RefreshTokenRepository extends JpaRepository<RefreshToken, Long> {

    Optional<RefreshToken> findByToken(String token);

    // Al hacer login nuevamente, revocamos los tokens anteriores del usuario
    @Modifying
    @Query("UPDATE RefreshToken r SET r.revocado = true WHERE r.usuario = :usuario AND r.revocado = false")
    void revocarTodosDeUsuario(Usuario usuario);
}
```

### Endpoint de refresh

```java
// Dentro de AuthController — agregar este endpoint:

/**
 * POST /api/v1/auth/refresh
 * Cuerpo: { refreshToken: "eyJ..." }
 * Respuesta: nuevo accessToken (y opcionalmente nuevo refreshToken)
 */
@PostMapping("/refresh")
@Operation(summary = "Renovar access token usando el refresh token")
public ResponseEntity<LoginResponse> refresh(
        @Valid @RequestBody TokenRefreshRequest request) {
    return ResponseEntity.ok(authService.renovarToken(request.refreshToken()));
}
```

```java
// src/main/java/co/uniquindio/triage/security/dto/TokenRefreshRequest.java

public record TokenRefreshRequest(
    @NotBlank(message = "El refresh token es obligatorio")
    String refreshToken
) {}
```

```java
// En AuthService — agregar el método renovarToken:

public LoginResponse renovarToken(String refreshTokenString) {
    var refreshToken = refreshTokenRepository
        .findByToken(refreshTokenString)
        .orElseThrow(() -> new IllegalArgumentException("Refresh token no encontrado"));

    if (!refreshToken.esValido()) {
        throw new IllegalArgumentException(
            refreshToken.isRevocado()
                ? "El refresh token ha sido revocado"
                : "El refresh token ha expirado. Por favor inicia sesión nuevamente."
        );
    }

    var usuario = refreshToken.getUsuario();
    var nuevoAccessToken = jwtService.generarToken(usuario);

    return construirRespuesta(usuario, nuevoAccessToken, refreshTokenString);
}
```

---

## 18. Flujo completo de prueba en Postman

Aquí tienes el flujo completo para probar tu API de seguridad:

### Paso 1: Registro de nuevo usuario

```
POST http://localhost:8080/api/v1/auth/register
Content-Type: application/json

{
  "nombre": "Juan",
  "apellido": "García López",
  "email": "juan.garcia@uniquindio.edu.co",
  "password": "MiClave2026!",
  "codigoEstudiante": "1094567890"
}
```

Respuesta esperada `201 Created`:
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiJ9...",
  "tipo": "Bearer",
  "expiresIn": 86400000,
  "email": "juan.garcia@uniquindio.edu.co",
  "nombre": "Juan García López",
  "rol": "ESTUDIANTE"
}
```

### Paso 2: Login

```
POST http://localhost:8080/api/v1/auth/login
Content-Type: application/json

{
  "email": "juan.garcia@uniquindio.edu.co",
  "password": "MiClave2026!"
}
```

### Paso 3: Usar el token en un endpoint protegido

Copia el `accessToken` de la respuesta y úsalo así:

```
GET http://localhost:8080/api/v1/solicitudes
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

### Paso 4: Probar un endpoint que requiere FUNCIONARIO (siendo ESTUDIANTE)

```
PATCH http://localhost:8080/api/v1/solicitudes/1/clasificar
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...  ← token de ESTUDIANTE
Content-Type: application/json

{ "tipo": "HOMOLOGACION", "prioridad": "ALTA" }
```

Respuesta esperada `403 Forbidden`:
```json
{
  "status": 403,
  "error": "Forbidden",
  "message": "No tienes permiso para realizar esta operación",
  "path": "/api/v1/solicitudes/1/clasificar"
}
```

### Paso 5: Login como admin y repetir la clasificación

```
POST http://localhost:8080/api/v1/auth/login
{
  "email": "admin@uniquindio.edu.co",
  "password": "Admin2026!"
}
```

Con el token del ADMIN, el PATCH a `/clasificar` ahora devuelve `200 OK`.

### Paso 6: Token expirado

Si esperas a que el token expire (o cambias la expiración a 1 segundo para probar), verás:

```
Respuesta: 401 Unauthorized
{
  "status": 401,
  "error": "Unauthorized",
  "message": "El token de acceso ha expirado. Por favor renuévalo."
}
```

### Paso 7: Renovar con el refresh token

```
POST http://localhost:8080/api/v1/auth/refresh
Content-Type: application/json

{
  "refreshToken": "eyJhbGciOiJIUzI1NiJ9..."  ← el refreshToken guardado
}
```

Obtienes un nuevo `accessToken` sin necesidad de hacer login de nuevo.

### Configurar Swagger para enviar JWT

Agrega esta configuración a tu `OpenApiConfig` para que Swagger UI tenga el botón "Authorize":

```java
@Bean
public OpenAPI openAPI() {
    return new OpenAPI()
        .info(new Info()
            .title("API Triage - Sistema de Gestión de Solicitudes Académicas")
            .version("1.0.0")
            .description("Universidad del Quindío — Programación Avanzada 2026"))
        .addSecurityItem(new SecurityRequirement().addList("bearerAuth"))
        .components(new Components()
            .addSecuritySchemes("bearerAuth", new SecurityScheme()
                .type(SecurityScheme.Type.HTTP)
                .scheme("bearer")
                .bearerFormat("JWT")
                .description("Ingresa el token JWT obtenido del endpoint /auth/login")
            )
        );
}
```

---

## 19. Errores comunes y Troubleshooting

### Error 1: 401 cuando debería ser 200

**Síntoma:** Haces petición con JWT válido y obtienes 401.  
**Causas posibles:**

```
1. Olvidaste el prefijo "Bearer " en el header:
   ❌ Authorization: eyJhbGciOiJIUzI1NiJ9...
   ✅ Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...

2. El JWT_SECRET en el servidor es diferente al que usaste para generar el token.
   Solución: reinicia el servidor y genera un token nuevo.

3. El token ya expiró (revisa la fecha en jwt.io)

4. El endpoint no está en la lista PUBLIC_URLS pero sí debería estar.
```

### Error 2: 403 cuando debería ser 200

**Síntoma:** Estás autenticado pero obtienes 403.  
**Causas posibles:**

```
1. Tu rol no tiene permiso para ese endpoint.
   Revisa el @PreAuthorize del método.

2. En getAuthorities() estás devolviendo "ESTUDIANTE" sin el prefijo "ROLE_".
   hasRole('ESTUDIANTE') busca "ROLE_ESTUDIANTE".
   Solución: asegúrate de devolver new SimpleGrantedAuthority("ROLE_" + rol.name())

3. No pusiste @EnableMethodSecurity en SecurityConfig.
   El @PreAuthorize silenciosamente no funciona sin esta anotación.
```

### Error 3: CORS bloqueado con Spring Security activo

**Síntoma:** Las peticiones de Angular son bloqueadas con error CORS incluso con corsConfigurationSource configurado.  
**Causa:** Spring Security rechaza la petición antes de que el filtro CORS procese los headers.  
**Solución:** Asegúrate de que la configuración CORS esté en el `SecurityFilterChain`, no solo en un `@CrossOrigin` o `WebMvcConfigurer`:

```java
// ✅ Configurar CORS dentro del SecurityFilterChain:
.cors(cors -> cors.configurationSource(corsConfigurationSource()))

// ❌ No es suficiente tener solo @CrossOrigin en el controller cuando Security está activo
```

### Error 4: BCrypt de distinta versión

**Síntoma:** El login falla con credenciales correctas. En logs: `Password did not match stored value`.  
**Causa:** El hash en la BD fue generado con una versión de BCrypt y se está verificando con otra.  
**Solución:** Siempre usa el mismo `PasswordEncoder` Bean para hashear y verificar. Si cambias el `strength` de BCrypt, re-hashea todas las contraseñas.

```java
// Para generar el hash del admin en la migración, puedes usar este test temporal:
@Test
void generarHashAdmin() {
    var encoder = new BCryptPasswordEncoder(12);
    System.out.println(encoder.encode("Admin2026!"));
    // Copia el resultado y pégalo en el script SQL de migración
}
```

### Error 5: 401 vs 403 — diferencia importante

```
401 Unauthorized → El usuario NO está autenticado (no hay JWT, o el JWT es inválido)
                   Mensaje implícito: "No sé quién eres"

403 Forbidden     → El usuario SÍ está autenticado pero NO tiene permiso
                   Mensaje implícito: "Sé quién eres, pero no puedes hacer esto"
```

### Error 6: Token expirado — mensajes claros al cliente

Spring Security por defecto devuelve un 401 genérico cuando el token expira. Para dar un mensaje más específico, puedes crear un `AuthenticationEntryPoint` personalizado:

```java
@Component
public class JwtAuthEntryPoint implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
            AuthenticationException authException) throws IOException {

        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);

        var body = Map.of(
            "status", 401,
            "error", "No autenticado",
            "message", "El token de acceso ha expirado o es inválido. "
                     + "Por favor inicia sesión nuevamente.",
            "path", request.getServletPath()
        );

        new ObjectMapper().writeValue(response.getOutputStream(), body);
    }
}
```

Y regístralo en `SecurityConfig`:

```java
.exceptionHandling(ex ->
    ex.authenticationEntryPoint(jwtAuthEntryPoint)
)
```

### Error 7: `spring-boot-starter-security` bloquea todo desde el inicio

Este no es un error, es el comportamiento esperado. Cuando agregas la dependencia sin configurar nada, Spring Security protege todo con autenticación básica HTTP. La solución es completar la configuración de `SecurityConfig` como la mostramos en esta guía.

---

## 20. Resumen y Cheat Sheet

### Flujo completo de autenticación

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    FLUJO COMPLETO JWT EN EL TRIAGE                      │
│                                                                         │
│  1. REGISTRO                                                            │
│  POST /auth/register → AuthController → AuthService                     │
│     → validar email único → hashear contraseña → guardar Usuario        │
│     → generar JWT → responder con tokens                                │
│                                                                         │
│  2. LOGIN                                                               │
│  POST /auth/login → AuthController → AuthService                        │
│     → AuthenticationManager.authenticate() → verifica BD + BCrypt      │
│     → generar JWT → responder con tokens                                │
│                                                                         │
│  3. USAR LA API                                                         │
│  GET /api/v1/solicitudes                                                │
│  Header: Authorization: Bearer <accessToken>                            │
│     → JwtAuthenticationFilter intercepta                                │
│     → extrae email del token                                            │
│     → carga Usuario desde BD (UserDetailsService)                       │
│     → valida token (firma + expiración)                                 │
│     → registra autenticación en SecurityContext                         │
│     → llega al @RestController                                          │
│     → @PreAuthorize verifica el rol si aplica                           │
│                                                                         │
│  4. TOKEN EXPIRADO                                                      │
│  POST /auth/refresh { refreshToken }                                    │
│     → validar refresh token en BD                                       │
│     → generar nuevo accessToken                                         │
│     → responder                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Anotaciones de seguridad de un vistazo

```java
// En SecurityConfig
@Configuration
@EnableWebSecurity           // activa Spring Security
@EnableMethodSecurity        // activa @PreAuthorize

// En los @RestController
@PreAuthorize("hasRole('ADMIN')")
@PreAuthorize("hasAnyRole('FUNCIONARIO', 'ADMIN')")
@PreAuthorize("isAuthenticated()")

// Para obtener el usuario actual
@AuthenticationPrincipal UserDetails usuario

// En los endpoints públicos (en SecurityConfig.PUBLIC_URLS)
.requestMatchers("/api/v1/auth/**").permitAll()
```

### Archivos que creaste en esta guía

| Archivo | Función |
|---------|---------|
| `domain/model/Usuario.java` | Entidad que implementa UserDetails |
| `domain/model/Rol.java` | Enum con los tres roles |
| `domain/model/RefreshToken.java` | Token de renovación persistido en BD |
| `domain/repository/UsuarioRepository.java` | Acceso a datos de usuarios |
| `domain/repository/RefreshTokenRepository.java` | Acceso a refresh tokens |
| `security/service/UserDetailsServiceImpl.java` | Carga usuario desde BD para Spring Security |
| `security/service/JwtService.java` | Genera y valida JWT |
| `security/filter/JwtAuthenticationFilter.java` | Intercepta peticiones y valida el JWT |
| `security/dto/LoginRequest.java` | Record de entrada para login |
| `security/dto/LoginResponse.java` | Record de respuesta con tokens |
| `security/dto/RegisterRequest.java` | Record de entrada para registro |
| `security/dto/TokenRefreshRequest.java` | Record para el refresh |
| `config/ApplicationConfig.java` | Beans: PasswordEncoder, AuthManager |
| `config/SecurityConfig.java` | Configuración central de seguridad |
| `auth/AuthService.java` | Lógica de autenticación |
| `auth/AuthController.java` | Endpoints /auth/login y /auth/register |
| `db/migration/V3__create_tabla_usuarios.sql` | Script Flyway para la tabla |

---

## 21. Ejercicios prácticos

Estos retos te ayudarán a consolidar lo aprendido. Trabájalos de manera autónoma, en el orden sugerido:

### Reto 1 — Verificación básica
Implementa toda la guía paso a paso. Al final, debes poder:
- Registrar un usuario nuevo
- Hacer login y obtener un JWT
- Usar ese JWT para consultar `/api/v1/solicitudes`
- Verificar que sin el JWT obtienes 401
- Verificar que con JWT de ESTUDIANTE y endpoint de FUNCIONARIO obtienes 403

### Reto 2 — Endpoint de perfil
Crea el endpoint `GET /api/v1/usuarios/me` que devuelva los datos del usuario actualmente autenticado (usando `@AuthenticationPrincipal`). El usuario solo puede ver su propio perfil, nunca el de otro.

### Reto 3 — Crear funcionario (solo ADMIN)
Crea el endpoint `POST /api/v1/usuarios/funcionario` que permita a un ADMIN crear un usuario con rol FUNCIONARIO. Protégelo con `@PreAuthorize("hasRole('ADMIN')")`. Un FUNCIONARIO o ESTUDIANTE que intente usarlo debe recibir 403.

### Reto 4 — Logout con invalidación de refresh token
Implementa `POST /api/v1/auth/logout` que:
- Recibe el refreshToken en el body
- Lo marca como revocado en la BD
- Responde con 204 No Content

### Reto 5 — Filtrado por usuario en consultas
En `SolicitudService.listar()`, implementa la lógica de que:
- Si el usuario tiene rol ESTUDIANTE, solo se devuelven sus propias solicitudes
- Si tiene rol FUNCIONARIO o ADMIN, se devuelven todas

Pista: usa `@AuthenticationPrincipal` para obtener el usuario actual en el Controller y pásalo al Service.

### Reto 6 — Manejo elegante de token expirado
Implementa el `JwtAuthEntryPoint` personalizado (mencionado en la sección 19) para que cuando el token esté expirado, la respuesta 401 incluya un campo `expired: true` en el JSON. Esto le permite a Angular detectar específicamente la expiración y redirigir al flujo de refresh.

---

## 22. Referencias

- **Spring Security Reference Documentation** — [docs.spring.io/spring-security/reference](https://docs.spring.io/spring-security/reference/index.html)
- **jjwt GitHub** — [github.com/jwtk/jjwt](https://github.com/jwtk/jjwt) — siempre usa la versión más reciente
- **JWT.io** — Herramienta para decodificar y depurar tokens JWT en el navegador
- **RFC 7519** — La especificación oficial de JWT
- **BCrypt** — Niels Provos y David Mazières, 1999 — el algoritmo de hashing de contraseñas más recomendado
- **Baeldung Spring Security Series** — [baeldung.com/security-spring](https://www.baeldung.com/security-spring) — artículos complementarios de referencia
- **Ruíz, Carlos (2018)** — *Spring Boot & Angular: Desarrollo de WebApps Seguras.* 0xWord.
- **Sharma, Sourabh (2021)** — *Modern API Development with Spring and Spring Boot.* Packt Publishing.

---

> **Autor:** José Alfredo Ramírez Espinosa  
> **Materia:** Programación Avanzada  
> **Programa:** Ingeniería de Sistemas y Computación  
> **Universidad del Quindío** | Armenia, Colombia  
> **Año:** 2026  
> *Material de uso académico. Todos los derechos reservados al autor.*
