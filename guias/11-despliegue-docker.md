# Guía 11 — Despliegue: Docker, CI/CD y la Nube

> **Materia:** Programación Avanzada  
> **Programa:** Ingeniería de Sistemas y Computación — Universidad del Quindío  
> **Núcleo Temático 5:** Despliegue en la Nube  
> **Proyecto base:** Sistema de Triage y Gestión de Solicitudes Académicas  
> **Versiones de referencia:** Docker 27+, Docker Compose v2, Java 21, Angular 19+, Node 22 LTS, Nginx 1.27+, GitHub Actions, Railway, Vercel (2026)

---

## Prerrequisitos

Antes de continuar con esta guía, asegúrate de haber completado:

- **Guía 04** — Spring Boot desde Cero (proyecto backend funcional)
- **Guía 05** — JPA y Persistencia (entidades y repositorios del Triage)
- **Guía 06** — API REST con Spring Boot (endpoints del Triage)
- **Guía 07** — Spring Security + JWT (seguridad implementada)
- **Guía 08 y 09** — Angular (frontend del Triage funcional)
- **Guía 10** — Integración Full-Stack (frontend y backend comunicándose)

Conocimientos previos que se asumen:

- Sabes crear y ejecutar un proyecto Spring Boot con Maven
- Tienes un proyecto Angular que consume la API REST del backend
- Sabes usar la terminal (cmd, bash o PowerShell)
- Tienes cuenta en GitHub

---

## Tabla de Contenidos

1. [¿Por qué necesitamos despliegue? El problema que resuelve Docker](#1-por-qué-necesitamos-despliegue)
2. [Docker: conceptos fundamentales](#2-docker-conceptos-fundamentales)
3. [Instalar Docker Desktop](#3-instalar-docker-desktop)
4. [Tu primera imagen y contenedor](#4-tu-primera-imagen-y-contenedor)
5. [Dockerfile para Spring Boot (multi-stage build)](#5-dockerfile-para-spring-boot)
6. [Dockerfile para Angular con Nginx](#6-dockerfile-para-angular-con-nginx)
7. [Configurar Nginx para Angular SPA](#7-configurar-nginx-para-angular-spa)
8. [Docker Compose: levantando todo el sistema](#8-docker-compose)
9. [Variables de entorno y secrets](#9-variables-de-entorno-y-secrets)
10. [Introducción a CI/CD](#10-introducción-a-cicd)
11. [GitHub Actions: pipeline completo](#11-github-actions)
12. [Despliegue en Railway (backend + base de datos)](#12-despliegue-en-railway)
13. [Despliegue del frontend en Vercel](#13-despliegue-en-vercel)
14. [ng build en producción](#14-ng-build-en-producción)
15. [Checklist completo pre-despliegue](#15-checklist-completo-pre-despliegue)
16. [Errores comunes y troubleshooting](#16-errores-comunes-y-troubleshooting)
17. [Resumen y cheat sheet](#17-resumen-y-cheat-sheet)
18. [Ejercicios prácticos](#18-ejercicios-prácticos)
19. [Referencias y recursos adicionales](#19-referencias-y-recursos-adicionales)

---

## 1. ¿Por qué Necesitamos Despliegue?

### El problema clásico del desarrollador

Imagina esta situación: llevas semanas trabajando en el Sistema de Triage. Funciona perfectamente en tu computador. Le compartes el código a un compañero y... nada funciona. El error dice algo como `Java version mismatch`, o `port 5432 already in use`, o simplemente `Cannot find module`. ¿Te suena familiar?

Ahora imagina el escenario real: terminás el sistema, el coordinador académico quiere probarlo desde la oficina, los administrativos lo van a usar desde sus computadores, y los estudiantes desde sus celulares. Nadie va a instalar Java 21, PostgreSQL, Node 22 y Angular CLI solo para usar tu aplicación.

**Ese es exactamente el problema que resuelve el despliegue.**

### ¿Qué significa "desplegar" una aplicación?

Desplegar (en inglés, *deploy*) significa poner tu aplicación en un lugar donde otras personas puedan acceder a ella, sin importar qué sistema operativo usen, qué versiones de software tengan instaladas, ni dónde estén físicamente.

Para el Sistema de Triage necesitas:

```
┌─────────────────────────────────────────────────────┐
│            ANTES DEL DESPLIEGUE                     │
│                                                     │
│  Tu PC:                                             │
│  ┌──────────────┐  ┌──────────────┐                │
│  │  Spring Boot │  │   Angular    │                │
│  │  localhost   │  │  localhost   │                │
│  │   :8080      │  │    :4200     │                │
│  └──────────────┘  └──────────────┘                │
│        ↑ Solo tú puedes acceder                    │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│            DESPUÉS DEL DESPLIEGUE                   │
│                                                     │
│  Internet:                                          │
│  https://triage-api.railway.app   ← Backend        │
│  https://triage-app.vercel.app    ← Frontend        │
│        ↑ Cualquier persona con internet puede       │
│          acceder desde cualquier dispositivo        │
└─────────────────────────────────────────────────────┘
```

### ¿Qué necesitamos desplegar para el Triage?

| Componente | Tecnología | Dónde vive |
|---|---|---|
| Base de datos | PostgreSQL | Railway (servicio gestionado) |
| Backend | Spring Boot JAR | Railway (contenedor Docker) |
| Frontend | HTML/CSS/JS compilado | Vercel (CDN global) |

---

## 2. Docker: Conceptos Fundamentales

### La analogía del contenedor de barco

Antes de que existieran los contenedores de barco estándar (los grandes cajones metálicos que ves en los puertos), cada empresa empacaba sus mercancías de forma diferente. Una caja de madera, un barril, una bolsa... los estibadores tenían que manipular cada producto de forma distinta. Era lento, caro y propenso a errores.

En 1956 se estandarizó el contenedor intermodal: una caja metálica de dimensiones fijas que se puede poner en un barco, un tren o un camión sin cambiar nada. El barco no sabe si adentro hay zapatos o electrónicos. Solo sabe cómo mover contenedores.

**Docker hace exactamente lo mismo para el software.**

Un contenedor Docker empaca tu aplicación con TODO lo que necesita para funcionar: el código, el runtime (Java 21, Node 22), las librerías, las configuraciones. Luego ese contenedor se puede ejecutar en cualquier máquina que tenga Docker instalado, y se comporta exactamente igual.

### Conceptos clave explicados

#### Imagen (Image)

Una imagen es la **receta** o el **molde** para crear contenedores. Es un archivo de solo lectura que contiene el sistema operativo base, el runtime, el código de tu aplicación y las instrucciones de cómo ejecutarla.

Analogía: la imagen es como el plano de un apartamento. El plano no es el apartamento, pero a partir de él puedes construir muchos apartamentos idénticos.

```
Imagen de Spring Boot del Triage:
┌──────────────────────────────────┐
│  Linux Alpine (SO base ligero)   │
│  ↓                               │
│  Java 21 JRE                     │
│  ↓                               │
│  triage-backend.jar              │
│  ↓                               │
│  EXPONER PUERTO 8080             │
│  EJECUTAR java -jar triage.jar   │
└──────────────────────────────────┘
```

#### Contenedor (Container)

Un contenedor es una **instancia en ejecución** de una imagen. Es el apartamento construido a partir del plano.

- De una imagen puedes crear múltiples contenedores
- Cada contenedor está aislado de los demás y del sistema operativo anfitrión
- Si el contenedor falla o lo eliminas, la imagen sigue intacta

#### Dockerfile

Es el archivo de texto con las instrucciones para construir una imagen. Es la receta. Lo escribes tú, lo interpreta Docker.

#### Docker Compose

Es una herramienta para definir y ejecutar aplicaciones multi-contenedor. En lugar de ejecutar cada contenedor manualmente con comandos largos, defines todo en un archivo `docker-compose.yml` y con un solo comando levantas todo el sistema.

Para el Triage: un solo `docker compose up` levanta PostgreSQL + Spring Boot + Angular/Nginx.

#### Volumen (Volume)

Un mecanismo para persistir datos más allá del ciclo de vida de un contenedor. Sin volúmenes, cuando un contenedor de PostgreSQL se elimina, toda la base de datos se pierde.

#### Red (Network)

Docker crea redes virtuales para que los contenedores se comuniquen entre sí de forma segura, sin exponer todo al exterior.

#### Registry

Es un repositorio de imágenes Docker. El registry público más conocido es **Docker Hub** (hub.docker.com). Puedes subir tus imágenes ahí y descargarlas desde cualquier servidor.

### El flujo completo de Docker

```
┌────────────────────────────────────────────────────────────┐
│                   FLUJO DE DOCKER                          │
│                                                            │
│  1. ESCRIBES      2. CONSTRUYES     3. EJECUTAS            │
│                                                            │
│  Dockerfile   →  docker build   →  docker run             │
│  (receta)        (imagen)          (contenedor)            │
│                      ↓                                     │
│                  docker push    →  Docker Hub / Registry   │
│                      ↓                                     │
│                  docker pull    →  Servidor en la nube     │
│                      ↓                                     │
│                  docker run     →  ¡Aplicación corriendo!  │
└────────────────────────────────────────────────────────────┘
```

---

## 3. Instalar Docker Desktop

Docker Desktop es la forma más sencilla de tener Docker en Windows o macOS. En Linux se instala el motor directamente.

### En Windows

1. Ve a [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
2. Descarga **Docker Desktop for Windows**
3. Ejecuta el instalador `.exe`
4. Durante la instalación, asegúrate de que esté marcada la opción **"Use WSL 2 instead of Hyper-V"** (recomendado)
5. Reinicia el computador cuando se solicite
6. Abre Docker Desktop desde el menú de inicio

### En macOS

1. Ve a [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
2. Descarga la versión según tu chip: **Intel** o **Apple Silicon (M1/M2/M3/M4)**
3. Abre el `.dmg` y arrastra Docker a Aplicaciones
4. Abre Docker Desktop desde Launchpad

### En Linux (Ubuntu/Debian)

```bash
# Desinstalar versiones antiguas si las hubiera
sudo apt-get remove docker docker-engine docker.io containerd runc

# Actualizar índice de paquetes
sudo apt-get update

# Instalar dependencias
sudo apt-get install ca-certificates curl gnupg lsb-release

# Agregar la clave GPG oficial de Docker
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Agregar el repositorio de Docker
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instalar Docker Engine y Docker Compose
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# (Opcional) Ejecutar Docker sin sudo
sudo usermod -aG docker $USER
newgrp docker
```

### Verificar la instalación

Abre una terminal y ejecuta:

```bash
docker --version
# Docker version 27.x.x, build xxxxxxx

docker compose version
# Docker Compose version v2.x.x

docker run hello-world
# Debe imprimir: "Hello from Docker!"
```

Si ves esos mensajes, Docker está correctamente instalado.

---

## 4. Tu Primera Imagen y Contenedor

Antes de hacer el Dockerfile del Triage, vamos a practicar con algo sencillo para entender los comandos básicos.

### Comandos Docker esenciales

```bash
# Ver imágenes descargadas localmente
docker images

# Descargar una imagen del registry (sin crear contenedor)
docker pull ubuntu:24.04

# Crear y ejecutar un contenedor de forma interactiva
docker run -it ubuntu:24.04 bash

# Listar contenedores en ejecución
docker ps

# Listar TODOS los contenedores (incluidos los detenidos)
docker ps -a

# Detener un contenedor
docker stop <nombre-o-id>

# Eliminar un contenedor detenido
docker rm <nombre-o-id>

# Eliminar una imagen
docker rmi <nombre:tag>

# Ver logs de un contenedor
docker logs <nombre-o-id>

# Ejecutar un comando dentro de un contenedor en ejecución
docker exec -it <nombre-o-id> bash

# Construir una imagen desde un Dockerfile
docker build -t mi-imagen:1.0 .

# Ejecutar un contenedor con configuraciones comunes
docker run \
  --name mi-contenedor \       # nombre amigable
  -d \                         # modo daemon (background)
  -p 8080:8080 \               # mapear puerto host:contenedor
  -e VARIABLE=valor \          # variable de entorno
  -v /host/path:/container/path \ # montar volumen
  mi-imagen:1.0
```

### Prueba rápida: PostgreSQL en Docker

Esto es exactamente lo que haremos para el Triage pero de forma manual primero, para entender qué hace Docker Compose automáticamente:

```bash
# Ejecutar PostgreSQL en un contenedor
docker run \
  --name postgres-triage \
  -d \
  -p 5432:5432 \
  -e POSTGRES_DB=triage_db \
  -e POSTGRES_USER=triage_user \
  -e POSTGRES_PASSWORD=triage_pass \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:16-alpine

# Verificar que está corriendo
docker ps

# Conectarse a la base de datos
docker exec -it postgres-triage psql -U triage_user -d triage_db

# Dentro de psql
\dt        -- listar tablas
\q         -- salir
```

Si podes conectarte, PostgreSQL está corriendo dentro de Docker. Ahora tu Spring Boot en `application.yml` puede apuntar a `localhost:5432` y funcionará exactamente igual que si tuvieras PostgreSQL instalado directamente.

---

## 5. Dockerfile para Spring Boot

### ¿Qué es un multi-stage build?

El concepto de **multi-stage build** (construcción en múltiples etapas) es uno de los mejores patrones para imágenes de producción. La idea es:

**Etapa 1 (Build):** Usa una imagen grande con Maven y el JDK completo para compilar y construir el JAR. Esta imagen pesa ~500MB pero solo la usamos para compilar.

**Etapa 2 (Runtime):** Copia solamente el JAR resultante a una imagen nueva, mucho más pequeña, que solo tiene el JRE (Java Runtime Environment, sin herramientas de compilación). Esta imagen pesa ~150MB.

El resultado final es una imagen pequeña y segura, sin código fuente ni herramientas de compilación expuestas.

```
┌─────────────────────────────────────────────────────┐
│             MULTI-STAGE BUILD                       │
│                                                     │
│  ETAPA 1: BUILD                                     │
│  ┌─────────────────────────────────────┐            │
│  │  maven:3.9-eclipse-temurin-21       │            │
│  │  (~500MB)                           │            │
│  │  mvn clean package -DskipTests      │            │
│  │  → target/triage-backend.jar        │            │
│  └─────────────────────────────────────┘            │
│                      ↓ Solo el JAR                  │
│  ETAPA 2: RUNTIME                                   │
│  ┌─────────────────────────────────────┐            │
│  │  eclipse-temurin:21-jre-alpine      │            │
│  │  (~150MB)                           │            │
│  │  java -jar triage-backend.jar       │            │
│  └─────────────────────────────────────┘            │
│         ↑ Imagen final ligera y segura              │
└─────────────────────────────────────────────────────┘
```

### Estructura del proyecto backend antes de dockerizar

Asegúrate de que tu proyecto Spring Boot tenga esta estructura:

```
triage-backend/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── co/edu/uniquindio/triage/
│   │   │       ├── TriageApplication.java
│   │   │       ├── config/
│   │   │       ├── controller/
│   │   │       ├── domain/
│   │   │       ├── dto/
│   │   │       ├── repository/
│   │   │       └── service/
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       ├── application-prod.yml
│   │       └── db/migration/
│   └── test/
├── pom.xml
├── Dockerfile              ← NUEVO: aquí va el Dockerfile
└── .dockerignore           ← NUEVO: archivos a excluir
```

### El archivo `.dockerignore`

Antes de escribir el Dockerfile, crea `.dockerignore` en la raíz del proyecto backend. Este archivo le dice a Docker qué archivos NO copiar al contexto de construcción (similar a `.gitignore`):

```dockerignore
# .dockerignore para triage-backend

# Directorio de compilación (Docker recompilará dentro del contenedor)
target/

# Archivos del IDE
.idea/
*.iml
.vscode/
*.class

# Git
.git/
.gitignore

# Docker (no copiar el propio Dockerfile)
Dockerfile
.dockerignore

# Variables de entorno locales (NUNCA en producción)
.env
.env.local

# Logs
*.log
logs/

# Archivos del sistema operativo
.DS_Store
Thumbs.db
```

### El Dockerfile del backend (multi-stage)

```dockerfile
# ============================================================
# Dockerfile para triage-backend (Spring Boot)
# Multi-stage build: compila con Maven, ejecuta con JRE ligero
# ============================================================

# ─────────────────────────────────────────────────────────────
# ETAPA 1: BUILD
# Usamos la imagen oficial de Maven con Java 21
# Esta imagen tiene todo lo necesario para compilar
# ─────────────────────────────────────────────────────────────
FROM maven:3.9-eclipse-temurin-21-alpine AS build

# Definir el directorio de trabajo dentro del contenedor
WORKDIR /app

# OPTIMIZACIÓN DE CACHÉ DE DOCKER:
# Primero copiamos SOLO el pom.xml y descargamos dependencias.
# Docker guarda esta capa en caché. Si el pom.xml no cambia,
# en el próximo build Docker reutiliza esta capa y NO vuelve a
# descargar internet. Esto hace los builds mucho más rápidos.
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Ahora copiamos el código fuente
COPY src ./src

# Compilar y empaquetar. -DskipTests omite los tests en el build
# (en CI/CD los tests se corren en un paso separado)
RUN mvn clean package -DskipTests -B

# ─────────────────────────────────────────────────────────────
# ETAPA 2: RUNTIME
# Imagen mínima: solo el JRE de Java 21 sobre Alpine Linux
# Alpine es una distribución Linux ultra-ligera (~5MB de base)
# ─────────────────────────────────────────────────────────────
FROM eclipse-temurin:21-jre-alpine AS runtime

# Etiquetas de metadatos (buena práctica)
LABEL maintainer="Jose Alfredo Ramirez <jramirez@uniquindio.edu.co>"
LABEL description="Backend del Sistema de Triage - Programación Avanzada UniQuindío"
LABEL version="1.0"

# Crear un usuario no-root para ejecutar la aplicación
# NUNCA ejecutar aplicaciones de producción como root
RUN addgroup -S triage && adduser -S triage -G triage

# Directorio de trabajo en runtime
WORKDIR /app

# Copiar el JAR compilado desde la etapa de build
# El nombre del JAR lo define el pom.xml (artifactId-version.jar)
# Con el comodín *.jar evitamos hardcodear la versión
COPY --from=build /app/target/*.jar triage-backend.jar

# Asignar permisos al usuario triage
RUN chown triage:triage triage-backend.jar

# Cambiar al usuario no-root
USER triage

# Exponer el puerto en el que Spring Boot escucha
# NOTA: EXPOSE es solo documentación. El binding real se hace en docker run o compose.
EXPOSE 8080

# Variable de entorno para el perfil de Spring
# Se puede sobreescribir al ejecutar el contenedor
ENV SPRING_PROFILES_ACTIVE=prod

# Comando de inicio
# Opciones JVM para entornos containerizados:
# -XX:+UseContainerSupport: usar los límites de CPU/RAM del contenedor
# -XX:MaxRAMPercentage=75.0: usar máximo 75% de la RAM disponible
ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-Djava.security.egd=file:/dev/./urandom", \
  "-jar", "triage-backend.jar"]
```

### Configurar application-prod.yml

Para que Spring Boot funcione correctamente en el contenedor, necesitamos un perfil de producción que use variables de entorno en lugar de valores hardcodeados:

```yaml
# src/main/resources/application-prod.yml

spring:
  datasource:
    url: ${DATABASE_URL}
    username: ${DATABASE_USERNAME}
    password: ${DATABASE_PASSWORD}
    driver-class-name: org.postgresql.Driver
    hikari:
      maximum-pool-size: 10
      minimum-idle: 2
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000

  jpa:
    hibernate:
      ddl-auto: validate    # En producción NUNCA usar create o create-drop
    show-sql: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: false

  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true

# JWT - usar variables de entorno SIEMPRE en producción
security:
  jwt:
    secret: ${JWT_SECRET}
    expiration: ${JWT_EXPIRATION:3600000}   # 1 hora por defecto

# Deshabilitar Swagger en producción (opcional pero recomendado)
springdoc:
  api-docs:
    enabled: ${SWAGGER_ENABLED:false}
  swagger-ui:
    enabled: ${SWAGGER_ENABLED:false}

# Logging más conciso en producción
logging:
  level:
    root: WARN
    co.edu.uniquindio.triage: INFO
    org.springframework.security: WARN
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"

# Actuator - solo health check expuesto
management:
  endpoints:
    web:
      exposure:
        include: health
  endpoint:
    health:
      show-details: never
```

### Construir y probar la imagen del backend

```bash
# Desde la raíz de triage-backend/
cd triage-backend

# Construir la imagen
# -t: tag (nombre:version)
# .: usar el directorio actual como contexto
docker build -t triage-backend:1.0 .

# Verificar que la imagen se creó
docker images | grep triage-backend

# Ejecutar el contenedor para probar
# (requiere que PostgreSQL esté corriendo, usamos el del paso anterior)
docker run \
  --name triage-backend-test \
  -d \
  -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e DATABASE_URL=jdbc:postgresql://host.docker.internal:5432/triage_db \
  -e DATABASE_USERNAME=triage_user \
  -e DATABASE_PASSWORD=triage_pass \
  -e JWT_SECRET=mi-secreto-super-largo-para-produccion-minimo-32-caracteres \
  triage-backend:1.0

# Ver logs en tiempo real
docker logs -f triage-backend-test

# Probar que responde
curl http://localhost:8080/actuator/health
# {"status":"UP"}
```

> **Nota sobre `host.docker.internal`:** Cuando un contenedor Docker necesita conectarse a un servicio que corre en la máquina anfitriona (como el PostgreSQL que instalaste directamente), usa `host.docker.internal` en lugar de `localhost`. En Docker Compose esto no es necesario porque todos los contenedores están en la misma red interna.

---

## 6. Dockerfile para Angular con Nginx

### El proceso de build de Angular

A diferencia de Spring Boot que se empaqueta como un JAR ejecutable, Angular se compila en **archivos estáticos**: HTML, CSS y JavaScript. Un servidor web como Nginx los sirve directamente al navegador.

```
┌────────────────────────────────────────────────────┐
│         PROCESO DE BUILD DE ANGULAR                │
│                                                    │
│  ETAPA 1: BUILD                                    │
│  ┌─────────────────────────────────────────────┐   │
│  │  node:22-alpine                             │   │
│  │  npm install                                │   │
│  │  ng build --configuration production        │   │
│  │  → dist/triage-frontend/browser/           │   │
│  │     ├── index.html                         │   │
│  │     ├── main-ABC123.js                     │   │
│  │     ├── polyfills-DEF456.js                │   │
│  │     └── styles-GHI789.css                  │   │
│  └─────────────────────────────────────────────┘   │
│                       ↓ Solo los archivos dist     │
│  ETAPA 2: SERVE                                    │
│  ┌─────────────────────────────────────────────┐   │
│  │  nginx:1.27-alpine                          │   │
│  │  (~10MB)                                    │   │
│  │  Sirve los archivos estáticos               │   │
│  └─────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────┘
```

### Estructura del proyecto frontend

```
triage-frontend/
├── src/
│   ├── app/
│   │   ├── core/
│   │   ├── features/
│   │   └── shared/
│   ├── environments/
│   │   ├── environment.ts          ← desarrollo
│   │   └── environment.prod.ts     ← producción
│   └── index.html
├── angular.json
├── package.json
├── tsconfig.json
├── nginx.conf                ← NUEVO: config de Nginx
├── Dockerfile                ← NUEVO
└── .dockerignore             ← NUEVO
```

### El archivo `.dockerignore` del frontend

```dockerignore
# .dockerignore para triage-frontend

# Dependencias (se instalarán dentro del contenedor)
node_modules/

# Resultado del build previo (Docker hará su propio build)
dist/

# Git
.git/
.gitignore

# Archivos de entorno locales
.env
.env.local

# Cache de Angular
.angular/

# Archivos del IDE
.idea/
.vscode/

# Archivos del sistema
.DS_Store
Thumbs.db
```

### Los archivos de entorno de Angular

Angular usa archivos `environment.ts` para manejar configuraciones por entorno. Al hacer `ng build --configuration production`, Angular reemplaza automáticamente `environment.ts` por `environment.prod.ts`.

```typescript
// src/environments/environment.ts  (desarrollo)
export const environment = {
  production: false,
  apiUrl: 'http://localhost:8080/api/v1',
  appName: 'Sistema de Triage - DEV'
};
```

```typescript
// src/environments/environment.prod.ts  (producción)
export const environment = {
  production: true,
  // En producción, la URL viene configurada en el build
  // NUNCA hardcodear URLs de producción en el código
  apiUrl: '/api/v1',  // Relativo: Nginx hará proxy al backend
  appName: 'Sistema de Triage Académico - UniQuindío'
};
```

> **¿Por qué `/api/v1` relativo?** Cuando frontend y backend están en el mismo dominio (o Nginx hace de proxy), usar rutas relativas es más limpio. Nginx intercepta las peticiones a `/api/v1/...` y las redirige al backend. Esto evita problemas de CORS en producción.

### El Dockerfile del frontend

```dockerfile
# ============================================================
# Dockerfile para triage-frontend (Angular + Nginx)
# ============================================================

# ─────────────────────────────────────────────────────────────
# ETAPA 1: BUILD
# Compilar Angular con Node.js
# ─────────────────────────────────────────────────────────────
FROM node:22-alpine AS build

# Instalar Angular CLI globalmente
RUN npm install -g @angular/cli@19

WORKDIR /app

# Copiar package.json y package-lock.json ANTES del código fuente
# Igual que con Maven: cachear la instalación de dependencias
COPY package.json package-lock.json ./

# Instalar dependencias (--frozen-lockfile garantiza reproducibilidad)
RUN npm ci --frozen-lockfile

# Copiar el resto del código fuente
COPY . .

# Compilar en modo producción
# --output-path: dónde poner los archivos compilados
RUN ng build --configuration production

# ─────────────────────────────────────────────────────────────
# ETAPA 2: SERVE
# Servir los archivos estáticos con Nginx
# ─────────────────────────────────────────────────────────────
FROM nginx:1.27-alpine AS runtime

LABEL maintainer="Jose Alfredo Ramirez <jramirez@uniquindio.edu.co>"
LABEL description="Frontend del Sistema de Triage - Angular con Nginx"

# Eliminar la configuración por defecto de Nginx
RUN rm /etc/nginx/conf.d/default.conf

# Copiar nuestra configuración personalizada de Nginx
COPY nginx.conf /etc/nginx/conf.d/triage.conf

# Copiar los archivos compilados de Angular al directorio web de Nginx
# Angular 17+ pone los archivos en dist/<nombre-proyecto>/browser/
COPY --from=build /app/dist/triage-frontend/browser /usr/share/nginx/html

# Nginx escucha en el puerto 80 por defecto
EXPOSE 80

# Nginx se inicia automáticamente. El flag "daemon off" es necesario
# para que Nginx corra en primer plano (Docker necesita que el proceso
# principal no se demonize para poder gestionar el contenedor)
CMD ["nginx", "-g", "daemon off;"]
```

---

## 7. Configurar Nginx para Angular SPA

Este es uno de los pasos más importantes y donde muchos estudiantes cometen errores. Nginx necesita configuración específica para servir una Single Page Application (SPA) como Angular.

### El problema del routing en SPAs

En una SPA, el routing lo maneja JavaScript en el navegador. Cuando el usuario navega a `https://triage.app/solicitudes/123`, el servidor recibe esa petición. Nginx busca un archivo en `/usr/share/nginx/html/solicitudes/123/index.html`... que no existe. Resultado: **404 Not Found**.

La solución es decirle a Nginx: "si el archivo no existe, devuelve siempre `index.html` y deja que Angular maneje el routing".

### El archivo `nginx.conf`

```nginx
# nginx.conf para el Sistema de Triage
# Va en: triage-frontend/nginx.conf

server {
    listen 80;
    server_name _;   # Acepta cualquier nombre de host

    # Directorio raíz donde están los archivos de Angular
    root /usr/share/nginx/html;
    index index.html;

    # Habilitar compresión gzip para reducir el tamaño de las respuestas
    # Esto mejora significativamente el tiempo de carga
    gzip on;
    gzip_vary on;
    gzip_min_length 1000;
    gzip_proxied expired no-cache no-store private auth;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/javascript
        application/x-javascript
        application/json
        application/xml
        application/rss+xml
        application/atom+xml
        image/svg+xml;

    # ─────────────────────────────────────────────────────────
    # PROXY INVERSO AL BACKEND
    # Todas las peticiones a /api/* se redirigen al backend
    # Esto elimina los problemas de CORS en producción porque
    # tanto el frontend como el backend están en el mismo "origen"
    # ─────────────────────────────────────────────────────────
    location /api/ {
        proxy_pass http://triage-backend:8080;   # nombre del servicio en Docker Compose

        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Connection "";

        # Timeouts para peticiones largas (ej: generación con IA)
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;

        # Tamaño máximo del cuerpo de la petición (para subir archivos)
        client_max_body_size 10M;
    }

    # ─────────────────────────────────────────────────────────
    # ARCHIVOS ESTÁTICOS CON CACHÉ LARGA
    # Angular genera nombres únicos (hashing) para sus bundles:
    # main-ABC123.js, styles-XYZ789.css
    # Podemos cachearlos por 1 año porque si cambia el contenido,
    # cambia el nombre del archivo
    # ─────────────────────────────────────────────────────────
    location ~* \.(js|css|png|jpg|jpeg|gif|svg|ico|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        add_header Vary Accept-Encoding;
        try_files $uri =404;
    }

    # ─────────────────────────────────────────────────────────
    # ROUTING DE ANGULAR (SPA)
    # Si el archivo solicitado no existe, devolver index.html
    # Angular maneja el routing internamente
    # ─────────────────────────────────────────────────────────
    location / {
        try_files $uri $uri/ /index.html;
    }

    # ─────────────────────────────────────────────────────────
    # HEADERS DE SEGURIDAD
    # ─────────────────────────────────────────────────────────
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline' 'unsafe-eval'" always;

    # Ocultar la versión de Nginx (seguridad por oscuridad)
    server_tokens off;
}
```

---

## 8. Docker Compose: Levantando Todo el Sistema

Docker Compose te permite definir y ejecutar múltiples contenedores como si fueran un único sistema. Para el Triage necesitamos:

1. **PostgreSQL** — base de datos
2. **triage-backend** — Spring Boot
3. **triage-frontend** — Angular + Nginx
4. **pgAdmin** — herramienta visual para gestionar la base de datos (solo en desarrollo)

### Estructura final del proyecto completo

```
triage-sistema/                    ← Carpeta raíz del monorepo
├── triage-backend/
│   ├── src/
│   ├── pom.xml
│   ├── Dockerfile
│   └── .dockerignore
├── triage-frontend/
│   ├── src/
│   ├── angular.json
│   ├── package.json
│   ├── nginx.conf
│   ├── Dockerfile
│   └── .dockerignore
├── docker-compose.yml             ← Configuración de todos los servicios
├── docker-compose.dev.yml         ← Overrides solo para desarrollo
├── docker-compose.prod.yml        ← Overrides para producción
├── .env.example                   ← Plantilla de variables (SÍ va en git)
├── .env                           ← Variables reales (NUNCA en git)
└── .gitignore
```

### El archivo `docker-compose.yml` (base)

```yaml
# docker-compose.yml
# Configuración base válida para todos los entornos

name: triage-sistema

services:

  # ────────────────────────────────────────────────────────
  # BASE DE DATOS: PostgreSQL
  # ────────────────────────────────────────────────────────
  postgres:
    image: postgres:16-alpine
    container_name: triage-postgres
    restart: unless-stopped      # reiniciar automáticamente si falla
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      # Configuración de rendimiento para producción
      POSTGRES_INITDB_ARGS: "--encoding=UTF-8 --lc-collate=es_CO.UTF-8 --lc-ctype=es_CO.UTF-8"
    volumes:
      # Volumen nombrado para persistir los datos de la BD
      # Si el contenedor se elimina, los datos se mantienen en el volumen
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      # Verificar que PostgreSQL está listo para aceptar conexiones
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    networks:
      - triage-network

  # ────────────────────────────────────────────────────────
  # BACKEND: Spring Boot
  # ────────────────────────────────────────────────────────
  triage-backend:
    build:
      context: ./triage-backend
      dockerfile: Dockerfile
    container_name: triage-backend
    restart: unless-stopped
    environment:
      SPRING_PROFILES_ACTIVE: prod
      # Usamos el nombre del servicio 'postgres' como host
      # Docker Compose crea DNS automáticamente para los servicios
      DATABASE_URL: jdbc:postgresql://postgres:5432/${POSTGRES_DB}
      DATABASE_USERNAME: ${POSTGRES_USER}
      DATABASE_PASSWORD: ${POSTGRES_PASSWORD}
      JWT_SECRET: ${JWT_SECRET}
      JWT_EXPIRATION: ${JWT_EXPIRATION:-3600000}
    depends_on:
      postgres:
        condition: service_healthy   # Solo iniciar cuando postgres esté sano
    healthcheck:
      test: ["CMD-SHELL", "wget -q --spider http://localhost:8080/actuator/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s    # Spring Boot necesita tiempo para iniciar
    networks:
      - triage-network

  # ────────────────────────────────────────────────────────
  # FRONTEND: Angular + Nginx
  # ────────────────────────────────────────────────────────
  triage-frontend:
    build:
      context: ./triage-frontend
      dockerfile: Dockerfile
    container_name: triage-frontend
    restart: unless-stopped
    ports:
      # Solo el frontend expone un puerto al exterior
      # El acceso es: http://localhost:80 → Nginx → Angular
      # Las peticiones a /api/* → Nginx → triage-backend:8080
      - "${FRONTEND_PORT:-80}:80"
    depends_on:
      triage-backend:
        condition: service_healthy
    networks:
      - triage-network

# ────────────────────────────────────────────────────────────
# VOLÚMENES NOMBRADOS
# Los volúmenes nombrados persisten entre reinicios y
# actualizaciones de contenedores
# ────────────────────────────────────────────────────────────
volumes:
  postgres_data:
    name: triage_postgres_data

# ────────────────────────────────────────────────────────────
# REDES
# Una red privada para comunicación interna entre contenedores
# Solo el frontend expone puertos al exterior
# ────────────────────────────────────────────────────────────
networks:
  triage-network:
    name: triage_network
    driver: bridge
```

### Override para desarrollo (`docker-compose.dev.yml`)

```yaml
# docker-compose.dev.yml
# Agrega herramientas de desarrollo al stack base

services:

  # En desarrollo, exponemos el puerto de postgres para conectarnos con DBeaver/pgAdmin
  postgres:
    ports:
      - "5432:5432"

  # En desarrollo, exponemos el puerto del backend directamente
  # para poder usar Swagger UI y probar con Postman sin pasar por Nginx
  triage-backend:
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: dev
      SWAGGER_ENABLED: "true"

  # pgAdmin: herramienta visual para PostgreSQL
  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: triage-pgadmin
    restart: unless-stopped
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_EMAIL:-admin@triage.local}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_PASSWORD:-admin123}
    ports:
      - "5050:80"
    volumes:
      - pgadmin_data:/var/lib/pgadmin
    depends_on:
      - postgres
    networks:
      - triage-network

volumes:
  pgadmin_data:
    name: triage_pgadmin_data
```

### Override para producción (`docker-compose.prod.yml`)

```yaml
# docker-compose.prod.yml
# Configuraciones adicionales de seguridad y rendimiento para producción

services:

  postgres:
    # En producción NO exponemos el puerto de postgres al exterior
    # Solo accesible internamente por la red de Docker
    deploy:
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M

  triage-backend:
    # En producción NO exponemos el 8080 directamente
    # Todo el tráfico entra por Nginx
    deploy:
      resources:
        limits:
          memory: 768M
        reservations:
          memory: 512M

  triage-frontend:
    # En producción podría usarse un puerto diferente si hay un balanceador de carga
    ports:
      - "80:80"
    deploy:
      resources:
        limits:
          memory: 128M
        reservations:
          memory: 64M
```

---

## 9. Variables de Entorno y Secrets

### El principio fundamental

**Nunca, bajo ninguna circunstancia, guardes credenciales en el código fuente o en archivos que se suban a Git.**

Esto incluye: contraseñas de base de datos, claves JWT, API keys, tokens de acceso.

### El archivo `.env`

Docker Compose carga automáticamente las variables del archivo `.env` en el mismo directorio:

```bash
# .env
# ESTE ARCHIVO NUNCA VA EN GIT
# Copia .env.example, renómbralo a .env y completa los valores

# Base de datos
POSTGRES_DB=triage_db
POSTGRES_USER=triage_user
POSTGRES_PASSWORD=UnaContraseñaMuySegura2026!

# JWT - Generar con: openssl rand -base64 64
JWT_SECRET=eyJhbGciOiJIUzUxMiJ9.superSecretKeyGeneradaConOpenSSL.queSeaLargaDeMas64Bytes
JWT_EXPIRATION=3600000

# Puertos
FRONTEND_PORT=80

# pgAdmin (solo dev)
PGADMIN_EMAIL=admin@triage.local
PGADMIN_PASSWORD=Admin123Local!
```

```bash
# .env.example
# PLANTILLA — Copia este archivo como .env y completa los valores
# Este archivo SÍ va en git, sirve de documentación

POSTGRES_DB=triage_db
POSTGRES_USER=triage_user
POSTGRES_PASSWORD=CAMBIAR_POR_PASSWORD_SEGURO

JWT_SECRET=CAMBIAR_POR_SECRET_DE_MINIMO_64_BYTES_GENERADO_CON_OPENSSL
JWT_EXPIRATION=3600000

FRONTEND_PORT=80

PGADMIN_EMAIL=admin@triage.local
PGADMIN_PASSWORD=CAMBIAR_POR_PASSWORD_SEGURO
```

```bash
# .gitignore (en la raíz del proyecto)
.env
.env.local
.env.production
*.env

# Nunca subir el target de Maven
triage-backend/target/

# Nunca subir node_modules
triage-frontend/node_modules/
triage-frontend/dist/
triage-frontend/.angular/
```

### Generar un JWT secret seguro

```bash
# Usando openssl (disponible en Linux, macOS, y Git Bash en Windows)
openssl rand -base64 64

# Salida ejemplo:
# K9mP+2XvL1rQfJ8nHcD0uWzYbA5sT3eG7iN4oM6pV9kR+qE1lF2jI0hU8wC/XnBD...
```

### Comandos de Docker Compose

```bash
# Levantar todos los servicios (modo background)
docker compose up -d

# Levantar con override de desarrollo
docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d

# Ver logs de todos los servicios
docker compose logs -f

# Ver logs de un servicio específico
docker compose logs -f triage-backend

# Detener todos los servicios (sin eliminar contenedores)
docker compose stop

# Detener y eliminar contenedores (los volúmenes persisten)
docker compose down

# Detener, eliminar contenedores Y volúmenes (BORRA LA BD)
docker compose down -v

# Reconstruir imágenes y levantar
docker compose up -d --build

# Ver el estado de los servicios
docker compose ps

# Ejecutar un comando en un servicio
docker compose exec triage-backend bash
docker compose exec postgres psql -U triage_user -d triage_db

# Escalar un servicio (múltiples instancias del backend)
docker compose up -d --scale triage-backend=3
```

### Flujo de trabajo en desarrollo

```bash
# Primera vez (configuración inicial)
cp .env.example .env
# Editar .env con tus valores

# Levantar el sistema por primera vez
docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d --build

# Verificar que todo está corriendo
docker compose ps

# Acceder al sistema:
# Frontend:  http://localhost
# Backend:   http://localhost:8080/swagger-ui.html
# pgAdmin:   http://localhost:5050

# Cuando modificas el código:
# Solo reconstruir el servicio modificado
docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d --build triage-backend

# Ver logs del backend
docker compose logs -f triage-backend
```

---

## 10. Introducción a CI/CD

### ¿Qué es la Integración Continua y la Entrega Continua?

**CI (Continuous Integration — Integración Continua)** es la práctica de integrar cambios de código frecuentemente (varias veces al día) en un repositorio compartido, y verificar automáticamente que esos cambios no rompan nada mediante tests automatizados.

**CD (Continuous Delivery/Deployment — Entrega/Despliegue Continuo)** es la práctica de automatizar el proceso de llevar el código desde el repositorio hasta un servidor de producción.

```
┌────────────────────────────────────────────────────────────────┐
│                    FLUJO CI/CD                                 │
│                                                                │
│  1. DEVELOPER         2. CI (tests)        3. CD (deploy)      │
│                                                                │
│  git push    →    Tests automáticos  →  Build de imágenes      │
│  (código)         ¿Pasaron?              Docker                │
│                   ├── SÍ → continúa  →  Push al registry      │
│                   └── NO → notifica      ↓                     │
│                       al dev y          Despliegue automático   │
│                       para el pipeline  en Railway / servidor   │
│                                         ↓                      │
│                                         ¡App actualizada!       │
└────────────────────────────────────────────────────────────────┘
```

### ¿Por qué CI/CD?

Sin CI/CD, el proceso de despliegue es manual, lento y propenso a errores:

1. El desarrollador compila manualmente
2. Copia el JAR al servidor
3. Detiene el servidor, reemplaza el JAR, reinicia
4. Reza para que funcione

Con CI/CD:

1. `git push`
2. Pipeline se ejecuta automáticamente (2-5 minutos)
3. Si los tests pasan, la aplicación se actualiza automáticamente en producción

### Herramientas de CI/CD

| Herramienta | Tipo | Gratuito para proyectos open source |
|---|---|---|
| **GitHub Actions** | SaaS integrado con GitHub | ✅ (2000 min/mes en privados) |
| Jenkins | Self-hosted | ✅ (necesitas servidor propio) |
| GitLab CI | SaaS / Self-hosted | ✅ |
| CircleCI | SaaS | Limitado |
| Travis CI | SaaS | No (fue gratuito, ya no) |

Para el Triage usaremos **GitHub Actions** por su integración nativa con GitHub y su generosidad con repositorios de estudiantes.

---

## 11. GitHub Actions: Pipeline Completo

### Conceptos de GitHub Actions

- **Workflow:** El archivo YAML que define el pipeline completo
- **Trigger (on):** El evento que dispara el workflow (push, pull_request, schedule)
- **Job:** Una unidad de trabajo que corre en un runner (máquina virtual)
- **Step:** Un paso dentro de un job (ejecutar un comando, usar una action)
- **Action:** Una tarea reutilizable publicada por la comunidad (como `actions/checkout@v4`)
- **Runner:** La máquina virtual donde corre el job (ubuntu-latest, windows-latest, macos-latest)
- **Secrets:** Variables sensibles configuradas en GitHub, no en el código

### Configurar los Secrets en GitHub

Antes de crear el pipeline, necesitas configurar los secrets en GitHub:

1. Ve a tu repositorio en GitHub
2. Settings → Secrets and variables → Actions
3. Haz clic en "New repository secret"
4. Agrega estos secrets:

```
DOCKER_HUB_USERNAME     → tu usuario de Docker Hub
DOCKER_HUB_TOKEN        → token de acceso de Docker Hub (no tu password)
RAILWAY_TOKEN           → token de despliegue de Railway
POSTGRES_PASSWORD       → contraseña de la BD en producción
JWT_SECRET              → secreto JWT de producción
```

Para obtener el token de Docker Hub: hub.docker.com → Account Settings → Security → New Access Token

### Estructura de carpetas para GitHub Actions

```
triage-sistema/
├── .github/
│   └── workflows/
│       ├── ci.yml          ← Pipeline de CI (tests en cada push)
│       └── cd.yml          ← Pipeline de CD (deploy en main)
├── triage-backend/
├── triage-frontend/
└── docker-compose.yml
```

### El pipeline de CI (`ci.yml`)

```yaml
# .github/workflows/ci.yml
# Se ejecuta en CADA push a CUALQUIER rama y en pull requests
# Su único objetivo: verificar que el código compila y los tests pasan

name: CI — Integración Continua

on:
  push:
    branches: ['**']       # Cualquier rama
  pull_request:
    branches: [main, develop]

jobs:

  # ──────────────────────────────────────────────────────────
  # JOB 1: Tests del Backend (Spring Boot)
  # ──────────────────────────────────────────────────────────
  test-backend:
    name: Tests Backend (Java 21 + Maven)
    runs-on: ubuntu-latest

    # Servicio de PostgreSQL para los tests de integración
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_DB: triage_test
          POSTGRES_USER: test_user
          POSTGRES_PASSWORD: test_pass
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      # Paso 1: Obtener el código del repositorio
      - name: Checkout del código
        uses: actions/checkout@v4

      # Paso 2: Configurar Java 21
      - name: Configurar Java 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: 'maven'    # Cachear dependencias Maven entre ejecuciones

      # Paso 3: Ejecutar tests
      - name: Ejecutar tests del backend
        working-directory: ./triage-backend
        env:
          # Variables de entorno para los tests
          DATABASE_URL: jdbc:postgresql://localhost:5432/triage_test
          DATABASE_USERNAME: test_user
          DATABASE_PASSWORD: test_pass
          JWT_SECRET: test-secret-key-para-ci-minimo-64-caracteres-de-largo-ok
          SPRING_PROFILES_ACTIVE: test
        run: mvn test -B

      # Paso 4: Publicar reporte de tests (aparece en la pestaña Actions de GitHub)
      - name: Publicar reporte de tests
        uses: dorny/test-reporter@v1
        if: always()    # Ejecutar aunque los tests fallen
        with:
          name: Tests Backend
          path: triage-backend/target/surefire-reports/*.xml
          reporter: java-junit

      # Paso 5: Verificar cobertura de código (opcional pero recomendado)
      - name: Verificar cobertura con JaCoCo
        working-directory: ./triage-backend
        run: mvn verify -DskipTests jacoco:report

  # ──────────────────────────────────────────────────────────
  # JOB 2: Build del Frontend (Angular)
  # ──────────────────────────────────────────────────────────
  build-frontend:
    name: Build Frontend (Angular 19 + Node 22)
    runs-on: ubuntu-latest

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Configurar Node.js 22
        uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
          cache-dependency-path: triage-frontend/package-lock.json

      - name: Instalar dependencias
        working-directory: ./triage-frontend
        run: npm ci --frozen-lockfile

      - name: Ejecutar linting
        working-directory: ./triage-frontend
        run: npx ng lint

      - name: Ejecutar tests del frontend
        working-directory: ./triage-frontend
        run: npx ng test --watch=false --browsers=ChromeHeadless

      - name: Build de producción
        working-directory: ./triage-frontend
        run: npx ng build --configuration production

      # Verificar que el build generó los archivos esperados
      - name: Verificar output del build
        run: ls -la triage-frontend/dist/triage-frontend/browser/

  # ──────────────────────────────────────────────────────────
  # JOB 3: Análisis de seguridad de dependencias
  # ──────────────────────────────────────────────────────────
  security-scan:
    name: Análisis de Seguridad
    runs-on: ubuntu-latest
    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Escanear vulnerabilidades en backend (OWASP)
        working-directory: ./triage-backend
        run: mvn dependency-check:check -DfailBuildOnCVSS=8
        continue-on-error: true   # No fallar el pipeline, solo reportar

      - name: Escanear vulnerabilidades en frontend
        working-directory: ./triage-frontend
        run: npm audit --audit-level=high
        continue-on-error: true
```

### El pipeline de CD (`cd.yml`)

```yaml
# .github/workflows/cd.yml
# Se ejecuta SOLO cuando hay un push a la rama 'main'
# Construye imágenes Docker y despliega en Railway

name: CD — Despliegue Continuo

on:
  push:
    branches: [main]
  # También se puede disparar manualmente desde la UI de GitHub
  workflow_dispatch:

# Solo permitir un deploy a la vez (evitar condiciones de carrera)
concurrency:
  group: production-deploy
  cancel-in-progress: false

jobs:

  # ──────────────────────────────────────────────────────────
  # JOB 1: Build y Push de imágenes Docker
  # ──────────────────────────────────────────────────────────
  build-and-push:
    name: Build y Push Docker Images
    runs-on: ubuntu-latest
    # Necesita que CI haya pasado primero (referencia al workflow de CI)
    # En un monorepo real usarías needs: [ci] con workflow_call

    outputs:
      # Pasar el tag de la imagen al siguiente job
      image-tag: ${{ steps.meta.outputs.version }}

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      # Configurar QEMU para builds multi-arquitectura (arm64 para M1/M2)
      - name: Configurar QEMU
        uses: docker/setup-qemu-action@v3

      # Buildx: herramienta avanzada de build de Docker
      - name: Configurar Docker Buildx
        uses: docker/setup-buildx-action@v3

      # Autenticarse en Docker Hub
      - name: Login en Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_TOKEN }}

      # Generar tags automáticos basados en el commit
      # Ejemplo: main-sha-abc123f, latest
      - name: Generar metadata de la imagen (backend)
        id: meta-backend
        uses: docker/metadata-action@v5
        with:
          images: ${{ secrets.DOCKER_HUB_USERNAME }}/triage-backend
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-sha-
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Generar metadata de la imagen (frontend)
        id: meta-frontend
        uses: docker/metadata-action@v5
        with:
          images: ${{ secrets.DOCKER_HUB_USERNAME }}/triage-frontend
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-sha-
            type=raw,value=latest,enable={{is_default_branch}}

      # Construir y subir la imagen del backend
      - name: Build y Push — triage-backend
        uses: docker/build-push-action@v5
        with:
          context: ./triage-backend
          dockerfile: ./triage-backend/Dockerfile
          push: true
          tags: ${{ steps.meta-backend.outputs.tags }}
          labels: ${{ steps.meta-backend.outputs.labels }}
          # Caché de capas para acelerar builds futuros
          cache-from: type=gha
          cache-to: type=gha,mode=max
          # Build multi-plataforma (linux/amd64 para servidores, linux/arm64 para M-series Mac)
          platforms: linux/amd64,linux/arm64

      # Construir y subir la imagen del frontend
      - name: Build y Push — triage-frontend
        uses: docker/build-push-action@v5
        with:
          context: ./triage-frontend
          dockerfile: ./triage-frontend/Dockerfile
          push: true
          tags: ${{ steps.meta-frontend.outputs.tags }}
          labels: ${{ steps.meta-frontend.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          platforms: linux/amd64,linux/arm64

  # ──────────────────────────────────────────────────────────
  # JOB 2: Desplegar en Railway
  # ──────────────────────────────────────────────────────────
  deploy-to-railway:
    name: Deploy a Railway
    runs-on: ubuntu-latest
    needs: build-and-push
    environment: production    # Requiere aprobación manual si lo configuras

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Instalar Railway CLI
        run: |
          curl -fsSL https://railway.app/install.sh | sh

      - name: Deploy backend en Railway
        env:
          RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}
        run: |
          railway up --service triage-backend --detach

      - name: Verificar deploy
        env:
          RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}
        run: |
          sleep 30   # Esperar a que Railway procese el deploy
          railway status --service triage-backend

  # ──────────────────────────────────────────────────────────
  # JOB 3: Notificación del resultado
  # ──────────────────────────────────────────────────────────
  notify:
    name: Notificar Resultado
    runs-on: ubuntu-latest
    needs: [build-and-push, deploy-to-railway]
    if: always()    # Ejecutar aunque los jobs anteriores fallen

    steps:
      - name: Notificación de éxito
        if: needs.deploy-to-railway.result == 'success'
        run: |
          echo "✅ Despliegue exitoso a producción"
          echo "Backend: https://triage-api.railway.app"

      - name: Notificación de fallo
        if: needs.deploy-to-railway.result == 'failure'
        run: |
          echo "❌ El despliegue falló. Revisar los logs."
          exit 1
```

---

## 12. Despliegue en Railway

Railway es una plataforma de despliegue moderna que simplifica enormemente el proceso. Ofrece una capa gratuita generosa para proyectos de estudiantes.

### Crear cuenta y proyecto en Railway

1. Ve a [https://railway.app](https://railway.app)
2. Haz clic en "Start a New Project"
3. Autentícate con tu cuenta de GitHub (recomendado: simplifica el despliegue)
4. Verás el dashboard de Railway

### Paso 1: Crear el servicio de PostgreSQL en Railway

1. En tu proyecto de Railway, haz clic en **"+ New"**
2. Selecciona **"Database"** → **"Add PostgreSQL"**
3. Railway aprovisiona automáticamente una instancia de PostgreSQL
4. Una vez creada, haz clic en el servicio de PostgreSQL
5. Ve a la pestaña **"Variables"** y copia los valores de:
   - `DATABASE_URL` (URL completa de conexión JDBC)
   - `PGPASSWORD`
   - `PGUSER`
   - `PGDATABASE`

### Paso 2: Crear el servicio del backend en Railway

**Opción A: Desplegar desde Docker Hub (recomendado si tienes CI/CD)**

1. En tu proyecto, haz clic en **"+ New"** → **"Docker Image"**
2. Ingresa: `tu-usuario/triage-backend:latest`
3. Railway descarga y ejecuta la imagen

**Opción B: Desplegar directamente desde GitHub**

1. Haz clic en **"+ New"** → **"GitHub Repo"**
2. Selecciona tu repositorio
3. Configura el directorio raíz como `/triage-backend`
4. Railway detecta el Dockerfile automáticamente y construye la imagen

### Paso 3: Configurar las variables de entorno en Railway

Con el servicio del backend seleccionado, ve a la pestaña **"Variables"** y agrega:

```bash
SPRING_PROFILES_ACTIVE=prod
DATABASE_URL=jdbc:postgresql://<host-de-railway>:<puerto>/<nombre-bd>
DATABASE_USERNAME=<usuario-de-railway>
DATABASE_PASSWORD=<password-de-railway>
JWT_SECRET=<tu-secreto-jwt-super-seguro>
JWT_EXPIRATION=3600000
SWAGGER_ENABLED=false
```

> **Tip:** Railway tiene una función "Reference Variable" que te permite referenciar variables de otros servicios del mismo proyecto. Puedes hacer que `DATABASE_URL` del backend referencie directamente la URL de la base de datos del servicio PostgreSQL.

### Paso 4: Configurar el dominio público

1. En el servicio del backend, ve a la pestaña **"Settings"**
2. En la sección **"Networking"**, haz clic en **"Generate Domain"**
3. Railway genera una URL pública como: `triage-backend-production.up.railway.app`

### Paso 5: Obtener el Railway Token para CI/CD

1. Ve a [https://railway.app/account/tokens](https://railway.app/account/tokens)
2. Haz clic en **"Create Token"**
3. Dale un nombre como `github-actions-triage`
4. Copia el token y guárdalo como secret en GitHub (`RAILWAY_TOKEN`)

### Verificar que el backend está funcionando

```bash
# Probar el health check
curl https://triage-backend-production.up.railway.app/actuator/health
# {"status":"UP"}

# Probar el login
curl -X POST https://triage-backend-production.up.railway.app/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@uniquindio.edu.co","password":"Admin123!"}'
# {"token":"eyJhbGci...","type":"Bearer","expiresIn":3600000}
```

---

## 13. Despliegue del Frontend en Vercel

Vercel es la plataforma perfecta para frontends: tiene integración nativa con Angular, despliega automáticamente en cada push, y tiene un CDN global que sirve tu aplicación súper rápido desde servidores cercanos al usuario.

### Crear cuenta en Vercel

1. Ve a [https://vercel.com](https://vercel.com)
2. Haz clic en **"Sign Up"** → **"Continue with GitHub"**
3. Autoriza a Vercel a acceder a tus repositorios

### Importar el proyecto a Vercel

1. En el dashboard de Vercel, haz clic en **"Add New..."** → **"Project"**
2. Importa tu repositorio de GitHub
3. Vercel detecta que es un proyecto Angular y sugiere la configuración

### Configuración del build en Vercel

Si Vercel no detecta automáticamente la configuración correcta, ajústala manualmente:

```
Framework Preset: Angular
Root Directory: triage-frontend/
Build Command: ng build --configuration production
Output Directory: dist/triage-frontend/browser
Install Command: npm ci
```

### Variables de entorno en Vercel

En la configuración del proyecto en Vercel, ve a **"Environment Variables"** y agrega:

```bash
# Nota: Con Nginx como proxy, la apiUrl en environment.prod.ts es '/api/v1' (relativo)
# y no necesita variables de entorno adicionales para la URL del backend.
# Sin embargo, si tu frontend no usa Nginx proxy, necesitas:
NG_APP_API_URL=https://triage-backend-production.up.railway.app/api/v1
```

> **Importante:** Si estás desplegando el frontend en Vercel Y el backend en Railway (en dominios diferentes), vas a tener problemas de CORS. Hay dos soluciones:
>
> **Opción 1 (recomendada):** Configura CORS en Spring Boot para permitir el dominio de Vercel. Vercel te da una URL fija cuando asignas un dominio personalizado.
>
> **Opción 2:** Usa las [Rewrites de Vercel](https://vercel.com/docs/projects/project-configuration) como proxy para el backend.

### Configurar Vercel como proxy del backend (evitar CORS)

Crea un archivo `vercel.json` en la raíz de `triage-frontend/`:

```json
{
  "rewrites": [
    {
      "source": "/api/:path*",
      "destination": "https://triage-backend-production.up.railway.app/api/:path*"
    }
  ]
}
```

Con esto, las peticiones a `https://triage.vercel.app/api/v1/solicitudes` se redirigen automáticamente a `https://triage-backend-production.up.railway.app/api/v1/solicitudes` sin pasar por el navegador. No hay CORS porque la redirección ocurre en el servidor de Vercel.

### Deploy automático desde GitHub

Una vez configurado el proyecto en Vercel, cada `git push` a la rama `main` dispara automáticamente:

1. Vercel clona el repositorio
2. Ejecuta `npm ci` e instala dependencias
3. Ejecuta `ng build --configuration production`
4. Despliega los archivos estáticos en su CDN global
5. La URL de producción se actualiza en ~30 segundos

Vercel también crea **Preview Deployments** para cada Pull Request, permitiéndote probar los cambios antes de mergear a main.

---

## 14. `ng build` en Producción

### La diferencia entre desarrollo y producción en Angular

Cuando ejecutas `ng serve` (modo desarrollo), Angular:
- No minifica el código (el JS es legible)
- Incluye source maps completos (para depuración)
- No hace tree-shaking agresivo
- Activa el modo de detección de cambios extra-estricto para debugging
- El bundle puede pesar 5-10MB

Cuando ejecutas `ng build --configuration production` (modo producción), Angular:
- Minifica y uglifica todo el código (ilegible pero pequeño)
- Comprime con AOT (Ahead-of-Time compilation): el template HTML se convierte a TypeScript en tiempo de build
- Aplica tree-shaking: elimina el código que no se usa
- Usa hashing de archivos para cache-busting
- El bundle típicamente pesa 200-500KB

### Configurar el build de producción en `angular.json`

```json
{
  "projects": {
    "triage-frontend": {
      "architect": {
        "build": {
          "configurations": {
            "production": {
              "budgets": [
                {
                  "type": "initial",
                  "maximumWarning": "500kB",
                  "maximumError": "1MB"
                },
                {
                  "type": "anyComponentStyle",
                  "maximumWarning": "4kB",
                  "maximumError": "8kB"
                }
              ],
              "outputHashing": "all",
              "optimization": true,
              "sourceMap": false,
              "namedChunks": false,
              "extractLicenses": true,
              "buildOptimizer": true,
              "fileReplacements": [
                {
                  "replace": "src/environments/environment.ts",
                  "with": "src/environments/environment.prod.ts"
                }
              ]
            }
          }
        }
      }
    }
  }
}
```

### Analizar el tamaño del bundle

Una herramienta muy útil para identificar qué librerías están inflando el bundle:

```bash
# Instalar el analizador de bundles de Angular
npm install -g webpack-bundle-analyzer

# Generar el reporte de estadísticas durante el build
ng build --configuration production --stats-json

# Abrir el analizador visual
npx webpack-bundle-analyzer dist/triage-frontend/browser/stats.json
```

Esto abre una visualización interactiva donde puedes ver exactamente qué está ocupando espacio en tu bundle.

### Comandos de build útiles

```bash
# Build de producción estándar
ng build --configuration production

# Build con reporte de presupuesto de tamaño
ng build --configuration production --verbose

# Build para un entorno específico (si tienes múltiples entornos)
ng build --configuration staging

# Previsualizar el build de producción localmente
npm install -g http-server
ng build --configuration production
http-server dist/triage-frontend/browser -p 4200
# Ahora puedes probar en http://localhost:4200

# Verificar que el routing funciona en modo producción local
# (ngx-quicklink o el servidor debe manejar SPA routing)
http-server dist/triage-frontend/browser -p 4200 --proxy http://localhost:4200?
```

---

## 15. Checklist Completo Pre-Despliegue

Antes de hacer tu primer despliegue a producción, verifica cada punto de esta lista:

### Seguridad

```
[ ] El JWT_SECRET tiene al menos 64 bytes de longitud
[ ] El JWT_SECRET fue generado con openssl rand (no inventado a mano)
[ ] Las contraseñas de BD usan caracteres especiales y son largas
[ ] El .env NUNCA fue commiteado a Git (verificar con: git log --all -- .env)
[ ] Swagger/OpenAPI está DESHABILITADO en el perfil de producción
[ ] application-prod.yml usa variables de entorno (${VAR}) en lugar de valores hardcodeados
[ ] spring.jpa.hibernate.ddl-auto está en 'validate' o 'none' en producción (NUNCA 'create')
[ ] Los endpoints de Actuator exponen SOLO /health (no /env, /beans, /mappings)
[ ] Nginx tiene los headers de seguridad configurados (X-Frame-Options, etc.)
[ ] Los contenedores NO corren como root
[ ] Las imágenes Docker usan tags específicos, no latest en producción
```

### CORS en producción

```
[ ] Spring Boot tiene configurado exactamente el dominio de Vercel en allowedOrigins
[ ] NO hay allowedOrigins con "*" en el perfil de producción
[ ] Se probó hacer una petición desde el dominio de Vercel al backend de Railway
[ ] Los métodos HTTP permitidos coinciden con lo que Angular usa (GET, POST, PUT, PATCH, DELETE)
[ ] Se incluye el header Authorization en allowedHeaders
```

Configuración de CORS en Spring Boot para producción:

```java
// src/main/java/co/edu/uniquindio/triage/config/CorsConfig.java
@Configuration
public class CorsConfig {

    @Value("${app.cors.allowed-origins}")
    private List<String> allowedOrigins;

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        var configuration = new CorsConfiguration();

        // En producción, esto viene de la variable de entorno
        // APP_CORS_ALLOWED_ORIGINS=https://triage.vercel.app
        configuration.setAllowedOrigins(allowedOrigins);
        configuration.setAllowedMethods(List.of("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"));
        configuration.setAllowedHeaders(List.of("Authorization", "Content-Type", "X-Requested-With"));
        configuration.setExposedHeaders(List.of("Authorization"));
        configuration.setAllowCredentials(true);
        configuration.setMaxAge(3600L);

        var source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", configuration);
        return source;
    }
}
```

```yaml
# En application-prod.yml
app:
  cors:
    allowed-origins: ${CORS_ALLOWED_ORIGINS}

# Variable de entorno en Railway:
# CORS_ALLOWED_ORIGINS=https://triage.vercel.app
```

### Variables de entorno

```
[ ] Todas las variables de entorno están configuradas en Railway
[ ] DATABASE_URL usa el host interno de Railway (no localhost)
[ ] JWT_SECRET está configurado en Railway
[ ] SPRING_PROFILES_ACTIVE=prod está configurado en Railway
[ ] CORS_ALLOWED_ORIGINS tiene el dominio exacto de Vercel (con https://)
[ ] En Vercel, las variables de entorno del frontend están configuradas
```

### Base de datos

```
[ ] Flyway está habilitado y los scripts de migración están en classpath:db/migration
[ ] Los scripts de migración son inmutables (nunca modificar un script ya aplicado)
[ ] Se tiene un script de respaldo de la base de datos
[ ] Se probó que la BD se restaura correctamente desde el respaldo
```

### Monitoring y logs

```
[ ] Se puede acceder a los logs del backend en Railway Dashboard
[ ] El endpoint /actuator/health responde con {"status":"UP"}
[ ] Se tiene algún mecanismo de alerta si el servicio cae (Railway tiene alertas básicas)
[ ] Los logs de errores son claros y contienen suficiente información para debug
```

### Testing antes del deploy

```
[ ] Todos los tests unitarios pasan localmente (mvn test)
[ ] Se probó el flujo completo: registro → login → crear solicitud → clasificar → cerrar
[ ] Se probó la aplicación en modo producción local (docker compose up)
[ ] Se verificó que la paginación de solicitudes funciona correctamente
[ ] Se verificó la autorización por roles (ESTUDIANTE no puede clasificar)
[ ] Se probó con múltiples usuarios simultáneos
```

### Performance

```
[ ] ng build --configuration production genera un bundle < 1MB
[ ] Las imágenes en el frontend están optimizadas (WebP cuando sea posible)
[ ] Nginx tiene compresión gzip habilitada
[ ] Spring Boot HikariCP tiene pool de conexiones configurado correctamente
[ ] Las consultas JPA frecuentes tienen índices en la BD
```

---

## 16. Errores Comunes y Troubleshooting

### Error 1: `docker: Cannot connect to the Docker daemon`

**Síntoma:** Al ejecutar cualquier comando `docker`, aparece este error.

**Causa:** Docker Desktop no está corriendo.

**Solución:**
```bash
# En macOS/Windows: Abrir Docker Desktop desde el menú de aplicaciones
# En Linux:
sudo systemctl start docker
sudo systemctl enable docker  # Para que inicie automáticamente

# Verificar
docker ps
```

---

### Error 2: El contenedor de Spring Boot no inicia: `Connection refused` a PostgreSQL

**Síntoma:** El backend inicia pero inmediatamente falla con `Connection to localhost:5432 refused`.

**Causa:** El backend está intentando conectarse a `localhost`, pero dentro de un contenedor Docker, `localhost` es el propio contenedor, no la máquina anfitriona ni otro contenedor.

**Solución:**
```yaml
# INCORRECTO en docker-compose.yml:
DATABASE_URL: jdbc:postgresql://localhost:5432/triage_db

# CORRECTO — usar el nombre del servicio de postgres:
DATABASE_URL: jdbc:postgresql://postgres:5432/triage_db
#                               ↑ nombre del servicio en docker-compose.yml

# Si postgres corre fuera de Docker (en tu máquina local):
DATABASE_URL: jdbc:postgresql://host.docker.internal:5432/triage_db
#                               ↑ dirección especial que Docker resuelve al host
```

---

### Error 3: CORS bloqueado cuando el frontend está en Vercel y el backend en Railway

**Síntoma:** En la consola del navegador: `Access to XMLHttpRequest at 'https://triage-api.railway.app/api/v1/solicitudes' from origin 'https://triage.vercel.app' has been blocked by CORS policy`.

**Causa común 1:** El `allowedOrigins` en Spring Boot no incluye exactamente el dominio de Vercel.

**Causa común 2:** Spring Security está interfiriendo con la configuración de CORS. La configuración de CORS debe estar en la cadena de filtros de Security.

**Solución correcta:**
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
        // ¡CORS debe ser lo PRIMERO antes que cualquier otra configuración de seguridad!
        .cors(cors -> cors.configurationSource(corsConfigurationSource()))
        .csrf(csrf -> csrf.disable())
        // ... resto de la configuración
        .build();
}
```

**Verificación rápida:** Abre las DevTools del navegador → Red → haz clic en la petición fallida → Headers → Response Headers. Busca el header `Access-Control-Allow-Origin`. Si no está, el problema es en el backend. Si está pero tiene el dominio equivocado, revisa la variable de entorno `CORS_ALLOWED_ORIGINS` en Railway.

---

### Error 4: Angular SPA — 404 al recargar la página en producción

**Síntoma:** Al navegar a `https://triage.vercel.app/solicitudes/123` directamente (o al recargar F5), aparece un 404.

**Causa:** El servidor busca un archivo `/solicitudes/123/index.html` que no existe. El routing de Angular solo funciona cuando JavaScript ya está cargado.

**Solución en Nginx:**
```nginx
location / {
    try_files $uri $uri/ /index.html;
    #                    ↑ Si el archivo no existe, servir index.html
}
```

**Solución en Vercel (`vercel.json`):**
```json
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ]
}
```

---

### Error 5: Imagen de Docker demasiado grande

**Síntoma:** La imagen del backend pesa 800MB o más.

**Causa:** No se usó multi-stage build, o se usó una imagen base innecesariamente grande.

**Diagnóstico:**
```bash
# Ver el historial de capas de la imagen
docker history triage-backend:1.0

# Ver el tamaño de cada capa
docker image inspect triage-backend:1.0 --format='{{json .RootFS.Layers}}'
```

**Solución:** Asegurarte de usar el multi-stage build correctamente:
```dockerfile
# Etapa 1: Build con Maven (~500MB, DESCARTADA)
FROM maven:3.9-eclipse-temurin-21-alpine AS build
# ... compilar ...

# Etapa 2: Solo JRE + JAR (~150MB, IMAGEN FINAL)
FROM eclipse-temurin:21-jre-alpine AS runtime
COPY --from=build /app/target/*.jar triage-backend.jar
# El FROM en la última etapa determina el tamaño de la imagen final
```

---

### Error 6: `flyway.migrate()` falla en producción

**Síntoma:** Spring Boot inicia pero falla con `Flyway found non-empty schema without schema history table`.

**Causa:** La base de datos ya tiene tablas (creadas por Hibernate en un ambiente anterior) pero no tiene la tabla de historial de Flyway (`flyway_schema_history`).

**Solución:**
```yaml
# application-prod.yml
spring:
  flyway:
    enabled: true
    baseline-on-migrate: true    # Marca el estado actual como baseline
    baseline-version: 0          # Versión del baseline
```

---

### Error 7: GitHub Actions falla con `Permission denied` al hacer push a Docker Hub

**Síntoma:** El workflow falla en el paso "Build y Push" con `unauthorized: authentication required`.

**Causa:** El token de Docker Hub no tiene permisos de escritura, o el secret no está configurado correctamente.

**Solución:**
1. Ve a Docker Hub → Account Settings → Security → Access Tokens
2. Crea un nuevo token con permisos "Read, Write, Delete"
3. En GitHub → Settings → Secrets → Actions → actualiza el secret `DOCKER_HUB_TOKEN`

---

### Error 8: `OOMKilled` — el contenedor se detiene por falta de memoria

**Síntoma:** El contenedor de Spring Boot se detiene inesperadamente. `docker inspect <container>` muestra `"OOMKilled": true`.

**Causa:** Spring Boot está usando más memoria de la asignada al contenedor.

**Solución:**

```dockerfile
# En el ENTRYPOINT del Dockerfile, configurar la JVM para respetar los límites del contenedor
ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \          # Usar los límites del contenedor
  "-XX:MaxRAMPercentage=75.0", \         # Usar máximo 75% de la RAM disponible
  "-jar", "triage-backend.jar"]
```

```yaml
# En docker-compose.yml, asignar memoria suficiente
services:
  triage-backend:
    deploy:
      resources:
        limits:
          memory: 512M    # Mínimo recomendado para Spring Boot en producción
        reservations:
          memory: 256M
```

---

## 17. Resumen y Cheat Sheet

### Docker — Comandos esenciales

```bash
# ─── IMÁGENES ───────────────────────────────────────────────
docker images                           # Listar imágenes locales
docker pull ubuntu:24.04                # Descargar imagen
docker build -t nombre:tag .            # Construir imagen
docker rmi nombre:tag                   # Eliminar imagen
docker push nombre:tag                  # Subir imagen al registry

# ─── CONTENEDORES ───────────────────────────────────────────
docker run -d -p 8080:8080 nombre:tag   # Ejecutar en background
docker ps                               # Contenedores corriendo
docker ps -a                            # Todos los contenedores
docker stop nombre                      # Detener contenedor
docker start nombre                     # Iniciar contenedor detenido
docker rm nombre                        # Eliminar contenedor
docker logs -f nombre                   # Ver logs en tiempo real
docker exec -it nombre bash             # Terminal dentro del contenedor

# ─── DOCKER COMPOSE ─────────────────────────────────────────
docker compose up -d                    # Levantar servicios
docker compose up -d --build            # Reconstruir y levantar
docker compose down                     # Detener y eliminar contenedores
docker compose down -v                  # + eliminar volúmenes
docker compose logs -f                  # Logs de todos los servicios
docker compose logs -f backend          # Logs de un servicio
docker compose ps                       # Estado de los servicios
docker compose exec backend bash        # Terminal en un servicio

# ─── LIMPIEZA ───────────────────────────────────────────────
docker system prune                     # Eliminar todo lo no usado
docker volume prune                     # Eliminar volúmenes no usados
docker image prune                      # Eliminar imágenes colgadas
```

### Estructura del `docker-compose.yml`

```yaml
name: mi-proyecto

services:
  mi-servicio:
    image: imagen:tag              # Usar imagen existente
    build: ./carpeta              # O construir desde Dockerfile
    container_name: nombre-fijo
    restart: unless-stopped
    environment:
      VARIABLE: ${VAR_DE_ENV}
    ports:
      - "host:contenedor"
    volumes:
      - volumen_nombrado:/ruta/en/contenedor
    depends_on:
      otro-servicio:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "comando-de-verificacion"]
      interval: 10s
      timeout: 5s
      retries: 3
    networks:
      - mi-red

volumes:
  volumen_nombrado:

networks:
  mi-red:
    driver: bridge
```

### Estructura de Dockerfile (multi-stage)

```dockerfile
# Etapa de build
FROM imagen-con-herramientas AS build
WORKDIR /app
COPY archivos-de-config .
RUN instalar-dependencias
COPY . .
RUN compilar

# Etapa de runtime
FROM imagen-minima AS runtime
WORKDIR /app
COPY --from=build /app/artefacto ./artefacto
EXPOSE puerto
USER usuario-no-root
ENTRYPOINT ["comando", "para", "iniciar"]
```

### GitHub Actions — estructura básica

```yaml
name: Mi Pipeline
on:
  push:
    branches: [main]

jobs:
  mi-job:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Mi paso
        run: echo "Hola CI/CD"
        env:
          MI_SECRET: ${{ secrets.MI_SECRET }}
```

### Comparativa de plataformas de despliegue

| Característica | Railway | Render | Fly.io | Heroku |
|---|---|---|---|---|
| **Capa gratuita** | $5 crédito/mes | Sí (con limitaciones) | Sí | No |
| **PostgreSQL** | ✅ Gestionado | ✅ Gestionado | ✅ | ✅ |
| **Deploy desde Docker Hub** | ✅ | ✅ | ✅ | ✅ |
| **Deploy desde GitHub** | ✅ | ✅ | Partial | ✅ |
| **Facilidad de uso** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Ideal para** | Proyectos estudiantiles | APIs y servicios | Apps de producción | Proyectos maduros |

---

## 18. Ejercicios Prácticos

### Reto 1 — Básico: Dockerizar el Backend

Crea el `Dockerfile` multi-stage para el backend del Triage y verifica que funciona:

1. Crea el `Dockerfile` y `.dockerignore` en `triage-backend/`
2. Crea `application-prod.yml` con todas las configuraciones
3. Construye la imagen: `docker build -t triage-backend:local .`
4. Levanta un PostgreSQL con Docker
5. Ejecuta el contenedor del backend apuntando al PostgreSQL
6. Verifica que `/actuator/health` responde `{"status":"UP"}`

**Criterio de éxito:** La imagen pesa menos de 300MB y el backend inicia en menos de 30 segundos.

---

### Reto 2 — Intermedio: Docker Compose completo

Crea el `docker-compose.yml` que levante todo el sistema con un solo comando:

1. Crea el `docker-compose.yml` con PostgreSQL, backend y frontend
2. Crea el `docker-compose.dev.yml` con pgAdmin y puertos adicionales
3. Crea `.env.example` y `.env` con todas las variables
4. Ejecuta: `docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d --build`
5. Verifica que:
   - Puedes acceder al frontend en `http://localhost`
   - El login funciona desde el frontend
   - Puedes crear una solicitud desde el frontend
   - pgAdmin en `http://localhost:5050` muestra la base de datos

**Criterio de éxito:** Cualquier persona con Docker Desktop puede levantar el sistema completo en menos de 5 minutos ejecutando solo 2 comandos: `cp .env.example .env` y `docker compose up -d --build`.

---

### Reto 3 — Avanzado: Pipeline CI/CD con GitHub Actions

Configura el pipeline completo:

1. Crea `.github/workflows/ci.yml` que ejecute los tests en cada push
2. Crea una cuenta en Docker Hub y genera un token
3. Configura los secrets en tu repositorio de GitHub
4. Crea `.github/workflows/cd.yml` que construya y suba las imágenes a Docker Hub
5. Haz un push a `main` y verifica que el pipeline se ejecuta correctamente en GitHub → Actions
6. Verifica que la imagen aparece en Docker Hub

**Criterio de éxito:** Al hacer `git push origin main`, el pipeline se ejecuta automáticamente, los tests pasan, y la imagen queda disponible en Docker Hub.

---

### Reto 4 — Despliegue Real

Despliega el Sistema de Triage en producción:

1. Crea un proyecto en Railway con PostgreSQL y el backend
2. Configura todas las variables de entorno en Railway
3. Despliega el frontend en Vercel conectado a tu repositorio de GitHub
4. Configura el proxy reverso (Vercel rewrites o Nginx proxy_pass) para evitar CORS
5. Verifica el flujo completo en producción:
   - Registro de usuario
   - Login con JWT
   - Crear solicitud académica
   - Clasificar y priorizar (como FUNCIONARIO)
   - Consultar historial

**Criterio de éxito:** La URL pública de Vercel funciona correctamente y cualquier persona con un navegador puede usar el Sistema de Triage.

---

### Reto 5 — Extra: Monitoreo básico

Agrega monitoreo básico al sistema:

1. Configura Spring Boot Actuator con métricas de Prometheus:
   ```xml
   <dependency>
     <groupId>io.micrometer</groupId>
     <artifactId>micrometer-registry-prometheus</artifactId>
   </dependency>
   ```
2. Agrega un servicio de Prometheus al `docker-compose.yml`
3. Agrega Grafana al `docker-compose.yml` con un dashboard básico
4. Crea un dashboard que muestre:
   - Número de solicitudes HTTP por segundo
   - Tiempo de respuesta promedio
   - Uso de memoria del backend

---

## 19. Referencias y Recursos Adicionales

### Documentación oficial

- **Docker:** [docs.docker.com](https://docs.docker.com) — La documentación oficial más completa
- **Docker Compose:** [docs.docker.com/compose/](https://docs.docker.com/compose/)
- **GitHub Actions:** [docs.github.com/en/actions](https://docs.github.com/en/actions)
- **Railway:** [docs.railway.app](https://docs.railway.app)
- **Vercel:** [vercel.com/docs](https://vercel.com/docs)
- **Nginx:** [nginx.org/en/docs/](https://nginx.org/en/docs/)

### Libros y recursos de estudio

- **"Docker Deep Dive"** — Nigel Poulton (2023) — El mejor libro introductorio a Docker
- **"Kubernetes in Action"** — Marko Lukša — Para cuando quieras escalar más allá de Docker Compose
- **"The DevOps Handbook"** — Kim, Humble, Debois, Willis — Fundamentos de cultura DevOps

### Herramientas complementarias

- **[Play with Docker](https://labs.play-with-docker.com/)** — Practica Docker en el navegador sin instalación
- **[Docker Hub](https://hub.docker.com)** — Registry oficial de imágenes
- **[Dive](https://github.com/wagoodman/dive)** — Herramienta para inspeccionar capas de imágenes Docker y optimizar su tamaño
- **[DBeaver](https://dbeaver.io)** — Cliente de base de datos para conectarte a PostgreSQL en Docker

### Canales de aprendizaje

- **TechWorld with Nana** (YouTube) — Tutoriales de DevOps y Docker en inglés, muy didácticos
- **Pelado Nerd** (YouTube) — Contenido de DevOps en español latinoamericano
- **Fireship** (YouTube) — Videos cortos y concisos sobre tecnología moderna

---

> **Autor:** José Alfredo Ramírez Espinosa  
> **Materia:** Programación Avanzada  
> **Programa:** Ingeniería de Sistemas y Computación  
> **Universidad del Quindío** | Armenia, Colombia  
> **Año:** 2026  
> *Material de uso académico. Todos los derechos reservados al autor.*
