# Guía 10 — Integración Full-Stack: Angular + Spring Boot de Extremo a Extremo

> **Materia:** Programación Avanzada  
> **Programa:** Ingeniería de Sistemas y Computación — Universidad del Quindío  
> **Núcleo temático:** 3 (Servicios de Negocio) + 4 (Capa de Presentación)  
> **Guía:** 10 de 12  
> **Duración estimada de estudio:** 6–8 horas  

---

## Prerrequisitos

Antes de trabajar esta guía, debes haber completado:

- **Guía 04** — Spring Boot desde Cero
- **Guía 05** — JPA, Hibernate y Persistencia
- **Guía 06** — API REST con Spring Boot
- **Guía 07** — Spring Security 6 + JWT
- **Guía 08** — Angular desde Cero hasta Intermedio
- **Guía 09** — Angular Avanzado: Consumiendo el Backend del Triage

Tener corriendo localmente:
- El proyecto backend en `http://localhost:8080`
- El proyecto frontend Angular en `http://localhost:4200`
- PostgreSQL con la base de datos `triage_db` inicializada

---

## Tabla de Contenidos

1. [Objetivos de Aprendizaje](#1-objetivos-de-aprendizaje)
2. [El Flujo Completo de Extremo a Extremo](#2-el-flujo-completo-de-extremo-a-extremo)
3. [CORS: Configuración Definitiva](#3-cors-configuración-definitiva)
4. [Variables de Entorno en Angular](#4-variables-de-entorno-en-angular)
5. [Flujo de Autenticación Completo Implementado](#5-flujo-de-autenticación-completo-implementado)
6. [Caso de Uso E2E #1: Estudiante Crea una Solicitud](#6-caso-de-uso-e2e-1-estudiante-crea-una-solicitud-de-homologación)
7. [Caso de Uso E2E #2: Funcionario Clasifica y Prioriza](#7-caso-de-uso-e2e-2-funcionario-clasifica-y-prioriza-una-solicitud)
8. [Caso de Uso E2E #3: Consultar con Filtros y Paginación](#8-caso-de-uso-e2e-3-consultar-solicitudes-con-filtros-y-paginación)
9. [Manejo de Errores de Punta a Punta](#9-manejo-de-errores-de-punta-a-punta)
10. [Testing Básico: JUnit 5 + Mockito y HttpClientTestingModule](#10-testing-básico-junit-5--mockito-y-httpclienttestingmodule)
11. [Errores Comunes y Troubleshooting](#11-errores-comunes-y-troubleshooting)
12. [Resumen y Cheat Sheet](#12-resumen-y-cheat-sheet)
13. [Referencias y Recursos Adicionales](#13-referencias-y-recursos-adicionales)

---

## 1. Objetivos de Aprendizaje

Al terminar esta guía serás capaz de:

- Explicar y trazar el flujo completo de una petición desde el navegador hasta la base de datos y de regreso.
- Configurar CORS correctamente en Spring Boot para que Angular pueda consumir la API sin errores.
- Manejar variables de entorno en Angular para apuntar al backend según el ambiente (desarrollo/producción).
- Implementar el flujo de autenticación JWT de extremo a extremo: login → token → interceptor → guard de rutas.
- Codificar al menos tres casos de uso completos del Sistema de Triage, conectando cada capa sin fisuras.
- Propagar errores desde el backend (Problem Details RFC 7807) hasta el usuario en el frontend con mensajes claros.
- Escribir pruebas unitarias básicas para el Service del backend con JUnit 5 y Mockito.
- Escribir pruebas unitarias básicas para servicios Angular con `HttpClientTestingModule`.

---

## 2. El Flujo Completo de Extremo a Extremo

Antes de escribir una sola línea de código de integración, necesitas tener completamente claro qué pasa cuando el usuario hace clic en un botón en el navegador. Vamos a trazar ese camino en detalle.

### 2.1 El Mapa Mental del Viaje

Imagina que eres un paquete de datos. Tu viaje empieza cuando el usuario hace clic en "Enviar Solicitud" y termina cuando la pantalla se actualiza con el resultado. Esto es lo que te ocurre:

```
NAVEGADOR (Angular - Puerto 4200)
│
│  1. El usuario hace clic en un botón
│  2. El componente llama a un método del servicio Angular
│  3. El servicio usa HttpClient para construir una petición HTTP
│  4. El interceptor JWT adjunta el token Bearer al header
│  5. La petición sale del navegador hacia el servidor
│
▼ ──── RED ──── (HTTP / HTTPS)
│
│  6. La petición llega al servidor Spring Boot (Puerto 8080)
│  7. Spring Security intercepta ANTES de que llegue al Controller
│  8. El filtro JWT extrae y valida el token del header
│  9. Si el token es válido, carga el usuario y sus roles en el contexto
│ 10. Spring Security verifica si el usuario tiene permiso para esa ruta
│
▼ ──── SPRING BOOT ────
│
│ 11. La petición llega al @RestController correspondiente
│ 12. @Valid valida el cuerpo de la petición (si aplica)
│ 13. El Controller delega al @Service
│ 14. El Service aplica las reglas de negocio
│ 15. El Service llama al @Repository
│ 16. Spring Data JPA traduce la llamada a SQL
│
▼ ──── BASE DE DATOS (PostgreSQL) ────
│
│ 17. PostgreSQL ejecuta la consulta o inserción
│ 18. Devuelve el resultado al Repository
│
▲ ──── DE VUELTA ────
│
│ 19. El Repository devuelve la entidad al Service
│ 20. El Service mapea la entidad a un DTO (Record)
│ 21. El Controller empaqueta el DTO en ResponseEntity
│ 22. Spring serializa el DTO a JSON
│ 23. La respuesta HTTP viaja de vuelta al navegador
│
▲ ──── ANGULAR ────
│
│ 24. HttpClient recibe la respuesta JSON
│ 25. El Observable emite el valor
│ 26. El componente recibe el dato en el subscribe/async pipe
│ 27. Angular actualiza el DOM automáticamente
│ 28. El usuario ve el resultado en pantalla
```

### 2.2 Diagrama ASCII Detallado

```
┌─────────────────────────────────────────────────────────┐
│                  NAVEGADOR (Angular)                     │
│                                                         │
│  ┌──────────────┐    ┌──────────────┐   ┌────────────┐  │
│  │  Componente  │───▶│   Servicio   │──▶│ HttpClient │  │
│  │  (UI/Vista)  │◀───│  (Lógica UI) │   │            │  │
│  └──────────────┘    └──────────────┘   └─────┬──────┘  │
│                                               │         │
│                                    ┌──────────▼──────┐  │
│                                    │  JWT Interceptor │  │
│                                    │  Error Interceptor│ │
│                                    └──────────┬──────┘  │
└───────────────────────────────────────────────┼─────────┘
                                                │ HTTP Request
                                                │ Authorization: Bearer <token>
                                                ▼
┌─────────────────────────────────────────────────────────┐
│                  SPRING BOOT (8080)                      │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │            Spring Security Filter Chain           │   │
│  │  ┌───────────────┐  ┌───────────────────────┐    │   │
│  │  │CorsFilter     │  │JwtAuthenticationFilter│    │   │
│  │  └───────────────┘  └───────────────────────┘    │   │
│  └──────────────────────────────────────────────────┘   │
│                          │                              │
│  ┌───────────────┐  ┌────▼──────┐  ┌────────────────┐  │
│  │  @Repository  │◀─│ @Service  │◀─│ @RestController│  │
│  │  (JPA/BD)     │  │ (Negocio) │  │  (@Valid,DTO)  │  │
│  └───────┬───────┘  └───────────┘  └────────────────┘  │
└──────────┼──────────────────────────────────────────────┘
           │ SQL
           ▼
┌─────────────────────────┐
│   PostgreSQL             │
│   (triage_db)            │
└─────────────────────────┘
```

### 2.3 Las Responsabilidades de Cada Capa

Es importante que nunca mezcles responsabilidades. Aquí está el contrato de cada capa:

| Capa | Responsabilidad | Lo que NO debe hacer |
|---|---|---|
| **Componente Angular** | Mostrar datos, capturar eventos del usuario, llamar al servicio | Lógica de negocio, llamadas HTTP directas |
| **Servicio Angular** | Llamadas HTTP, transformar datos para el componente | Manipular el DOM, lógica de presentación |
| **Interceptor Angular** | Modificar peticiones/respuestas de forma transversal (token, errores) | Lógica de negocio específica |
| **Controller Spring** | Recibir petición, validar entrada, delegar al Service, devolver respuesta | Lógica de negocio, acceso a BD directo |
| **Service Spring** | Reglas de negocio, orquestación, transacciones | Acceso directo a BD, serialización HTTP |
| **Repository Spring** | Acceso a base de datos | Lógica de negocio |
| **Entidad JPA** | Representar datos en BD | Lógica de negocio compleja, serialización |
| **DTO (Record)** | Transferir datos entre capas o hacia el cliente | Persistencia, lógica de negocio |

---

## 3. CORS: Configuración Definitiva

### 3.1 ¿Qué es CORS y por qué causa tantos problemas?

CORS significa *Cross-Origin Resource Sharing* (Intercambio de Recursos entre Orígenes Cruzados). Es una política de seguridad que implementan los **navegadores** (no los servidores) para evitar que una página web haga peticiones a un dominio diferente al que la sirvió, sin permiso explícito.

Piénsalo así: tu Angular corre en `http://localhost:4200` y tu Spring Boot en `http://localhost:8080`. Para el navegador, son dos "orígenes" distintos porque tienen puertos diferentes. El navegador entonces pregunta al servidor: "¿Puedo hacer una petición desde 4200 hacia 8080?" Si el servidor no responde afirmativamente con los headers correctos, el navegador bloquea la respuesta.

**El error más temido:**
```
Access to XMLHttpRequest at 'http://localhost:8080/api/v1/solicitudes'
from origin 'http://localhost:4200' has been blocked by CORS policy:
No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

Lo más confuso para los principiantes: **el servidor SÍ recibió y procesó la petición**, pero el navegador bloquea la respuesta antes de que Angular la vea.

### 3.2 El Preflight Request (Petición OPTIONS)

Para peticiones que modifican datos (POST, PUT, DELETE) o que llevan headers personalizados (como `Authorization`), el navegador primero envía una petición de "preflight" usando el método `OPTIONS`. Solo si el servidor responde correctamente al preflight, el navegador envía la petición real.

```
Navegador ──── OPTIONS /api/v1/solicitudes ──── ▶ Spring Boot
               Origin: http://localhost:4200
               Access-Control-Request-Method: POST
               Access-Control-Request-Headers: authorization, content-type

Spring Boot ◀─────────────────────────────────── responde con:
               Access-Control-Allow-Origin: http://localhost:4200
               Access-Control-Allow-Methods: GET, POST, PUT, ...
               Access-Control-Allow-Headers: authorization, content-type

Navegador ──── POST /api/v1/solicitudes ──────── ▶ Spring Boot
               Authorization: Bearer <token>
               (¡Ahora sí la petición real!)
```

### 3.3 Configuración en Spring Boot (sin Spring Security)

Si no usas Spring Security, la configuración más limpia es un bean de tipo `WebMvcConfigurer`:

```java
// src/main/java/co/uniquindio/triage/config/CorsConfig.java

package co.uniquindio.triage.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class CorsConfig {

    // Leemos los orígenes permitidos desde application.yml
    // Así no hay credenciales hardcodeadas en el código
    @Value("${app.cors.allowed-origins}")
    private String[] allowedOrigins;

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")          // Aplica a todas las rutas bajo /api/
                    .allowedOrigins(allowedOrigins)     // Orígenes permitidos (desde config)
                    .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS")
                    .allowedHeaders("*")                // Permite cualquier header
                    .allowCredentials(true)             // Permite enviar cookies y Authorization
                    .maxAge(3600);                      // Cache del preflight por 1 hora
            }
        };
    }
}
```

### 3.4 Configuración CORS integrada con Spring Security (la correcta para nuestro proyecto)

Cuando Spring Security está activo, **debes** configurar CORS dentro del `SecurityFilterChain`. Si lo configuras solo en `WebMvcConfigurer`, Spring Security puede bloquear las peticiones preflight antes de que lleguen a tu configuración CORS.

La razón: Spring Security tiene su propio filter chain que actúa antes que Spring MVC.

```java
// src/main/java/co/uniquindio/triage/config/SecurityConfig.java

package co.uniquindio.triage.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import java.util.List;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity   // Habilita @PreAuthorize en los controllers
public class SecurityConfig {

    @Value("${app.cors.allowed-origins}")
    private List<String> allowedOrigins;

    private final JwtAuthenticationFilter jwtAuthenticationFilter;

    public SecurityConfig(JwtAuthenticationFilter jwtAuthenticationFilter) {
        this.jwtAuthenticationFilter = jwtAuthenticationFilter;
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
            // 1. Activar y configurar CORS — debe ir ANTES del csrf
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))

            // 2. Deshabilitar CSRF: usamos JWT stateless, no necesitamos CSRF
            .csrf(csrf -> csrf.disable())

            // 3. No guardar sesión en servidor: somos stateless con JWT
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))

            // 4. Reglas de autorización
            .authorizeHttpRequests(auth -> auth
                // Rutas públicas: no necesitan autenticación
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers("/v3/api-docs/**", "/swagger-ui/**").permitAll()
                .requestMatchers("/actuator/health").permitAll()
                // Todo lo demás requiere autenticación
                .anyRequest().authenticated()
            )

            // 5. Agregar nuestro filtro JWT antes del filtro de autenticación por usuario/clave
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)

            .build();
    }

    /**
     * Fuente de configuración CORS.
     * Se define como Bean para poder reutilizarlo si es necesario.
     */
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        var configuration = new CorsConfiguration();

        // Orígenes permitidos: Angular en dev y la URL de producción
        configuration.setAllowedOrigins(allowedOrigins);

        // Métodos HTTP permitidos
        configuration.setAllowedMethods(List.of(
            "GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"
        ));

        // Headers que el cliente puede enviar
        // "Authorization" es fundamental para el JWT
        configuration.setAllowedHeaders(List.of(
            "Authorization",
            "Content-Type",
            "Accept",
            "X-Requested-With"
        ));

        // Headers que el cliente puede leer de la respuesta
        configuration.setExposedHeaders(List.of(
            "Authorization",
            "X-Total-Count"    // Útil para paginación
        ));

        // Permitir credenciales (necesario para JWT en header Authorization)
        configuration.setAllowCredentials(true);

        // Cuánto tiempo puede el navegador cachear el resultado del preflight
        configuration.setMaxAge(3600L);

        var source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", configuration);
        return source;
    }
}
```

### 3.5 Variables de configuración CORS en application.yml

```yaml
# src/main/resources/application.yml

app:
  cors:
    allowed-origins:
      - "http://localhost:4200"     # Angular en desarrollo
      - "http://localhost:3000"     # Por si usas otro puerto
      # En producción añadirías la URL real del frontend:
      # - "https://triage.uniquindio.edu.co"

---
# src/main/resources/application-prod.yml
# Este archivo sobreescribe la configuración en perfil de producción

app:
  cors:
    allowed-origins:
      - "https://triage-frontend.railway.app"
      - "https://triage.uniquindio.edu.co"
```

### 3.6 Antes vs Ahora: WebSecurityConfigurerAdapter (deprecado)

| Forma antigua (Spring Security 5.x) | Forma nueva (Spring Security 6.x) |
|---|---|
| Extender `WebSecurityConfigurerAdapter` | Crear un `@Bean SecurityFilterChain` |
| `@Override configure(HttpSecurity http)` | Lambda con `HttpSecurity` como parámetro |
| `.cors().and().csrf().disable()` | `.cors(cors -> ...)` y `.csrf(csrf -> ...)` |
| Configuración CORS separada podía funcionar | CORS **debe** estar en el `SecurityFilterChain` |
| `HttpSecurity.antMatchers()` | `HttpSecurity.requestMatchers()` |

```java
// ❌ FORMA ANTIGUA — DEPRECADA — NO USAR
@Configuration
public class OldSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.cors().and().csrf().disable()
            .authorizeRequests()
            .antMatchers("/api/auth/**").permitAll()
            .anyRequest().authenticated();
        // ⚠️ WebSecurityConfigurerAdapter fue eliminado en Spring Boot 3.x
    }
}

// ✅ FORMA NUEVA — Spring Boot 3.4+ / Spring Security 6+
@Configuration
@EnableWebSecurity
public class NewSecurityConfig {
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .build();
    }
}
```

---

## 4. Variables de Entorno en Angular

### 4.1 ¿Por qué necesitas variables de entorno?

Imagina que en tu código Angular tienes la URL del backend hardcodeada:

```typescript
// ❌ Mal: URL fija en el código
this.http.get('http://localhost:8080/api/v1/solicitudes')
```

Cuando subas el proyecto a producción, el backend ya no está en `localhost:8080`. Tendrías que buscar y reemplazar esa URL en cada servicio. Con variables de entorno, cambias un solo lugar y toda la app se adapta.

### 4.2 Estructura de archivos de entorno en Angular 19+

```
triage-frontend/
└── src/
    └── environments/
        ├── environment.ts          ← Configuración para DESARROLLO
        └── environment.prod.ts     ← Configuración para PRODUCCIÓN
```

```typescript
// src/environments/environment.ts — DESARROLLO

export const environment = {
  production: false,
  apiUrl: 'http://localhost:8080/api/v1',
  appName: 'Sistema de Triage — DEV',
  // Tiempo de expiración del token en milisegundos (para refrescar)
  tokenExpirationBuffer: 60000,   // 1 minuto antes de que expire
};
```

```typescript
// src/environments/environment.prod.ts — PRODUCCIÓN

export const environment = {
  production: true,
  apiUrl: 'https://triage-backend.railway.app/api/v1',
  appName: 'Sistema de Triage Académico',
  tokenExpirationBuffer: 60000,
};
```

### 4.3 Configurar el file replacement en angular.json

Angular sabe qué archivo de entorno usar gracias a `angular.json`. Cuando ejecutas `ng build --configuration production`, Angular automáticamente reemplaza `environment.ts` con `environment.prod.ts`.

```json
// angular.json (fragmento relevante)
{
  "configurations": {
    "production": {
      "fileReplacements": [
        {
          "replace": "src/environments/environment.ts",
          "with": "src/environments/environment.prod.ts"
        }
      ],
      "optimization": true,
      "outputHashing": "all"
    }
  }
}
```

### 4.4 Crear un token de inyección para la URL base

En lugar de importar `environment` directamente en cada servicio (lo cual dificulta los tests), es buena práctica usar un token de inyección:

```typescript
// src/app/core/tokens/api.tokens.ts

import { InjectionToken } from '@angular/core';

export const API_URL = new InjectionToken<string>('API_URL');
```

```typescript
// src/app/app.config.ts

import { ApplicationConfig } from '@angular/core';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';
import { environment } from '../environments/environment';
import { API_URL } from './core/tokens/api.tokens';
import { jwtInterceptor } from './core/interceptors/jwt.interceptor';
import { errorInterceptor } from './core/interceptors/error.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(
      withInterceptors([jwtInterceptor, errorInterceptor])
    ),
    // Registrar la URL base como valor inyectable
    { provide: API_URL, useValue: environment.apiUrl },
  ],
};
```

```typescript
// src/app/core/services/solicitud.service.ts

import { inject, Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { API_URL } from '../tokens/api.tokens';

@Injectable({ providedIn: 'root' })
export class SolicitudService {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = `${inject(API_URL)}/solicitudes`;

  // Ahora la URL base viene de la inyección, no hardcodeada
  obtenerTodas() {
    return this.http.get(this.baseUrl);
  }
}
```

---

## 5. Flujo de Autenticación Completo Implementado

Vamos a implementar el flujo completo de autenticación de extremo a extremo. Esto incluye el backend (Spring Boot) y el frontend (Angular) trabajando juntos.

### 5.1 Estructura de carpetas del proyecto completo

Antes de empezar, veamos cómo están organizados ambos proyectos:

```
triage-backend/
├── src/main/java/co/uniquindio/triage/
│   ├── config/
│   │   ├── SecurityConfig.java
│   │   ├── CorsConfig.java            ← (Ya no lo necesitas si CORS está en SecurityConfig)
│   │   └── OpenApiConfig.java
│   ├── auth/
│   │   ├── controller/
│   │   │   └── AuthController.java
│   │   ├── dto/
│   │   │   ├── LoginRequest.java      ← Record
│   │   │   ├── LoginResponse.java     ← Record
│   │   │   └── RegisterRequest.java   ← Record
│   │   ├── service/
│   │   │   └── AuthService.java
│   │   └── filter/
│   │       └── JwtAuthenticationFilter.java
│   ├── solicitud/
│   │   ├── controller/
│   │   ├── dto/
│   │   ├── entity/
│   │   ├── repository/
│   │   └── service/
│   └── shared/
│       ├── exception/
│       ├── dto/
│       └── util/
│
triage-frontend/
├── src/app/
│   ├── core/
│   │   ├── interceptors/
│   │   │   ├── jwt.interceptor.ts
│   │   │   └── error.interceptor.ts
│   │   ├── guards/
│   │   │   ├── auth.guard.ts
│   │   │   └── role.guard.ts
│   │   ├── services/
│   │   │   ├── auth.service.ts
│   │   │   └── solicitud.service.ts
│   │   ├── models/
│   │   │   ├── solicitud.model.ts
│   │   │   ├── usuario.model.ts
│   │   │   └── auth.model.ts
│   │   └── tokens/
│   │       └── api.tokens.ts
│   ├── features/
│   │   ├── auth/
│   │   │   ├── login/
│   │   │   └── register/
│   │   └── solicitudes/
│   │       ├── lista-solicitudes/
│   │       ├── crear-solicitud/
│   │       ├── detalle-solicitud/
│   │       └── clasificar-solicitud/
│   └── shared/
│       ├── components/
│       └── pipes/
```

### 5.2 Backend: AuthController y AuthService

```java
// src/main/java/co/uniquindio/triage/auth/dto/LoginRequest.java
// Record Java 21: inmutable, compacto, perfecto para DTOs de entrada

package co.uniquindio.triage.auth.dto;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public record LoginRequest(
    @NotBlank(message = "El correo es obligatorio")
    @Email(message = "Formato de correo inválido")
    String correo,

    @NotBlank(message = "La contraseña es obligatoria")
    @Size(min = 6, message = "La contraseña debe tener al menos 6 caracteres")
    String password
) {}
```

```java
// src/main/java/co/uniquindio/triage/auth/dto/LoginResponse.java

package co.uniquindio.triage.auth.dto;

public record LoginResponse(
    String token,
    String tipo,           // "Bearer"
    Long expiracionMs,     // Tiempo de expiración en milisegundos
    String nombre,
    String correo,
    String rol
) {}
```

```java
// src/main/java/co/uniquindio/triage/auth/dto/RegisterRequest.java

package co.uniquindio.triage.auth.dto;

import jakarta.validation.constraints.*;

public record RegisterRequest(
    @NotBlank(message = "El nombre es obligatorio")
    @Size(min = 2, max = 100)
    String nombre,

    @NotBlank(message = "El correo es obligatorio")
    @Email(message = "Formato de correo inválido")
    String correo,

    @NotBlank(message = "La contraseña es obligatoria")
    @Size(min = 6, max = 50)
    String password,

    @NotBlank(message = "El código de estudiante es obligatorio")
    @Pattern(regexp = "\\d{8}", message = "El código debe tener 8 dígitos")
    String codigoEstudiante
) {}
```

```java
// src/main/java/co/uniquindio/triage/auth/controller/AuthController.java

package co.uniquindio.triage.auth.controller;

import co.uniquindio.triage.auth.dto.LoginRequest;
import co.uniquindio.triage.auth.dto.LoginResponse;
import co.uniquindio.triage.auth.dto.RegisterRequest;
import co.uniquindio.triage.auth.service.AuthService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/auth")
@Tag(name = "Autenticación", description = "Endpoints de registro e inicio de sesión")
public class AuthController {

    private final AuthService authService;

    public AuthController(AuthService authService) {
        this.authService = authService;
    }

    @PostMapping("/login")
    @Operation(summary = "Iniciar sesión", description = "Devuelve un token JWT si las credenciales son válidas")
    public ResponseEntity<LoginResponse> login(@Valid @RequestBody LoginRequest request) {
        var response = authService.login(request);
        return ResponseEntity.ok(response);
    }

    @PostMapping("/register")
    @Operation(summary = "Registrar nuevo estudiante")
    public ResponseEntity<Void> register(@Valid @RequestBody RegisterRequest request) {
        authService.registrar(request);
        // 201 Created: el recurso fue creado exitosamente
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
}
```

```java
// src/main/java/co/uniquindio/triage/auth/service/AuthService.java

package co.uniquindio.triage.auth.service;

import co.uniquindio.triage.auth.dto.LoginRequest;
import co.uniquindio.triage.auth.dto.LoginResponse;
import co.uniquindio.triage.auth.dto.RegisterRequest;
import co.uniquindio.triage.shared.exception.ConflictoException;
import co.uniquindio.triage.shared.exception.CredencialesInvalidasException;
import co.uniquindio.triage.usuario.entity.Rol;
import co.uniquindio.triage.usuario.entity.Usuario;
import co.uniquindio.triage.usuario.repository.UsuarioRepository;
import co.uniquindio.triage.shared.util.JwtUtil;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class AuthService {

    private final UsuarioRepository usuarioRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtUtil jwtUtil;

    public AuthService(UsuarioRepository usuarioRepository,
                       PasswordEncoder passwordEncoder,
                       JwtUtil jwtUtil) {
        this.usuarioRepository = usuarioRepository;
        this.passwordEncoder = passwordEncoder;
        this.jwtUtil = jwtUtil;
    }

    /**
     * Autentica al usuario y genera un token JWT.
     * Proceso:
     * 1. Buscar el usuario por correo
     * 2. Verificar que la contraseña coincida (BCrypt)
     * 3. Generar y devolver el token JWT
     */
    @Transactional(readOnly = true)
    public LoginResponse login(LoginRequest request) {
        // Buscar usuario: si no existe, credenciales inválidas
        var usuario = usuarioRepository.findByCorreo(request.correo())
            .orElseThrow(() -> new CredencialesInvalidasException(
                "Correo o contraseña incorrectos"
            ));

        // Verificar contraseña con BCrypt
        // NUNCA comparar strings directamente: siempre usar passwordEncoder.matches()
        if (!passwordEncoder.matches(request.password(), usuario.getPasswordHash())) {
            throw new CredencialesInvalidasException("Correo o contraseña incorrectos");
        }

        // Generar JWT
        var token = jwtUtil.generarToken(usuario);
        var expiracion = jwtUtil.obtenerExpiracionMs();

        return new LoginResponse(
            token,
            "Bearer",
            expiracion,
            usuario.getNombre(),
            usuario.getCorreo(),
            usuario.getRol().name()
        );
    }

    /**
     * Registra un nuevo estudiante.
     * Los nuevos registros siempre tienen rol ESTUDIANTE.
     */
    @Transactional
    public void registrar(RegisterRequest request) {
        // Verificar que el correo no esté en uso
        if (usuarioRepository.existsByCorreo(request.correo())) {
            throw new ConflictoException(
                "Ya existe una cuenta con el correo: " + request.correo()
            );
        }

        var usuario = new Usuario();
        usuario.setNombre(request.nombre());
        usuario.setCorreo(request.correo());
        // Hashear la contraseña ANTES de guardar
        usuario.setPasswordHash(passwordEncoder.encode(request.password()));
        usuario.setCodigoEstudiante(request.codigoEstudiante());
        usuario.setRol(Rol.ESTUDIANTE);   // Siempre empieza como ESTUDIANTE
        usuario.setActivo(true);

        usuarioRepository.save(usuario);
    }
}
```

### 5.3 Frontend: AuthService en Angular

```typescript
// src/app/core/models/auth.model.ts

export interface LoginRequest {
  correo: string;
  password: string;
}

export interface RegisterRequest {
  nombre: string;
  correo: string;
  password: string;
  codigoEstudiante: string;
}

export interface LoginResponse {
  token: string;
  tipo: string;           // "Bearer"
  expiracionMs: number;
  nombre: string;
  correo: string;
  rol: string;
}

// El estado de autenticación que compartimos en la app
export interface AuthState {
  autenticado: boolean;
  usuario: UsuarioAutenticado | null;
  token: string | null;
}

export interface UsuarioAutenticado {
  nombre: string;
  correo: string;
  rol: string;
}
```

```typescript
// src/app/core/services/auth.service.ts

import { inject, Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Router } from '@angular/router';
import { BehaviorSubject, Observable, tap } from 'rxjs';
import { API_URL } from '../tokens/api.tokens';
import {
  AuthState,
  LoginRequest,
  LoginResponse,
  RegisterRequest,
  UsuarioAutenticado,
} from '../models/auth.model';

@Injectable({ providedIn: 'root' })
export class AuthService {
  private readonly http = inject(HttpClient);
  private readonly router = inject(Router);
  private readonly baseUrl = `${inject(API_URL)}/auth`;

  // Claves para localStorage
  private readonly TOKEN_KEY = 'triage_token';
  private readonly USER_KEY = 'triage_user';

  // BehaviorSubject: mantiene el estado actual y notifica a todos los suscriptores
  // cuando cambia. Es como un "almacén de estado" reactivo.
  private readonly authStateSubject = new BehaviorSubject<AuthState>(
    this.recuperarEstadoInicial()
  );

  // Observable público: los componentes se suscriben a esto para reaccionar a cambios
  readonly authState$ = this.authStateSubject.asObservable();

  /**
   * Verifica si hay una sesión guardada al iniciar la app.
   * Esto permite que si el usuario recarga la página, siga autenticado.
   */
  private recuperarEstadoInicial(): AuthState {
    const token = localStorage.getItem(this.TOKEN_KEY);
    const usuarioJson = localStorage.getItem(this.USER_KEY);

    if (token && usuarioJson) {
      try {
        const usuario: UsuarioAutenticado = JSON.parse(usuarioJson);
        // Verificar que el token no haya expirado
        if (this.tokenEsValido(token)) {
          return { autenticado: true, usuario, token };
        }
      } catch {
        // JSON inválido en localStorage: limpiar y empezar de cero
        this.limpiarStorage();
      }
    }

    return { autenticado: false, usuario: null, token: null };
  }

  /**
   * Decodifica el JWT y verifica si no ha expirado.
   * El JWT tiene formato: header.payload.signature
   * El payload está en base64 y contiene el campo "exp" (expiración en segundos Unix)
   */
  private tokenEsValido(token: string): boolean {
    try {
      const payload = JSON.parse(atob(token.split('.')[1]));
      const ahora = Math.floor(Date.now() / 1000);
      return payload.exp > ahora;
    } catch {
      return false;
    }
  }

  /**
   * Realiza el login: llama al backend y guarda el token si tiene éxito.
   * El operador tap() ejecuta un efecto secundario sin alterar el observable.
   */
  login(credenciales: LoginRequest): Observable<LoginResponse> {
    return this.http
      .post<LoginResponse>(`${this.baseUrl}/login`, credenciales)
      .pipe(
        tap((response) => {
          // Guardar token en localStorage para persistencia
          localStorage.setItem(this.TOKEN_KEY, response.token);

          const usuario: UsuarioAutenticado = {
            nombre: response.nombre,
            correo: response.correo,
            rol: response.rol,
          };
          localStorage.setItem(this.USER_KEY, JSON.stringify(usuario));

          // Actualizar el estado global de autenticación
          this.authStateSubject.next({
            autenticado: true,
            usuario,
            token: response.token,
          });
        })
      );
  }

  register(datos: RegisterRequest): Observable<void> {
    return this.http.post<void>(`${this.baseUrl}/register`, datos);
  }

  /**
   * Cierra la sesión: limpia localStorage y actualiza el estado.
   */
  logout(): void {
    this.limpiarStorage();
    this.authStateSubject.next({
      autenticado: false,
      usuario: null,
      token: null,
    });
    this.router.navigate(['/login']);
  }

  private limpiarStorage(): void {
    localStorage.removeItem(this.TOKEN_KEY);
    localStorage.removeItem(this.USER_KEY);
  }

  obtenerToken(): string | null {
    return localStorage.getItem(this.TOKEN_KEY);
  }

  get estaAutenticado(): boolean {
    return this.authStateSubject.value.autenticado;
  }

  get usuarioActual(): UsuarioAutenticado | null {
    return this.authStateSubject.value.usuario;
  }

  tieneRol(rol: string): boolean {
    return this.usuarioActual?.rol === rol;
  }
}
```

### 5.4 El Interceptor JWT

```typescript
// src/app/core/interceptors/jwt.interceptor.ts

import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { AuthService } from '../services/auth.service';

/**
 * Interceptor funcional (Angular 17+): adjunta el token JWT a todas las peticiones.
 *
 * Este es un "functional interceptor": una función, no una clase.
 * Es la forma moderna recomendada en Angular 17+.
 *
 * Compara con la forma antigua basada en clases que implementaban HttpInterceptor.
 */
export const jwtInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.obtenerToken();

  // Si hay token, clonar la petición y agregar el header Authorization
  // IMPORTANTE: Las peticiones HTTP son inmutables en Angular.
  // Por eso debemos clonarlas (clone()) para modificarlas.
  if (token) {
    const requestConToken = req.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`,
      },
    });
    return next(requestConToken);
  }

  // Si no hay token (usuario no autenticado), pasar la petición sin modificar
  // Esto aplica para las rutas públicas como /auth/login y /auth/register
  return next(req);
};
```

### 5.5 El Interceptor de Errores

```typescript
// src/app/core/interceptors/error.interceptor.ts

import { HttpInterceptorFn, HttpErrorResponse } from '@angular/common/http';
import { inject } from '@angular/core';
import { Router } from '@angular/router';
import { catchError, throwError } from 'rxjs';
import { AuthService } from '../services/auth.service';

export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const router = inject(Router);
  const authService = inject(AuthService);

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      switch (error.status) {
        case 401:
          // No autenticado: el token venció o no se envió
          // Limpiar la sesión y redirigir al login
          authService.logout();
          router.navigate(['/login'], {
            queryParams: { motivo: 'sesion-expirada' },
          });
          break;

        case 403:
          // Autenticado pero sin permiso: redirigir a página de acceso denegado
          router.navigate(['/acceso-denegado']);
          break;

        case 0:
          // Error de red: el servidor no respondió (sin conexión, CORS bloqueado)
          console.error('Error de conexión: el servidor no está disponible');
          break;

        case 500:
          console.error('Error interno del servidor:', error.error);
          break;
      }

      // Propagar el error para que los componentes puedan manejarlo si quieren
      return throwError(() => error);
    })
  );
};
```

### 5.6 Guards de Rutas

```typescript
// src/app/core/guards/auth.guard.ts

import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

/**
 * Guard funcional (Angular 17+): protege rutas que requieren autenticación.
 * Si el usuario no está autenticado, lo redirige al login.
 */
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.estaAutenticado) {
    return true;  // Permitir acceso
  }

  // Guardar la URL a la que intentaba acceder para redirigir después del login
  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url },
  });
};
```

```typescript
// src/app/core/guards/role.guard.ts

import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

/**
 * Guard de roles: verifica que el usuario tenga el rol requerido.
 * El rol permitido se pasa en el campo "data" de la ruta.
 *
 * Ejemplo de uso en routes:
 * {
 *   path: 'clasificar',
 *   component: ClasificarComponent,
 *   canActivate: [authGuard, roleGuard],
 *   data: { roles: ['FUNCIONARIO', 'ADMIN'] }
 * }
 */
export const roleGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  // Obtener los roles permitidos de la configuración de la ruta
  const rolesPermitidos: string[] = route.data?.['roles'] ?? [];

  if (rolesPermitidos.length === 0) {
    return true;  // Sin restricción de roles
  }

  const rolUsuario = authService.usuarioActual?.rol;

  if (rolUsuario && rolesPermitidos.includes(rolUsuario)) {
    return true;
  }

  return router.createUrlTree(['/acceso-denegado']);
};
```

```typescript
// src/app/app.routes.ts

import { Routes } from '@angular/router';
import { authGuard } from './core/guards/auth.guard';
import { roleGuard } from './core/guards/role.guard';

export const routes: Routes = [
  {
    path: '',
    redirectTo: '/dashboard',
    pathMatch: 'full',
  },
  {
    path: 'login',
    loadComponent: () =>
      import('./features/auth/login/login.component').then(
        (m) => m.LoginComponent
      ),
  },
  {
    path: 'register',
    loadComponent: () =>
      import('./features/auth/register/register.component').then(
        (m) => m.RegisterComponent
      ),
  },
  {
    path: 'dashboard',
    canActivate: [authGuard],
    loadComponent: () =>
      import('./features/dashboard/dashboard.component').then(
        (m) => m.DashboardComponent
      ),
  },
  {
    path: 'solicitudes',
    canActivate: [authGuard],
    children: [
      {
        path: '',
        loadComponent: () =>
          import('./features/solicitudes/lista-solicitudes/lista-solicitudes.component').then(
            (m) => m.ListaSolicitudesComponent
          ),
      },
      {
        path: 'nueva',
        // Solo ESTUDIANTE puede crear solicitudes
        canActivate: [roleGuard],
        data: { roles: ['ESTUDIANTE'] },
        loadComponent: () =>
          import('./features/solicitudes/crear-solicitud/crear-solicitud.component').then(
            (m) => m.CrearSolicitudComponent
          ),
      },
      {
        path: ':id',
        loadComponent: () =>
          import('./features/solicitudes/detalle-solicitud/detalle-solicitud.component').then(
            (m) => m.DetalleSolicitudComponent
          ),
      },
      {
        path: ':id/clasificar',
        // Solo FUNCIONARIO y ADMIN pueden clasificar
        canActivate: [roleGuard],
        data: { roles: ['FUNCIONARIO', 'ADMIN'] },
        loadComponent: () =>
          import('./features/solicitudes/clasificar-solicitud/clasificar-solicitud.component').then(
            (m) => m.ClasificarSolicitudComponent
          ),
      },
    ],
  },
  {
    path: 'acceso-denegado',
    loadComponent: () =>
      import('./shared/components/acceso-denegado/acceso-denegado.component').then(
        (m) => m.AccesoDenegadoComponent
      ),
  },
  {
    path: '**',
    redirectTo: '/dashboard',
  },
];
```

### 5.7 Componente de Login Completo

```typescript
// src/app/features/auth/login/login.component.ts

import { Component, inject, OnInit } from '@angular/core';
import { FormBuilder, ReactiveFormsModule, Validators } from '@angular/forms';
import { Router, ActivatedRoute, RouterLink } from '@angular/router';
import { CommonModule } from '@angular/common';
import { AuthService } from '../../../core/services/auth.service';
import { HttpErrorResponse } from '@angular/common/http';

@Component({
  selector: 'app-login',
  standalone: true,
  imports: [ReactiveFormsModule, CommonModule, RouterLink],
  template: `
    <div class="login-container">
      <div class="login-card">
        <h1>Sistema de Triage</h1>
        <h2>Iniciar Sesión</h2>

        <!-- Mensaje de sesión expirada -->
        @if (sesionExpirada) {
          <div class="alert alert-warning">
            Tu sesión ha expirado. Por favor inicia sesión nuevamente.
          </div>
        }

        <!-- Mensaje de error de credenciales -->
        @if (errorMensaje) {
          <div class="alert alert-error">{{ errorMensaje }}</div>
        }

        <form [formGroup]="loginForm" (ngSubmit)="onSubmit()">
          <div class="form-group">
            <label for="correo">Correo institucional</label>
            <input
              id="correo"
              type="email"
              formControlName="correo"
              placeholder="usuario@uniquindio.edu.co"
              [class.input-error]="correo.invalid && correo.touched"
            />
            @if (correo.invalid && correo.touched) {
              <span class="error-text">
                @if (correo.errors?.['required']) { El correo es obligatorio }
                @if (correo.errors?.['email']) { Formato de correo inválido }
              </span>
            }
          </div>

          <div class="form-group">
            <label for="password">Contraseña</label>
            <input
              id="password"
              type="password"
              formControlName="password"
              [class.input-error]="password.invalid && password.touched"
            />
            @if (password.invalid && password.touched) {
              <span class="error-text">
                @if (password.errors?.['required']) { La contraseña es obligatoria }
                @if (password.errors?.['minlength']) { Mínimo 6 caracteres }
              </span>
            }
          </div>

          <button
            type="submit"
            [disabled]="loginForm.invalid || cargando"
            class="btn-primary"
          >
            @if (cargando) {
              <span>Ingresando...</span>
            } @else {
              <span>Ingresar</span>
            }
          </button>
        </form>

        <p>¿No tienes cuenta? <a routerLink="/register">Regístrate aquí</a></p>
      </div>
    </div>
  `,
  styleUrl: './login.component.scss',
})
export class LoginComponent implements OnInit {
  private readonly fb = inject(FormBuilder);
  private readonly authService = inject(AuthService);
  private readonly router = inject(Router);
  private readonly route = inject(ActivatedRoute);

  cargando = false;
  errorMensaje = '';
  sesionExpirada = false;
  private returnUrl = '/dashboard';

  loginForm = this.fb.group({
    correo: ['', [Validators.required, Validators.email]],
    password: ['', [Validators.required, Validators.minLength(6)]],
  });

  // Getters para acceder fácilmente a los controles en el template
  get correo() { return this.loginForm.controls.correo; }
  get password() { return this.loginForm.controls.password; }

  ngOnInit(): void {
    // Si el usuario ya está autenticado, redirigir al dashboard
    if (this.authService.estaAutenticado) {
      this.router.navigate(['/dashboard']);
      return;
    }

    // Revisar si viene de una sesión expirada o de una URL protegida
    this.route.queryParams.subscribe((params) => {
      this.sesionExpirada = params['motivo'] === 'sesion-expirada';
      this.returnUrl = params['returnUrl'] ?? '/dashboard';
    });
  }

  onSubmit(): void {
    if (this.loginForm.invalid) return;

    this.cargando = true;
    this.errorMensaje = '';

    const { correo, password } = this.loginForm.value;

    this.authService.login({ correo: correo!, password: password! }).subscribe({
      next: () => {
        // Login exitoso: redirigir a la URL de destino
        this.router.navigate([this.returnUrl]);
      },
      error: (err: HttpErrorResponse) => {
        this.cargando = false;
        if (err.status === 401) {
          this.errorMensaje = 'Correo o contraseña incorrectos';
        } else {
          this.errorMensaje = 'Error al conectar con el servidor. Intenta más tarde.';
        }
      },
      complete: () => {
        this.cargando = false;
      },
    });
  }
}
```

---

## 6. Caso de Uso E2E #1: Estudiante Crea una Solicitud de Homologación

Este es el caso de uso más importante del sistema. Vamos a implementarlo de extremo a extremo, capa por capa.

### 6.1 Backend: Entidad, DTO, Repository, Service y Controller

#### Las entidades del dominio

```java
// src/main/java/co/uniquindio/triage/solicitud/entity/TipoSolicitud.java

package co.uniquindio.triage.solicitud.entity;

public enum TipoSolicitud {
    HOMOLOGACION("Homologación de asignatura"),
    REGISTRO_ASIGNATURA("Registro de asignatura"),
    CANCELACION_ASIGNATURA("Cancelación de asignatura"),
    SOLICITUD_CUPO("Solicitud de cupo"),
    CONSULTA_ACADEMICA("Consulta académica"),
    OTRO("Otro");

    private final String descripcionAmigable;

    TipoSolicitud(String descripcionAmigable) {
        this.descripcionAmigable = descripcionAmigable;
    }

    public String getDescripcionAmigable() {
        return descripcionAmigable;
    }
}
```

```java
// src/main/java/co/uniquindio/triage/solicitud/entity/EstadoSolicitud.java

package co.uniquindio.triage.solicitud.entity;

public enum EstadoSolicitud {
    REGISTRADA,
    CLASIFICADA,
    EN_ATENCION,
    ATENDIDA,
    CERRADA
}
```

```java
// src/main/java/co/uniquindio/triage/solicitud/entity/Prioridad.java

package co.uniquindio.triage.solicitud.entity;

public enum Prioridad {
    BAJA(1),
    MEDIA(2),
    ALTA(3),
    CRITICA(4);

    private final int nivel;

    Prioridad(int nivel) { this.nivel = nivel; }

    public int getNivel() { return nivel; }
}
```

```java
// src/main/java/co/uniquindio/triage/solicitud/entity/CanalOrigen.java

package co.uniquindio.triage.solicitud.entity;

public enum CanalOrigen {
    PRESENCIAL,
    CORREO,
    SAC,
    TELEFONICO,
    PLATAFORMA_WEB
}
```

```java
// src/main/java/co/uniquindio/triage/solicitud/entity/Solicitud.java

package co.uniquindio.triage.solicitud.entity;

import co.uniquindio.triage.usuario.entity.Usuario;
import jakarta.persistence.*;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;
import java.util.UUID;

@Entity
@Table(name = "solicitudes", indexes = {
    @Index(name = "idx_solicitudes_estado", columnList = "estado"),
    @Index(name = "idx_solicitudes_tipo", columnList = "tipo"),
    @Index(name = "idx_solicitudes_solicitante", columnList = "solicitante_id"),
})
@EntityListeners(AuditingEntityListener.class)
public class Solicitud {

    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;

    @Column(nullable = false, length = 1000)
    private String descripcion;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private TipoSolicitud tipo;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private EstadoSolicitud estado;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private CanalOrigen canalOrigen;

    @Enumerated(EnumType.STRING)
    private Prioridad prioridad;

    // Quién hizo la solicitud (ESTUDIANTE)
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "solicitante_id", nullable = false)
    private Usuario solicitante;

    // A quién se le asignó para atender (FUNCIONARIO o ADMIN)
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "responsable_id")
    private Usuario responsable;

    // Historial de acciones sobre esta solicitud (RF-06)
    @OneToMany(mappedBy = "solicitud", cascade = CascadeType.ALL, orphanRemoval = true)
    @OrderBy("fechaAccion ASC")
    private List<HistorialSolicitud> historial = new ArrayList<>();

    @CreatedDate
    @Column(nullable = false, updatable = false)
    private LocalDateTime fechaRegistro;

    @LastModifiedDate
    @Column(nullable = false)
    private LocalDateTime fechaUltimaModificacion;

    private String observacionCierre;

    // Getters y Setters (puedes usar Lombok @Getter @Setter si lo prefieres)
    public UUID getId() { return id; }
    public String getDescripcion() { return descripcion; }
    public void setDescripcion(String descripcion) { this.descripcion = descripcion; }
    public TipoSolicitud getTipo() { return tipo; }
    public void setTipo(TipoSolicitud tipo) { this.tipo = tipo; }
    public EstadoSolicitud getEstado() { return estado; }
    public void setEstado(EstadoSolicitud estado) { this.estado = estado; }
    public CanalOrigen getCanalOrigen() { return canalOrigen; }
    public void setCanalOrigen(CanalOrigen canalOrigen) { this.canalOrigen = canalOrigen; }
    public Prioridad getPrioridad() { return prioridad; }
    public void setPrioridad(Prioridad prioridad) { this.prioridad = prioridad; }
    public Usuario getSolicitante() { return solicitante; }
    public void setSolicitante(Usuario solicitante) { this.solicitante = solicitante; }
    public Usuario getResponsable() { return responsable; }
    public void setResponsable(Usuario responsable) { this.responsable = responsable; }
    public List<HistorialSolicitud> getHistorial() { return historial; }
    public LocalDateTime getFechaRegistro() { return fechaRegistro; }
    public LocalDateTime getFechaUltimaModificacion() { return fechaUltimaModificacion; }
    public String getObservacionCierre() { return observacionCierre; }
    public void setObservacionCierre(String observacionCierre) { this.observacionCierre = observacionCierre; }
}
```

#### DTOs del dominio de solicitud (con Records de Java 21)

```java
// src/main/java/co/uniquindio/triage/solicitud/dto/CrearSolicitudRequest.java

package co.uniquindio.triage.solicitud.dto;

import co.uniquindio.triage.solicitud.entity.CanalOrigen;
import co.uniquindio.triage.solicitud.entity.TipoSolicitud;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;

/**
 * RF-01: Registro de solicitudes académicas
 * Los campos requeridos son: tipo, descripción, canal de origen.
 * El solicitante se toma del usuario autenticado en el contexto de seguridad.
 */
public record CrearSolicitudRequest(
    @NotNull(message = "El tipo de solicitud es obligatorio")
    TipoSolicitud tipo,

    @NotBlank(message = "La descripción es obligatoria")
    @Size(min = 10, max = 1000, message = "La descripción debe tener entre 10 y 1000 caracteres")
    String descripcion,

    @NotNull(message = "El canal de origen es obligatorio")
    CanalOrigen canalOrigen
) {}
```

```java
// src/main/java/co/uniquindio/triage/solicitud/dto/SolicitudResponse.java

package co.uniquindio.triage.solicitud.dto;

import co.uniquindio.triage.solicitud.entity.CanalOrigen;
import co.uniquindio.triage.solicitud.entity.EstadoSolicitud;
import co.uniquindio.triage.solicitud.entity.Prioridad;
import co.uniquindio.triage.solicitud.entity.TipoSolicitud;

import java.time.LocalDateTime;
import java.util.List;
import java.util.UUID;

/**
 * DTO de respuesta para una solicitud completa.
 * NUNCA exponemos la entidad JPA directamente: siempre usamos DTOs.
 * Razones:
 * 1. Evitar serialización de datos sensibles o innecesarios
 * 2. Desacoplar el modelo de BD del contrato de la API
 * 3. Evitar problemas con lazy loading de JPA (LazyInitializationException)
 */
public record SolicitudResponse(
    UUID id,
    TipoSolicitud tipo,
    String tipoDescripcion,         // "Homologación de asignatura"
    String descripcion,
    EstadoSolicitud estado,
    Prioridad prioridad,
    CanalOrigen canalOrigen,
    UsuarioResumenResponse solicitante,
    UsuarioResumenResponse responsable,
    LocalDateTime fechaRegistro,
    LocalDateTime fechaUltimaModificacion,
    List<HistorialSolicitudResponse> historial
) {}
```

```java
// src/main/java/co/uniquindio/triage/solicitud/dto/UsuarioResumenResponse.java

package co.uniquindio.triage.solicitud.dto;

import java.util.UUID;

// DTO compacto para representar un usuario dentro de una solicitud
public record UsuarioResumenResponse(
    UUID id,
    String nombre,
    String correo,
    String rol
) {}
```

```java
// src/main/java/co/uniquindio/triage/solicitud/dto/HistorialSolicitudResponse.java

package co.uniquindio.triage.solicitud.dto;

import java.time.LocalDateTime;
import java.util.UUID;

public record HistorialSolicitudResponse(
    UUID id,
    String accion,
    String observaciones,
    LocalDateTime fechaAccion,
    String usuarioResponsable
) {}
```

#### El Repository

```java
// src/main/java/co/uniquindio/triage/solicitud/repository/SolicitudRepository.java

package co.uniquindio.triage.solicitud.repository;

import co.uniquindio.triage.solicitud.entity.EstadoSolicitud;
import co.uniquindio.triage.solicitud.entity.Solicitud;
import co.uniquindio.triage.solicitud.entity.TipoSolicitud;
import co.uniquindio.triage.solicitud.entity.Prioridad;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.Optional;
import java.util.UUID;

@Repository
public interface SolicitudRepository extends JpaRepository<Solicitud, UUID> {

    /**
     * Buscar con filtros opcionales y paginación.
     * Usamos JPQL para hacer el JOIN FETCH del historial y evitar N+1 queries.
     *
     * Los parámetros opcionales se manejan con COALESCE o IS NULL checks:
     * si el parámetro es null, se ignora ese filtro.
     */
    @Query("""
        SELECT s FROM Solicitud s
        LEFT JOIN FETCH s.solicitante
        LEFT JOIN FETCH s.responsable
        WHERE (:estado IS NULL OR s.estado = :estado)
          AND (:tipo IS NULL OR s.tipo = :tipo)
          AND (:prioridad IS NULL OR s.prioridad = :prioridad)
        ORDER BY s.fechaRegistro DESC
        """)
    Page<Solicitud> buscarConFiltros(
        @Param("estado") EstadoSolicitud estado,
        @Param("tipo") TipoSolicitud tipo,
        @Param("prioridad") Prioridad prioridad,
        Pageable pageable
    );

    /**
     * Buscar por ID cargando el historial en la misma query (evita N+1)
     */
    @Query("""
        SELECT s FROM Solicitud s
        LEFT JOIN FETCH s.solicitante
        LEFT JOIN FETCH s.responsable
        LEFT JOIN FETCH s.historial h
        LEFT JOIN FETCH h.usuario
        WHERE s.id = :id
        """)
    Optional<Solicitud> findByIdConHistorial(@Param("id") UUID id);

    /**
     * Contar solicitudes por estado (útil para el dashboard)
     */
    long countByEstado(EstadoSolicitud estado);
}
```

#### El Service con lógica de negocio

```java
// src/main/java/co/uniquindio/triage/solicitud/service/SolicitudService.java

package co.uniquindio.triage.solicitud.service;

import co.uniquindio.triage.shared.exception.RecursoNoEncontradoException;
import co.uniquindio.triage.shared.exception.TransicionEstadoInvalidaException;
import co.uniquindio.triage.solicitud.dto.*;
import co.uniquindio.triage.solicitud.entity.*;
import co.uniquindio.triage.solicitud.repository.SolicitudRepository;
import co.uniquindio.triage.usuario.entity.Usuario;
import co.uniquindio.triage.usuario.repository.UsuarioRepository;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.UUID;

@Service
@Transactional   // Todas las operaciones de este Service son transaccionales por defecto
public class SolicitudService {

    private final SolicitudRepository solicitudRepository;
    private final UsuarioRepository usuarioRepository;
    private final SolicitudMapper mapper;

    public SolicitudService(SolicitudRepository solicitudRepository,
                            UsuarioRepository usuarioRepository,
                            SolicitudMapper mapper) {
        this.solicitudRepository = solicitudRepository;
        this.usuarioRepository = usuarioRepository;
        this.mapper = mapper;
    }

    /**
     * RF-01: Registrar una nueva solicitud académica.
     *
     * El solicitante es el usuario autenticado que está haciendo la petición.
     * No lo recibimos en el DTO: lo tomamos del contexto de seguridad de Spring.
     * Esto es más seguro: un usuario no puede crear solicitudes en nombre de otro.
     */
    public SolicitudResponse crear(CrearSolicitudRequest request) {
        // Obtener el usuario autenticado desde el contexto de Spring Security
        var correoAutenticado = SecurityContextHolder.getContext()
            .getAuthentication()
            .getName();

        var solicitante = usuarioRepository.findByCorreo(correoAutenticado)
            .orElseThrow(() -> new RecursoNoEncontradoException(
                "Usuario autenticado no encontrado: " + correoAutenticado
            ));

        // Construir la entidad
        var solicitud = new Solicitud();
        solicitud.setDescripcion(request.descripcion());
        solicitud.setTipo(request.tipo());
        solicitud.setCanalOrigen(request.canalOrigen());
        solicitud.setEstado(EstadoSolicitud.REGISTRADA);   // Estado inicial siempre REGISTRADA
        solicitud.setSolicitante(solicitante);

        // Registrar el primer evento en el historial
        agregarHistorial(solicitud, "Solicitud registrada", null, solicitante);

        var guardada = solicitudRepository.save(solicitud);
        return mapper.toResponse(guardada);
    }

    /**
     * Obtener una solicitud por ID con su historial completo.
     */
    @Transactional(readOnly = true)
    public SolicitudResponse obtenerPorId(UUID id) {
        var solicitud = solicitudRepository.findByIdConHistorial(id)
            .orElseThrow(() -> new RecursoNoEncontradoException(
                "Solicitud no encontrada con ID: " + id
            ));
        return mapper.toResponse(solicitud);
    }

    /**
     * RF-07: Consultar solicitudes con filtros y paginación.
     */
    @Transactional(readOnly = true)
    public Page<SolicitudResponse> buscarConFiltros(
        EstadoSolicitud estado,
        TipoSolicitud tipo,
        Prioridad prioridad,
        Pageable pageable
    ) {
        return solicitudRepository
            .buscarConFiltros(estado, tipo, prioridad, pageable)
            .map(mapper::toResponse);
    }

    /**
     * Método auxiliar para agregar una entrada al historial de la solicitud.
     * Se llama en cada cambio de estado o acción relevante.
     */
    private void agregarHistorial(Solicitud solicitud, String accion,
                                   String observaciones, Usuario usuario) {
        var entrada = new HistorialSolicitud();
        entrada.setSolicitud(solicitud);
        entrada.setAccion(accion);
        entrada.setObservaciones(observaciones);
        entrada.setUsuario(usuario);
        solicitud.getHistorial().add(entrada);
    }
}
```

#### El Controller

```java
// src/main/java/co/uniquindio/triage/solicitud/controller/SolicitudController.java

package co.uniquindio.triage.solicitud.controller;

import co.uniquindio.triage.solicitud.dto.CrearSolicitudRequest;
import co.uniquindio.triage.solicitud.dto.SolicitudResponse;
import co.uniquindio.triage.solicitud.entity.EstadoSolicitud;
import co.uniquindio.triage.solicitud.entity.Prioridad;
import co.uniquindio.triage.solicitud.entity.TipoSolicitud;
import co.uniquindio.triage.solicitud.service.SolicitudService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.security.SecurityRequirement;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

import java.util.UUID;

@RestController
@RequestMapping("/api/v1/solicitudes")
@Tag(name = "Solicitudes", description = "Gestión del ciclo de vida de solicitudes académicas")
@SecurityRequirement(name = "bearerAuth")   // Swagger sabe que necesita token JWT
public class SolicitudController {

    private final SolicitudService solicitudService;

    public SolicitudController(SolicitudService solicitudService) {
        this.solicitudService = solicitudService;
    }

    /**
     * RF-01: Crear una nueva solicitud.
     * Solo ESTUDIANTE puede crear solicitudes.
     */
    @PostMapping
    @PreAuthorize("hasRole('ESTUDIANTE')")
    @Operation(summary = "Registrar nueva solicitud académica (RF-01)")
    public ResponseEntity<SolicitudResponse> crear(
        @Valid @RequestBody CrearSolicitudRequest request
    ) {
        var solicitud = solicitudService.crear(request);
        // 201 Created con el recurso creado en el cuerpo
        return ResponseEntity.status(HttpStatus.CREATED).body(solicitud);
    }

    /**
     * Obtener una solicitud por ID.
     * Cualquier usuario autenticado puede consultar.
     */
    @GetMapping("/{id}")
    @Operation(summary = "Obtener solicitud por ID con historial completo")
    public ResponseEntity<SolicitudResponse> obtenerPorId(@PathVariable UUID id) {
        var solicitud = solicitudService.obtenerPorId(id);
        return ResponseEntity.ok(solicitud);
    }

    /**
     * RF-07: Consultar solicitudes con filtros.
     * Todos los usuarios autenticados pueden consultar, pero el Service
     * puede filtrar según el rol (el ESTUDIANTE solo ve sus propias solicitudes).
     */
    @GetMapping
    @Operation(summary = "Listar solicitudes con filtros opcionales (RF-07)")
    public ResponseEntity<Page<SolicitudResponse>> listar(
        @RequestParam(required = false) EstadoSolicitud estado,
        @RequestParam(required = false) TipoSolicitud tipo,
        @RequestParam(required = false) Prioridad prioridad,
        Pageable pageable
    ) {
        var solicitudes = solicitudService.buscarConFiltros(estado, tipo, prioridad, pageable);
        return ResponseEntity.ok(solicitudes);
    }
}
```

### 6.2 Frontend: Formulario de Creación de Solicitud

```typescript
// src/app/core/models/solicitud.model.ts

export type TipoSolicitud =
  | 'HOMOLOGACION'
  | 'REGISTRO_ASIGNATURA'
  | 'CANCELACION_ASIGNATURA'
  | 'SOLICITUD_CUPO'
  | 'CONSULTA_ACADEMICA'
  | 'OTRO';

export type EstadoSolicitud =
  | 'REGISTRADA'
  | 'CLASIFICADA'
  | 'EN_ATENCION'
  | 'ATENDIDA'
  | 'CERRADA';

export type CanalOrigen =
  | 'PRESENCIAL'
  | 'CORREO'
  | 'SAC'
  | 'TELEFONICO'
  | 'PLATAFORMA_WEB';

export type Prioridad = 'BAJA' | 'MEDIA' | 'ALTA' | 'CRITICA';

export interface UsuarioResumen {
  id: string;
  nombre: string;
  correo: string;
  rol: string;
}

export interface HistorialSolicitud {
  id: string;
  accion: string;
  observaciones: string | null;
  fechaAccion: string;
  usuarioResponsable: string;
}

export interface SolicitudResponse {
  id: string;
  tipo: TipoSolicitud;
  tipoDescripcion: string;
  descripcion: string;
  estado: EstadoSolicitud;
  prioridad: Prioridad | null;
  canalOrigen: CanalOrigen;
  solicitante: UsuarioResumen;
  responsable: UsuarioResumen | null;
  fechaRegistro: string;
  fechaUltimaModificacion: string;
  historial: HistorialSolicitud[];
}

export interface CrearSolicitudRequest {
  tipo: TipoSolicitud;
  descripcion: string;
  canalOrigen: CanalOrigen;
}

export interface PageResponse<T> {
  content: T[];
  totalElements: number;
  totalPages: number;
  number: number;        // Página actual (0-indexado)
  size: number;
  first: boolean;
  last: boolean;
}
```

```typescript
// src/app/core/services/solicitud.service.ts

import { inject, Injectable } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable } from 'rxjs';
import { API_URL } from '../tokens/api.tokens';
import {
  CrearSolicitudRequest,
  EstadoSolicitud,
  PageResponse,
  SolicitudResponse,
  TipoSolicitud,
  Prioridad,
} from '../models/solicitud.model';

@Injectable({ providedIn: 'root' })
export class SolicitudService {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = `${inject(API_URL)}/solicitudes`;

  /**
   * RF-01: Crear una nueva solicitud.
   * El JWT adjuntado por el interceptor identifica al solicitante en el backend.
   */
  crear(request: CrearSolicitudRequest): Observable<SolicitudResponse> {
    return this.http.post<SolicitudResponse>(this.baseUrl, request);
  }

  obtenerPorId(id: string): Observable<SolicitudResponse> {
    return this.http.get<SolicitudResponse>(`${this.baseUrl}/${id}`);
  }

  /**
   * RF-07: Listar con filtros opcionales.
   * HttpParams construye los query params solo para los filtros que tengan valor.
   */
  listar(filtros: {
    estado?: EstadoSolicitud;
    tipo?: TipoSolicitud;
    prioridad?: Prioridad;
    page?: number;
    size?: number;
    sort?: string;
  }): Observable<PageResponse<SolicitudResponse>> {
    let params = new HttpParams();

    if (filtros.estado) params = params.set('estado', filtros.estado);
    if (filtros.tipo) params = params.set('tipo', filtros.tipo);
    if (filtros.prioridad) params = params.set('prioridad', filtros.prioridad);
    if (filtros.page !== undefined) params = params.set('page', filtros.page);
    if (filtros.size !== undefined) params = params.set('size', filtros.size);
    if (filtros.sort) params = params.set('sort', filtros.sort);

    return this.http.get<PageResponse<SolicitudResponse>>(this.baseUrl, { params });
  }
}
```

```typescript
// src/app/features/solicitudes/crear-solicitud/crear-solicitud.component.ts

import { Component, inject } from '@angular/core';
import { FormBuilder, ReactiveFormsModule, Validators } from '@angular/forms';
import { Router } from '@angular/router';
import { CommonModule } from '@angular/common';
import { SolicitudService } from '../../../core/services/solicitud.service';
import { HttpErrorResponse } from '@angular/common/http';
import { CanalOrigen, TipoSolicitud } from '../../../core/models/solicitud.model';

@Component({
  selector: 'app-crear-solicitud',
  standalone: true,
  imports: [ReactiveFormsModule, CommonModule],
  template: `
    <div class="page-container">
      <div class="form-card">
        <h2>Nueva Solicitud Académica</h2>
        <p class="subtitle">Completa el formulario para registrar tu solicitud (RF-01)</p>

        @if (errorMensaje) {
          <div class="alert alert-error">{{ errorMensaje }}</div>
        }

        <form [formGroup]="solicitudForm" (ngSubmit)="onSubmit()">

          <!-- Tipo de solicitud -->
          <div class="form-group">
            <label for="tipo">Tipo de solicitud *</label>
            <select id="tipo" formControlName="tipo"
                    [class.input-error]="tipo.invalid && tipo.touched">
              <option value="">-- Selecciona un tipo --</option>
              @for (opcion of tiposSolicitud; track opcion.valor) {
                <option [value]="opcion.valor">{{ opcion.etiqueta }}</option>
              }
            </select>
            @if (tipo.invalid && tipo.touched) {
              <span class="error-text">Debes seleccionar un tipo de solicitud</span>
            }
          </div>

          <!-- Descripción -->
          <div class="form-group">
            <label for="descripcion">Descripción *</label>
            <textarea
              id="descripcion"
              formControlName="descripcion"
              rows="5"
              placeholder="Describe detalladamente tu solicitud (mínimo 10 caracteres)..."
              [class.input-error]="descripcion.invalid && descripcion.touched"
            ></textarea>
            <span class="char-count">
              {{ descripcion.value?.length ?? 0 }} / 1000
            </span>
            @if (descripcion.invalid && descripcion.touched) {
              <span class="error-text">
                @if (descripcion.errors?.['required']) { La descripción es obligatoria }
                @if (descripcion.errors?.['minlength']) { Mínimo 10 caracteres }
                @if (descripcion.errors?.['maxlength']) { Máximo 1000 caracteres }
              </span>
            }
          </div>

          <!-- Canal de origen -->
          <div class="form-group">
            <label for="canalOrigen">Canal de ingreso *</label>
            <select id="canalOrigen" formControlName="canalOrigen"
                    [class.input-error]="canalOrigen.invalid && canalOrigen.touched">
              <option value="">-- Selecciona un canal --</option>
              @for (opcion of canalesOrigen; track opcion.valor) {
                <option [value]="opcion.valor">{{ opcion.etiqueta }}</option>
              }
            </select>
            @if (canalOrigen.invalid && canalOrigen.touched) {
              <span class="error-text">Debes seleccionar el canal de ingreso</span>
            }
          </div>

          <div class="form-actions">
            <button type="button" class="btn-secondary" (click)="cancelar()">
              Cancelar
            </button>
            <button
              type="submit"
              class="btn-primary"
              [disabled]="solicitudForm.invalid || cargando"
            >
              @if (cargando) { Enviando... } @else { Registrar Solicitud }
            </button>
          </div>
        </form>
      </div>
    </div>
  `,
  styleUrl: './crear-solicitud.component.scss',
})
export class CrearSolicitudComponent {
  private readonly fb = inject(FormBuilder);
  private readonly solicitudService = inject(SolicitudService);
  private readonly router = inject(Router);

  cargando = false;
  errorMensaje = '';

  tiposSolicitud: { valor: TipoSolicitud; etiqueta: string }[] = [
    { valor: 'HOMOLOGACION', etiqueta: 'Homologación de asignatura' },
    { valor: 'REGISTRO_ASIGNATURA', etiqueta: 'Registro de asignatura' },
    { valor: 'CANCELACION_ASIGNATURA', etiqueta: 'Cancelación de asignatura' },
    { valor: 'SOLICITUD_CUPO', etiqueta: 'Solicitud de cupo' },
    { valor: 'CONSULTA_ACADEMICA', etiqueta: 'Consulta académica' },
    { valor: 'OTRO', etiqueta: 'Otro' },
  ];

  canalesOrigen: { valor: CanalOrigen; etiqueta: string }[] = [
    { valor: 'PLATAFORMA_WEB', etiqueta: 'Plataforma web' },
    { valor: 'CORREO', etiqueta: 'Correo electrónico' },
    { valor: 'PRESENCIAL', etiqueta: 'Presencial' },
    { valor: 'SAC', etiqueta: 'SAC (Sistema de Atención al Ciudadano)' },
    { valor: 'TELEFONICO', etiqueta: 'Telefónico' },
  ];

  solicitudForm = this.fb.group({
    tipo: ['', Validators.required],
    descripcion: [
      '',
      [Validators.required, Validators.minLength(10), Validators.maxLength(1000)],
    ],
    canalOrigen: ['', Validators.required],
  });

  get tipo() { return this.solicitudForm.controls.tipo; }
  get descripcion() { return this.solicitudForm.controls.descripcion; }
  get canalOrigen() { return this.solicitudForm.controls.canalOrigen; }

  onSubmit(): void {
    if (this.solicitudForm.invalid) {
      // Marcar todos los campos como tocados para mostrar errores
      this.solicitudForm.markAllAsTouched();
      return;
    }

    this.cargando = true;
    this.errorMensaje = '';

    const { tipo, descripcion, canalOrigen } = this.solicitudForm.value;

    this.solicitudService.crear({
      tipo: tipo as TipoSolicitud,
      descripcion: descripcion!,
      canalOrigen: canalOrigen as CanalOrigen,
    }).subscribe({
      next: (solicitud) => {
        // Éxito: navegar al detalle de la solicitud recién creada
        this.router.navigate(['/solicitudes', solicitud.id], {
          queryParams: { creada: 'true' },  // Para mostrar mensaje de éxito
        });
      },
      error: (err: HttpErrorResponse) => {
        this.cargando = false;
        if (err.status === 400 && err.error?.errors) {
          // Error de validación del backend: mostrar el primer error
          const errores = err.error.errors as { field: string; message: string }[];
          this.errorMensaje = errores.map(e => e.message).join('. ');
        } else {
          this.errorMensaje = 'Error al registrar la solicitud. Intenta nuevamente.';
        }
      },
      complete: () => {
        this.cargando = false;
      },
    });
  }

  cancelar(): void {
    this.router.navigate(['/solicitudes']);
  }
}
```

---

## 7. Caso de Uso E2E #2: Funcionario Clasifica y Prioriza una Solicitud

### 7.1 Backend: Lógica de clasificación y motor de priorización

El proceso de clasificar una solicitud implica:
1. Cambiar el estado de `REGISTRADA` a `CLASIFICADA`
2. Asignar la prioridad (manualmente o por reglas de negocio)
3. Registrar la acción en el historial

Primero, necesitamos un validador de transiciones de estado. No todas las transiciones son válidas: por ejemplo, no puedes pasar de `CERRADA` a `EN_ATENCION`.

```java
// src/main/java/co/uniquindio/triage/solicitud/service/TransicionEstadoValidator.java

package co.uniquindio.triage.solicitud.service;

import co.uniquindio.triage.shared.exception.TransicionEstadoInvalidaException;
import co.uniquindio.triage.solicitud.entity.EstadoSolicitud;
import org.springframework.stereotype.Component;

import java.util.Map;
import java.util.Set;

/**
 * Valida que las transiciones de estado sean coherentes (RF-04).
 *
 * Transiciones permitidas:
 *   REGISTRADA → CLASIFICADA
 *   CLASIFICADA → EN_ATENCION
 *   EN_ATENCION → ATENDIDA
 *   ATENDIDA → CERRADA
 *
 * Ningún estado puede volver atrás.
 */
@Component
public class TransicionEstadoValidator {

    // Mapa de transiciones válidas: estado actual → estados a los que puede ir
    private static final Map<EstadoSolicitud, Set<EstadoSolicitud>> TRANSICIONES_VALIDAS =
        Map.of(
            EstadoSolicitud.REGISTRADA,   Set.of(EstadoSolicitud.CLASIFICADA),
            EstadoSolicitud.CLASIFICADA,  Set.of(EstadoSolicitud.EN_ATENCION),
            EstadoSolicitud.EN_ATENCION,  Set.of(EstadoSolicitud.ATENDIDA),
            EstadoSolicitud.ATENDIDA,     Set.of(EstadoSolicitud.CERRADA),
            EstadoSolicitud.CERRADA,      Set.of()   // Estado terminal: no puede avanzar
        );

    public void validar(EstadoSolicitud actual, EstadoSolicitud destino) {
        var permitidos = TRANSICIONES_VALIDAS.getOrDefault(actual, Set.of());

        if (!permitidos.contains(destino)) {
            throw new TransicionEstadoInvalidaException(
                String.format(
                    "No se puede cambiar de '%s' a '%s'. Transiciones permitidas desde '%s': %s",
                    actual, destino, actual, permitidos
                )
            );
        }
    }
}
```

```java
// src/main/java/co/uniquindio/triage/solicitud/service/PriorizacionService.java

package co.uniquindio.triage.solicitud.service;

import co.uniquindio.triage.solicitud.entity.Prioridad;
import co.uniquindio.triage.solicitud.entity.Solicitud;
import co.uniquindio.triage.solicitud.entity.TipoSolicitud;
import org.springframework.stereotype.Service;

import java.time.LocalDate;

/**
 * Motor de priorización de solicitudes.
 * Implementa el patrón Strategy: podríamos tener diferentes estrategias
 * de priorización e intercambiarlas.
 *
 * Reglas de negocio (RF-03):
 * - HOMOLOGACION en período de matrículas → CRITICA
 * - CANCELACION_ASIGNATURA → ALTA (siempre hay fecha límite)
 * - CONSULTA_ACADEMICA → BAJA (informativa)
 * - Por defecto → MEDIA
 */
@Service
public class PriorizacionService {

    public Prioridad calcularPrioridad(Solicitud solicitud) {
        var tipo = solicitud.getTipo();

        return switch (tipo) {
            case HOMOLOGACION -> estamosEnPeriodoMatriculas()
                ? Prioridad.CRITICA
                : Prioridad.ALTA;

            case CANCELACION_ASIGNATURA -> Prioridad.ALTA;

            case REGISTRO_ASIGNATURA -> estamosEnPeriodoMatriculas()
                ? Prioridad.ALTA
                : Prioridad.MEDIA;

            case SOLICITUD_CUPO -> Prioridad.MEDIA;

            case CONSULTA_ACADEMICA -> Prioridad.BAJA;

            case OTRO -> Prioridad.MEDIA;
        };
    }

    /**
     * Simula la verificación de período de matrículas.
     * En un sistema real, consultaría el calendario académico de la base de datos.
     */
    private boolean estamosEnPeriodoMatriculas() {
        var mesActual = LocalDate.now().getMonthValue();
        // Febrero y Agosto: típicos períodos de matrícula
        return mesActual == 2 || mesActual == 8;
    }
}
```

```java
// Agregar al SolicitudService — método para clasificar (RF-02, RF-03)

/**
 * RF-02 + RF-03: Clasificar y priorizar una solicitud.
 * Válido solo para FUNCIONARIO y ADMIN.
 * El Controller usa @PreAuthorize para esto.
 */
public SolicitudResponse clasificar(UUID id, ClasificarSolicitudRequest request) {
    var solicitud = solicitudRepository.findByIdConHistorial(id)
        .orElseThrow(() -> new RecursoNoEncontradoException("Solicitud no encontrada: " + id));

    // Validar transición de estado
    transicionValidator.validar(solicitud.getEstado(), EstadoSolicitud.CLASIFICADA);

    // Aplicar clasificación
    solicitud.setEstado(EstadoSolicitud.CLASIFICADA);

    // La prioridad puede ser manual o calculada automáticamente
    var prioridad = request.prioridadManual() != null
        ? request.prioridadManual()
        : priorizacionService.calcularPrioridad(solicitud);

    solicitud.setPrioridad(prioridad);

    // Obtener el funcionario que está haciendo la clasificación
    var correoFuncionario = SecurityContextHolder.getContext()
        .getAuthentication().getName();
    var funcionario = usuarioRepository.findByCorreo(correoFuncionario)
        .orElseThrow();

    // Registrar en el historial
    var accion = String.format("Solicitud clasificada. Prioridad asignada: %s. %s",
        prioridad,
        request.prioridadManual() != null ? "(Manual)" : "(Automática)"
    );
    agregarHistorial(solicitud, accion, request.justificacion(), funcionario);

    var guardada = solicitudRepository.save(solicitud);
    return mapper.toResponse(guardada);
}
```

### 7.2 Frontend: Componente de Clasificación

```typescript
// src/app/features/solicitudes/clasificar-solicitud/clasificar-solicitud.component.ts

import { Component, inject, OnInit } from '@angular/core';
import { FormBuilder, ReactiveFormsModule, Validators } from '@angular/forms';
import { ActivatedRoute, Router } from '@angular/router';
import { CommonModule } from '@angular/common';
import { SolicitudService } from '../../../core/services/solicitud.service';
import { Prioridad, SolicitudResponse } from '../../../core/models/solicitud.model';

@Component({
  selector: 'app-clasificar-solicitud',
  standalone: true,
  imports: [ReactiveFormsModule, CommonModule],
  template: `
    <div class="page-container">
      @if (solicitud) {
        <div class="clasificar-card">
          <h2>Clasificar Solicitud</h2>

          <!-- Resumen de la solicitud a clasificar -->
          <div class="solicitud-resumen">
            <span class="badge tipo">{{ solicitud.tipoDescripcion }}</span>
            <p><strong>Solicitante:</strong> {{ solicitud.solicitante.nombre }}</p>
            <p><strong>Descripción:</strong> {{ solicitud.descripcion }}</p>
            <p><strong>Fecha de registro:</strong>
              {{ solicitud.fechaRegistro | date:'dd/MM/yyyy HH:mm' }}
            </p>
          </div>

          <!-- Prioridad sugerida automáticamente -->
          @if (prioridadSugerida) {
            <div class="sugerencia-ia">
              <span class="badge sugerencia">⚡ Sugerencia del sistema</span>
              <p>Basado en el tipo de solicitud y el calendario académico,
                 la prioridad sugerida es: <strong>{{ prioridadSugerida }}</strong></p>
              <button class="btn-outline" (click)="aceptarSugerencia()">
                Aceptar sugerencia
              </button>
            </div>
          }

          <form [formGroup]="clasificarForm" (ngSubmit)="onSubmit()">
            <div class="form-group">
              <label for="prioridad">Prioridad *</label>
              <select id="prioridad" formControlName="prioridad">
                <option value="">-- Selecciona la prioridad --</option>
                <option value="BAJA">Baja</option>
                <option value="MEDIA">Media</option>
                <option value="ALTA">Alta</option>
                <option value="CRITICA">Crítica</option>
              </select>
            </div>

            <div class="form-group">
              <label for="justificacion">Justificación *</label>
              <textarea
                id="justificacion"
                formControlName="justificacion"
                rows="4"
                placeholder="Explica el criterio de clasificación..."
              ></textarea>
              @if (justificacion.invalid && justificacion.touched) {
                <span class="error-text">La justificación es obligatoria</span>
              }
            </div>

            <div class="form-actions">
              <button type="button" class="btn-secondary" (click)="volver()">
                Cancelar
              </button>
              <button
                type="submit"
                class="btn-primary"
                [disabled]="clasificarForm.invalid || cargando"
              >
                @if (cargando) { Clasificando... } @else { Confirmar Clasificación }
              </button>
            </div>
          </form>
        </div>
      } @else {
        <div class="loading">Cargando solicitud...</div>
      }
    </div>
  `,
  styleUrl: './clasificar-solicitud.component.scss',
})
export class ClasificarSolicitudComponent implements OnInit {
  private readonly fb = inject(FormBuilder);
  private readonly solicitudService = inject(SolicitudService);
  private readonly route = inject(ActivatedRoute);
  private readonly router = inject(Router);

  solicitud: SolicitudResponse | null = null;
  prioridadSugerida: Prioridad | null = null;
  cargando = false;
  solicitudId = '';

  clasificarForm = this.fb.group({
    prioridad: ['', Validators.required],
    justificacion: ['', [Validators.required, Validators.minLength(10)]],
  });

  get justificacion() { return this.clasificarForm.controls.justificacion; }

  ngOnInit(): void {
    this.solicitudId = this.route.snapshot.params['id'];
    this.cargarSolicitud();
  }

  cargarSolicitud(): void {
    this.solicitudService.obtenerPorId(this.solicitudId).subscribe({
      next: (sol) => {
        this.solicitud = sol;
        // Calcular sugerencia de prioridad en el frontend
        // (El backend también puede hacerlo; aquí se muestra como UI/UX)
        this.prioridadSugerida = this.calcularPrioridadSugerida(sol);
      },
    });
  }

  private calcularPrioridadSugerida(sol: SolicitudResponse): Prioridad {
    const mesActual = new Date().getMonth() + 1;  // 1-indexado
    const enMatriculas = mesActual === 2 || mesActual === 8;

    if (sol.tipo === 'HOMOLOGACION' && enMatriculas) return 'CRITICA';
    if (sol.tipo === 'CANCELACION_ASIGNATURA') return 'ALTA';
    if (sol.tipo === 'CONSULTA_ACADEMICA') return 'BAJA';
    return 'MEDIA';
  }

  aceptarSugerencia(): void {
    if (this.prioridadSugerida) {
      this.clasificarForm.patchValue({ prioridad: this.prioridadSugerida });
    }
  }

  onSubmit(): void {
    if (this.clasificarForm.invalid) {
      this.clasificarForm.markAllAsTouched();
      return;
    }

    this.cargando = true;
    const { prioridad, justificacion } = this.clasificarForm.value;

    this.solicitudService.clasificar(this.solicitudId, {
      prioridadManual: prioridad as Prioridad,
      justificacion: justificacion!,
    }).subscribe({
      next: () => {
        this.router.navigate(['/solicitudes', this.solicitudId], {
          queryParams: { clasificada: 'true' },
        });
      },
      error: (err) => {
        this.cargando = false;
        console.error('Error al clasificar:', err);
      },
      complete: () => { this.cargando = false; },
    });
  }

  volver(): void {
    this.router.navigate(['/solicitudes', this.solicitudId]);
  }
}
```

---

## 8. Caso de Uso E2E #3: Consultar Solicitudes con Filtros y Paginación

Este caso de uso muestra la tabla de solicitudes con filtros dinámicos y paginación del lado del servidor.

### 8.1 Backend: Controller con paginación (ya implementado)

La paginación del lado del servidor es gestionada por Spring Data JPA con el objeto `Pageable`. Cuando Angular manda los parámetros `?page=0&size=10&sort=fechaRegistro,desc`, Spring los deserializa automáticamente.

La respuesta de la API para una página tiene esta estructura:

```json
{
  "content": [ ... ],      // Array de solicitudes en esta página
  "totalElements": 47,     // Total de solicitudes que coinciden con el filtro
  "totalPages": 5,         // Número total de páginas
  "number": 0,             // Página actual (0-indexado)
  "size": 10,              // Tamaño de la página
  "first": true,           // ¿Es la primera página?
  "last": false            // ¿Es la última página?
}
```

### 8.2 Frontend: Componente de Lista con Filtros

```typescript
// src/app/features/solicitudes/lista-solicitudes/lista-solicitudes.component.ts

import { Component, inject, OnInit, OnDestroy } from '@angular/core';
import { FormBuilder, ReactiveFormsModule } from '@angular/forms';
import { RouterLink } from '@angular/router';
import { CommonModule, DatePipe } from '@angular/common';
import { debounceTime, distinctUntilChanged, Subject, takeUntil } from 'rxjs';
import { SolicitudService } from '../../../core/services/solicitud.service';
import { AuthService } from '../../../core/services/auth.service';
import {
  EstadoSolicitud,
  PageResponse,
  Prioridad,
  SolicitudResponse,
  TipoSolicitud,
} from '../../../core/models/solicitud.model';

@Component({
  selector: 'app-lista-solicitudes',
  standalone: true,
  imports: [ReactiveFormsModule, CommonModule, RouterLink, DatePipe],
  template: `
    <div class="lista-container">
      <div class="lista-header">
        <h2>Solicitudes Académicas</h2>
        <!-- Solo el ESTUDIANTE puede crear solicitudes -->
        @if (authService.tieneRol('ESTUDIANTE')) {
          <a routerLink="/solicitudes/nueva" class="btn-primary">
            + Nueva Solicitud
          </a>
        }
      </div>

      <!-- Panel de Filtros -->
      <div class="filtros-panel" [formGroup]="filtrosForm">
        <div class="filtros-grid">
          <div class="form-group">
            <label>Estado</label>
            <select formControlName="estado">
              <option value="">Todos</option>
              <option value="REGISTRADA">Registrada</option>
              <option value="CLASIFICADA">Clasificada</option>
              <option value="EN_ATENCION">En Atención</option>
              <option value="ATENDIDA">Atendida</option>
              <option value="CERRADA">Cerrada</option>
            </select>
          </div>

          <div class="form-group">
            <label>Tipo</label>
            <select formControlName="tipo">
              <option value="">Todos</option>
              <option value="HOMOLOGACION">Homologación</option>
              <option value="REGISTRO_ASIGNATURA">Registro de asignatura</option>
              <option value="CANCELACION_ASIGNATURA">Cancelación</option>
              <option value="SOLICITUD_CUPO">Cupo</option>
              <option value="CONSULTA_ACADEMICA">Consulta</option>
            </select>
          </div>

          <div class="form-group">
            <label>Prioridad</label>
            <select formControlName="prioridad">
              <option value="">Todas</option>
              <option value="BAJA">Baja</option>
              <option value="MEDIA">Media</option>
              <option value="ALTA">Alta</option>
              <option value="CRITICA">Crítica</option>
            </select>
          </div>
        </div>

        <button class="btn-outline" (click)="limpiarFiltros()">
          Limpiar filtros
        </button>
      </div>

      <!-- Estado de carga -->
      @if (cargando) {
        <div class="loading-state">
          <div class="spinner"></div>
          <p>Cargando solicitudes...</p>
        </div>
      } @else if (paginaActual?.content.length === 0) {
        <!-- Estado vacío -->
        <div class="empty-state">
          <p>No se encontraron solicitudes con los filtros seleccionados.</p>
        </div>
      } @else {
        <!-- Tabla de resultados -->
        <div class="tabla-wrapper">
          <table class="tabla-solicitudes">
            <thead>
              <tr>
                <th>Tipo</th>
                <th>Descripción</th>
                <th>Estado</th>
                <th>Prioridad</th>
                <th>Solicitante</th>
                <th>Fecha</th>
                <th>Acciones</th>
              </tr>
            </thead>
            <tbody>
              @for (sol of paginaActual?.content; track sol.id) {
                <tr>
                  <td>
                    <span class="badge tipo-{{ sol.tipo.toLowerCase() }}">
                      {{ sol.tipoDescripcion }}
                    </span>
                  </td>
                  <td class="descripcion-truncada" [title]="sol.descripcion">
                    {{ sol.descripcion | slice:0:80 }}
                    @if (sol.descripcion.length > 80) { ... }
                  </td>
                  <td>
                    <span class="badge estado estado-{{ sol.estado.toLowerCase() }}">
                      {{ sol.estado }}
                    </span>
                  </td>
                  <td>
                    @if (sol.prioridad) {
                      <span class="badge prioridad prioridad-{{ sol.prioridad.toLowerCase() }}">
                        {{ sol.prioridad }}
                      </span>
                    } @else {
                      <span class="text-muted">Sin asignar</span>
                    }
                  </td>
                  <td>{{ sol.solicitante.nombre }}</td>
                  <td>{{ sol.fechaRegistro | date:'dd/MM/yy HH:mm' }}</td>
                  <td>
                    <div class="acciones">
                      <a [routerLink]="['/solicitudes', sol.id]"
                         class="btn-icon" title="Ver detalle">👁</a>

                      @if (authService.tieneRol('FUNCIONARIO') || authService.tieneRol('ADMIN')) {
                        @if (sol.estado === 'REGISTRADA') {
                          <a [routerLink]="['/solicitudes', sol.id, 'clasificar']"
                             class="btn-icon" title="Clasificar">📋</a>
                        }
                      }
                    </div>
                  </td>
                </tr>
              }
            </tbody>
          </table>
        </div>

        <!-- Paginación -->
        @if (paginaActual && paginaActual.totalPages > 1) {
          <div class="paginacion">
            <button
              class="btn-icon"
              [disabled]="paginaActual.first"
              (click)="cambiarPagina(0)"
            >⏮</button>

            <button
              class="btn-icon"
              [disabled]="paginaActual.first"
              (click)="cambiarPagina(paginaActual.number - 1)"
            >◀</button>

            <span class="info-pagina">
              Página {{ paginaActual.number + 1 }} de {{ paginaActual.totalPages }}
              ({{ paginaActual.totalElements }} solicitudes en total)
            </span>

            <button
              class="btn-icon"
              [disabled]="paginaActual.last"
              (click)="cambiarPagina(paginaActual.number + 1)"
            >▶</button>

            <button
              class="btn-icon"
              [disabled]="paginaActual.last"
              (click)="cambiarPagina(paginaActual.totalPages - 1)"
            >⏭</button>
          </div>
        }
      }
    </div>
  `,
  styleUrl: './lista-solicitudes.component.scss',
})
export class ListaSolicitudesComponent implements OnInit, OnDestroy {
  private readonly fb = inject(FormBuilder);
  private readonly solicitudService = inject(SolicitudService);
  readonly authService = inject(AuthService);

  private readonly destroy$ = new Subject<void>();

  cargando = false;
  paginaActual: PageResponse<SolicitudResponse> | null = null;
  paginaIndex = 0;
  pageSize = 10;

  filtrosForm = this.fb.group({
    estado: [''],
    tipo: [''],
    prioridad: [''],
  });

  ngOnInit(): void {
    // Escuchar cambios en los filtros con debounce
    // debounceTime(300): esperar 300ms después del último cambio antes de hacer la petición
    // distinctUntilChanged: no hacer petición si los valores son iguales al anterior
    this.filtrosForm.valueChanges.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      takeUntil(this.destroy$)
    ).subscribe(() => {
      this.paginaIndex = 0;   // Al cambiar filtros, volver a la primera página
      this.cargarSolicitudes();
    });

    this.cargarSolicitudes();
  }

  cargarSolicitudes(): void {
    this.cargando = true;
    const { estado, tipo, prioridad } = this.filtrosForm.value;

    this.solicitudService.listar({
      estado: (estado || undefined) as EstadoSolicitud | undefined,
      tipo: (tipo || undefined) as TipoSolicitud | undefined,
      prioridad: (prioridad || undefined) as Prioridad | undefined,
      page: this.paginaIndex,
      size: this.pageSize,
      sort: 'fechaRegistro,desc',
    }).subscribe({
      next: (pagina) => {
        this.paginaActual = pagina;
        this.cargando = false;
      },
      error: () => {
        this.cargando = false;
      },
    });
  }

  cambiarPagina(numeroPagina: number): void {
    this.paginaIndex = numeroPagina;
    this.cargarSolicitudes();
    // Hacer scroll al inicio de la tabla
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }

  limpiarFiltros(): void {
    this.filtrosForm.reset({ estado: '', tipo: '', prioridad: '' });
  }

  ngOnDestroy(): void {
    // Limpiar suscripciones para evitar memory leaks
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

---

## 9. Manejo de Errores de Punta a Punta

El manejo de errores es crucial para una buena experiencia de usuario. Un error del servidor debe transformarse en un mensaje claro y útil en el frontend.

### 9.1 Backend: Estructura de errores con Problem Details (RFC 7807)

RFC 7807 es un estándar para representar errores en APIs HTTP. Spring Boot 3.x lo soporta nativamente.

```java
// src/main/java/co/uniquindio/triage/shared/exception/GlobalExceptionHandler.java

package co.uniquindio.triage.shared.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ProblemDetail;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.net.URI;
import java.time.Instant;
import java.util.List;
import java.util.Map;

@RestControllerAdvice
public class GlobalExceptionHandler {

    /**
     * Maneja errores de validación de @Valid.
     * Cuando el cuerpo de la petición tiene campos inválidos,
     * Spring lanza MethodArgumentNotValidException.
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ProblemDetail handleValidationErrors(MethodArgumentNotValidException ex) {
        var problema = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        problema.setType(URI.create("https://triage.uniquindio.edu.co/errors/validation"));
        problema.setTitle("Error de validación");
        problema.setDetail("Uno o más campos tienen valores inválidos");
        problema.setProperty("timestamp", Instant.now());

        // Extraer todos los errores de campo
        List<Map<String, String>> errores = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(fieldError -> Map.of(
                "field", fieldError.getField(),
                "message", fieldError.getDefaultMessage() != null
                    ? fieldError.getDefaultMessage()
                    : "Valor inválido"
            ))
            .toList();

        problema.setProperty("errors", errores);
        return problema;
    }

    /**
     * Maneja el caso en que un recurso no existe (404).
     */
    @ExceptionHandler(RecursoNoEncontradoException.class)
    public ProblemDetail handleRecursoNoEncontrado(RecursoNoEncontradoException ex) {
        var problema = ProblemDetail.forStatus(HttpStatus.NOT_FOUND);
        problema.setType(URI.create("https://triage.uniquindio.edu.co/errors/not-found"));
        problema.setTitle("Recurso no encontrado");
        problema.setDetail(ex.getMessage());
        problema.setProperty("timestamp", Instant.now());
        return problema;
    }

    /**
     * Maneja transiciones de estado inválidas (ej: de CERRADA a REGISTRADA).
     */
    @ExceptionHandler(TransicionEstadoInvalidaException.class)
    public ProblemDetail handleTransicionInvalida(TransicionEstadoInvalidaException ex) {
        var problema = ProblemDetail.forStatus(HttpStatus.UNPROCESSABLE_ENTITY);
        problema.setType(URI.create("https://triage.uniquindio.edu.co/errors/invalid-state-transition"));
        problema.setTitle("Transición de estado inválida");
        problema.setDetail(ex.getMessage());
        problema.setProperty("timestamp", Instant.now());
        return problema;
    }

    /**
     * Maneja credenciales incorrectas (401).
     */
    @ExceptionHandler(CredencialesInvalidasException.class)
    public ProblemDetail handleCredencialesInvalidas(CredencialesInvalidasException ex) {
        var problema = ProblemDetail.forStatus(HttpStatus.UNAUTHORIZED);
        problema.setType(URI.create("https://triage.uniquindio.edu.co/errors/invalid-credentials"));
        problema.setTitle("Credenciales inválidas");
        problema.setDetail(ex.getMessage());
        problema.setProperty("timestamp", Instant.now());
        return problema;
    }

    /**
     * Maneja conflictos (ej: correo ya registrado) (409).
     */
    @ExceptionHandler(ConflictoException.class)
    public ProblemDetail handleConflicto(ConflictoException ex) {
        var problema = ProblemDetail.forStatus(HttpStatus.CONFLICT);
        problema.setType(URI.create("https://triage.uniquindio.edu.co/errors/conflict"));
        problema.setTitle("Conflicto");
        problema.setDetail(ex.getMessage());
        problema.setProperty("timestamp", Instant.now());
        return problema;
    }

    /**
     * Handler genérico: cualquier excepción no manejada por los anteriores.
     */
    @ExceptionHandler(Exception.class)
    public ProblemDetail handleException(Exception ex) {
        var problema = ProblemDetail.forStatus(HttpStatus.INTERNAL_SERVER_ERROR);
        problema.setType(URI.create("https://triage.uniquindio.edu.co/errors/internal"));
        problema.setTitle("Error interno del servidor");
        // NUNCA exponer el mensaje de excepción real al cliente en producción
        problema.setDetail("Ocurrió un error inesperado. Contacta al administrador.");
        problema.setProperty("timestamp", Instant.now());
        // Sí loggearlo en el servidor
        System.err.println("Error no manejado: " + ex.getMessage());
        return problema;
    }
}
```

```java
// Clases de excepción personalizadas

// src/main/java/co/uniquindio/triage/shared/exception/RecursoNoEncontradoException.java
package co.uniquindio.triage.shared.exception;
public class RecursoNoEncontradoException extends RuntimeException {
    public RecursoNoEncontradoException(String message) { super(message); }
}

// src/main/java/co/uniquindio/triage/shared/exception/TransicionEstadoInvalidaException.java
package co.uniquindio.triage.shared.exception;
public class TransicionEstadoInvalidaException extends RuntimeException {
    public TransicionEstadoInvalidaException(String message) { super(message); }
}

// src/main/java/co/uniquindio/triage/shared/exception/CredencialesInvalidasException.java
package co.uniquindio.triage.shared.exception;
public class CredencialesInvalidasException extends RuntimeException {
    public CredencialesInvalidasException(String message) { super(message); }
}

// src/main/java/co/uniquindio/triage/shared/exception/ConflictoException.java
package co.uniquindio.triage.shared.exception;
public class ConflictoException extends RuntimeException {
    public ConflictoException(String message) { super(message); }
}
```

### 9.2 Frontend: Servicio de Notificaciones y Manejo de Errores

```typescript
// src/app/shared/services/notificacion.service.ts

import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';

export interface Notificacion {
  id: string;
  tipo: 'exito' | 'error' | 'advertencia' | 'info';
  titulo: string;
  mensaje: string;
  duracionMs?: number;
}

@Injectable({ providedIn: 'root' })
export class NotificacionService {
  private notificacionesSubject = new BehaviorSubject<Notificacion[]>([]);
  readonly notificaciones$ = this.notificacionesSubject.asObservable();

  exito(titulo: string, mensaje: string, duracionMs = 4000): void {
    this.agregar({ tipo: 'exito', titulo, mensaje, duracionMs });
  }

  error(titulo: string, mensaje: string, duracionMs = 6000): void {
    this.agregar({ tipo: 'error', titulo, mensaje, duracionMs });
  }

  advertencia(titulo: string, mensaje: string, duracionMs = 5000): void {
    this.agregar({ tipo: 'advertencia', titulo, mensaje, duracionMs });
  }

  private agregar(datos: Omit<Notificacion, 'id'>): void {
    const notificacion: Notificacion = {
      ...datos,
      id: crypto.randomUUID(),
    };

    const actuales = this.notificacionesSubject.value;
    this.notificacionesSubject.next([...actuales, notificacion]);

    // Auto-eliminar después de la duración
    if (datos.duracionMs) {
      setTimeout(() => this.eliminar(notificacion.id), datos.duracionMs);
    }
  }

  eliminar(id: string): void {
    this.notificacionesSubject.next(
      this.notificacionesSubject.value.filter((n) => n.id !== id)
    );
  }
}
```

```typescript
// src/app/core/interceptors/error.interceptor.ts (versión mejorada con notificaciones)

import { HttpInterceptorFn, HttpErrorResponse } from '@angular/common/http';
import { inject } from '@angular/core';
import { Router } from '@angular/router';
import { catchError, throwError } from 'rxjs';
import { AuthService } from '../services/auth.service';
import { NotificacionService } from '../../shared/services/notificacion.service';

/**
 * Interface para el formato Problem Details (RFC 7807).
 * Esto es lo que el backend devuelve en caso de error.
 */
interface ProblemDetail {
  type: string;
  title: string;
  status: number;
  detail: string;
  timestamp: string;
  errors?: { field: string; message: string }[];
}

export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const router = inject(Router);
  const authService = inject(AuthService);
  const notificaciones = inject(NotificacionService);

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      // Intentar extraer el Problem Detail del cuerpo del error
      const problema = error.error as ProblemDetail | null;

      switch (error.status) {
        case 400:
          // Error de validación: mostrar los mensajes específicos de cada campo
          if (problema?.errors && problema.errors.length > 0) {
            const mensajes = problema.errors
              .map((e) => `${e.field}: ${e.message}`)
              .join('\n');
            notificaciones.error('Datos inválidos', mensajes);
          } else {
            notificaciones.error(
              'Solicitud inválida',
              problema?.detail ?? 'Verifica los datos enviados'
            );
          }
          break;

        case 401:
          // No autenticado: sesión expirada
          authService.logout();
          router.navigate(['/login'], {
            queryParams: { motivo: 'sesion-expirada' },
          });
          // No mostramos notificación aquí: el componente de login lo maneja
          break;

        case 403:
          notificaciones.error(
            'Acceso denegado',
            'No tienes permisos para realizar esta acción.'
          );
          router.navigate(['/acceso-denegado']);
          break;

        case 404:
          notificaciones.error(
            'No encontrado',
            problema?.detail ?? 'El recurso solicitado no existe.'
          );
          break;

        case 409:
          notificaciones.advertencia(
            problema?.title ?? 'Conflicto',
            problema?.detail ?? 'Ya existe un registro con esos datos.'
          );
          break;

        case 422:
          notificaciones.error(
            problema?.title ?? 'Operación inválida',
            problema?.detail ?? 'La operación no puede realizarse en el estado actual.'
          );
          break;

        case 0:
          notificaciones.error(
            'Sin conexión',
            'No se puede conectar al servidor. Verifica tu conexión a internet.'
          );
          break;

        case 500:
          notificaciones.error(
            'Error del servidor',
            'Ocurrió un error inesperado. Si persiste, contacta al administrador.'
          );
          break;

        default:
          notificaciones.error(
            'Error',
            `Error inesperado (${error.status}). Intenta nuevamente.`
          );
      }

      // Siempre propagar el error para que los componentes puedan reaccionar
      return throwError(() => error);
    })
  );
};
```

---

## 10. Testing Básico: JUnit 5 + Mockito y HttpClientTestingModule

Las pruebas automatizadas son una buena práctica esencial. En este nivel nos enfocaremos en pruebas unitarias: probar una clase de forma aislada, reemplazando sus dependencias con "mocks" (objetos falsos que simulan el comportamiento real).

### 10.1 Pruebas del AuthService con JUnit 5 y Mockito

```java
// src/test/java/co/uniquindio/triage/auth/service/AuthServiceTest.java

package co.uniquindio.triage.auth.service;

import co.uniquindio.triage.auth.dto.LoginRequest;
import co.uniquindio.triage.auth.dto.RegisterRequest;
import co.uniquindio.triage.shared.exception.ConflictoException;
import co.uniquindio.triage.shared.exception.CredencialesInvalidasException;
import co.uniquindio.triage.shared.util.JwtUtil;
import co.uniquindio.triage.usuario.entity.Rol;
import co.uniquindio.triage.usuario.entity.Usuario;
import co.uniquindio.triage.usuario.repository.UsuarioRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.security.crypto.password.PasswordEncoder;

import java.util.Optional;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.anyString;
import static org.mockito.Mockito.*;

/**
 * Pruebas unitarias para AuthService.
 *
 * @ExtendWith(MockitoExtension.class): Habilita Mockito en JUnit 5.
 * @Mock: Crea un mock (objeto falso) para la dependencia.
 * @InjectMocks: Crea la instancia real de AuthService e inyecta los mocks.
 *
 * Usamos AssertJ (assertThat) en lugar de assertEquals de JUnit:
 * - Más legible y expresivo
 * - Mejores mensajes de error al fallar
 * - Más fluente: assertThat(valor).isEqualTo(esperado)
 */
@ExtendWith(MockitoExtension.class)
@DisplayName("AuthService — Pruebas unitarias")
class AuthServiceTest {

    @Mock
    private UsuarioRepository usuarioRepository;

    @Mock
    private PasswordEncoder passwordEncoder;

    @Mock
    private JwtUtil jwtUtil;

    @InjectMocks
    private AuthService authService;

    // Datos de prueba reutilizables
    private Usuario usuarioValido;
    private final String CORREO_VALIDO = "estudiante@uniquindio.edu.co";
    private final String PASSWORD_VALIDA = "password123";
    private final String HASH_PASSWORD = "$2a$10$hasheado";

    @BeforeEach
    void setUp() {
        // Crear un usuario de prueba antes de cada test
        usuarioValido = new Usuario();
        usuarioValido.setNombre("Juan Pérez");
        usuarioValido.setCorreo(CORREO_VALIDO);
        usuarioValido.setPasswordHash(HASH_PASSWORD);
        usuarioValido.setRol(Rol.ESTUDIANTE);
        usuarioValido.setActivo(true);
    }

    @Nested
    @DisplayName("Login")
    class LoginTests {

        @Test
        @DisplayName("Debe retornar LoginResponse cuando las credenciales son válidas")
        void debeRetornarTokenCuandoCredencialesValidas() {
            // ARRANGE: preparar el escenario (qué responden los mocks)
            var request = new LoginRequest(CORREO_VALIDO, PASSWORD_VALIDA);

            when(usuarioRepository.findByCorreo(CORREO_VALIDO))
                .thenReturn(Optional.of(usuarioValido));
            when(passwordEncoder.matches(PASSWORD_VALIDA, HASH_PASSWORD))
                .thenReturn(true);
            when(jwtUtil.generarToken(usuarioValido))
                .thenReturn("jwt-token-falso-para-test");
            when(jwtUtil.obtenerExpiracionMs())
                .thenReturn(86400000L);

            // ACT: ejecutar el método bajo prueba
            var respuesta = authService.login(request);

            // ASSERT: verificar el resultado
            assertThat(respuesta).isNotNull();
            assertThat(respuesta.token()).isEqualTo("jwt-token-falso-para-test");
            assertThat(respuesta.correo()).isEqualTo(CORREO_VALIDO);
            assertThat(respuesta.rol()).isEqualTo("ESTUDIANTE");
            assertThat(respuesta.tipo()).isEqualTo("Bearer");

            // Verificar que se llamaron los métodos esperados
            verify(usuarioRepository).findByCorreo(CORREO_VALIDO);
            verify(passwordEncoder).matches(PASSWORD_VALIDA, HASH_PASSWORD);
            verify(jwtUtil).generarToken(usuarioValido);
        }

        @Test
        @DisplayName("Debe lanzar CredencialesInvalidasException cuando el correo no existe")
        void debeLanzarExcepcionCuandoCorreoNoExiste() {
            // ARRANGE
            var request = new LoginRequest("noexiste@test.com", PASSWORD_VALIDA);

            when(usuarioRepository.findByCorreo("noexiste@test.com"))
                .thenReturn(Optional.empty());

            // ASSERT + ACT: verificar que se lanza la excepción esperada
            assertThatThrownBy(() -> authService.login(request))
                .isInstanceOf(CredencialesInvalidasException.class)
                .hasMessageContaining("incorrectos");

            // Verificar que NO se llamó al passwordEncoder (falló antes)
            verify(passwordEncoder, never()).matches(anyString(), anyString());
        }

        @Test
        @DisplayName("Debe lanzar CredencialesInvalidasException cuando la contraseña no coincide")
        void debeLanzarExcepcionCuandoPasswordIncorrecta() {
            // ARRANGE
            var request = new LoginRequest(CORREO_VALIDO, "passwordMal");

            when(usuarioRepository.findByCorreo(CORREO_VALIDO))
                .thenReturn(Optional.of(usuarioValido));
            when(passwordEncoder.matches("passwordMal", HASH_PASSWORD))
                .thenReturn(false);   // La contraseña NO coincide

            // ACT + ASSERT
            assertThatThrownBy(() -> authService.login(request))
                .isInstanceOf(CredencialesInvalidasException.class);

            // Verificar que NO se generó un token
            verify(jwtUtil, never()).generarToken(any());
        }
    }

    @Nested
    @DisplayName("Registro")
    class RegistroTests {

        @Test
        @DisplayName("Debe registrar un nuevo estudiante correctamente")
        void debeRegistrarEstudianteCorrecto() {
            // ARRANGE
            var request = new RegisterRequest(
                "Ana García", CORREO_VALIDO, PASSWORD_VALIDA, "12345678"
            );

            when(usuarioRepository.existsByCorreo(CORREO_VALIDO)).thenReturn(false);
            when(passwordEncoder.encode(PASSWORD_VALIDA)).thenReturn(HASH_PASSWORD);
            when(usuarioRepository.save(any(Usuario.class))).thenAnswer(i -> i.getArgument(0));

            // ACT
            assertThatCode(() -> authService.registrar(request))
                .doesNotThrowAnyException();

            // ASSERT: verificar que se guardó un usuario con los datos correctos
            verify(usuarioRepository).save(argThat(usuario ->
                usuario.getCorreo().equals(CORREO_VALIDO) &&
                usuario.getPasswordHash().equals(HASH_PASSWORD) &&
                usuario.getRol() == Rol.ESTUDIANTE &&
                usuario.isActivo()
            ));
        }

        @Test
        @DisplayName("Debe lanzar ConflictoException si el correo ya está registrado")
        void debeLanzarExcepcionCuandoCorreoDuplicado() {
            // ARRANGE
            var request = new RegisterRequest(
                "Pedro", CORREO_VALIDO, PASSWORD_VALIDA, "11111111"
            );

            when(usuarioRepository.existsByCorreo(CORREO_VALIDO)).thenReturn(true);

            // ACT + ASSERT
            assertThatThrownBy(() -> authService.registrar(request))
                .isInstanceOf(ConflictoException.class)
                .hasMessageContaining(CORREO_VALIDO);

            // Verificar que NO se intentó guardar
            verify(usuarioRepository, never()).save(any());
        }
    }
}
```

### 10.2 Pruebas del SolicitudService

```java
// src/test/java/co/uniquindio/triage/solicitud/service/SolicitudServiceTest.java

package co.uniquindio.triage.solicitud.service;

import co.uniquindio.triage.shared.exception.RecursoNoEncontradoException;
import co.uniquindio.triage.shared.exception.TransicionEstadoInvalidaException;
import co.uniquindio.triage.solicitud.dto.SolicitudResponse;
import co.uniquindio.triage.solicitud.entity.EstadoSolicitud;
import co.uniquindio.triage.solicitud.entity.Solicitud;
import co.uniquindio.triage.solicitud.entity.TipoSolicitud;
import co.uniquindio.triage.solicitud.repository.SolicitudRepository;
import co.uniquindio.triage.usuario.entity.Usuario;
import co.uniquindio.triage.usuario.repository.UsuarioRepository;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContext;
import org.springframework.security.core.context.SecurityContextHolder;

import java.util.Optional;
import java.util.UUID;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
@DisplayName("SolicitudService — Pruebas unitarias")
class SolicitudServiceTest {

    @Mock private SolicitudRepository solicitudRepository;
    @Mock private UsuarioRepository usuarioRepository;
    @Mock private SolicitudMapper mapper;
    @Mock private TransicionEstadoValidator transicionValidator;
    @Mock private PriorizacionService priorizacionService;

    @InjectMocks
    private SolicitudService solicitudService;

    @Test
    @DisplayName("obtenerPorId debe lanzar excepción si la solicitud no existe")
    void obtenerPorIdDebeLanzarExcepcionSiNoExiste() {
        // ARRANGE
        var idInexistente = UUID.randomUUID();
        when(solicitudRepository.findByIdConHistorial(idInexistente))
            .thenReturn(Optional.empty());

        // ACT + ASSERT
        assertThatThrownBy(() -> solicitudService.obtenerPorId(idInexistente))
            .isInstanceOf(RecursoNoEncontradoException.class)
            .hasMessageContaining(idInexistente.toString());
    }

    @Test
    @DisplayName("obtenerPorId debe devolver la solicitud si existe")
    void obtenerPorIdDebeDevolverSolicitudSiExiste() {
        // ARRANGE
        var id = UUID.randomUUID();
        var solicitud = new Solicitud();
        var responseEsperada = mock(SolicitudResponse.class);

        when(solicitudRepository.findByIdConHistorial(id))
            .thenReturn(Optional.of(solicitud));
        when(mapper.toResponse(solicitud)).thenReturn(responseEsperada);

        // ACT
        var resultado = solicitudService.obtenerPorId(id);

        // ASSERT
        assertThat(resultado).isEqualTo(responseEsperada);
        verify(mapper).toResponse(solicitud);
    }

    /**
     * Mockear el SecurityContext es un poco más elaborado.
     * Spring Security guarda el usuario en un ThreadLocal (SecurityContextHolder).
     * En el test, tenemos que simular que hay un usuario autenticado.
     */
    private void simularUsuarioAutenticado(String correo) {
        var authentication = mock(Authentication.class);
        var securityContext = mock(SecurityContext.class);

        when(authentication.getName()).thenReturn(correo);
        when(securityContext.getAuthentication()).thenReturn(authentication);
        SecurityContextHolder.setContext(securityContext);
    }
}
```

### 10.3 Pruebas del AuthService Angular con HttpClientTestingModule

```typescript
// src/app/core/services/auth.service.spec.ts

import { TestBed } from '@angular/core/testing';
import {
  HttpClientTestingModule,
  HttpTestingController,
} from '@angular/common/http/testing';
import { RouterTestingModule } from '@angular/router/testing';
import { AuthService } from './auth.service';
import { API_URL } from '../tokens/api.tokens';
import { LoginResponse } from '../models/auth.model';

describe('AuthService', () => {
  let service: AuthService;
  let httpMock: HttpTestingController;
  const API_BASE = 'http://localhost:8080/api/v1';

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [
        HttpClientTestingModule,   // Módulo de prueba para HttpClient
        RouterTestingModule,       // Módulo de prueba para Router
      ],
      providers: [
        AuthService,
        // Proveer el token de inyección con un valor de prueba
        { provide: API_URL, useValue: API_BASE },
      ],
    });

    service = TestBed.inject(AuthService);
    httpMock = TestBed.inject(HttpTestingController);

    // Limpiar localStorage antes de cada prueba
    localStorage.clear();
  });

  afterEach(() => {
    // Verificar que no hayan quedado peticiones HTTP pendientes sin responder
    httpMock.verify();
    localStorage.clear();
  });

  it('debe crearse correctamente', () => {
    expect(service).toBeTruthy();
  });

  describe('login()', () => {
    it('debe hacer POST a /auth/login y guardar el token', () => {
      // ARRANGE: respuesta simulada del servidor
      const respuestaSimulada: LoginResponse = {
        token: 'jwt-test-token',
        tipo: 'Bearer',
        expiracionMs: 86400000,
        nombre: 'Juan Pérez',
        correo: 'juan@test.com',
        rol: 'ESTUDIANTE',
      };

      // ACT: llamar al método
      let resultadoObtenido: LoginResponse | undefined;
      service.login({ correo: 'juan@test.com', password: '123456' }).subscribe({
        next: (res) => (resultadoObtenido = res),
      });

      // ASSERT: verificar la petición HTTP
      const req = httpMock.expectOne(`${API_BASE}/auth/login`);
      expect(req.request.method).toBe('POST');
      expect(req.request.body).toEqual({
        correo: 'juan@test.com',
        password: '123456',
      });

      // Simular la respuesta del servidor
      req.flush(respuestaSimulada);

      // Verificar que el token se guardó en localStorage
      expect(localStorage.getItem('triage_token')).toBe('jwt-test-token');

      // Verificar que el estado de autenticación se actualizó
      expect(service.estaAutenticado).toBeTrue();
      expect(service.usuarioActual?.correo).toBe('juan@test.com');
    });

    it('debe propagar el error cuando las credenciales son inválidas', () => {
      // ACT
      let errorObtenido: any;
      service.login({ correo: 'bad@test.com', password: 'mal' }).subscribe({
        error: (err) => (errorObtenido = err),
      });

      // Simular error 401 del servidor
      const req = httpMock.expectOne(`${API_BASE}/auth/login`);
      req.flush(
        { title: 'Credenciales inválidas', detail: 'Correo o contraseña incorrectos' },
        { status: 401, statusText: 'Unauthorized' }
      );

      // Verificar que el error llegó al suscriptor
      expect(errorObtenido.status).toBe(401);

      // El usuario NO debe estar autenticado
      expect(service.estaAutenticado).toBeFalse();
      expect(localStorage.getItem('triage_token')).toBeNull();
    });
  });

  describe('logout()', () => {
    it('debe limpiar el estado y el localStorage', () => {
      // Simular que hay un usuario logueado
      localStorage.setItem('triage_token', 'un-token');
      localStorage.setItem('triage_user', JSON.stringify({
        nombre: 'Test',
        correo: 'test@test.com',
        rol: 'ESTUDIANTE',
      }));

      // ACT
      service.logout();

      // ASSERT
      expect(localStorage.getItem('triage_token')).toBeNull();
      expect(localStorage.getItem('triage_user')).toBeNull();
      expect(service.estaAutenticado).toBeFalse();
    });
  });
});
```

```typescript
// src/app/core/services/solicitud.service.spec.ts

import { TestBed } from '@angular/core/testing';
import {
  HttpClientTestingModule,
  HttpTestingController,
} from '@angular/common/http/testing';
import { SolicitudService } from './solicitud.service';
import { API_URL } from '../tokens/api.tokens';
import { SolicitudResponse } from '../models/solicitud.model';

describe('SolicitudService', () => {
  let service: SolicitudService;
  let httpMock: HttpTestingController;
  const API_BASE = 'http://localhost:8080/api/v1';

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [
        SolicitudService,
        { provide: API_URL, useValue: API_BASE },
      ],
    });

    service = TestBed.inject(SolicitudService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => { httpMock.verify(); });

  it('crear() debe hacer POST a /solicitudes', () => {
    const solicitudCreada = { id: 'uuid-123', tipo: 'HOMOLOGACION' } as any;

    service.crear({
      tipo: 'HOMOLOGACION',
      descripcion: 'Necesito homologar Cálculo I',
      canalOrigen: 'PLATAFORMA_WEB',
    }).subscribe((res) => {
      expect(res).toEqual(solicitudCreada);
    });

    const req = httpMock.expectOne(`${API_BASE}/solicitudes`);
    expect(req.request.method).toBe('POST');
    req.flush(solicitudCreada);
  });

  it('listar() debe incluir los parámetros de filtro en la URL', () => {
    service.listar({
      estado: 'REGISTRADA',
      page: 0,
      size: 10,
    }).subscribe();

    const req = httpMock.expectOne(
      (request) => request.url === `${API_BASE}/solicitudes`
    );
    expect(req.request.params.get('estado')).toBe('REGISTRADA');
    expect(req.request.params.get('page')).toBe('0');
    expect(req.request.params.get('size')).toBe('10');

    req.flush({ content: [], totalElements: 0 });
  });
});
```

---

## 11. Errores Comunes y Troubleshooting

### 11.1 Error: CORS bloqueado incluso con configuración correcta

**Síntoma:**
```
Access to XMLHttpRequest at 'http://localhost:8080/api/v1/solicitudes'
from origin 'http://localhost:4200' has been blocked by CORS policy
```

**Causa más común:** Tienes CORS configurado en `WebMvcConfigurer` pero Spring Security está interceptando las peticiones antes. La petición OPTIONS (preflight) está siendo rechazada por Spring Security.

**Solución:**
```java
// Asegúrate de que CORS esté configurado DENTRO del SecurityFilterChain
// y que las peticiones OPTIONS sean permitidas:
http.cors(cors -> cors.configurationSource(corsConfigurationSource()))
```

**Verificación rápida:** Usa curl para hacer el preflight manualmente:
```bash
curl -X OPTIONS http://localhost:8080/api/v1/solicitudes \
  -H "Origin: http://localhost:4200" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: authorization, content-type" \
  -v
```
Si la respuesta tiene `Access-Control-Allow-Origin: http://localhost:4200`, está bien configurado.

---

### 11.2 Error: 401 Unauthorized en cada petición después del login

**Síntoma:** El login funciona y devuelve un token, pero todas las peticiones siguientes dan 401.

**Causas posibles:**

1. **El interceptor JWT no está registrado:**
```typescript
// Verificar en app.config.ts:
provideHttpClient(
  withInterceptors([jwtInterceptor, errorInterceptor])  // ← debe estar aquí
)
```

2. **El token no se está guardando en localStorage:**
```typescript
// En AuthService.login():
tap((response) => {
  localStorage.setItem(this.TOKEN_KEY, response.token);  // ← ¿está esto?
})
```

3. **El filtro JWT en el backend no está extrayendo el token correctamente:**
```java
// Verificar que el filtro busca el header "Authorization" con prefijo "Bearer "
String authHeader = request.getHeader("Authorization");
if (authHeader != null && authHeader.startsWith("Bearer ")) {
    String token = authHeader.substring(7);  // Quitar "Bearer "
}
```

---

### 11.3 Error: LazyInitializationException en la serialización

**Síntoma:**
```
org.hibernate.LazyInitializationException: could not initialize proxy - no Session
```

**Causa:** Estás devolviendo una entidad JPA directamente desde el Controller, y cuando Jackson intenta serializar las relaciones con `LAZY` loading, ya no hay una sesión de Hibernate activa.

**Solución:** Siempre usa DTOs. El mapper transforma la entidad a un Record antes de salir del `@Service`, donde la sesión todavía está activa.

```java
// ❌ MAL: devolver entidad JPA directamente
@GetMapping("/{id}")
public Solicitud obtenerSolicitud(@PathVariable UUID id) {
    return solicitudRepository.findById(id).get();
}

// ✅ BIEN: devolver DTO
@GetMapping("/{id}")
public SolicitudResponse obtenerSolicitud(@PathVariable UUID id) {
    return solicitudService.obtenerPorId(id);  // El service mapea a DTO
}
```

---

### 11.4 Error: Angular no puede parsear la respuesta JSON

**Síntoma:**
```
SyntaxError: Unexpected token '<', "<!DOCTYPE..." is not valid JSON
```

**Causa:** Angular está recibiendo una página HTML (probablemente una página de error del servidor o del proxy) en lugar de JSON.

**Solución:**
- Verifica que la URL base del backend esté correcta en `environment.ts`
- Verifica que el servidor Spring Boot esté corriendo
- Verifica que no haya un proxy inverso en medio que esté devolviendo HTML

---

### 11.5 Error: ExpressionChangedAfterItHasBeenCheckedError

**Síntoma:** Error en la consola de Angular durante el desarrollo, aunque la app parece funcionar.

**Causa:** Estás modificando el estado del componente en `ngAfterViewInit` o en una operación asíncrona que Angular no puede detectar.

**Solución:**
```typescript
import { ChangeDetectorRef } from '@angular/core';

// Inyectar ChangeDetectorRef e invocar detectChanges() después de modificar el estado
constructor(private cdr: ChangeDetectorRef) {}

ngAfterViewInit() {
  this.datos = [...];
  this.cdr.detectChanges();  // Forzar detección de cambios
}
```

---

### 11.6 Tabla de códigos HTTP y su significado en el contexto del Triage

| Código | Nombre | Cuándo ocurre | Qué hacer en Angular |
|---|---|---|---|
| 200 | OK | Petición exitosa | Usar el cuerpo de la respuesta |
| 201 | Created | Solicitud creada exitosamente | Navegar al detalle del nuevo recurso |
| 400 | Bad Request | Error de validación | Mostrar mensajes de error específicos |
| 401 | Unauthorized | Token inválido o vencido | Redirigir al login |
| 403 | Forbidden | Sin permiso para esa acción | Redirigir a acceso-denegado |
| 404 | Not Found | Solicitud no existe | Mostrar mensaje "no encontrado" |
| 409 | Conflict | Correo duplicado, estado inválido | Mostrar advertencia |
| 422 | Unprocessable | Transición de estado no permitida | Explicar al usuario qué no es posible |
| 500 | Server Error | Bug en el backend | Mensaje genérico + notificar al equipo |

---

## 12. Resumen y Cheat Sheet

### El flujo en 3 líneas

1. **Angular** construye una petición HTTP → el interceptor JWT le pone el token → la petición viaja al backend.
2. **Spring Security** valida el token → el Controller recibe la petición validada → el Service aplica lógica → el Repository habla con la BD.
3. El resultado viaja de vuelta como JSON → el Observable en Angular emite el valor → el componente actualiza la UI.

### Configuración CORS mínima funcional

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
        .cors(cors -> cors.configurationSource(corsConfigurationSource()))
        .csrf(csrf -> csrf.disable())
        .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
        .authorizeHttpRequests(a -> a
            .requestMatchers("/api/v1/auth/**").permitAll()
            .anyRequest().authenticated())
        .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
        .build();
}
```

### Interceptor JWT en Angular (forma funcional)

```typescript
export const jwtInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthService).obtenerToken();
  if (token) {
    return next(req.clone({ setHeaders: { Authorization: `Bearer ${token}` } }));
  }
  return next(req);
};
```

### Guard funcional en Angular

```typescript
export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);
  return auth.estaAutenticado
    ? true
    : router.createUrlTree(['/login'], { queryParams: { returnUrl: state.url } });
};
```

### Variables de entorno en Angular

```typescript
// environment.ts (dev)
export const environment = { production: false, apiUrl: 'http://localhost:8080/api/v1' };
// environment.prod.ts (prod)
export const environment = { production: true, apiUrl: 'https://tu-backend.railway.app/api/v1' };
// Uso en el servicio:
private readonly baseUrl = `${inject(API_URL)}/solicitudes`;
```

### Paginación en Spring Data JPA

```java
// Repositorio:
Page<Solicitud> buscarConFiltros(EstadoSolicitud estado, Pageable pageable);
// Service:
return repository.buscarConFiltros(estado, pageable).map(mapper::toResponse);
// Angular:
this.solicitudService.listar({ page: 0, size: 10, sort: 'fechaRegistro,desc' }).subscribe(...)
```

### Manejo de errores con Problem Details

```java
// Backend:
@ExceptionHandler(RecursoNoEncontradoException.class)
public ProblemDetail handleNotFound(RecursoNoEncontradoException ex) {
    var p = ProblemDetail.forStatus(HttpStatus.NOT_FOUND);
    p.setDetail(ex.getMessage());
    return p;
}
```

```typescript
// Angular interceptor:
catchError((error: HttpErrorResponse) => {
  const problema = error.error as ProblemDetail;
  notificaciones.error(problema?.title, problema?.detail);
  return throwError(() => error);
})
```

### Antes vs Ahora — Resumen de la guía

| Concepto | Forma antigua | Forma nueva (2026) |
|---|---|---|
| Configuración de seguridad | `WebSecurityConfigurerAdapter` | `@Bean SecurityFilterChain` |
| CORS con Security | Config separada (fallaba) | Dentro de `SecurityFilterChain` |
| Interceptor Angular | Clase con `HttpInterceptor` | Función `HttpInterceptorFn` |
| Guard Angular | Clase con `CanActivate` | Función `CanActivateFn` |
| DTOs | Clases con getters/setters | Records de Java 21 |
| Query params Angular | Manual con strings | `HttpParams` tipado |
| Control de flujo template | `*ngIf`, `*ngFor` | `@if`, `@for` |
| DI en Angular | `constructor(private s: S)` | `inject(S)` |

---

## 13. Referencias y Recursos Adicionales

- [Spring Security Reference](https://docs.spring.io/spring-security/reference/index.html)
- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/reference/jpa.html)
- [Angular HTTP Client Guide](https://angular.dev/guide/http)
- [Angular Signals y Reactive Patterns](https://angular.dev/guide/signals)
- [RFC 7807 — Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc7807)
- [JWT.io — Debugger y herramienta de aprendizaje](https://jwt.io)
- [AssertJ Documentation](https://assertj.github.io/doc/)
- [Mockito Documentation](https://site.mockito.org/)

---

> **Autor:** José Alfredo Ramírez Espinosa  
> **Materia:** Programación Avanzada  
> **Programa:** Ingeniería de Sistemas y Computación  
> **Universidad del Quindío** | Armenia, Colombia  
> **Año:** 2026  
> *Material de uso académico. Todos los derechos reservados al autor.*
