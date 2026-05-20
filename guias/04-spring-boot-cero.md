# Guía 04 — Spring Boot desde Cero: Tu Primer Proyecto

> **Materia:** Programación Avanzada | **Programa:** Ingeniería de Sistemas y Computación  
> **Universidad del Quindío** | Armenia, Colombia | **Año:** 2026  
> **Stack:** Java 21+ · Spring Boot 3.4+ · PostgreSQL · Maven

---

## Tabla de Contenidos

1. [Objetivos de Aprendizaje](#1-objetivos-de-aprendizaje)
2. [Prerrequisitos](#2-prerrequisitos)
3. [¿Qué es Spring Framework y qué es Spring Boot?](#3-qué-es-spring-framework-y-qué-es-spring-boot)
4. [Inversión de Control (IoC) e Inyección de Dependencias (DI)](#4-inversión-de-control-ioc-e-inyección-de-dependencias-di)
5. [Crear el Proyecto con Spring Initializr](#5-crear-el-proyecto-con-spring-initializr)
6. [Anatomía del Proyecto: Cada Carpeta y Archivo Explicado](#6-anatomía-del-proyecto-cada-carpeta-y-archivo-explicado)
7. [application.properties vs application.yml](#7-applicationproperties-vs-applicationyml)
8. [Anotaciones Fundamentales de Spring Boot](#8-anotaciones-fundamentales-de-spring-boot)
9. [Ciclo de Vida de un Bean](#9-ciclo-de-vida-de-un-bean)
10. [Perfiles de Configuración: dev, prod, test](#10-perfiles-de-configuración-dev-prod-test)
11. [Tu Primer Endpoint Funcional Paso a Paso](#11-tu-primer-endpoint-funcional-paso-a-paso)
12. [Logging con SLF4J y Logback](#12-logging-con-slf4j-y-logback)
13. [Manejo Global de Errores](#13-manejo-global-de-errores)
14. [Correr el Proyecto y Probar con Swagger UI](#14-correr-el-proyecto-y-probar-con-swagger-ui)
15. [Errores Comunes y Troubleshooting](#15-errores-comunes-y-troubleshooting)
16. [Ejercicios Prácticos](#16-ejercicios-prácticos)
17. [Resumen y Cheat Sheet](#17-resumen-y-cheat-sheet)
18. [Referencias y Recursos Adicionales](#18-referencias-y-recursos-adicionales)

---

## 1. Objetivos de Aprendizaje

Al terminar esta guía serás capaz de:

- Explicar con tus propias palabras qué es Spring Boot y en qué se diferencia de Spring Framework.
- Comprender y aplicar los conceptos de Inversión de Control (IoC) e Inyección de Dependencias (DI).
- Crear un proyecto Spring Boot desde Spring Initializr con las dependencias correctas.
- Navegar la estructura de carpetas del proyecto y saber para qué sirve cada archivo.
- Configurar la aplicación usando `application.yml` con perfiles para distintos entornos.
- Reconocer y usar las anotaciones principales de Spring Boot (`@SpringBootApplication`, `@RestController`, `@Service`, `@Repository`, etc.).
- Entender el ciclo de vida de un bean administrado por Spring.
- Crear tu primer endpoint REST funcional y probarlo con Swagger UI.
- Configurar logging con SLF4J.
- Manejar errores de forma global con `@ControllerAdvice`.

---

## 2. Prerrequisitos

Antes de comenzar, asegúrate de tener lo siguiente instalado y verificado:

| Herramienta | Versión mínima | Verificar con |
|---|---|---|
| JDK | 21+ | `java -version` |
| Maven | 3.9+ | `mvn -version` |
| IDE | IntelliJ IDEA (recomendado) o VS Code con extensión Java | — |
| PostgreSQL | 16+ | `psql --version` |
| Postman o Bruno | Cualquier versión reciente | — |
| Docker (opcional) | 25+ | `docker --version` |

**Conocimiento previo necesario:**
- Java básico: clases, objetos, herencia, interfaces, polimorfismo.
- Conocimiento de la Guía 03 (Java Moderno: Records, Optional, Stream API).
- Comprensión básica de qué es HTTP (petición/respuesta, GET, POST, códigos 200/404/500).

---

## 3. ¿Qué es Spring Framework y qué es Spring Boot?

### 3.1 El Problema que Resuelve Spring

Imagina que vas a construir una casa. Podrías fabricar tú mismo cada ladrillo, mezclar el cemento, cortar la madera y soldar el acero. Técnicamente es posible, pero nadie lo hace así en la industria: se usan materiales prefabricados y procesos estandarizados.

En el desarrollo de software empresarial pasa lo mismo. Antes de Spring (en los años 2000), construir una aplicación Java empresarial requería escribir toneladas de código repetitivo: configurar conexiones a bases de datos manualmente, gestionar transacciones a mano, crear objetos en el lugar correcto y en el orden correcto, y mantener todo ese "pegamento" entre componentes.

**Spring Framework** (lanzado en 2003 por Rod Johnson) llegó para resolver ese problema. Es un ecosistema de herramientas que te permite construir aplicaciones Java empresariales de forma más limpia, modular y con mucho menos código repetitivo.

### 3.2 Spring Framework vs Spring Boot — La Diferencia Clave

Aunque en la práctica usarás **Spring Boot** directamente, es importante entender por qué existe:

```
Spring Framework (2003)
├── Es el motor: contiene todas las funcionalidades
├── Requería XML extenso o configuración Java explícita
├── Tú debías configurar: servidor de aplicaciones, conexión BD, 
│   manejo de errores, serialización JSON, etc.
└── Era poderoso pero tedioso de arrancar

Spring Boot (2014)
├── Es una CAPA SOBRE Spring Framework
├── Lema: "Convention over Configuration" (Convención sobre Configuración)
├── Incluye un servidor web embebido (Tomcat, por defecto)
├── Auto-configura todo lo que puede basándose en las dependencias presentes
├── Un solo punto de arranque: tu método main()
└── Listo para producción: métricas, health checks, actuator
```

**Analogía perfecta:** Spring Framework es el motor de un carro. Spring Boot es el carro completo con el motor ya instalado, con la gasolina puesta, el GPS configurado y las llaves en la ignición. Solo tienes que arrancar.

### 3.3 El Ecosistema Spring

Spring no es solo un framework; es un ecosistema completo. En este semestre vas a usar:

| Módulo | Para qué sirve | Guía |
|---|---|---|
| Spring Core | IoC, DI, gestión de beans | Guía 04 (esta) |
| Spring Web MVC | Controladores REST, HTTP | Guía 06 |
| Spring Data JPA | Acceso a base de datos con JPA | Guía 05 |
| Spring Security | Autenticación y autorización | Guía 07 |
| Spring Boot Actuator | Monitoreo y métricas | Guía 11 |

---

## 4. Inversión de Control (IoC) e Inyección de Dependencias (DI)

Este es **el concepto más importante** de Spring. Si lo entiendes bien, todo lo demás tiene sentido automáticamente.

### 4.1 El Problema: El Acoplamiento Fuerte

Observa este código Java que seguramente has escrito antes:

```java
// ❌ El problema: acoplamiento fuerte
public class GestorSolicitudes {
    
    // Aquí estamos CREANDO la dependencia dentro de la misma clase
    private SolicitudRepository repositorio = new SolicitudRepositoryImpl();
    
    public void registrar(Solicitud solicitud) {
        repositorio.guardar(solicitud);
    }
}
```

¿Cuál es el problema aquí? `GestorSolicitudes` está **fuertemente acoplado** a `SolicitudRepositoryImpl`. Esto significa:

1. **No puedes probar** `GestorSolicitudes` sin también usar la base de datos real.
2. **No puedes cambiar** la implementación del repositorio sin modificar `GestorSolicitudes`.
3. **No puedes reutilizar** `GestorSolicitudes` con un repositorio diferente.

### 4.2 La Solución: Inversión de Control

La **Inversión de Control (IoC)** dice: "No crees tus dependencias. Pídelas. Alguien más se encargará de crearlas y dártelas."

Ese "alguien más" en Spring se llama el **Contenedor IoC** (también llamado **Application Context**).

```
Sin IoC:                        Con IoC:
┌─────────────────────┐         ┌──────────────────────┐
│  GestorSolicitudes  │         │  Contenedor IoC       │
│                     │         │  (Application Context)│
│  new Repository()   │         │                       │
│  ─────────────────  │  ──►    │  Crea: Repository     │
│  Yo controlo mis    │         │  Crea: Gestor         │
│  dependencias       │         │  Inyecta en: Gestor   │
└─────────────────────┘         └──────────────────────┘
                                        │
                                        ▼
                                 GestorSolicitudes
                                 recibe su Repository
                                 ya creado y listo
```

El **control de la creación de objetos** se **invierte**: antes lo hacías tú, ahora lo hace el framework.

### 4.3 Inyección de Dependencias (DI)

La **Inyección de Dependencias** es el mecanismo concreto de la IoC. El contenedor "inyecta" (entrega) las dependencias en los objetos que las necesitan. Hay tres formas:

#### Inyección por Constructor (✅ Recomendada en 2026)

```java
// ✅ FORMA CORRECTA Y MODERNA
@Service
public class SolicitudService {
    
    // La dependencia se declara como final (inmutable)
    private final SolicitudRepository solicitudRepository;
    
    // El contenedor IoC llama a este constructor automáticamente
    // y le pasa el SolicitudRepository que él mismo creó
    public SolicitudService(SolicitudRepository solicitudRepository) {
        this.solicitudRepository = solicitudRepository;
    }
    
    public void registrar(Solicitud solicitud) {
        solicitudRepository.guardar(solicitud);
    }
}
```

> **¿Por qué constructor?** Porque el objeto es inmutable (campo `final`), las dependencias son visibles desde afuera (facilita pruebas), y el compilador te obliga a proveerlas (no puedes "olvidar" una dependencia).

#### Inyección por Campo con @Autowired (⚠️ Evitar)

```java
// ⚠️ FUNCIONA PERO NO SE RECOMIENDA
@Service
public class SolicitudService {
    
    @Autowired  // Spring inyecta esto mediante reflexión
    private SolicitudRepository solicitudRepository;
    
    // Problema: el campo NO es final, y no puedes probar
    // fácilmente esta clase sin Spring
}
```

#### Inyección por Setter (🔄 Solo para dependencias opcionales)

```java
// 🔄 Úsalo solo cuando la dependencia es realmente opcional
@Service
public class SolicitudService {
    
    private NotificacionService notificacionService;
    
    @Autowired(required = false)
    public void setNotificacionService(NotificacionService notificacionService) {
        this.notificacionService = notificacionService;
    }
}
```

### 4.4 Cómo Funciona Internamente (Simplificado)

Cuando arrancas tu aplicación Spring Boot, pasa esto:

```
1. Se ejecuta main() en TuAplicacion.java
         │
         ▼
2. Spring Boot escanea todos los packages buscando
   clases con anotaciones como @Component, @Service,
   @Repository, @Controller, @Configuration
         │
         ▼
3. Para cada clase encontrada, Spring:
   a) Analiza sus dependencias (constructor o @Autowired)
   b) Crea las instancias en el orden correcto
   c) Las conecta entre sí
   d) Las registra en el Application Context
         │
         ▼
4. El Application Context queda listo con todos los
   "beans" (objetos administrados) disponibles
         │
         ▼
5. Cuando un componente necesita otro, Spring lo
   provee automáticamente del Application Context
```

---

## 5. Crear el Proyecto con Spring Initializr

### 5.1 Usando Spring Initializr Web

Ve a [https://start.spring.io](https://start.spring.io) y configura así:

| Campo | Valor |
|---|---|
| **Project** | Maven |
| **Language** | Java |
| **Spring Boot** | 3.4.x (la más reciente estable) |
| **Group** | `co.edu.uniquindio` |
| **Artifact** | `triage` |
| **Name** | `triage` |
| **Description** | Sistema de Triage y Gestión de Solicitudes Académicas |
| **Package name** | `co.edu.uniquindio.triage` |
| **Packaging** | Jar |
| **Java** | 21 |

**Dependencias a seleccionar** (clic en "ADD DEPENDENCIES"):

| Dependencia | Para qué la necesitamos |
|---|---|
| Spring Web | Crear endpoints REST con Spring MVC |
| Spring Data JPA | Acceso a BD con JPA/Hibernate |
| PostgreSQL Driver | Conectar a PostgreSQL |
| Spring Security | Autenticación y autorización con JWT |
| Validation | Validar entradas con Bean Validation |
| Lombok | Reducir código boilerplate (getters, constructors) |
| Spring Boot DevTools | Recarga automática en desarrollo |
| Spring Boot Actuator | Health checks y métricas |

Haz clic en **GENERATE**, descarga el ZIP y descomprímelo.

### 5.2 Abrirlo en IntelliJ IDEA

1. Abre IntelliJ → `File → Open` → selecciona la carpeta descomprimida.
2. IntelliJ detectará automáticamente que es un proyecto Maven y descargará las dependencias.
3. Espera a que la barra de progreso en la parte inferior desaparezca (~2-5 minutos la primera vez).

> **Importante:** Si usas IntelliJ IDEA Community Edition, tiene soporte completo para Spring Boot. IntelliJ IDEA Ultimate tiene soporte adicional con asistentes visuales, pero no es obligatorio.

---

## 6. Anatomía del Proyecto: Cada Carpeta y Archivo Explicado

Una vez abierto el proyecto, verás esta estructura. Vamos a entender cada parte:

```
triage/
├── .mvn/                          ← Wrapper de Maven (no tocar)
│   └── wrapper/
│       └── maven-wrapper.properties
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── co/edu/uniquindio/triage/
│   │   │       └── TriageApplication.java     ← Punto de entrada de la app
│   │   └── resources/
│   │       ├── application.properties         ← Configuración principal
│   │       ├── static/                        ← Archivos estáticos (HTML, CSS, JS)
│   │       └── templates/                     ← Plantillas Thymeleaf (no usaremos)
│   └── test/
│       └── java/
│           └── co/edu/uniquindio/triage/
│               └── TriageApplicationTests.java ← Tests de integración
├── .gitignore                     ← Archivos a ignorar por Git
├── mvnw                           ← Maven wrapper para Linux/Mac
├── mvnw.cmd                       ← Maven wrapper para Windows
└── pom.xml                        ← Configuración del proyecto Maven
```

Ahora vamos a entender cada elemento importante:

### 6.1 `TriageApplication.java` — El Punto de Entrada

```java
package co.edu.uniquindio.triage;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication  // Esta anotación hace mucho (lo veremos en la sección 8)
public class TriageApplication {

    public static void main(String[] args) {
        // Esta línea arranca TODO el contenedor Spring
        SpringApplication.run(TriageApplication.class, args);
    }
}
```

**¿Qué hace `SpringApplication.run()`?**
1. Arranca el servidor Tomcat embebido (por defecto en el puerto 8080).
2. Escanea el package actual y todos los sub-packages en busca de componentes Spring.
3. Crea y configura el Application Context.
4. Auto-configura todo lo que puede basándose en las dependencias en el classpath.

### 6.2 `pom.xml` — El Corazón del Proyecto Maven

El `pom.xml` es como el "recibo de compras" de tu proyecto: lista todas las librerías que necesitas y Maven las descarga automáticamente.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <!-- El "padre" es Spring Boot, que define versiones compatibles -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.4.1</version>
        <relativePath/>
    </parent>
    
    <!-- Identidad de tu proyecto -->
    <groupId>co.edu.uniquindio</groupId>
    <artifactId>triage</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>triage</name>
    <description>Sistema de Triage y Gestión de Solicitudes Académicas</description>
    
    <properties>
        <java.version>21</java.version>  <!-- Usamos Java 21 -->
    </properties>
    
    <dependencies>
        <!-- Spring Web: para crear APIs REST -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- Spring Data JPA: para persistencia con ORM -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        
        <!-- PostgreSQL Driver: para conectar con PostgreSQL -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Spring Security: para autenticación/autorización -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        
        <!-- Validation: para @NotNull, @NotBlank, etc. -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        
        <!-- Lombok: para reducir boilerplate -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <!-- DevTools: recarga automática en desarrollo -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        
        <!-- Actuator: health checks y métricas -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-actuator</artifactId>
        </dependency>
        
        <!-- SpringDoc OpenAPI: para Swagger UI -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.7.0</version>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <!-- Plugin para compilar y crear el JAR ejecutable -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

> **Nota sobre los "starters":** Las dependencias con nombre `spring-boot-starter-XXX` son paquetes que agrupan múltiples librerías relacionadas. Por ejemplo, `spring-boot-starter-web` incluye Spring MVC, Jackson (para JSON), Tomcat embebido y otras librerías que necesitas para crear una API REST. No tienes que agregar cada una por separado.

---

## 7. `application.properties` vs `application.yml`

### 7.1 ¿Por Qué Preferirnos `application.yml`?

Spring Boot acepta configuración en dos formatos. Vamos a comparar el mismo fragmento en ambos:

```properties
# application.properties — Formato plano, difícil de leer cuando crece
spring.datasource.url=jdbc:postgresql://localhost:5432/triage_db
spring.datasource.username=postgres
spring.datasource.password=secreto123
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
server.port=8080
```

```yaml
# application.yml — Formato jerárquico, mucho más legible
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/triage_db
    username: postgres
    password: secreto123
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true

server:
  port: 8080
```

La estructura jerárquica de YAML es más legible cuando hay muchas propiedades relacionadas. **Usaremos `.yml` en todas las guías.**

> **Atención con la indentación en YAML:** A diferencia de Java, en YAML la indentación tiene significado. Usa **2 espacios** (nunca tabs). Un error de indentación puede romper toda la configuración.

### 7.2 Configuración Completa para el Proyecto de Triage

Elimina el archivo `application.properties` que viene por defecto y crea `application.yml` con el siguiente contenido:

```yaml
# src/main/resources/application.yml

# ─────────────────────────────────────────────
#  Configuración del servidor
# ─────────────────────────────────────────────
server:
  port: 8080
  # Prefijo para todos los endpoints de la API
  servlet:
    context-path: /

# ─────────────────────────────────────────────
#  Información de la aplicación
# ─────────────────────────────────────────────
spring:
  application:
    name: triage-api

  # ─────────────────────────────────────────────
  #  Base de datos PostgreSQL
  # ─────────────────────────────────────────────
  datasource:
    url: jdbc:postgresql://localhost:5432/triage_db
    username: postgres
    password: postgres  # Cambiar en producción
    driver-class-name: org.postgresql.Driver
    # Pool de conexiones HikariCP (viene con Spring Boot)
    hikari:
      pool-name: TriagePool
      maximum-pool-size: 10
      minimum-idle: 2
      connection-timeout: 30000

  # ─────────────────────────────────────────────
  #  JPA / Hibernate
  # ─────────────────────────────────────────────
  jpa:
    hibernate:
      # create-drop: elimina y recrea al iniciar (SOLO DESARROLLO INICIAL)
      # update: actualiza el esquema sin borrar datos (DESARROLLO)
      # validate: solo valida, no modifica (PRODUCCIÓN)
      # none: no hace nada (cuando usas Flyway/Liquibase)
      ddl-auto: update
    show-sql: true  # Muestra las queries SQL en consola
    open-in-view: false  # Buena práctica: desactivar OSIV
    properties:
      hibernate:
        format_sql: true  # Formatea el SQL para legibilidad
        use_sql_comments: true
        jdbc:
          time_zone: UTC

  # ─────────────────────────────────────────────
  #  Mensajes de validación en español
  # ─────────────────────────────────────────────
  messages:
    basename: messages
    encoding: UTF-8

# ─────────────────────────────────────────────
#  Swagger / OpenAPI
# ─────────────────────────────────────────────
springdoc:
  api-docs:
    path: /api-docs
  swagger-ui:
    path: /swagger-ui
    tags-sorter: alpha
    operations-sorter: method

# ─────────────────────────────────────────────
#  Spring Boot Actuator
# ─────────────────────────────────────────────
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics
  endpoint:
    health:
      show-details: when-authorized

# ─────────────────────────────────────────────
#  Logging
# ─────────────────────────────────────────────
logging:
  level:
    root: INFO
    co.edu.uniquindio.triage: DEBUG  # Más detalle para nuestro código
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql: TRACE
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
```

---

## 8. Anotaciones Fundamentales de Spring Boot

Las anotaciones son la forma en que le dices a Spring cómo debe tratar cada clase y método. Aquí está la referencia completa de las que usarás en este semestre.

### 8.1 `@SpringBootApplication` — La Anotación Maestra

```java
@SpringBootApplication
public class TriageApplication {
    public static void main(String[] args) {
        SpringApplication.run(TriageApplication.class, args);
    }
}
```

Esta sola anotación equivale a tres anotaciones combinadas:

```
@SpringBootApplication
       │
       ├── @SpringBootConfiguration
       │       └── Equivale a @Configuration: esta clase puede
       │           definir beans con métodos @Bean
       │
       ├── @EnableAutoConfiguration
       │       └── Le dice a Spring Boot: "auto-configúrate
       │           basándote en las dependencias presentes"
       │           (si ve Spring Data JPA + PostgreSQL Driver,
       │           auto-configura la conexión a BD)
       │
       └── @ComponentScan
               └── Escanea este package y todos los sub-packages
                   en busca de clases con @Component, @Service,
                   @Repository, @Controller, etc.
```

### 8.2 Anotaciones de Estereotipos — Para Declarar Beans

Estas anotaciones le dicen a Spring: "esta clase es un componente que debes administrar."

```java
// ─────────────────────────────────────────────────────────────
// @Component — Anotación genérica para cualquier bean
// Úsala cuando el componente no encaja en las categorías abajo
// ─────────────────────────────────────────────────────────────
@Component
public class TokenGenerator {
    public String generar() {
        return UUID.randomUUID().toString();
    }
}
```

```java
// ─────────────────────────────────────────────────────────────
// @Service — Para clases con lógica de negocio
// Semánticamente indica: "aquí viven las reglas del dominio"
// ─────────────────────────────────────────────────────────────
@Service
public class SolicitudService {
    
    private final SolicitudRepository solicitudRepository;
    
    // Inyección por constructor (Spring la hace automáticamente)
    public SolicitudService(SolicitudRepository solicitudRepository) {
        this.solicitudRepository = solicitudRepository;
    }
    
    public Solicitud registrar(Solicitud solicitud) {
        // Aquí va la lógica de negocio
        solicitud.setEstado(EstadoSolicitud.REGISTRADA);
        solicitud.setFechaRegistro(LocalDateTime.now());
        return solicitudRepository.save(solicitud);
    }
}
```

```java
// ─────────────────────────────────────────────────────────────
// @Repository — Para clases que acceden a la base de datos
// Además de ser un bean, Spring traduce las excepciones de BD
// a excepciones de Spring (DataAccessException)
// ─────────────────────────────────────────────────────────────
@Repository  // Cuando extiendes JpaRepository, esto es opcional
             // pero es buena práctica dejarlo explícito
public interface SolicitudRepository extends JpaRepository<Solicitud, Long> {
    // Spring Data JPA genera la implementación automáticamente
    List<Solicitud> findByEstado(EstadoSolicitud estado);
}
```

```java
// ─────────────────────────────────────────────────────────────
// @Controller — Para controladores web que devuelven vistas HTML
// (No lo usaremos en nuestra API REST)
// ─────────────────────────────────────────────────────────────
@Controller
public class HomeController {
    @GetMapping("/")
    public String home() {
        return "index"; // Devuelve una vista HTML (Thymeleaf)
    }
}
```

```java
// ─────────────────────────────────────────────────────────────
// @RestController — Para controladores de API REST
// Equivale a @Controller + @ResponseBody en cada método
// @ResponseBody indica que el retorno se serializa como JSON
// ─────────────────────────────────────────────────────────────
@RestController
@RequestMapping("/api/v1/solicitudes")  // URL base para todos los métodos
public class SolicitudController {
    
    private final SolicitudService solicitudService;
    
    public SolicitudController(SolicitudService solicitudService) {
        this.solicitudService = solicitudService;
    }
    
    @GetMapping("/{id}")  // GET /api/v1/solicitudes/{id}
    public ResponseEntity<Solicitud> obtener(@PathVariable Long id) {
        return ResponseEntity.ok(solicitudService.buscarPorId(id));
    }
}
```

```java
// ─────────────────────────────────────────────────────────────
// @Configuration + @Bean — Para configurar beans complejos
// Cuando no puedes poner @Component en una clase (por ejemplo,
// en clases de librerías externas)
// ─────────────────────────────────────────────────────────────
@Configuration
public class SecurityConfig {
    
    // Spring registrará lo que retorne este método como un bean
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) 
            throws Exception {
        // Configuración detallada en Guía 07
        return http.build();
    }
}
```

### 8.3 Anotaciones de Inyección

```java
// ─────────────────────────────────────────────────────────────
// @Autowired — Inyección automática (preferir inyección por constructor)
// ─────────────────────────────────────────────────────────────

// Forma 1: Por constructor (✅ RECOMENDADA - no necesita @Autowired con 1 constructor)
@Service
public class NotificacionService {
    private final EmailSender emailSender;
    
    // Spring detecta que hay 1 constructor y lo usa automáticamente
    // @Autowired es opcional cuando hay UN SOLO constructor
    public NotificacionService(EmailSender emailSender) {
        this.emailSender = emailSender;
    }
}

// Forma 2: Por campo (❌ EVITAR en código de producción)
@Service
public class NotificacionService {
    @Autowired
    private EmailSender emailSender;  // No es final, dificulta pruebas
}
```

```java
// ─────────────────────────────────────────────────────────────
// @Qualifier — Cuando hay varias implementaciones del mismo tipo
// ─────────────────────────────────────────────────────────────
public interface PriorizacionStrategy {
    Prioridad calcular(Solicitud solicitud);
}

@Component("priorizacionUrgente")
public class PriorizacionUrgenteStrategy implements PriorizacionStrategy {
    @Override
    public Prioridad calcular(Solicitud solicitud) {
        return Prioridad.ALTA;
    }
}

@Component("priorizacionNormal")
public class PriorizacionNormalStrategy implements PriorizacionStrategy {
    @Override
    public Prioridad calcular(Solicitud solicitud) {
        return Prioridad.MEDIA;
    }
}

// Cuando hay dos implementaciones, debes especificar cuál quieres:
@Service
public class SolicitudService {
    private final PriorizacionStrategy strategy;
    
    public SolicitudService(@Qualifier("priorizacionUrgente") 
                            PriorizacionStrategy strategy) {
        this.strategy = strategy;
    }
}
```

### 8.4 Anotaciones de Configuración y Valores

```java
// ─────────────────────────────────────────────────────────────
// @Value — Inyectar valores del application.yml directamente
// ─────────────────────────────────────────────────────────────
@Service
public class JwtService {
    
    // Lee el valor de application.yml: jwt.secret
    @Value("${jwt.secret}")
    private String jwtSecret;
    
    // Lee el valor con un valor por defecto si no está definido
    @Value("${jwt.expiracion:86400000}")  // default: 24 horas en ms
    private long jwtExpiracion;
}
```

```java
// ─────────────────────────────────────────────────────────────
// @ConfigurationProperties — Para grupos de propiedades (mejor que @Value)
// ─────────────────────────────────────────────────────────────

// En application.yml agregas:
// triage:
//   jwt:
//     secret: miSecretoSuperSeguro
//     expiracion: 86400000
//   upload:
//     ruta: /uploads
//     max-size: 10MB

@ConfigurationProperties(prefix = "triage.jwt")
@Component
public record JwtProperties(
    String secret,
    long expiracion
) {}

// Inyección normal en cualquier servicio:
@Service
public class JwtService {
    private final JwtProperties jwtProperties;
    
    public JwtService(JwtProperties jwtProperties) {
        this.jwtProperties = jwtProperties;
    }
    
    public String generarToken(String username) {
        // Usa jwtProperties.secret() y jwtProperties.expiracion()
        return "token...";
    }
}
```

### 8.5 Tabla de Referencia Rápida

| Anotación | Dónde va | Para qué sirve |
|---|---|---|
| `@SpringBootApplication` | Clase principal | Habilita todo Spring Boot |
| `@Component` | Clase | Bean genérico administrado por Spring |
| `@Service` | Clase | Bean con lógica de negocio |
| `@Repository` | Clase/Interface | Bean de acceso a datos |
| `@Controller` | Clase | Controlador web (vistas HTML) |
| `@RestController` | Clase | Controlador API REST (devuelve JSON) |
| `@Configuration` | Clase | Clase de configuración con @Bean |
| `@Bean` | Método en @Configuration | Registra el retorno como bean |
| `@Autowired` | Campo, constructor, setter | Inyecta una dependencia |
| `@Qualifier("nombre")` | Parámetro de constructor | Especifica qué implementación inyectar |
| `@Value("${prop}")` | Campo | Inyecta valor del application.yml |
| `@ConfigurationProperties` | Clase Record | Mapea grupo de propiedades |
| `@RequestMapping` | Clase/Método | URL base para el controlador/método |
| `@GetMapping` | Método | Maneja peticiones GET |
| `@PostMapping` | Método | Maneja peticiones POST |
| `@PutMapping` | Método | Maneja peticiones PUT |
| `@PatchMapping` | Método | Maneja peticiones PATCH |
| `@DeleteMapping` | Método | Maneja peticiones DELETE |
| `@PathVariable` | Parámetro | Extrae variable de la URL `/api/{id}` |
| `@RequestParam` | Parámetro | Extrae parámetro de query `?estado=ACTIVO` |
| `@RequestBody` | Parámetro | Deserializa el body JSON al objeto |
| `@ResponseBody` | Clase/Método | Serializa el retorno como JSON |
| `@ResponseStatus` | Método | Define el HTTP status code del retorno |
| `@Profile("dev")` | Clase | Bean activo solo en el perfil especificado |
| `@Scope("prototype")` | Clase | Crea nueva instancia cada vez (no singleton) |

---

## 9. Ciclo de Vida de un Bean

### 9.1 El Alcance de un Bean (Scope)

Por defecto, todos los beans de Spring son **Singleton**: Spring crea **una única instancia** y la reutiliza para todos los que la necesiten. Esto es diferente al Singleton pattern de GoF (que has visto en la Guía 02) — aquí Spring gestiona la singularidad.

```
Scope Singleton (por defecto):
┌───────────────────────────────────────────────────────┐
│                Application Context                     │
│                                                        │
│  ┌─────────────────┐     ┌─────────────────────────┐  │
│  │  SolicitudService│◄────│ SolicitudController (A) │  │
│  │  (una instancia) │     └─────────────────────────┘  │
│  │                  │◄────┌─────────────────────────┐  │
│  └─────────────────┘     │ SolicitudController (B) │  │
│                           └─────────────────────────┘  │
└───────────────────────────────────────────────────────┘

Ambos controladores comparten LA MISMA instancia de SolicitudService.
Por eso, los campos de un @Service NO deben tener estado mutable.
```

Otros scopes menos comunes:

```java
// Prototype: nueva instancia cada vez que se inyecta o se pide
@Scope("prototype")
@Component
public class SolicitudBuilder {
    // Nuevo objeto por cada uso
}

// Request: una instancia por petición HTTP (en apps web)
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
@Component
public class SolicitudContextHolder {
    private Solicitud solicitudActual;
}
```

### 9.2 Hooks de Ciclo de Vida

Puedes ejecutar código justo después de que Spring crea el bean, o justo antes de destruirlo:

```java
@Service
public class CacheService {
    
    private Map<Long, Solicitud> cache;
    
    // Se ejecuta DESPUÉS de que Spring inyecta todas las dependencias
    @PostConstruct
    public void inicializar() {
        this.cache = new ConcurrentHashMap<>();
        System.out.println("CacheService inicializado - cache lista");
    }
    
    // Se ejecuta ANTES de que Spring destruya el bean
    // (cuando se cierra la aplicación)
    @PreDestroy
    public void limpiar() {
        cache.clear();
        System.out.println("CacheService destruido - cache limpiada");
    }
    
    public void guardar(Long id, Solicitud solicitud) {
        cache.put(id, solicitud);
    }
}
```

### 9.3 El Flujo Completo de Arranque

```
Ejecutar main()
      │
      ▼
SpringApplication.run()
      │
      ├─ 1. Crear ApplicationContext
      │
      ├─ 2. Cargar application.yml (y perfiles activos)
      │
      ├─ 3. @ComponentScan: escanear packages, encontrar beans
      │
      ├─ 4. Resolver dependencias (¿qué necesita qué?)
      │
      ├─ 5. Crear beans en orden (primero los que no dependen de nada)
      │
      ├─ 6. Inyectar dependencias en cada bean
      │
      ├─ 7. Ejecutar @PostConstruct en cada bean
      │
      ├─ 8. Auto-configurar (Tomcat, JPA, Security...)
      │
      └─ 9. ¡Aplicación lista! Tomcat escucha en :8080

Al apagar (Ctrl+C o SIGTERM):
      │
      ├─ Ejecutar @PreDestroy en cada bean
      │
      └─ Cerrar conexiones, liberar recursos
```

---

## 10. Perfiles de Configuración: dev, prod, test

### 10.1 ¿Por Qué Necesitamos Perfiles?

Tu aplicación necesita comportarse diferente según el entorno:

| Entorno | Base de Datos | Logging | ddl-auto | Seguridad |
|---|---|---|---|---|
| **dev** | Local PostgreSQL | DEBUG | update | Relajada (Swagger sin auth) |
| **test** | H2 en memoria | WARN | create-drop | Sin seguridad |
| **prod** | PostgreSQL en la nube | INFO/ERROR | validate | Estricta, HTTPS |

### 10.2 Estructura de Archivos de Perfil

```
src/main/resources/
├── application.yml          ← Configuración base (compartida por todos los perfiles)
├── application-dev.yml      ← Sobreescribe/agrega para perfil 'dev'
├── application-prod.yml     ← Sobreescribe/agrega para perfil 'prod'
└── application-test.yml     ← Sobreescribe/agrega para perfil 'test'
```

### 10.3 Implementación Completa

```yaml
# application.yml — Solo propiedades COMUNES a todos los entornos
spring:
  application:
    name: triage-api
  jpa:
    open-in-view: false
    properties:
      hibernate:
        jdbc:
          time_zone: UTC

springdoc:
  api-docs:
    path: /api-docs
  swagger-ui:
    path: /swagger-ui

management:
  endpoints:
    web:
      exposure:
        include: health, info
```

```yaml
# application-dev.yml — Desarrollo local
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/triage_db
    username: postgres
    password: postgres
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true

server:
  port: 8080

logging:
  level:
    co.edu.uniquindio.triage: DEBUG
    org.hibernate.SQL: DEBUG

# En dev, exponemos más endpoints de actuator
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

```yaml
# application-prod.yml — Producción
spring:
  datasource:
    # En producción, las credenciales vienen de variables de entorno
    url: ${DATABASE_URL}
    username: ${DATABASE_USERNAME}
    password: ${DATABASE_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: validate  # NUNCA update en producción
    show-sql: false

server:
  port: ${PORT:8080}

logging:
  level:
    root: WARN
    co.edu.uniquindio.triage: INFO
```

### 10.4 Activar un Perfil

**Forma 1 — En `application.yml`** (para desarrollo local):
```yaml
spring:
  profiles:
    active: dev
```

**Forma 2 — Variable de entorno** (para producción, Docker):
```bash
export SPRING_PROFILES_ACTIVE=prod
java -jar triage.jar
```

**Forma 3 — Argumento de línea de comandos**:
```bash
java -jar triage.jar --spring.profiles.active=prod
```

### 10.5 Beans Condicionales por Perfil

```java
// Este bean SOLO se crea cuando el perfil 'dev' está activo
@Profile("dev")
@Component
public class DatosEjemploInitializer implements CommandLineRunner {
    
    private final SolicitudRepository repository;
    
    public DatosEjemploInitializer(SolicitudRepository repository) {
        this.repository = repository;
    }
    
    @Override
    public void run(String... args) {
        // Carga datos de ejemplo al arrancar en dev
        System.out.println("Cargando datos de ejemplo para desarrollo...");
    }
}

// Este bean se crea en TODOS los perfiles EXCEPTO 'dev'
@Profile("!dev")
@Component
public class SeguridadEstrictaConfig {
    // Configuración de seguridad para prod/test
}
```

---

## 11. Tu Primer Endpoint Funcional Paso a Paso

Vamos a construir paso a paso el primer endpoint real del proyecto de Triage: obtener el estado de salud del sistema y registrar una solicitud académica.

### 11.1 Estructura de Paquetes del Proyecto de Triage

Antes de escribir código, hay que definir la arquitectura de paquetes. Esta estructura seguirá la arquitectura en capas que estudiaste en la Guía 01:

```
src/main/java/co/edu/uniquindio/triage/
├── TriageApplication.java                    ← Punto de entrada
│
├── config/                                   ← Configuración de Spring
│   ├── SecurityConfig.java
│   ├── SwaggerConfig.java
│   └── WebConfig.java
│
├── domain/                                   ← Capa de dominio
│   ├── model/                                ← Entidades JPA
│   │   ├── Solicitud.java
│   │   ├── TipoSolicitud.java
│   │   ├── Usuario.java
│   │   └── HistorialSolicitud.java
│   └── enums/                                ← Enumeraciones del dominio
│       ├── EstadoSolicitud.java
│       ├── Prioridad.java
│       ├── CanalOrigen.java
│       └── RolUsuario.java
│
├── repository/                               ← Capa de persistencia
│   ├── SolicitudRepository.java
│   ├── UsuarioRepository.java
│   └── HistorialRepository.java
│
├── service/                                  ← Capa de negocio
│   ├── SolicitudService.java
│   ├── UsuarioService.java
│   └── impl/                                 ← Implementaciones (si necesitas interfaces)
│
├── controller/                               ← Capa de presentación (API)
│   ├── SolicitudController.java
│   ├── AuthController.java
│   └── UsuarioController.java
│
├── dto/                                      ← Data Transfer Objects
│   ├── request/                              ← DTOs de entrada (lo que recibe la API)
│   │   ├── CrearSolicitudRequest.java
│   │   └── LoginRequest.java
│   └── response/                             ← DTOs de salida (lo que devuelve la API)
│       ├── SolicitudResponse.java
│       └── ErrorResponse.java
│
└── exception/                                ← Excepciones personalizadas
    ├── SolicitudNoEncontradaException.java
    ├── EstadoInvalidoException.java
    └── GlobalExceptionHandler.java
```

### 11.2 Crear las Enumeraciones del Dominio

Primero definimos los valores posibles para los campos del dominio:

```java
// src/main/java/co/edu/uniquindio/triage/domain/enums/EstadoSolicitud.java
package co.edu.uniquindio.triage.domain.enums;

/**
 * Define los estados posibles de una solicitud académica.
 * Las transiciones válidas son:
 * REGISTRADA → CLASIFICADA → EN_ATENCION → ATENDIDA → CERRADA
 */
public enum EstadoSolicitud {
    REGISTRADA,
    CLASIFICADA,
    EN_ATENCION,
    ATENDIDA,
    CERRADA
}
```

```java
// src/main/java/co/edu/uniquindio/triage/domain/enums/Prioridad.java
package co.edu.uniquindio.triage.domain.enums;

public enum Prioridad {
    BAJA,
    MEDIA,
    ALTA,
    URGENTE
}
```

```java
// src/main/java/co/edu/uniquindio/triage/domain/enums/CanalOrigen.java
package co.edu.uniquindio.triage.domain.enums;

/**
 * Canal por el que llegó la solicitud al sistema (RF-01).
 */
public enum CanalOrigen {
    CSU,          // Centro de Servicios Universitarios
    CORREO,       // Correo electrónico
    SAC,          // Sistema de Atención al Cliente
    TELEFONICO,   // Atención telefónica
    PRESENCIAL,   // Atención presencial
    SISTEMA       // Registro directo en el sistema
}
```

### 11.3 Crear la Entidad Solicitud (Simplificada para esta Guía)

> **Nota:** En esta guía vamos a crear una versión simplificada de la entidad para poder construir el primer endpoint. La versión completa con todas las relaciones JPA, validaciones y auditoría la verás en la **Guía 05**.

```java
// src/main/java/co/edu/uniquindio/triage/domain/model/Solicitud.java
package co.edu.uniquindio.triage.domain.model;

import co.edu.uniquindio.triage.domain.enums.CanalOrigen;
import co.edu.uniquindio.triage.domain.enums.EstadoSolicitud;
import co.edu.uniquindio.triage.domain.enums.Prioridad;
import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;

/**
 * Entidad que representa una solicitud académica en el sistema de triage.
 * Almacena toda la información de un trámite desde su registro hasta su cierre.
 * 
 * Corresponde al RF-01 (Registro de solicitudes académicas).
 */
@Entity
@Table(name = "solicitudes")
@Data               // Lombok: genera getters, setters, toString, equals, hashCode
@Builder            // Lombok: genera el patrón Builder para construcción fluida
@NoArgsConstructor  // Lombok: genera constructor vacío (requerido por JPA)
@AllArgsConstructor // Lombok: genera constructor con todos los campos
public class Solicitud {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String tipoSolicitud;  // Versión completa en Guía 05: @ManyToOne TipoSolicitud

    @Column(nullable = false, columnDefinition = "TEXT")
    private String descripcion;

    @Enumerated(EnumType.STRING)  // Guarda el nombre del enum, no el número ordinal
    @Column(nullable = false, length = 20)
    private CanalOrigen canalOrigen;

    @Column(nullable = false)
    private LocalDateTime fechaRegistro;

    @Column(nullable = false, length = 20)
    private String identificacionSolicitante;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private EstadoSolicitud estado;

    @Enumerated(EnumType.STRING)
    @Column(length = 10)
    private Prioridad prioridad;

    @Column(columnDefinition = "TEXT")
    private String observacionCierre;
}
```

### 11.4 Crear los DTOs con Records de Java 21

Recuerda de la Guía 03: nunca expongas las entidades JPA directamente. Usa DTOs.

```java
// src/main/java/co/edu/uniquindio/triage/dto/request/CrearSolicitudRequest.java
package co.edu.uniquindio.triage.dto.request;

import co.edu.uniquindio.triage.domain.enums.CanalOrigen;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;

/**
 * DTO para recibir los datos de una nueva solicitud académica.
 * 
 * Usamos un Record de Java 21: inmutable, sin getters/setters explícitos,
 * validaciones con Bean Validation.
 * 
 * Corresponde a RF-01: Registro de solicitudes académicas.
 */
public record CrearSolicitudRequest(

    @NotBlank(message = "El tipo de solicitud es obligatorio")
    @Size(max = 100, message = "El tipo no puede superar 100 caracteres")
    String tipoSolicitud,

    @NotBlank(message = "La descripción es obligatoria")
    @Size(min = 10, max = 2000, message = "La descripción debe tener entre 10 y 2000 caracteres")
    String descripcion,

    @NotNull(message = "El canal de origen es obligatorio")
    CanalOrigen canalOrigen,

    @NotBlank(message = "La identificación del solicitante es obligatoria")
    @Size(max = 20, message = "La identificación no puede superar 20 caracteres")
    String identificacionSolicitante
) {}
```

```java
// src/main/java/co/edu.uniquindio/triage/dto/response/SolicitudResponse.java
package co.edu.uniquindio.triage.dto.response;

import co.edu.uniquindio.triage.domain.enums.CanalOrigen;
import co.edu.uniquindio.triage.domain.enums.EstadoSolicitud;
import co.edu.uniquindio.triage.domain.enums.Prioridad;

import java.time.LocalDateTime;

/**
 * DTO de respuesta con la información de una solicitud.
 * Solo expone los campos que el cliente necesita ver.
 */
public record SolicitudResponse(
    Long id,
    String tipoSolicitud,
    String descripcion,
    CanalOrigen canalOrigen,
    LocalDateTime fechaRegistro,
    String identificacionSolicitante,
    EstadoSolicitud estado,
    Prioridad prioridad
) {}
```

### 11.5 Crear el Repository

```java
// src/main/java/co/edu/uniquindio/triage/repository/SolicitudRepository.java
package co.edu.uniquindio.triage.repository;

import co.edu.uniquindio.triage.domain.enums.EstadoSolicitud;
import co.edu.uniquindio.triage.domain.model.Solicitud;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

/**
 * Repository para acceder a los datos de solicitudes en la BD.
 * Spring Data JPA genera la implementación automáticamente.
 * 
 * JpaRepository<Solicitud, Long>:
 *   - Solicitud: la entidad que maneja
 *   - Long: el tipo del ID
 */
@Repository
public interface SolicitudRepository extends JpaRepository<Solicitud, Long> {
    
    // Spring Data JPA genera el SQL automáticamente por el nombre del método:
    // SELECT * FROM solicitudes WHERE estado = ?
    List<Solicitud> findByEstado(EstadoSolicitud estado);
    
    // SELECT * FROM solicitudes WHERE identificacion_solicitante = ?
    List<Solicitud> findByIdentificacionSolicitante(String identificacion);
}
```

### 11.6 Crear las Excepciones Personalizadas

```java
// src/main/java/co/edu/uniquindio/triage/exception/SolicitudNoEncontradaException.java
package co.edu.uniquindio.triage.exception;

/**
 * Se lanza cuando se busca una solicitud por ID y no existe.
 * Al extender RuntimeException, no es necesario declararla con 'throws'.
 */
public class SolicitudNoEncontradaException extends RuntimeException {
    
    private final Long solicitudId;
    
    public SolicitudNoEncontradaException(Long solicitudId) {
        super("No existe una solicitud con ID: " + solicitudId);
        this.solicitudId = solicitudId;
    }
    
    public Long getSolicitudId() {
        return solicitudId;
    }
}
```

```java
// src/main/java/co/edu/uniquindio/triage/exception/EstadoInvalidoException.java
package co.edu.uniquindio.triage.exception;

import co.edu.uniquindio.triage.domain.enums.EstadoSolicitud;

/**
 * Se lanza cuando se intenta hacer una transición de estado inválida.
 * Ejemplo: intentar cerrar una solicitud que aún está REGISTRADA.
 */
public class EstadoInvalidoException extends RuntimeException {
    
    public EstadoInvalidoException(EstadoSolicitud estadoActual, EstadoSolicitud estadoDeseado) {
        super(String.format(
            "No se puede cambiar el estado de '%s' a '%s'",
            estadoActual.name(),
            estadoDeseado.name()
        ));
    }
}
```

### 11.7 Crear el Service

```java
// src/main/java/co/edu/uniquindio/triage/service/SolicitudService.java
package co.edu.uniquindio.triage.service;

import co.edu.uniquindio.triage.domain.enums.EstadoSolicitud;
import co.edu.uniquindio.triage.domain.model.Solicitud;
import co.edu.uniquindio.triage.dto.request.CrearSolicitudRequest;
import co.edu.uniquindio.triage.dto.response.SolicitudResponse;
import co.edu.uniquindio.triage.exception.SolicitudNoEncontradaException;
import co.edu.uniquindio.triage.repository.SolicitudRepository;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDateTime;
import java.util.List;

/**
 * Servicio con la lógica de negocio para gestión de solicitudes académicas.
 * 
 * @Transactional: asegura que las operaciones de BD sean atómicas.
 * @Slf4j (Lombok): genera automáticamente el logger 'log'.
 */
@Service
@Slf4j
public class SolicitudService {
    
    // Inyección por constructor: la mejor práctica
    private final SolicitudRepository solicitudRepository;
    
    public SolicitudService(SolicitudRepository solicitudRepository) {
        this.solicitudRepository = solicitudRepository;
    }
    
    /**
     * Registra una nueva solicitud académica en el sistema.
     * Implementa RF-01: Registro de solicitudes académicas.
     * 
     * @param request Datos de la nueva solicitud
     * @return La solicitud registrada como DTO de respuesta
     */
    @Transactional
    public SolicitudResponse registrar(CrearSolicitudRequest request) {
        log.info("Registrando nueva solicitud de tipo '{}' para el solicitante '{}'",
                 request.tipoSolicitud(), request.identificacionSolicitante());
        
        // Convertir el DTO de request a entidad usando Builder (Guía 02)
        var solicitud = Solicitud.builder()
                .tipoSolicitud(request.tipoSolicitud())
                .descripcion(request.descripcion())
                .canalOrigen(request.canalOrigen())
                .identificacionSolicitante(request.identificacionSolicitante())
                .fechaRegistro(LocalDateTime.now())  // La fecha la asignamos aquí, no el cliente
                .estado(EstadoSolicitud.REGISTRADA)  // Estado inicial siempre REGISTRADA
                .build();
        
        var solicitudGuardada = solicitudRepository.save(solicitud);
        
        log.info("Solicitud registrada exitosamente con ID: {}", solicitudGuardada.getId());
        
        return toResponse(solicitudGuardada);
    }
    
    /**
     * Busca una solicitud por su ID.
     * 
     * @param id ID de la solicitud
     * @return La solicitud encontrada como DTO de respuesta
     * @throws SolicitudNoEncontradaException si no existe la solicitud
     */
    @Transactional(readOnly = true)  // readOnly mejora el rendimiento en consultas
    public SolicitudResponse buscarPorId(Long id) {
        log.debug("Buscando solicitud con ID: {}", id);
        
        // findById devuelve Optional<Solicitud>
        // Si no existe, lanzamos nuestra excepción personalizada
        return solicitudRepository.findById(id)
                .map(this::toResponse)
                .orElseThrow(() -> new SolicitudNoEncontradaException(id));
    }
    
    /**
     * Lista todas las solicitudes, opcionalmente filtradas por estado.
     * Implementa RF-07: Consulta de solicitudes.
     */
    @Transactional(readOnly = true)
    public List<SolicitudResponse> listar(EstadoSolicitud estado) {
        log.debug("Listando solicitudes. Filtro por estado: {}", estado);
        
        var solicitudes = (estado != null)
                ? solicitudRepository.findByEstado(estado)
                : solicitudRepository.findAll();
        
        return solicitudes.stream()
                .map(this::toResponse)
                .toList();
    }
    
    /**
     * Convierte una entidad Solicitud a su DTO de respuesta.
     * Este método es privado porque es un detalle de implementación del servicio.
     */
    private SolicitudResponse toResponse(Solicitud solicitud) {
        return new SolicitudResponse(
                solicitud.getId(),
                solicitud.getTipoSolicitud(),
                solicitud.getDescripcion(),
                solicitud.getCanalOrigen(),
                solicitud.getFechaRegistro(),
                solicitud.getIdentificacionSolicitante(),
                solicitud.getEstado(),
                solicitud.getPrioridad()
        );
    }
}
```

### 11.8 Crear el Controller

```java
// src/main/java/co/edu/uniquindio/triage/controller/SolicitudController.java
package co.edu.uniquindio.triage.controller;

import co.edu.uniquindio.triage.domain.enums.EstadoSolicitud;
import co.edu.uniquindio.triage.dto.request.CrearSolicitudRequest;
import co.edu.uniquindio.triage.dto.response.SolicitudResponse;
import co.edu.uniquindio.triage.service.SolicitudService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * Controlador REST para la gestión de solicitudes académicas.
 * 
 * @RestController: indica que esta clase es un controlador y que todos
 *                  sus métodos devuelven JSON (no vistas HTML)
 * @RequestMapping: prefijo de URL para todos los métodos de este controlador
 * @Tag: metadato para Swagger UI
 */
@RestController
@RequestMapping("/api/v1/solicitudes")
@Tag(name = "Solicitudes", description = "Gestión del ciclo de vida de solicitudes académicas")
@Slf4j
public class SolicitudController {
    
    // El Service se inyecta por constructor
    private final SolicitudService solicitudService;
    
    public SolicitudController(SolicitudService solicitudService) {
        this.solicitudService = solicitudService;
    }
    
    /**
     * Endpoint: POST /api/v1/solicitudes
     * Registra una nueva solicitud académica (RF-01).
     */
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)  // Devuelve 201 Created
    @Operation(summary = "Registrar solicitud", 
               description = "Crea una nueva solicitud académica en estado REGISTRADA")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "201", description = "Solicitud creada exitosamente"),
        @ApiResponse(responseCode = "400", description = "Datos de entrada inválidos"),
        @ApiResponse(responseCode = "401", description = "No autenticado")
    })
    public ResponseEntity<SolicitudResponse> registrar(
            @Valid @RequestBody CrearSolicitudRequest request) {
        
        log.info("Petición recibida: registrar solicitud - {}", request.tipoSolicitud());
        
        var response = solicitudService.registrar(request);
        
        // ResponseEntity nos permite controlar el código de respuesta y los headers
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
    
    /**
     * Endpoint: GET /api/v1/solicitudes/{id}
     * Obtiene una solicitud por su ID.
     */
    @GetMapping("/{id}")
    @Operation(summary = "Obtener solicitud por ID")
    public ResponseEntity<SolicitudResponse> obtenerPorId(
            @Parameter(description = "ID de la solicitud", example = "1")
            @PathVariable Long id) {
        
        return ResponseEntity.ok(solicitudService.buscarPorId(id));
    }
    
    /**
     * Endpoint: GET /api/v1/solicitudes
     * Lista solicitudes, opcionalmente filtradas por estado (RF-07).
     * 
     * Ejemplos:
     *   GET /api/v1/solicitudes               → todas las solicitudes
     *   GET /api/v1/solicitudes?estado=REGISTRADA → solo las registradas
     */
    @GetMapping
    @Operation(summary = "Listar solicitudes", 
               description = "Lista todas las solicitudes, con filtro opcional por estado")
    public ResponseEntity<List<SolicitudResponse>> listar(
            @Parameter(description = "Filtrar por estado (opcional)")
            @RequestParam(required = false) EstadoSolicitud estado) {
        
        return ResponseEntity.ok(solicitudService.listar(estado));
    }
}
```

---

## 12. Logging con SLF4J y Logback

### 12.1 ¿Por Qué Logging y No System.out.println?

El `System.out.println` que usas para depurar en ejercicios básicos tiene limitaciones severas en aplicaciones reales:

| Característica | `System.out.println` | SLF4J + Logback |
|---|---|---|
| Niveles de severidad | No | Sí (TRACE, DEBUG, INFO, WARN, ERROR) |
| Filtrar por nivel | No | Sí (en producción, solo INFO+) |
| Timestamp automático | No | Sí |
| Nombre de la clase | No | Sí |
| Escribir en archivos | No | Sí |
| Rendimiento | Malo (sincrónico) | Excelente (asincrónico disponible) |
| Apagar en producción | Requiere eliminar código | Solo cambiar configuración |

### 12.2 Cómo Usar el Logger

Con Lombok, la forma más limpia es:

```java
@Service
@Slf4j  // Lombok genera: private static final Logger log = LoggerFactory.getLogger(SolicitudService.class);
public class SolicitudService {
    
    public void registrar(Solicitud solicitud) {
        
        // ❌ NUNCA uses concatenación de strings en logs:
        // log.debug("Registrando solicitud " + solicitud.getId() + " de " + solicitud.getTipo());
        // Esto evalúa la concatenación AUNQUE el nivel DEBUG esté desactivado
        
        // ✅ USA siempre placeholders {}:
        // Solo se evalúa si el nivel está activo
        log.debug("Registrando solicitud ID={} tipo={}", solicitud.getId(), solicitud.getTipoSolicitud());
        
        // Niveles de log y cuándo usarlos:
        log.trace("Nivel más detallado - para depuración muy profunda");
        log.debug("Información útil para depuración en desarrollo");
        log.info("Eventos importantes del flujo normal: 'Solicitud 42 registrada exitosamente'");
        log.warn("Algo inesperado pero no crítico: 'Timeout en servicio externo, reintentando...'");
        log.error("Error que requiere atención: 'Fallo al conectar con BD'");
        
        // Para loggear excepciones:
        try {
            solicitudRepository.save(solicitud);
        } catch (Exception e) {
            // El segundo parámetro es la excepción; imprime el stack trace completo
            log.error("Error al guardar la solicitud ID={}", solicitud.getId(), e);
            throw e;
        }
    }
}
```

### 12.3 Configuración del Logging en application.yml

Ya la incluimos en la sección 7.2, pero aquí la explicamos:

```yaml
logging:
  level:
    root: INFO                              # Nivel base para todo
    co.edu.uniquindio.triage: DEBUG         # Más detalle para nuestro código
    org.hibernate.SQL: DEBUG                # Muestra las queries SQL
    org.hibernate.type.descriptor.sql: TRACE # Muestra los parámetros de las queries
  pattern:
    # Formato: fecha [hilo] NIVEL paquete.Clase - mensaje
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
```

---

## 13. Manejo Global de Errores

### 13.1 El Problema Sin Manejo Global

Sin configuración especial, cuando ocurre una excepción en un endpoint, Spring Boot devuelve un JSON genérico poco útil:

```json
{
  "timestamp": "2026-01-15T14:30:00.000+00:00",
  "status": 500,
  "error": "Internal Server Error",
  "path": "/api/v1/solicitudes/999"
}
```

El cliente no sabe qué salió mal. Con manejo global podemos devolver respuestas estructuradas y útiles.

### 13.2 Crear la Estructura de Respuesta de Error

```java
// src/main/java/co/edu/uniquindio/triage/dto/response/ErrorResponse.java
package co.edu.uniquindio.triage.dto.response;

import java.time.LocalDateTime;
import java.util.List;

/**
 * Estructura estándar para respuestas de error.
 * Sigue el estándar RFC 7807 (Problem Details for HTTP APIs).
 */
public record ErrorResponse(
    int status,          // Código HTTP (400, 404, 500...)
    String error,        // Nombre del error ("Not Found", "Bad Request"...)
    String message,      // Mensaje descriptivo para el desarrollador
    String path,         // Endpoint que generó el error
    LocalDateTime timestamp,
    List<String> detalles  // Errores de validación detallados (si aplica)
) {
    // Constructor conveniente para errores simples (sin lista de detalles)
    public ErrorResponse(int status, String error, String message, String path) {
        this(status, error, message, path, LocalDateTime.now(), null);
    }
}
```

### 13.3 Crear el Manejador Global con @ControllerAdvice

```java
// src/main/java/co/edu/uniquindio/triage/exception/GlobalExceptionHandler.java
package co.edu.uniquindio.triage.exception;

import co.edu.uniquindio.triage.dto.response.ErrorResponse;
import jakarta.servlet.http.HttpServletRequest;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.time.LocalDateTime;
import java.util.List;

/**
 * Manejador global de excepciones para toda la API.
 * 
 * @RestControllerAdvice: intercepta excepciones de TODOS los controladores
 *                        y permite devolver respuestas JSON personalizadas.
 * 
 * Funciona como un "try-catch global" que captura excepciones que no
 * fueron manejadas en los controladores y las convierte en respuestas HTTP.
 */
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    /**
     * Maneja el caso en que una solicitud no existe (404 Not Found).
     */
    @ExceptionHandler(SolicitudNoEncontradaException.class)
    public ResponseEntity<ErrorResponse> handleSolicitudNoEncontrada(
            SolicitudNoEncontradaException ex, 
            HttpServletRequest request) {
        
        log.warn("Solicitud no encontrada: {}", ex.getMessage());
        
        var error = new ErrorResponse(
                HttpStatus.NOT_FOUND.value(),
                "Not Found",
                ex.getMessage(),
                request.getRequestURI()
        );
        
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    /**
     * Maneja errores de validación del @Valid en los Request DTOs (400 Bad Request).
     * Se activa cuando un campo obligatorio está vacío, es muy largo, etc.
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidacion(
            MethodArgumentNotValidException ex, 
            HttpServletRequest request) {
        
        // Extraer todos los mensajes de error de validación
        List<String> detalles = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(fieldError -> 
                    String.format("Campo '%s': %s", 
                        fieldError.getField(), 
                        fieldError.getDefaultMessage()))
                .toList();
        
        log.warn("Error de validación en {}: {}", request.getRequestURI(), detalles);
        
        var error = new ErrorResponse(
                HttpStatus.BAD_REQUEST.value(),
                "Bad Request",
                "Los datos enviados no son válidos",
                request.getRequestURI(),
                LocalDateTime.now(),
                detalles
        );
        
        return ResponseEntity.badRequest().body(error);
    }
    
    /**
     * Maneja transiciones de estado inválidas (400 Bad Request).
     */
    @ExceptionHandler(EstadoInvalidoException.class)
    public ResponseEntity<ErrorResponse> handleEstadoInvalido(
            EstadoInvalidoException ex, 
            HttpServletRequest request) {
        
        log.warn("Transición de estado inválida: {}", ex.getMessage());
        
        var error = new ErrorResponse(
                HttpStatus.BAD_REQUEST.value(),
                "Bad Request",
                ex.getMessage(),
                request.getRequestURI()
        );
        
        return ResponseEntity.badRequest().body(error);
    }
    
    /**
     * Manejador de último recurso: captura cualquier excepción no manejada (500).
     * En producción, NO deberías exponer el mensaje interno de la excepción.
     */
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(
            Exception ex, 
            HttpServletRequest request) {
        
        // Loggeamos el error completo (con stack trace) para análisis interno
        log.error("Error no manejado en {}: {}", request.getRequestURI(), ex.getMessage(), ex);
        
        var error = new ErrorResponse(
                HttpStatus.INTERNAL_SERVER_ERROR.value(),
                "Internal Server Error",
                "Ocurrió un error inesperado. Por favor contacta al administrador.",
                request.getRequestURI()
        );
        
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

### 13.4 Ejemplo de Respuestas de Error

Con este manejador, las respuestas de error ahora son así:

**404 — Solicitud no encontrada:**
```json
{
  "status": 404,
  "error": "Not Found",
  "message": "No existe una solicitud con ID: 999",
  "path": "/api/v1/solicitudes/999",
  "timestamp": "2026-01-15T14:30:00",
  "detalles": null
}
```

**400 — Error de validación:**
```json
{
  "status": 400,
  "error": "Bad Request",
  "message": "Los datos enviados no son válidos",
  "path": "/api/v1/solicitudes",
  "timestamp": "2026-01-15T14:30:00",
  "detalles": [
    "Campo 'descripcion': La descripción debe tener entre 10 y 2000 caracteres",
    "Campo 'canalOrigen': El canal de origen es obligatorio"
  ]
}
```

---

## 14. Correr el Proyecto y Probar con Swagger UI

### 14.1 Crear la Base de Datos en PostgreSQL

Antes de arrancar la app, necesitas crear la base de datos:

```sql
-- Conecta a PostgreSQL como usuario postgres y ejecuta:
CREATE DATABASE triage_db;
```

Con Docker es más fácil:
```bash
docker run --name triage-postgres \
  -e POSTGRES_DB=triage_db \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -p 5432:5432 \
  -d postgres:16
```

### 14.2 Configurar el Perfil Activo

En `application.yml`, asegúrate de tener:
```yaml
spring:
  profiles:
    active: dev
```

### 14.3 Correr la Aplicación

**Desde IntelliJ:**
- Haz clic derecho en `TriageApplication.java` → `Run 'TriageApplication'`

**Desde la terminal:**
```bash
./mvnw spring-boot:run
```

**Salida esperada en consola:**
```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

 :: Spring Boot ::                (v3.4.1)

2026-01-15 09:00:00 [main] INFO  c.e.u.triage.TriageApplication - Starting TriageApplication
2026-01-15 09:00:01 [main] INFO  o.s.b.w.embedded.tomcat.TomcatWebServer - Tomcat started on port 8080
2026-01-15 09:00:01 [main] INFO  c.e.u.triage.TriageApplication - Started TriageApplication in 3.5 seconds
```

### 14.4 Probar con Swagger UI

Abre tu navegador y ve a: **http://localhost:8080/swagger-ui**

Verás la interfaz interactiva de Swagger con todos tus endpoints documentados. Puedes probar directamente desde el navegador:

1. Expande el endpoint `POST /api/v1/solicitudes`.
2. Haz clic en "Try it out".
3. Reemplaza el body de ejemplo con:

```json
{
  "tipoSolicitud": "Homologación",
  "descripcion": "Solicito homologar la asignatura Algoritmos cursada en la Universidad EAFIT con código EST-201",
  "canalOrigen": "SISTEMA",
  "identificacionSolicitante": "1094567890"
}
```

4. Haz clic en "Execute".
5. Deberías recibir un **201 Created** con la solicitud creada.

### 14.5 Probar con Postman

**Registrar solicitud:**
```
POST http://localhost:8080/api/v1/solicitudes
Content-Type: application/json

{
  "tipoSolicitud": "Cupo en asignatura",
  "descripcion": "Necesito un cupo adicional en Programación Avanzada grupo 01",
  "canalOrigen": "CORREO",
  "identificacionSolicitante": "1094123456"
}
```

**Obtener solicitud:**
```
GET http://localhost:8080/api/v1/solicitudes/1
```

**Listar por estado:**
```
GET http://localhost:8080/api/v1/solicitudes?estado=REGISTRADA
```

### 14.6 Verificar el Actuator

```
GET http://localhost:8080/actuator/health
```

Respuesta esperada:
```json
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP",
      "details": {
        "database": "PostgreSQL",
        "validationQuery": "isValid()"
      }
    }
  }
}
```

---

## 15. Errores Comunes y Troubleshooting

### Error 1: `Failed to configure a DataSource`

```
APPLICATION FAILED TO START
Failed to configure a DataSource: 'url' attribute is not specified 
and no embedded datasource could be configured.
```

**Causa:** Spring Boot no encuentra la configuración de la base de datos.

**Solución:**
1. Verifica que `application-dev.yml` exista y tenga la configuración de `datasource`.
2. Verifica que el perfil `dev` esté activo en `application.yml`:
   ```yaml
   spring:
     profiles:
       active: dev
   ```
3. Verifica que la base de datos `triage_db` exista en PostgreSQL.

---

### Error 2: `Connection refused` a PostgreSQL

```
org.postgresql.util.PSQLException: Connection refused. 
Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.
```

**Causa:** PostgreSQL no está corriendo o la configuración de conexión es incorrecta.

**Solución:**
1. Verifica que PostgreSQL esté corriendo: `pg_isready -h localhost -p 5432`
2. Verifica usuario y contraseña en `application-dev.yml`.
3. Si usas Docker, verifica que el contenedor esté corriendo: `docker ps`.

---

### Error 3: `Consider defining a bean of type X in your configuration`

```
Parameter 0 of constructor in SolicitudService required a bean of type 
'SolicitudRepository' that could not be found.
```

**Causa:** Spring no encuentra el bean que necesita inyectar.

**Soluciones:**
1. Verifica que la clase tiene la anotación correcta (`@Service`, `@Repository`, etc.).
2. Verifica que la clase está en un sub-package de `co.edu.uniquindio.triage` (para que `@ComponentScan` la detecte).
3. Si es una interfaz de Spring Data JPA, debe extender `JpaRepository`.

---

### Error 4: `Whitelabel Error Page` al acceder a un endpoint

**Causa:** Puede ser varias cosas:
1. La URL está mal escrita.
2. Spring Security está bloqueando el acceso (lo configuraremos en la Guía 07).
3. El endpoint no existe.

**Solución temporal para esta guía** (deshabilitar Security mientras desarrollas):

```java
// src/main/java/co/edu/uniquindio/triage/config/SecurityConfig.java
@Configuration
@Profile("dev")  // Solo en desarrollo
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth.anyRequest().permitAll())
            .build();
    }
}
```

> **Importante:** Este código es SOLO para desarrollar más cómodo. La configuración correcta de seguridad la verás en la **Guía 07**.

---

### Error 5: Error de validación no esperado

```
400 Bad Request: "Los datos enviados no son válidos"
```

**Causa:** Uno o más campos del request no pasan la validación.

**Solución:** Revisa el campo `detalles` en la respuesta JSON. Te dirá exactamente qué campos fallaron y por qué.

---

### Error 6: `LazyInitializationException`

```
org.hibernate.LazyInitializationException: 
failed to lazily initialize a collection: could not initialize proxy
```

**Causa:** Estás tratando de acceder a una relación lazy (cargada perezosamente) fuera de una transacción activa.

**Soluciones:**
1. Asegúrate de que el método del Service tenga `@Transactional`.
2. En el `application.yml`, verifica que `spring.jpa.open-in-view: false`.
3. Usa `@EntityGraph` para cargar la relación junto con la entidad (ver Guía 05).

---

## 16. Ejercicios Prácticos

### Ejercicio 1 — Agregar el Endpoint de Actualización de Estado

**Objetivo:** Implementar la transición de estado de una solicitud (RF-04).

**Lo que debes lograr:**

```
PATCH /api/v1/solicitudes/{id}/estado

Body:
{
  "nuevoEstado": "CLASIFICADA"
}

Respuesta 200:
{
  "id": 1,
  "estado": "CLASIFICADA",
  ...
}
```

**Restricciones de negocio:**
- REGISTRADA solo puede pasar a CLASIFICADA.
- CLASIFICADA solo puede pasar a EN_ATENCION.
- EN_ATENCION solo puede pasar a ATENDIDA.
- ATENDIDA solo puede pasar a CERRADA.
- Las transiciones inversas no son válidas.
- Una solicitud CERRADA no puede cambiar de estado.

**Pistas:**
1. Crea el record `CambiarEstadoRequest` con el campo `EstadoSolicitud nuevoEstado`.
2. Crea el método `cambiarEstado(Long id, CambiarEstadoRequest request)` en `SolicitudService`.
3. Implementa la lógica de validación de transiciones.
4. Agrega el endpoint `@PatchMapping("/{id}/estado")` en `SolicitudController`.
5. Si la transición es inválida, lanza `EstadoInvalidoException`.

---

### Ejercicio 2 — Agregar la Asignación de Prioridad

**Objetivo:** Implementar la priorización de solicitudes (RF-03).

**Lo que debes lograr:**

```
PATCH /api/v1/solicitudes/{id}/prioridad

Body:
{
  "prioridad": "ALTA",
  "justificacion": "El estudiante presenta fecha límite de matrícula inminente"
}

Respuesta 200: SolicitudResponse con la prioridad actualizada
```

**Pistas:**
1. Crea el record `AsignarPrioridadRequest`.
2. El campo `justificacion` debe ser obligatorio (`@NotBlank`).
3. Guarda la justificación... ¿en qué campo de la entidad? (Pista: podrías usar el campo `observacionCierre` temporalmente o agregar un nuevo campo `justificacionPrioridad`).

---

### Ejercicio 3 — Crear un Endpoint de Estadísticas

**Objetivo:** Practicar consultas con Spring Data JPA y construir respuestas complejas.

**Lo que debes lograr:**

```
GET /api/v1/solicitudes/estadisticas

Respuesta:
{
  "totalSolicitudes": 15,
  "porEstado": {
    "REGISTRADA": 5,
    "CLASIFICADA": 3,
    "EN_ATENCION": 4,
    "ATENDIDA": 2,
    "CERRADA": 1
  },
  "porCanal": {
    "CORREO": 8,
    "SISTEMA": 4,
    "PRESENCIAL": 3
  }
}
```

**Pistas:**
1. Crea el record `EstadisticasResponse` con los campos `totalSolicitudes`, `porEstado` (Map<EstadoSolicitud, Long>) y `porCanal` (Map<CanalOrigen, Long>).
2. En el Repository, investiga el método `count()` y cómo hacer `countByEstado()`.
3. En el Service, usa Streams y `Collectors.groupingBy` con `Collectors.counting()`.

---

### Ejercicio 4 — Personalizar la Documentación Swagger

**Objetivo:** Practicar la configuración de OpenAPI/Swagger.

**Crea la clase `SwaggerConfig.java`** en el paquete `config` que configure:
- Título: "Sistema de Triage - API"
- Versión: "1.0.0"
- Descripción: "API REST para la gestión de solicitudes académicas de Ingeniería de Sistemas"
- Información de contacto: tu nombre y correo institucional
- Licencia: "Uso académico - Universidad del Quindío"

**Pistas:** Investiga las clases `OpenAPI`, `Info`, `Contact` y `License` de `io.swagger.v3.oas.models`.

---

## 17. Resumen y Cheat Sheet

### Flujo de una Petición HTTP en Spring Boot

```
Cliente (Postman/Angular)
        │
        │  POST /api/v1/solicitudes  + JSON body
        ▼
┌─────────────────────────────────────────────────┐
│  Tomcat Embebido (puerto 8080)                   │
│  Recibe la petición HTTP                        │
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│  DispatcherServlet (Spring MVC)                  │
│  Decide qué controlador maneja esta petición    │
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│  SolicitudController                             │
│  @PostMapping → método registrar()              │
│  @Valid → valida el body                        │
│  Convierte JSON → CrearSolicitudRequest (DTO)   │
└────────────────────┬────────────────────────────┘
                     │  llama
                     ▼
┌─────────────────────────────────────────────────┐
│  SolicitudService                                │
│  Contiene la lógica de negocio                  │
│  Crea la entidad Solicitud                      │
└────────────────────┬────────────────────────────┘
                     │  llama
                     ▼
┌─────────────────────────────────────────────────┐
│  SolicitudRepository                             │
│  Genera el INSERT SQL                           │
│  Ejecuta contra PostgreSQL                      │
└────────────────────┬────────────────────────────┘
                     │  resultado
                     ▼
    Entidad guardada → Service → DTO Response
                     │
                     ▼
    Controller → ResponseEntity<SolicitudResponse>
                     │
                     ▼
    Jackson serializa el DTO a JSON
                     │
                     ▼
    201 Created + JSON body → Cliente
```

### Cheat Sheet de Anotaciones

```java
// Clase principal
@SpringBootApplication          // = @Configuration + @EnableAutoConfiguration + @ComponentScan

// Declarar beans
@Component                      // Bean genérico
@Service                        // Lógica de negocio
@Repository                     // Acceso a datos
@RestController                 // Controlador API REST
@Configuration                  // Clase de configuración
@Bean                           // Método que produce un bean

// Inyección
// (preferir inyección por constructor, sin anotación)
@Autowired                      // Inyección de dependencia
@Qualifier("nombreBean")        // Especificar qué bean inyectar
@Value("${prop.key}")           // Inyectar valor de yml

// Endpoints
@RequestMapping("/ruta")        // URL base
@GetMapping("/{id}")            // GET
@PostMapping                    // POST
@PutMapping("/{id}")            // PUT
@PatchMapping("/{id}/campo")    // PATCH
@DeleteMapping("/{id}")         // DELETE
@PathVariable Long id           // Variable de URL
@RequestParam String filtro     // Parámetro de query ?filtro=valor
@RequestBody @Valid MiDto dto   // Body JSON + validación

// Ciclo de vida
@PostConstruct                  // Ejecutar después de crear el bean
@PreDestroy                     // Ejecutar antes de destruir el bean
@Profile("dev")                 // Bean activo solo en el perfil 'dev'

// Manejo de errores
@RestControllerAdvice           // Manejador global de excepciones
@ExceptionHandler(MiEx.class)   // Maneja una excepción específica
```

---

## 18. Referencias y Recursos Adicionales

**Documentación Oficial:**
- Spring Boot Reference: https://docs.spring.io/spring-boot/docs/current/reference/html/
- Spring Data JPA: https://docs.spring.io/spring-data/jpa/docs/current/reference/html/
- Baeldung Spring Boot: https://www.baeldung.com/spring-boot

**Herramientas Usadas:**
- Spring Initializr: https://start.spring.io
- SpringDoc OpenAPI: https://springdoc.org
- Postman: https://www.postman.com
- Lombok: https://projectlombok.org

**Libros Recomendados:**
- Sharma, Sourabh (2021). *Modern API Development with Spring and Spring Boot.* Packt Publishing.
- Ruíz, Carlos (2018). *Spring Boot & Angular: Desarrollo de WebApps Seguras.* 0xWord.

**Próximas Guías:**
- **Guía 05:** JPA, Hibernate y Persistencia: Del Modelo al Código (modelar todas las entidades del Triage con relaciones completas, Flyway, auditoría).
- **Guía 06:** API REST con Spring Boot: Diseño, implementación y buenas prácticas avanzadas.
- **Guía 07:** Spring Security 6 + JWT desde Cero.

---

> **Autor:** José Alfredo Ramírez Espinosa  
> **Materia:** Programación Avanzada  
> **Programa:** Ingeniería de Sistemas y Computación  
> **Universidad del Quindío** | Armenia, Colombia  
> **Año:** 2026  
> *Material de uso académico. Todos los derechos reservados al autor.*
