# Guía 12 — Integración con IA (LLM) desde Spring Boot

> **Materia:** Programación Avanzada | **Programa:** Ingeniería de Sistemas y Computación | **Universidad del Quindío**  
> **Semestre:** 6to | **Núcleo temático:** Servicios de Negocio — Valor Agregado con IA  
> **Guía:** 12 de 12 | **Stack:** Java 21 · Spring Boot 3.4+ · Spring AI · Angular 19+ · OpenAI / Anthropic Claude

---

## Tabla de Contenidos

1. [Objetivos de aprendizaje](#1-objetivos-de-aprendizaje)
2. [Prerrequisitos](#2-prerrequisitos)
3. [¿Qué es un LLM y cómo funciona?](#3-qué-es-un-llm-y-cómo-funciona)
4. [APIs de LLM disponibles en 2026](#4-apis-de-llm-disponibles-en-2026)
5. [Spring AI: el módulo oficial de Spring](#5-spring-ai-el-módulo-oficial-de-spring)
6. [Configurar Spring AI en el proyecto de Triage](#6-configurar-spring-ai-en-el-proyecto-de-triage)
7. [Patrón de diseño: IA como servicio opcional](#7-patrón-de-diseño-ia-como-servicio-opcional)
8. [Implementar RF-10: Sugerencia de clasificación y prioridad](#8-implementar-rf-10-sugerencia-de-clasificación-y-prioridad)
9. [Implementar RF-09: Resumen del historial de una solicitud](#9-implementar-rf-09-resumen-del-historial-de-una-solicitud)
10. [RF-11: Garantizar funcionamiento sin IA](#10-rf-11-garantizar-funcionamiento-sin-ia)
11. [Endpoints REST para las funcionalidades de IA](#11-endpoints-rest-para-las-funcionalidades-de-ia)
12. [Manejo de errores, timeouts y resiliencia](#12-manejo-de-errores-timeouts-y-resiliencia)
13. [Angular: UI para interactuar con la IA](#13-angular-ui-para-interactuar-con-la-ia)
14. [Costos, privacidad y consideraciones éticas](#14-costos-privacidad-y-consideraciones-éticas)
15. [Errores comunes y troubleshooting](#15-errores-comunes-y-troubleshooting)
16. [Resumen y cheat sheet](#16-resumen-y-cheat-sheet)
17. [Ejercicios prácticos](#17-ejercicios-prácticos)
18. [Referencias y recursos adicionales](#18-referencias-y-recursos-adicionales)

---

## 1. Objetivos de aprendizaje

Al finalizar esta guía, serás capaz de:

- Explicar qué es un LLM, cómo funciona a alto nivel, y cuándo tiene sentido usarlo en una aplicación empresarial.
- Integrar Spring AI en un proyecto Spring Boot 3.4+ y conectarlo con APIs de LLM como OpenAI o Anthropic Claude.
- Implementar los requisitos funcionales RF-09 y RF-10 del proyecto de Triage de forma desacoplada y opcional.
- Aplicar el patrón **Strategy + Feature Flag** para que la IA sea un componente intercambiable y el sistema no dependa de ella (RF-11).
- Manejar errores, timeouts y fallos de APIs externas con resiliencia adecuada.
- Construir en Angular el flujo de "sugerir con IA → revisar → confirmar o editar" respetando que la decisión final siempre la toma un humano.
- Razonar sobre los aspectos éticos, de costo y de privacidad al integrar modelos de lenguaje en sistemas reales.

---

## 2. Prerrequisitos

Antes de trabajar esta guía debes tener claros los contenidos de las guías anteriores, en especial:

- **Guía 04** — Spring Boot: inyección de dependencias, anotaciones, configuración con `application.yml`.
- **Guía 05** — JPA/Hibernate: las entidades `Solicitud`, `HistorialSolicitud`, `TipoSolicitud`, `Prioridad`.
- **Guía 06** — API REST: `@RestController`, `ResponseEntity`, DTOs con Records, manejo global de errores.
- **Guía 07** — Spring Security: autenticación con JWT, autorización por roles.
- **Guía 09** — Angular: servicios HTTP, interceptores, formularios reactivos.

Además necesitas:
- Una API key de OpenAI (https://platform.openai.com) **o** de Anthropic (https://console.anthropic.com). Ambas tienen créditos gratuitos de prueba.
- El proyecto de Triage funcionando con las guías 04 a 07 ya implementadas.

---

## 3. ¿Qué es un LLM y cómo funciona?

### 3.1 La analogía del empleado que leyó todo internet

Imagina que contratas un empleado que durante años leyó miles de millones de documentos: libros, artículos, código fuente, foros, manuales académicos. Después de esa lectura, ese empleado puede responder preguntas, redactar textos, clasificar información o resumir documentos, simplemente porque ha visto tantos ejemplos similares que reconoce patrones.

Un **LLM (Large Language Model)** es exactamente eso: un modelo estadístico masivo entrenado sobre gigantescas cantidades de texto, que aprendió a predecir cuál es el siguiente token (fragmento de palabra) más probable dado un contexto. El resultado es que puede generar texto que parece comprensión real del lenguaje.

Lo que **no es** un LLM:
- No es una base de datos con respuestas precargadas.
- No "sabe" las cosas como un humano las sabe.
- No tiene memoria entre conversaciones independientes (a menos que se la des tú explícitamente).
- No es determinístico: la misma pregunta puede dar respuestas ligeramente distintas.

### 3.2 Conceptos clave que necesitas entender

| Concepto | Definición simple |
|---|---|
| **Token** | Fragmento de texto. Aproximadamente 1 token ≈ 0.75 palabras en español. |
| **Prompt** | El texto de entrada que le envías al modelo. |
| **Completion / Response** | El texto que genera el modelo como respuesta. |
| **Context window** | La cantidad máxima de tokens que el modelo puede procesar de una sola vez (prompt + respuesta). GPT-4o tiene ~128k tokens. |
| **Temperature** | Controla cuánta "creatividad" o variabilidad hay en la respuesta. 0 = determinístico. 1 = muy creativo. Para clasificación usamos valores bajos (0.1–0.3). |
| **System prompt** | Instrucciones previas que le das al modelo para que adopte un rol o siga reglas específicas. |
| **Few-shot prompting** | Incluir ejemplos en el prompt para que el modelo entienda el patrón de respuesta esperado. |
| **Streaming** | Recibir la respuesta token por token en tiempo real, en lugar de esperar toda la respuesta al final. |

### 3.3 El flujo de una petición a un LLM

```
Tu aplicación
     │
     │  HTTP POST con JSON
     │  {
     │    "model": "gpt-4o-mini",
     │    "messages": [
     │      {"role": "system", "content": "Eres un clasificador de solicitudes académicas..."},
     │      {"role": "user",   "content": "El estudiante dice: Quiero cambiar materias..."}
     │    ],
     │    "temperature": 0.2
     │  }
     ▼
  API del LLM (OpenAI / Anthropic / etc.)
     │
     │  HTTP 200 con JSON
     │  {
     │    "choices": [{
     │      "message": {
     │        "role": "assistant",
     │        "content": "{\"tipo\": \"CANCELACION_ASIGNATURAS\", \"prioridad\": \"MEDIA\"}"
     │      }
     │    }]
     │  }
     ▼
Tu aplicación procesa la respuesta
```

La complejidad real de trabajar con LLMs no está en la petición HTTP (eso es sencillo), sino en:

1. **Diseñar buenos prompts** que produzcan respuestas consistentes y parseables.
2. **Manejar respuestas inconsistentes**: el modelo a veces no sigue el formato que pediste.
3. **Controlar costos**: cada token cuesta dinero.
4. **Mantener el sistema funcional cuando la API falla** (RF-11).

---

## 4. APIs de LLM disponibles en 2026

Aquí tienes las principales opciones que puedes usar:

| Proveedor | Modelos populares | Fortalezas | Precio aproximado (2026) |
|---|---|---|---|
| **OpenAI** | GPT-4o, GPT-4o-mini, o1 | Ecosistema maduro, buena documentación, Spring AI lo soporta nativamente | GPT-4o-mini: muy económico. GPT-4o: medio. o1: costoso. |
| **Anthropic** | Claude 3.5 Haiku, Claude Sonnet 4, Claude Opus 4 | Excelente para instrucciones largas y texto estructurado, muy confiable con JSON | Haiku: económico. Sonnet: medio. Opus: costoso. |
| **Google** | Gemini 2.0 Flash, Gemini 2.5 Pro | Integración con Google Cloud, gran context window | Flash: económico. Pro: medio-alto. |
| **Modelos open-source** | Llama 3, Mistral, Phi-4 | Sin costo de API, privacidad total, corren on-premise | Infraestructura propia (RAM/GPU) |
| **Ollama (local)** | Cualquier modelo open-source | 100% local, sin costo, sin red | Solo hardware local |

### ¿Cuál usar en el Triage?

Para este proyecto académico te recomendamos en orden de conveniencia:

1. **OpenAI GPT-4o-mini** — Muy económico (fracciones de centavo por petición), soportado por Spring AI de forma nativa, y suficientemente capaz para clasificación de texto corto.
2. **Anthropic Claude 3.5 Haiku** — Excelente calidad para instrucciones estructuradas, también soportado por Spring AI.
3. **Ollama con Llama 3** — Si no quieres gastar nada y tienes una máquina con 8+ GB de RAM libre.

En esta guía usaremos **OpenAI** como proveedor principal y mostraremos cómo cambiar a Anthropic con un mínimo de cambios, lo que demostrará exactamente por qué el patrón de diseño que implementaremos es tan valioso.

---

## 5. Spring AI: el módulo oficial de Spring

### 5.1 ¿Qué es Spring AI?

Spring AI es el módulo del ecosistema Spring que te da una capa de abstracción sobre múltiples proveedores de LLMs. En lugar de hacer peticiones HTTP manuales a cada API y parsear sus respuestas con formatos distintos, Spring AI te da interfaces Java unificadas como `ChatClient`, `ChatModel` y `PromptTemplate`.

La analogía: así como Spring Data JPA te abstrae de si usas PostgreSQL, MySQL u Oracle, Spring AI te abstrae de si usas OpenAI, Anthropic o Google.

```
Tu código Java (usa interfaces de Spring AI)
            │
            ▼
    Spring AI (abstracción)
     │          │          │
     ▼          ▼          ▼
  OpenAI   Anthropic   Ollama/Local
```

### 5.2 Componentes principales de Spring AI que usaremos

| Componente | Para qué sirve |
|---|---|
| `ChatClient` | El cliente principal. Construyes peticiones de chat fluentemente. |
| `ChatModel` | La interfaz de bajo nivel que representa el modelo de lenguaje. |
| `Prompt` | Encapsula los mensajes que envías al modelo. |
| `SystemMessage` | El system prompt (instrucciones de rol/comportamiento). |
| `UserMessage` | El mensaje del usuario. |
| `BeanOutputConverter<T>` | Convierte automáticamente la respuesta del LLM a un Record/POJO Java. |
| `PromptTemplate` | Template con variables para construir prompts dinámicos. |

---

## 6. Configurar Spring AI en el proyecto de Triage

### 6.1 Estructura de carpetas actualizada

Antes de agregar código, veamos dónde vivirá todo lo relacionado con la IA:

```
triage-backend/
├── src/
│   └── main/
│       ├── java/
│       │   └── co/uniquindio/triage/
│       │       ├── ai/                         ← NUEVO: todo lo de IA aquí
│       │       │   ├── config/
│       │       │   │   └── AiConfig.java
│       │       │   ├── dto/
│       │       │   │   ├── SugerenciaClasificacionResponse.java
│       │       │   │   └── ResumenSolicitudResponse.java
│       │       │   ├── port/
│       │       │   │   └── AsistenteAcademicoPort.java   ← interfaz (puerto)
│       │       │   └── adapter/
│       │       │       ├── SpringAiAsistenteAdapter.java ← implementación con Spring AI
│       │       │       └── NoOpAsistenteAdapter.java     ← implementación vacía (sin IA)
│       │       ├── controller/
│       │       │   ├── SolicitudController.java
│       │       │   └── AsistenteController.java          ← NUEVO
│       │       ├── service/
│       │       │   └── AsistenteService.java             ← NUEVO
│       │       ├── domain/
│       │       │   └── ... (entidades existentes)
│       │       └── TriageApplication.java
│       └── resources/
│           ├── application.yml
│           └── prompts/                         ← NUEVO: templates de prompts
│               ├── clasificar-solicitud.st
│               └── resumir-historial.st
```

### 6.2 Agregar dependencias en `pom.xml`

```xml
<!-- pom.xml — agregar dentro de <dependencies> -->

<!-- Spring AI BOM (Bill of Materials) — gestiona versiones de todos los módulos -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>1.0.0</version>  <!-- versión estable 2026 -->
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- ...tus dependencias existentes... -->

    <!-- Spring AI con OpenAI -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
        <!-- la versión la gestiona el BOM -->
    </dependency>

    <!-- OPCIONAL: si también quieres usar Anthropic -->
    <!--
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-anthropic-spring-boot-starter</artifactId>
    </dependency>
    -->

    <!-- OPCIONAL: si quieres usar Ollama local -->
    <!--
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-ollama-spring-boot-starter</artifactId>
    </dependency>
    -->
</dependencies>
```

**Importante:** Para agregar el BOM de Spring AI necesitas también configurar el repositorio de Spring Milestones en tu `pom.xml` si usas versiones milestone (1.0.0-M6, etc.). Si la versión `1.0.0` ya está en Maven Central, no necesitas nada adicional. Verifica en https://spring.io/projects/spring-ai cuál es la última versión estable.

```xml
<!-- Solo si necesitas repositorios adicionales (versiones milestone) -->
<repositories>
    <repository>
        <id>spring-milestones</id>
        <name>Spring Milestones</name>
        <url>https://repo.spring.io/milestone</url>
        <snapshots><enabled>false</enabled></snapshots>
    </repository>
</repositories>
```

### 6.3 Configurar `application.yml`

Ahora vamos a configurar Spring AI. La configuración se divide en dos partes: la configuración base del proveedor y los feature flags que controlan si la IA está habilitada.

```yaml
# application.yml

spring:
  application:
    name: triage-backend

  # --- Configuración de Spring AI ---
  ai:
    openai:
      api-key: ${OPENAI_API_KEY:}          # Leer de variable de entorno. Vacío si no está definida.
      chat:
        options:
          model: gpt-4o-mini              # El modelo más económico de OpenAI
          temperature: 0.2                # Respuestas más determinísticas (bueno para clasificación)
          max-tokens: 500                 # Límite de tokens en la respuesta

# --- Feature flags del sistema de IA (propios del Triage) ---
triage:
  ai:
    enabled: ${TRIAGE_AI_ENABLED:false}   # Por defecto DESHABILITADO (RF-11)
    provider: openai                      # openai | anthropic | ollama | none
    timeout-seconds: 10                   # Timeout máximo para peticiones a la API

---
# application-dev.yml — sobrescribe para desarrollo local
spring:
  ai:
    openai:
      api-key: ${OPENAI_API_KEY:}

triage:
  ai:
    enabled: true    # En dev lo habilitamos si la API key está presente
```

**¿Por qué `enabled: false` por defecto?**

Porque el RF-11 establece que el sistema debe funcionar perfectamente sin IA. Si alguien clona el repositorio y no tiene una API key de OpenAI, el sistema no debe fallar al arrancar. El feature flag resuelve exactamente eso.

### 6.4 Clase de propiedades personalizadas

Vamos a mapear la sección `triage.ai` del YAML a una clase Java tipada:

```java
// src/main/java/co/uniquindio/triage/ai/config/AiProperties.java

package co.uniquindio.triage.ai.config;

import org.springframework.boot.context.properties.ConfigurationProperties;

/**
 * Propiedades de configuración del módulo de IA del sistema de Triage.
 * Se leen del bloque 'triage.ai' en application.yml.
 *
 * @param enabled        Si la integración con IA está habilitada (RF-11).
 * @param provider       Proveedor del LLM: openai, anthropic, ollama, none.
 * @param timeoutSeconds Timeout máximo en segundos para llamadas a la API externa.
 */
@ConfigurationProperties(prefix = "triage.ai")
public record AiProperties(
        boolean enabled,
        String provider,
        int timeoutSeconds
) {
    // Valor por defecto para timeoutSeconds si no se configura
    public AiProperties {
        if (timeoutSeconds <= 0) timeoutSeconds = 10;
        if (provider == null || provider.isBlank()) provider = "none";
    }
}
```

```java
// src/main/java/co/uniquindio/triage/ai/config/AiConfig.java

package co.uniquindio.triage.ai.config;

import co.uniquindio.triage.ai.adapter.NoOpAsistenteAdapter;
import co.uniquindio.triage.ai.adapter.SpringAiAsistenteAdapter;
import co.uniquindio.triage.ai.port.AsistenteAcademicoPort;
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Configuración central del módulo de IA.
 *
 * Aquí decidimos cuál implementación del puerto AsistenteAcademicoPort
 * se registra como Bean de Spring:
 *  - Si la IA está habilitada Y la API key está presente → SpringAiAsistenteAdapter
 *  - En cualquier otro caso → NoOpAsistenteAdapter (sin IA, RF-11)
 */
@Configuration
@EnableConfigurationProperties(AiProperties.class)
public class AiConfig {

    @Bean
    public AsistenteAcademicoPort asistenteAcademicoPort(
            AiProperties properties,
            // ChatClient.Builder lo inyecta Spring AI automáticamente si la dependencia está en el classpath
            // Usamos required = false para no fallar si la dependencia no está
            @org.springframework.beans.factory.annotation.Autowired(required = false)
            ChatClient.Builder chatClientBuilder
    ) {
        if (properties.enabled() && chatClientBuilder != null) {
            // La IA está habilitada y el ChatClient está disponible
            var chatClient = chatClientBuilder
                    .defaultSystem("Eres un asistente especializado en gestión académica universitaria. " +
                            "Respondes SIEMPRE en formato JSON válido, sin texto adicional, sin markdown.")
                    .build();

            return new SpringAiAsistenteAdapter(chatClient, properties);
        }

        // Fallback: retornamos el adaptador vacío que no hace nada (RF-11 garantizado)
        return new NoOpAsistenteAdapter();
    }
}
```

---

## 7. Patrón de diseño: IA como servicio opcional

### 7.1 El problema que necesitamos resolver

La IA es un servicio externo: puede no estar disponible, puede costar dinero, puede ser lenta, y puede que algunos entornos (producción, CI/CD) no tengan la API key configurada. Necesitamos un diseño donde la IA sea un **componente enchufable y desenchufable** sin que el resto del sistema lo sienta.

Esto se resuelve combinando dos patrones que ya conoces de la Guía 02:

- **Patrón Strategy**: define una interfaz con las operaciones de IA. Múltiples implementaciones concretas.
- **Feature Flag**: una bandera de configuración que determina en tiempo de inicio qué implementación usar.

### 7.2 El Puerto (la interfaz)

```java
// src/main/java/co/uniquindio/triage/ai/port/AsistenteAcademicoPort.java

package co.uniquindio.triage.ai.port;

import co.uniquindio.triage.ai.dto.ResumenSolicitudResponse;
import co.uniquindio.triage.ai.dto.SugerenciaClasificacionResponse;

import java.util.List;
import java.util.Optional;

/**
 * Puerto (interfaz) del asistente de IA para el sistema de Triage.
 *
 * Define las operaciones que el sistema puede solicitar al asistente inteligente.
 * Las implementaciones concretas pueden conectarse a OpenAI, Anthropic, Ollama
 * o simplemente no hacer nada (NoOp).
 *
 * REGLA: ninguna clase de servicio o controlador debe depender de una
 * implementación concreta; solo de esta interfaz.
 */
public interface AsistenteAcademicoPort {

    /**
     * Indica si la integración con IA está activa y disponible.
     * Úsalo para mostrar u ocultar opciones de IA en el frontend.
     */
    boolean estaDisponible();

    /**
     * RF-10: Sugiere el tipo de solicitud y la prioridad a partir
     * de la descripción libre ingresada por el estudiante.
     *
     * La sugerencia es solo una recomendación; la decisión final
     * la toma el funcionario humano.
     *
     * @param descripcionSolicitud Texto libre del estudiante describiendo su problema.
     * @return Optional con la sugerencia, o vacío si la IA no está disponible o falla.
     */
    Optional<SugerenciaClasificacionResponse> sugerirClasificacion(String descripcionSolicitud);

    /**
     * RF-09: Genera un resumen legible del historial de una solicitud.
     *
     * @param solicitudId       Identificador de la solicitud.
     * @param descripcionOriginal Descripción que ingresó el estudiante.
     * @param entradas          Lista de textos del historial (acciones, observaciones).
     * @return Optional con el resumen, o vacío si la IA no está disponible o falla.
     */
    Optional<ResumenSolicitudResponse> resumirHistorial(
            Long solicitudId,
            String descripcionOriginal,
            List<String> entradas
    );
}
```

### 7.3 Los DTOs de respuesta

```java
// src/main/java/co/uniquindio/triage/ai/dto/SugerenciaClasificacionResponse.java

package co.uniquindio.triage.ai.dto;

import co.uniquindio.triage.domain.enums.Prioridad;
import co.uniquindio.triage.domain.enums.TipoSolicitud;
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

/**
 * DTO que representa la sugerencia que genera la IA para clasificar
 * una solicitud académica (RF-10).
 *
 * Es un Record inmutable. Lo usamos tanto para parsear la respuesta JSON
 * del LLM como para enviarlo al frontend.
 *
 * @param tipoSolicitudSugerido  El tipo de solicitud que la IA sugiere.
 * @param prioridadSugerida      La prioridad que la IA recomienda.
 * @param justificacion          Explicación breve de por qué sugiere eso.
 * @param confianza              Nivel de confianza del 0 al 100 (la IA lo estima).
 */
@JsonIgnoreProperties(ignoreUnknown = true)
public record SugerenciaClasificacionResponse(
        TipoSolicitud tipoSolicitudSugerido,
        Prioridad prioridadSugerida,
        String justificacion,
        int confianza
) {}
```

```java
// src/main/java/co/uniquindio/triage/ai/dto/ResumenSolicitudResponse.java

package co.uniquindio.triage.ai.dto;

/**
 * DTO con el resumen generado por IA del historial de una solicitud (RF-09).
 *
 * @param resumen       Texto del resumen generado.
 * @param estadoActual  El estado actual inferido del historial.
 * @param siguientePaso Sugerencia de qué debería pasar a continuación.
 */
public record ResumenSolicitudResponse(
        String resumen,
        String estadoActual,
        String siguientePaso
) {}
```

### 7.4 El Adaptador NoOp (sin IA — RF-11)

```java
// src/main/java/co/uniquindio/triage/ai/adapter/NoOpAsistenteAdapter.java

package co.uniquindio.triage.ai.adapter;

import co.uniquindio.triage.ai.dto.ResumenSolicitudResponse;
import co.uniquindio.triage.ai.dto.SugerenciaClasificacionResponse;
import co.uniquindio.triage.ai.port.AsistenteAcademicoPort;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.List;
import java.util.Optional;

/**
 * Implementación "sin operación" del asistente de IA.
 *
 * Se activa cuando la IA está deshabilitada (triage.ai.enabled=false)
 * o cuando el proveedor no está configurado.
 *
 * Garantiza el RF-11: el sistema funciona perfectamente sin IA.
 * No lanza excepciones, no llama APIs externas, simplemente retorna Optional.empty().
 */
public class NoOpAsistenteAdapter implements AsistenteAcademicoPort {

    private static final Logger log = LoggerFactory.getLogger(NoOpAsistenteAdapter.class);

    public NoOpAsistenteAdapter() {
        log.info("==> AsistenteAcademicoPort: modo SIN IA activo (NoOpAsistenteAdapter). " +
                "Para habilitar, configura triage.ai.enabled=true y una API key válida.");
    }

    @Override
    public boolean estaDisponible() {
        return false;  // Le decimos al frontend que la IA no está activa
    }

    @Override
    public Optional<SugerenciaClasificacionResponse> sugerirClasificacion(String descripcionSolicitud) {
        log.debug("sugerirClasificacion() llamado pero la IA está deshabilitada (NoOp).");
        return Optional.empty();
    }

    @Override
    public Optional<ResumenSolicitudResponse> resumirHistorial(
            Long solicitudId, String descripcionOriginal, List<String> entradas) {
        log.debug("resumirHistorial() llamado pero la IA está deshabilitada (NoOp).");
        return Optional.empty();
    }
}
```

---

## 8. Implementar RF-10: Sugerencia de clasificación y prioridad

### 8.1 Los templates de prompts

Antes de escribir el adaptador, vamos a diseñar los prompts. En Spring AI puedes colocar templates en archivos `.st` (StringTemplate) dentro de `src/main/resources/prompts/`.

La clave de un buen prompt para obtener JSON es ser muy explícito sobre el formato exacto que esperas:

```
# src/main/resources/prompts/clasificar-solicitud.st

Eres un clasificador experto de solicitudes académicas universitarias en Colombia.
Tu tarea es analizar la descripción de una solicitud y determinar su tipo y prioridad.

TIPOS DE SOLICITUD DISPONIBLES (usa exactamente estos valores):
- REGISTRO_ASIGNATURAS: El estudiante quiere inscribir, agregar o cambiar materias.
- HOMOLOGACION: El estudiante quiere que le reconozcan materias de otra institución.
- CANCELACION_ASIGNATURAS: El estudiante quiere cancelar una o más materias.
- SOLICITUD_CUPOS: El estudiante necesita un cupo en un grupo/materia específica.
- CONSULTA_ACADEMICA: El estudiante tiene una pregunta general sobre su situación académica.
- OTRO: No encaja en ninguna categoría anterior.

NIVELES DE PRIORIDAD DISPONIBLES (usa exactamente estos valores):
- CRITICA: Afecta la continuidad del semestre o la graduación. Requiere atención inmediata.
- ALTA: Tiene fecha límite próxima (menos de 1 semana). Importante pero no emergencia.
- MEDIA: Trámite normal, puede atenderse en el flujo regular (1-2 semanas).
- BAJA: Sin urgencia. Puede resolverse en el próximo ciclo de atención.

DESCRIPCIÓN DEL ESTUDIANTE:
{descripcion}

Responde ÚNICAMENTE con un objeto JSON válido, sin texto adicional, sin explicaciones fuera del JSON, sin bloques de código markdown. El JSON debe tener exactamente esta estructura:
{
  "tipoSolicitudSugerido": "<UNO DE LOS TIPOS>",
  "prioridadSugerida": "<UNO DE LOS NIVELES>",
  "justificacion": "<Explicación breve en máximo 2 oraciones de por qué ese tipo y prioridad>",
  "confianza": <número entre 0 y 100>
}
```

```
# src/main/resources/prompts/resumir-historial.st

Eres un asistente de gestión académica. Tu tarea es leer el historial de acciones
de una solicitud académica y generar un resumen ejecutivo para que un funcionario
entienda rápidamente el estado del caso.

SOLICITUD #: {solicitudId}
DESCRIPCIÓN ORIGINAL DEL ESTUDIANTE:
{descripcionOriginal}

HISTORIAL DE ACCIONES (en orden cronológico):
{historial}

Responde ÚNICAMENTE con un objeto JSON válido con esta estructura exacta:
{
  "resumen": "<Párrafo de 2-3 oraciones resumiendo qué pasó con esta solicitud>",
  "estadoActual": "<Estado actual del trámite en una frase>",
  "siguientePaso": "<Qué debería pasar a continuación para resolver la solicitud>"
}
```

### 8.2 El Adaptador con Spring AI

Ahora implementamos el adaptador real que usa Spring AI para llamar a OpenAI:

```java
// src/main/java/co/uniquindio/triage/ai/adapter/SpringAiAsistenteAdapter.java

package co.uniquindio.triage.ai.adapter;

import co.uniquindio.triage.ai.config.AiProperties;
import co.uniquindio.triage.ai.dto.ResumenSolicitudResponse;
import co.uniquindio.triage.ai.dto.SugerenciaClasificacionResponse;
import co.uniquindio.triage.ai.port.AsistenteAcademicoPort;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.model.ChatResponse;
import org.springframework.core.io.ClassPathResource;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.List;
import java.util.Optional;

/**
 * Adaptador que implementa AsistenteAcademicoPort usando Spring AI y OpenAI.
 *
 * Principio de diseño: si CUALQUIER cosa falla (timeout, respuesta malformada,
 * error de red, JSON inválido), el método retorna Optional.empty() y loguea el error.
 * NUNCA propaga excepciones al llamador (el Service), porque el Service
 * no debe saber si la IA existe o no (RF-11).
 */
public class SpringAiAsistenteAdapter implements AsistenteAcademicoPort {

    private static final Logger log = LoggerFactory.getLogger(SpringAiAsistenteAdapter.class);

    private final ChatClient chatClient;
    private final AiProperties properties;
    private final ObjectMapper objectMapper;

    // Templates de prompts cargados al inicio (no en cada llamada, por rendimiento)
    private final String templateClasificar;
    private final String templateResumir;

    public SpringAiAsistenteAdapter(ChatClient chatClient, AiProperties properties) {
        this.chatClient = chatClient;
        this.properties = properties;
        this.objectMapper = new ObjectMapper();

        // Cargar templates de disco al construir el adaptador
        this.templateClasificar = cargarTemplate("prompts/clasificar-solicitud.st");
        this.templateResumir = cargarTemplate("prompts/resumir-historial.st");

        log.info("==> AsistenteAcademicoPort: modo CON IA activo (SpringAiAsistenteAdapter). " +
                "Proveedor: {}", properties.provider());
    }

    @Override
    public boolean estaDisponible() {
        return true;
    }

    /**
     * RF-10: Sugiere tipo de solicitud y prioridad a partir de texto libre.
     *
     * Flujo:
     * 1. Construye el prompt usando el template y la descripción del estudiante.
     * 2. Envía al LLM mediante ChatClient.
     * 3. Parsea la respuesta JSON al Record SugerenciaClasificacionResponse.
     * 4. Si cualquier paso falla → retorna Optional.empty().
     */
    @Override
    public Optional<SugerenciaClasificacionResponse> sugerirClasificacion(String descripcionSolicitud) {
        if (descripcionSolicitud == null || descripcionSolicitud.isBlank()) {
            log.warn("sugerirClasificacion: descripción vacía, retornando vacío.");
            return Optional.empty();
        }

        try {
            // Sustituir la variable {descripcion} en el template
            var prompt = templateClasificar.replace("{descripcion}", descripcionSolicitud);

            log.debug("Enviando solicitud de clasificación al LLM...");

            // Llamar al LLM usando la API fluida de Spring AI
            var respuestaJson = chatClient
                    .prompt()
                    .user(prompt)
                    .call()
                    .content();  // Obtener solo el texto de la respuesta

            log.debug("Respuesta del LLM (clasificación): {}", respuestaJson);

            // Parsear el JSON a nuestro Record
            var sugerencia = objectMapper.readValue(
                    limpiarJson(respuestaJson),
                    SugerenciaClasificacionResponse.class
            );

            return Optional.of(sugerencia);

        } catch (Exception e) {
            // Capturamos TODO: errores de red, timeout, JSON inválido, enums no reconocidos
            log.error("Error al sugerir clasificación con IA: {}", e.getMessage(), e);
            return Optional.empty();  // El sistema sigue funcionando sin IA (RF-11)
        }
    }

    /**
     * RF-09: Genera un resumen del historial de una solicitud.
     */
    @Override
    public Optional<ResumenSolicitudResponse> resumirHistorial(
            Long solicitudId,
            String descripcionOriginal,
            List<String> entradas
    ) {
        if (entradas == null || entradas.isEmpty()) {
            log.warn("resumirHistorial: historial vacío para solicitud #{}", solicitudId);
            return Optional.empty();
        }

        try {
            // Construir el texto del historial numerado
            var historialTexto = new StringBuilder();
            for (int i = 0; i < entradas.size(); i++) {
                historialTexto.append(i + 1).append(". ").append(entradas.get(i)).append("\n");
            }

            // Sustituir variables en el template
            var prompt = templateResumir
                    .replace("{solicitudId}", solicitudId.toString())
                    .replace("{descripcionOriginal}", descripcionOriginal)
                    .replace("{historial}", historialTexto.toString());

            log.debug("Enviando solicitud de resumen al LLM para solicitud #{}...", solicitudId);

            var respuestaJson = chatClient
                    .prompt()
                    .user(prompt)
                    .call()
                    .content();

            log.debug("Respuesta del LLM (resumen solicitud #{}): {}", solicitudId, respuestaJson);

            var resumen = objectMapper.readValue(
                    limpiarJson(respuestaJson),
                    ResumenSolicitudResponse.class
            );

            return Optional.of(resumen);

        } catch (Exception e) {
            log.error("Error al resumir historial de solicitud #{} con IA: {}", solicitudId, e.getMessage(), e);
            return Optional.empty();
        }
    }

    // ─── Métodos auxiliares privados ───────────────────────────────────────────

    /**
     * Carga un template de prompts desde el classpath (resources/).
     * Los templates se cargan una vez al construir el adaptador.
     */
    private String cargarTemplate(String rutaEnResources) {
        try {
            var resource = new ClassPathResource(rutaEnResources);
            return resource.getContentAsString(StandardCharsets.UTF_8);
        } catch (IOException e) {
            log.error("No se pudo cargar el template de prompt: {}", rutaEnResources, e);
            return "";  // Retornar vacío; los métodos que usen este template fallarán graciosamente
        }
    }

    /**
     * Los LLMs a veces envuelven el JSON en bloques de código markdown como ```json ... ```
     * aunque les pidamos que no lo hagan. Esta función limpia eso.
     */
    private String limpiarJson(String respuesta) {
        if (respuesta == null) return "{}";
        return respuesta
                .strip()
                .replaceAll("(?s)^```json\\s*", "")   // Quitar apertura de bloque markdown
                .replaceAll("(?s)\\s*```$", "")         // Quitar cierre de bloque markdown
                .strip();
    }
}
```

---

## 9. Implementar RF-09: Resumen del historial de una solicitud

### 9.1 El Service de IA

El service orquesta las operaciones: obtiene datos del repositorio y delega al puerto de IA.

```java
// src/main/java/co/uniquindio/triage/service/AsistenteService.java

package co.uniquindio.triage.service;

import co.uniquindio.triage.ai.dto.ResumenSolicitudResponse;
import co.uniquindio.triage.ai.dto.SugerenciaClasificacionResponse;
import co.uniquindio.triage.ai.port.AsistenteAcademicoPort;
import co.uniquindio.triage.domain.HistorialSolicitud;
import co.uniquindio.triage.domain.Solicitud;
import co.uniquindio.triage.exception.RecursoNoEncontradoException;
import co.uniquindio.triage.repository.SolicitudRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.format.DateTimeFormatter;
import java.util.List;
import java.util.Optional;

/**
 * Servicio de negocio que expone las funcionalidades de IA del sistema de Triage.
 *
 * Este servicio NO contiene lógica de IA; delega al puerto AsistenteAcademicoPort.
 * Su responsabilidad es:
 *  1. Obtener los datos necesarios del dominio (solicitud, historial).
 *  2. Transformarlos al formato que necesita el puerto.
 *  3. Retornar la respuesta (o vacío si la IA no está disponible).
 */
@Service
@Transactional(readOnly = true)
public class AsistenteService {

    private static final Logger log = LoggerFactory.getLogger(AsistenteService.class);
    private static final DateTimeFormatter FORMATTER = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm");

    private final AsistenteAcademicoPort asistentePort;
    private final SolicitudRepository solicitudRepository;

    // Usamos constructor injection (mejor práctica que @Autowired en el campo)
    public AsistenteService(AsistenteAcademicoPort asistentePort,
                             SolicitudRepository solicitudRepository) {
        this.asistentePort = asistentePort;
        this.solicitudRepository = solicitudRepository;
    }

    /**
     * Indica al cliente si la integración con IA está disponible.
     * El frontend usa este dato para mostrar u ocultar los botones de IA.
     */
    public boolean isIaDisponible() {
        return asistentePort.estaDisponible();
    }

    /**
     * RF-10: Sugiere clasificación y prioridad para un texto de solicitud.
     *
     * @param descripcion El texto libre del estudiante.
     * @return La sugerencia, o empty si la IA no está disponible.
     */
    public Optional<SugerenciaClasificacionResponse> sugerirClasificacion(String descripcion) {
        log.info("Solicitando sugerencia de clasificación para descripción de {} caracteres", descripcion.length());
        return asistentePort.sugerirClasificacion(descripcion);
    }

    /**
     * RF-09: Genera un resumen del historial de una solicitud específica.
     *
     * Primero carga la solicitud y su historial desde la base de datos,
     * luego formatea las entradas del historial y las envía al asistente de IA.
     *
     * @param solicitudId ID de la solicitud a resumir.
     * @return El resumen generado, o empty si la IA no está disponible.
     */
    public Optional<ResumenSolicitudResponse> resumirHistorial(Long solicitudId) {
        log.info("Solicitando resumen de historial para solicitud #{}", solicitudId);

        // Cargar la solicitud con su historial (usamos @EntityGraph para evitar N+1)
        var solicitud = solicitudRepository
                .findByIdWithHistorial(solicitudId)
                .orElseThrow(() -> new RecursoNoEncontradoException("Solicitud no encontrada: #" + solicitudId));

        // Transformar las entradas del historial a texto legible para el LLM
        var entradasHistorial = formatearHistorial(solicitud);

        return asistentePort.resumirHistorial(
                solicitudId,
                solicitud.getDescripcion(),
                entradasHistorial
        );
    }

    /**
     * Formatea las entradas del historial de una solicitud en textos descriptivos
     * que el LLM puede entender fácilmente.
     */
    private List<String> formatearHistorial(Solicitud solicitud) {
        return solicitud.getHistorial().stream()
                .sorted((a, b) -> a.getFechaAccion().compareTo(b.getFechaAccion()))
                .map(entrada -> String.format(
                        "[%s] %s realizó la acción '%s'. Observaciones: %s",
                        entrada.getFechaAccion().format(FORMATTER),
                        entrada.getUsuario() != null ? entrada.getUsuario().getNombre() : "Sistema",
                        entrada.getAccion(),
                        entrada.getObservaciones() != null ? entrada.getObservaciones() : "Sin observaciones"
                ))
                .toList();
    }
}
```

### 9.2 El repositorio con la consulta optimizada

Necesitamos una consulta que traiga la solicitud y su historial en una sola query para evitar el problema N+1:

```java
// src/main/java/co/uniquindio/triage/repository/SolicitudRepository.java
// Agregar este método al repositorio existente:

import org.springframework.data.jpa.repository.EntityGraph;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

public interface SolicitudRepository extends JpaRepository<Solicitud, Long> {

    // ...métodos existentes...

    /**
     * Carga la solicitud junto con su historial y el usuario de cada entrada
     * en una sola consulta SQL (evita N+1).
     *
     * Usamos LEFT JOIN FETCH para asegurar que el historial se cargue
     * aunque esté configurado como LAZY en la entidad.
     */
    @Query("""
        SELECT s FROM Solicitud s
        LEFT JOIN FETCH s.historial h
        LEFT JOIN FETCH h.usuario
        WHERE s.id = :id
    """)
    Optional<Solicitud> findByIdWithHistorial(@Param("id") Long id);
}
```

---

## 10. RF-11: Garantizar funcionamiento sin IA

Ya implementamos el `NoOpAsistenteAdapter`, pero necesitamos asegurarnos de que el `AsistenteService` y los controladores lo manejen correctamente. El contrato es claro: cuando la IA no está disponible, los métodos retornan `Optional.empty()`. Los controladores deben responder con un código HTTP apropiado.

Veamos cómo queda el flujo completo:

```
Con IA habilitada:
Cliente → AsistenteController → AsistenteService → SpringAiAsistenteAdapter → OpenAI API
                                                                              ↓
                                                              SugerenciaClasificacionResponse
                                                                              ↓
                                                           HTTP 200 con la sugerencia

Sin IA (NoOp):
Cliente → AsistenteController → AsistenteService → NoOpAsistenteAdapter → Optional.empty()
                                                                              ↓
                                                           HTTP 503 Service Unavailable
                                                           {"mensaje": "Asistente de IA no disponible"}

Con IA habilitada pero API falla:
Cliente → AsistenteController → AsistenteService → SpringAiAsistenteAdapter → ERROR de red
                                                                              ↓ (catch)
                                                                        Optional.empty()
                                                                              ↓
                                                           HTTP 503 Service Unavailable
```

En los tres casos, el `AsistenteController` recibe un `Optional`. Su trabajo es convertirlo en la respuesta HTTP adecuada.

---

## 11. Endpoints REST para las funcionalidades de IA

```java
// src/main/java/co/uniquindio/triage/controller/AsistenteController.java

package co.uniquindio.triage.controller;

import co.uniquindio.triage.ai.dto.ResumenSolicitudResponse;
import co.uniquindio.triage.ai.dto.SugerenciaClasificacionResponse;
import co.uniquindio.triage.service.AsistenteService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.security.SecurityRequirement;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import java.util.Map;

/**
 * Controlador REST para las funcionalidades de asistencia por IA.
 *
 * IMPORTANTE DE DISEÑO:
 * - Los endpoints retornan 503 (Service Unavailable) cuando la IA no está activa,
 *   NO un 500 (error del servidor). La IA es un servicio opcional, no crítico.
 * - Las sugerencias son solo eso: sugerencias. El usuario siempre decide.
 */
@RestController
@RequestMapping("/api/v1/asistente")
@Validated
@Tag(name = "Asistente IA", description = "Funcionalidades opcionales de asistencia con IA (RF-09, RF-10, RF-11)")
@SecurityRequirement(name = "bearerAuth")
public class AsistenteController {

    private final AsistenteService asistenteService;

    public AsistenteController(AsistenteService asistenteService) {
        this.asistenteService = asistenteService;
    }

    // ─── GET /api/v1/asistente/disponible ─────────────────────────────────────

    @GetMapping("/disponible")
    @Operation(
            summary = "Verifica si el asistente de IA está disponible",
            description = "El frontend usa este endpoint al iniciar para mostrar u ocultar los botones de IA."
    )
    @ApiResponse(responseCode = "200", description = "Estado del asistente")
    public ResponseEntity<Map<String, Boolean>> verificarDisponibilidad() {
        return ResponseEntity.ok(Map.of("disponible", asistenteService.isIaDisponible()));
    }

    // ─── POST /api/v1/asistente/clasificacion ─────────────────────────────────

    @PostMapping("/clasificacion")
    @PreAuthorize("hasAnyRole('ESTUDIANTE', 'FUNCIONARIO', 'ADMIN')")
    @Operation(
            summary = "RF-10: Sugiere clasificación y prioridad para una solicitud",
            description = """
                    Dado el texto descriptivo de una solicitud académica, la IA sugiere
                    el tipo de solicitud y la prioridad más adecuada.
                    
                    La sugerencia debe ser revisada y confirmada (o ajustada) por un usuario humano.
                    Retorna 503 si el asistente de IA no está disponible.
                    """
    )
    @ApiResponse(responseCode = "200", description = "Sugerencia generada exitosamente")
    @ApiResponse(responseCode = "503", description = "Asistente de IA no disponible")
    public ResponseEntity<?> sugerirClasificacion(
            @Parameter(description = "Texto descriptivo de la solicitud")
            @RequestParam
            @NotBlank(message = "La descripción no puede estar vacía")
            @Size(min = 20, max = 2000, message = "La descripción debe tener entre 20 y 2000 caracteres")
            String descripcion
    ) {
        return asistenteService.sugerirClasificacion(descripcion)
                .map(ResponseEntity::ok)
                .orElseGet(() -> ResponseEntity
                        .status(503)
                        .body((SugerenciaClasificacionResponse) null));
        // Nota: cuando la IA no está disponible, retornamos 503
        // El frontend mostrará un mensaje amigable al usuario
    }

    // ─── GET /api/v1/asistente/solicitudes/{id}/resumen ───────────────────────

    @GetMapping("/solicitudes/{id}/resumen")
    @PreAuthorize("hasAnyRole('FUNCIONARIO', 'ADMIN')")
    @Operation(
            summary = "RF-09: Genera un resumen del historial de una solicitud",
            description = """
                    Usando el historial de acciones de la solicitud, la IA genera
                    un resumen ejecutivo para que el funcionario entienda rápidamente
                    el estado del caso sin leer todas las entradas del historial.
                    
                    Solo accesible para FUNCIONARIO y ADMIN.
                    Retorna 503 si el asistente de IA no está disponible.
                    """
    )
    @ApiResponse(responseCode = "200", description = "Resumen generado exitosamente")
    @ApiResponse(responseCode = "404", description = "Solicitud no encontrada")
    @ApiResponse(responseCode = "503", description = "Asistente de IA no disponible")
    public ResponseEntity<?> resumirHistorial(
            @PathVariable Long id
    ) {
        return asistenteService.resumirHistorial(id)
                .map(ResponseEntity::ok)
                .orElseGet(() -> ResponseEntity.status(503).build());
    }
}
```

### 11.1 Respuesta de ejemplo de los endpoints

**POST /api/v1/asistente/clasificacion?descripcion=Quiero cancelar Cálculo 2 porque tengo problemas de salud**

```json
HTTP 200 OK
{
  "tipoSolicitudSugerido": "CANCELACION_ASIGNATURAS",
  "prioridadSugerida": "ALTA",
  "justificacion": "El estudiante indica razones de salud para cancelar una materia, lo que requiere atención prioritaria. La cancelación por causas médicas suele tener plazos específicos.",
  "confianza": 92
}
```

**GET /api/v1/asistente/solicitudes/42/resumen**

```json
HTTP 200 OK
{
  "resumen": "El estudiante Juan García registró el 15/03/2026 una solicitud de homologación para la materia Algoritmos cursada en la Universidad Nacional. La solicitud fue clasificada como ALTA prioridad y asignada al funcionario Pedro López el 16/03/2026.",
  "estadoActual": "En atención - asignada a funcionario responsable",
  "siguientePaso": "El funcionario debe revisar los documentos del pensum de origen y emitir concepto de equivalencia curricular"
}
```

**Cuando la IA no está disponible:**

```json
HTTP 503 Service Unavailable
(body vacío o con mensaje de error configurado globalmente)
```

---

## 12. Manejo de errores, timeouts y resiliencia

### 12.1 El problema de los servicios externos

Cuando dependes de una API externa como OpenAI tienes que planear para estas situaciones:

| Situación | Probabilidad | Impacto sin manejo |
|---|---|---|
| API key inválida | Baja | Error 401, tu app explota |
| Límite de rate excedido | Media | Error 429, tu app espera sin informar |
| Timeout de red | Media | Tu hilo queda bloqueado indefinidamente |
| Respuesta JSON malformada | Media | `JsonParseException`, tu app explota |
| API de OpenAI caída | Baja | Tu app completamente inoperativa |
| Enums no reconocidos en JSON | Media | `InvalidDefinitionException` |

Ya resolviste la mayoría con el `try-catch` genérico en el adaptador. Pero el timeout necesita configuración adicional.

### 12.2 Configurar timeout en Spring AI

Spring AI usa `RestClient` internamente. Puedes configurar el timeout de conexión y lectura:

```java
// src/main/java/co/uniquindio/triage/ai/config/AiConfig.java
// Versión actualizada con timeout configurado

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.openai.OpenAiChatModel;
import org.springframework.boot.web.client.RestClientCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.time.Duration;

@Configuration
@EnableConfigurationProperties(AiProperties.class)
public class AiConfig {

    /**
     * Personaliza el RestClient que Spring AI usa internamente para llamar a OpenAI.
     * Aquí configuramos el timeout.
     */
    @Bean
    public RestClientCustomizer openAiRestClientCustomizer(AiProperties properties) {
        return restClientBuilder -> restClientBuilder
                .requestInterceptor((request, body, execution) -> {
                    // Podríamos agregar headers adicionales aquí si fuera necesario
                    return execution.execute(request, body);
                });
        // Nota: la configuración de timeout específica depende del HttpClient subyacente.
        // Spring Boot 3.4+ permite configurar esto via properties:
        // spring.ai.openai.connect-timeout=10s
        // spring.ai.openai.read-timeout=30s
    }

    @Bean
    public AsistenteAcademicoPort asistenteAcademicoPort(
            AiProperties properties,
            @org.springframework.beans.factory.annotation.Autowired(required = false)
            ChatClient.Builder chatClientBuilder
    ) {
        if (properties.enabled() && chatClientBuilder != null) {
            var chatClient = chatClientBuilder
                    .defaultSystem("Eres un asistente especializado en gestión académica universitaria. " +
                            "Respondes SIEMPRE en formato JSON válido, sin texto adicional, sin markdown.")
                    .build();
            return new SpringAiAsistenteAdapter(chatClient, properties);
        }
        return new NoOpAsistenteAdapter();
    }
}
```

Y en el `application.yml`:

```yaml
spring:
  ai:
    openai:
      api-key: ${OPENAI_API_KEY:}
      chat:
        options:
          model: gpt-4o-mini
          temperature: 0.2
          max-tokens: 500
      # Timeouts para las llamadas HTTP a OpenAI
      connect-timeout: 5s     # Tiempo máximo para establecer conexión
      read-timeout: 30s       # Tiempo máximo para recibir la respuesta completa
```

### 12.3 Manejo de enums en JSON

Un problema frecuente: el LLM puede responder con `"CANCELACION_DE_ASIGNATURAS"` en lugar de `"CANCELACION_ASIGNATURAS"`, o en minúsculas. Jackson fallará al parsear.

Solución: configurar Jackson para ser tolerante con enums desconocidos:

```java
// Agregar en AiConfig.java o en una configuración de Jackson global:

@Bean
public ObjectMapper aiObjectMapper() {
    return JsonMapper.builder()
            // Si el enum no se reconoce, usa null en lugar de lanzar excepción
            .enable(DeserializationFeature.READ_UNKNOWN_ENUM_VALUES_AS_NULL)
            // Ignorar propiedades JSON que no existen en el Record
            .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
            // Case-insensitive para enums
            .enable(MapperFeature.ACCEPT_CASE_INSENSITIVE_ENUMS)
            .build();
}
```

Y en el adaptador, validar que los enums no sean null antes de retornar:

```java
// En SpringAiAsistenteAdapter.sugerirClasificacion(), después de parsear:

var sugerencia = objectMapper.readValue(limpiarJson(respuestaJson), SugerenciaClasificacionResponse.class);

// Validar que los campos críticos no sean null (el LLM pudo haber devuelto un valor no reconocido)
if (sugerencia.tipoSolicitudSugerido() == null || sugerencia.prioridadSugerida() == null) {
    log.warn("El LLM devolvió un tipo o prioridad no reconocida. Respuesta: {}", respuestaJson);
    return Optional.empty();
}

return Optional.of(sugerencia);
```

### 12.4 Logging de trazabilidad

Para poder depurar problemas en producción, es crucial loguear bien las llamadas a la IA:

```java
// Ejemplo de logging estructurado en el adaptador:

log.info("IA_REQUEST type=clasificacion descripcion_length={}", descripcionSolicitud.length());
// ... llamada al LLM ...
log.info("IA_RESPONSE type=clasificacion tipo={} prioridad={} confianza={} tokens_aprox={}",
        sugerencia.tipoSolicitudSugerido(),
        sugerencia.prioridadSugerida(),
        sugerencia.confianza(),
        respuestaJson.length() / 4  // estimación muy aproximada de tokens
);
```

---

## 13. Angular: UI para interactuar con la IA

### 13.1 Estructura de componentes Angular para la IA

```
src/app/features/solicitudes/
├── crear-solicitud/
│   ├── crear-solicitud.component.ts     ← Ya existe. Le agregaremos el botón de IA.
│   └── crear-solicitud.component.html
└── components/
    ├── sugerencia-ia/                    ← NUEVO: componente que muestra la sugerencia
    │   ├── sugerencia-ia.component.ts
    │   └── sugerencia-ia.component.html
    └── resumen-ia/                       ← NUEVO: componente que muestra el resumen
        ├── resumen-ia.component.ts
        └── resumen-ia.component.html

src/app/core/services/
└── asistente.service.ts                  ← NUEVO: servicio que llama al backend
```

### 13.2 Interfaces TypeScript

```typescript
// src/app/core/models/ia.models.ts

export type TipoSolicitud =
  | 'REGISTRO_ASIGNATURAS'
  | 'HOMOLOGACION'
  | 'CANCELACION_ASIGNATURAS'
  | 'SOLICITUD_CUPOS'
  | 'CONSULTA_ACADEMICA'
  | 'OTRO';

export type Prioridad = 'CRITICA' | 'ALTA' | 'MEDIA' | 'BAJA';

export interface SugerenciaClasificacion {
  tipoSolicitudSugerido: TipoSolicitud;
  prioridadSugerida: Prioridad;
  justificacion: string;
  confianza: number;
}

export interface ResumenSolicitud {
  resumen: string;
  estadoActual: string;
  siguientePaso: string;
}
```

### 13.3 El servicio Angular de IA

```typescript
// src/app/core/services/asistente.service.ts

import { Injectable, inject, signal } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, of, catchError } from 'rxjs';
import { environment } from '../../../environments/environment';
import { SugerenciaClasificacion, ResumenSolicitud } from '../models/ia.models';

@Injectable({ providedIn: 'root' })
export class AsistenteService {

  private readonly http = inject(HttpClient);
  private readonly baseUrl = `${environment.apiUrl}/asistente`;

  // Signal reactivo que guarda si la IA está disponible.
  // Los componentes pueden leer este signal para mostrar/ocultar botones de IA.
  readonly iaDisponible = signal<boolean>(false);

  constructor() {
    // Verificamos la disponibilidad de la IA al inicializar el servicio
    this.verificarDisponibilidad();
  }

  /**
   * Consulta al backend si el asistente de IA está activo.
   * El resultado se guarda en el signal `iaDisponible`.
   */
  verificarDisponibilidad(): void {
    this.http.get<{ disponible: boolean }>(`${this.baseUrl}/disponible`)
      .pipe(catchError(() => of({ disponible: false })))  // Si falla, asumimos no disponible
      .subscribe(response => this.iaDisponible.set(response.disponible));
  }

  /**
   * RF-10: Solicita al backend una sugerencia de clasificación.
   *
   * @param descripcion El texto descriptivo de la solicitud.
   * @returns Observable con la sugerencia, o null si la IA no está disponible.
   */
  sugerirClasificacion(descripcion: string): Observable<SugerenciaClasificacion | null> {
    return this.http
      .post<SugerenciaClasificacion>(
        `${this.baseUrl}/clasificacion`,
        null,
        { params: { descripcion } }
      )
      .pipe(
        catchError(error => {
          if (error.status === 503) {
            // La IA no está disponible — no es un error, es comportamiento esperado
            return of(null);
          }
          // Otros errores: dejar que el interceptor global los maneje
          throw error;
        })
      );
  }

  /**
   * RF-09: Solicita al backend un resumen del historial de una solicitud.
   *
   * @param solicitudId ID de la solicitud.
   * @returns Observable con el resumen, o null si la IA no está disponible.
   */
  resumirHistorial(solicitudId: number): Observable<ResumenSolicitud | null> {
    return this.http
      .get<ResumenSolicitud>(`${this.baseUrl}/solicitudes/${solicitudId}/resumen`)
      .pipe(
        catchError(error => {
          if (error.status === 503) {
            return of(null);
          }
          throw error;
        })
      );
  }
}
```

### 13.4 Componente de sugerencia de IA

Este componente se integra en el formulario de creación de solicitudes. El estudiante escribe su descripción, hace clic en "Sugerir con IA", y ve la sugerencia con la opción de aceptarla o editarla:

```typescript
// src/app/features/solicitudes/components/sugerencia-ia/sugerencia-ia.component.ts

import {
  Component, inject, input, output, signal, computed
} from '@angular/core';
import { CommonModule } from '@angular/common';
import { AsistenteService } from '../../../../core/services/asistente.service';
import { SugerenciaClasificacion, TipoSolicitud, Prioridad } from '../../../../core/models/ia.models';

@Component({
  selector: 'app-sugerencia-ia',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div class="sugerencia-ia-container">

      <!-- Botón para solicitar sugerencia — visible solo si la IA está disponible -->
      @if (asistenteService.iaDisponible()) {
        <button
          type="button"
          class="btn-ia"
          [disabled]="cargando() || !descripcionValida()"
          (click)="solicitarSugerencia()">

          @if (cargando()) {
            <span class="spinner"></span> Consultando IA...
          } @else {
            ✨ Sugerir clasificación con IA
          }
        </button>

        @if (!descripcionValida()) {
          <small class="hint">Escribe al menos 20 caracteres para obtener una sugerencia.</small>
        }
      }

      <!-- Tarjeta con la sugerencia — visible cuando hay resultado -->
      @if (sugerencia()) {
        <div class="sugerencia-card" [class.alta-confianza]="sugerencia()!.confianza >= 80">

          <div class="sugerencia-header">
            <span class="ia-badge">🤖 Sugerencia de IA</span>
            <span class="confianza-badge" [class]="badgeClass()">
              Confianza: {{ sugerencia()!.confianza }}%
            </span>
          </div>

          <div class="sugerencia-body">
            <div class="campo">
              <label>Tipo de solicitud sugerido:</label>
              <strong>{{ formatearTipo(sugerencia()!.tipoSolicitudSugerido) }}</strong>
            </div>
            <div class="campo">
              <label>Prioridad sugerida:</label>
              <strong [class]="'prioridad-' + sugerencia()!.prioridadSugerida.toLowerCase()">
                {{ sugerencia()!.prioridadSugerida }}
              </strong>
            </div>
            <div class="justificacion">
              <label>Justificación:</label>
              <p>{{ sugerencia()!.justificacion }}</p>
            </div>
          </div>

          <div class="sugerencia-acciones">
            <!-- El usuario SIEMPRE decide si acepta o ignora la sugerencia -->
            <button type="button" class="btn-aceptar" (click)="aceptarSugerencia()">
              ✓ Aceptar sugerencia
            </button>
            <button type="button" class="btn-ignorar" (click)="ignorarSugerencia()">
              ✗ Ignorar y clasificar manualmente
            </button>
          </div>

          <p class="disclaimer">
            ⚠️ Esta es una sugerencia automática. La clasificación final la determina el funcionario responsable.
          </p>
        </div>
      }

      <!-- Mensaje de error si la IA no está disponible -->
      @if (errorIa()) {
        <div class="ia-error">
          El asistente de IA no está disponible en este momento.
          Puedes clasificar la solicitud manualmente.
        </div>
      }

    </div>
  `,
  styleUrl: './sugerencia-ia.component.css'
})
export class SugerenciaIaComponent {

  // Input: la descripción actual del formulario padre
  readonly descripcion = input.required<string>();

  // Outputs: eventos hacia el componente padre
  readonly sugerenciaAceptada = output<SugerenciaClasificacion>();
  readonly sugerenciaIgnorada = output<void>();

  // Signals del estado interno del componente
  readonly cargando = signal(false);
  readonly sugerencia = signal<SugerenciaClasificacion | null>(null);
  readonly errorIa = signal(false);

  // Signal derivado: la descripción tiene la longitud mínima
  readonly descripcionValida = computed(() => this.descripcion().length >= 20);

  // Signal derivado: clase CSS según el nivel de confianza
  readonly badgeClass = computed(() => {
    const c = this.sugerencia()?.confianza ?? 0;
    if (c >= 80) return 'badge-verde';
    if (c >= 60) return 'badge-amarillo';
    return 'badge-rojo';
  });

  readonly asistenteService = inject(AsistenteService);

  solicitarSugerencia(): void {
    if (!this.descripcionValida() || this.cargando()) return;

    // Resetear estado previo
    this.cargando.set(true);
    this.sugerencia.set(null);
    this.errorIa.set(false);

    this.asistenteService.sugerirClasificacion(this.descripcion()).subscribe({
      next: (resultado) => {
        this.cargando.set(false);
        if (resultado) {
          this.sugerencia.set(resultado);
        } else {
          // La IA devolvió null → no disponible
          this.errorIa.set(true);
        }
      },
      error: () => {
        this.cargando.set(false);
        this.errorIa.set(true);
      }
    });
  }

  aceptarSugerencia(): void {
    if (this.sugerencia()) {
      // El componente padre recibe la sugerencia aceptada para pre-llenar los campos
      this.sugerenciaAceptada.emit(this.sugerencia()!);
      this.sugerencia.set(null);  // Limpiar la tarjeta
    }
  }

  ignorarSugerencia(): void {
    this.sugerencia.set(null);
    this.sugerenciaIgnorada.emit();
  }

  formatearTipo(tipo: TipoSolicitud): string {
    const nombres: Record<TipoSolicitud, string> = {
      'REGISTRO_ASIGNATURAS': 'Registro de Asignaturas',
      'HOMOLOGACION': 'Homologación',
      'CANCELACION_ASIGNATURAS': 'Cancelación de Asignaturas',
      'SOLICITUD_CUPOS': 'Solicitud de Cupos',
      'CONSULTA_ACADEMICA': 'Consulta Académica',
      'OTRO': 'Otro'
    };
    return nombres[tipo] ?? tipo;
  }
}
```

### 13.5 Integrar el componente en el formulario de crear solicitud

```typescript
// src/app/features/solicitudes/crear-solicitud/crear-solicitud.component.ts
// Fragmento relevante — muestra cómo integrar SugerenciaIaComponent

import { SugerenciaIaComponent } from '../components/sugerencia-ia/sugerencia-ia.component';
import { SugerenciaClasificacion } from '../../../core/models/ia.models';

@Component({
  selector: 'app-crear-solicitud',
  standalone: true,
  imports: [
    ReactiveFormsModule,
    CommonModule,
    SugerenciaIaComponent,  // ← Importar el componente de IA
  ],
  template: `
    <form [formGroup]="form" (ngSubmit)="enviarSolicitud()">

      <!-- Campo de descripción -->
      <div class="campo-form">
        <label for="descripcion">Descripción de tu solicitud *</label>
        <textarea
          id="descripcion"
          formControlName="descripcion"
          rows="5"
          placeholder="Describe tu solicitud con el mayor detalle posible...">
        </textarea>
      </div>

      <!-- Componente de sugerencia de IA — se integra justo después de la descripción -->
      <app-sugerencia-ia
        [descripcion]="form.get('descripcion')?.value ?? ''"
        (sugerenciaAceptada)="aplicarSugerencia($event)"
        (sugerenciaIgnorada)="limpiarSugerencia()">
      </app-sugerencia-ia>

      <!-- Selector de tipo de solicitud — se pre-llena si el usuario acepta la sugerencia -->
      <div class="campo-form">
        <label for="tipo">Tipo de solicitud *</label>
        <select id="tipo" formControlName="tipoSolicitud">
          <option value="">Seleccionar...</option>
          <option value="REGISTRO_ASIGNATURAS">Registro de Asignaturas</option>
          <option value="HOMOLOGACION">Homologación</option>
          <option value="CANCELACION_ASIGNATURAS">Cancelación de Asignaturas</option>
          <option value="SOLICITUD_CUPOS">Solicitud de Cupos</option>
          <option value="CONSULTA_ACADEMICA">Consulta Académica</option>
          <option value="OTRO">Otro</option>
        </select>
      </div>

      <!-- Selector de prioridad -->
      <div class="campo-form">
        <label for="prioridad">Prioridad</label>
        <select id="prioridad" formControlName="prioridad">
          <option value="BAJA">Baja</option>
          <option value="MEDIA">Media</option>
          <option value="ALTA">Alta</option>
          <option value="CRITICA">Crítica</option>
        </select>
      </div>

      <button type="submit" [disabled]="form.invalid || enviando()">
        Registrar Solicitud
      </button>

    </form>
  `
})
export class CrearSolicitudComponent {
  // ...código existente del formulario...

  /**
   * Cuando el usuario acepta la sugerencia de la IA,
   * pre-llenamos los campos del formulario.
   * El usuario puede modificarlos antes de enviar.
   */
  aplicarSugerencia(sugerencia: SugerenciaClasificacion): void {
    this.form.patchValue({
      tipoSolicitud: sugerencia.tipoSolicitudSugerido,
      prioridad: sugerencia.prioridadSugerida
    });
    // Nota: patchValue NO bloquea los campos. El usuario puede seguir editándolos.
  }

  limpiarSugerencia(): void {
    // El usuario ignoró la sugerencia — no hacemos nada con los campos
    // porque puede que ya los haya llenado manualmente
  }
}
```

### 13.6 Componente de resumen de historial

```typescript
// src/app/features/solicitudes/components/resumen-ia/resumen-ia.component.ts

import { Component, inject, input, signal } from '@angular/core';
import { CommonModule } from '@angular/common';
import { AsistenteService } from '../../../../core/services/asistente.service';
import { ResumenSolicitud } from '../../../../core/models/ia.models';

@Component({
  selector: 'app-resumen-ia',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div class="resumen-container">

      @if (asistenteService.iaDisponible()) {
        <button
          type="button"
          class="btn-resumen-ia"
          [disabled]="cargando()"
          (click)="generarResumen()">

          @if (cargando()) {
            <span class="spinner"></span> Generando resumen...
          } @else {
            📋 Generar resumen con IA
          }
        </button>
      }

      @if (resumen()) {
        <div class="resumen-card">
          <h4>📋 Resumen generado por IA</h4>

          <div class="resumen-seccion">
            <label>Resumen del caso:</label>
            <p>{{ resumen()!.resumen }}</p>
          </div>

          <div class="resumen-seccion">
            <label>Estado actual:</label>
            <p class="estado-actual">{{ resumen()!.estadoActual }}</p>
          </div>

          <div class="resumen-seccion">
            <label>Siguiente paso sugerido:</label>
            <p class="siguiente-paso">{{ resumen()!.siguientePaso }}</p>
          </div>

          <small class="disclaimer">
            Generado automáticamente. Puede contener imprecisiones.
          </small>
        </div>
      }

    </div>
  `
})
export class ResumenIaComponent {

  readonly solicitudId = input.required<number>();

  readonly cargando = signal(false);
  readonly resumen = signal<ResumenSolicitud | null>(null);

  readonly asistenteService = inject(AsistenteService);

  generarResumen(): void {
    this.cargando.set(true);
    this.resumen.set(null);

    this.asistenteService.resumirHistorial(this.solicitudId()).subscribe({
      next: (resultado) => {
        this.cargando.set(false);
        this.resumen.set(resultado);
      },
      error: () => {
        this.cargando.set(false);
      }
    });
  }
}
```

---

## 14. Costos, privacidad y consideraciones éticas

### 14.1 ¿Cuánto cuesta usar la IA?

Entender los costos es fundamental si vas a desplegar esto en un entorno real:

| Modelo | Costo por 1M tokens de entrada | Costo por 1M tokens de salida |
|---|---|---|
| GPT-4o-mini | ~$0.15 USD | ~$0.60 USD |
| GPT-4o | ~$5.00 USD | ~$15.00 USD |
| Claude 3.5 Haiku | ~$0.80 USD | ~$4.00 USD |
| Claude Sonnet 4 | ~$3.00 USD | ~$15.00 USD |
| Llama 3 (local) | $0 | $0 |

*Los precios cambian frecuentemente. Verifica siempre en la página oficial del proveedor.*

**Para nuestro caso de Triage:**

Un prompt de clasificación tiene aproximadamente:
- Template del sistema: ~200 tokens
- Descripción del estudiante: ~100 tokens promedio
- Respuesta del LLM: ~80 tokens

Total ≈ 380 tokens por clasificación. Con GPT-4o-mini, eso es $0.000057 USD por clasificación. Muy económico.

Para el resumen del historial el prompt es más largo (incluye todas las entradas del historial), pero sigue siendo centavos por solicitud.

**Estimación para 1.400 estudiantes usando el sistema:**

Si cada estudiante crea 2 solicitudes por semestre y usa la IA una vez por solicitud:
1.400 × 2 × 380 tokens ≈ 1.064.000 tokens ≈ $0.16 USD por semestre.

Para el sistema académico de la Universidad del Quindío, esto es prácticamente gratis.

### 14.2 Privacidad de los datos

**¿Qué datos enviamos a OpenAI/Anthropic?**

En nuestro sistema enviamos:
- La descripción del estudiante (texto libre).
- El historial de acciones (usuario que realizó la acción, fecha, observación).

**Esto puede incluir información personal identificable (PII).**

Acciones que debes considerar en un sistema real:

1. **Revisar los términos de servicio del proveedor:** OpenAI y Anthropic tienen políticas de datos diferentes. Verifica si usan tus datos para entrenar sus modelos (algunos contratos empresariales lo excluyen).

2. **Anonimizar antes de enviar:** En lugar de enviar `"Juan García solicitó..."`, enviar `"El estudiante solicitó..."` o usar IDs.

3. **Notificar a los usuarios:** Los estudiantes deben saber que su descripción puede ser procesada por un tercero.

4. **Almacenar solo localmente:** Considera modelos locales (Ollama) para datos sensibles.

```java
// Ejemplo de anonimización simple antes de enviar al LLM:
private List<String> formatearHistorialAnonimizado(Solicitud solicitud) {
    return solicitud.getHistorial().stream()
            .sorted(...)
            .map(entrada -> String.format(
                    "[%s] Un usuario realizó la acción '%s'. Observaciones: %s",
                    // No incluimos el nombre del usuario
                    entrada.getFechaAccion().format(FORMATTER),
                    entrada.getAccion(),
                    entrada.getObservaciones() != null ? entrada.getObservaciones() : "Sin observaciones"
            ))
            .toList();
}
```

### 14.3 Consideraciones éticas

La integración de IA en sistemas que afectan a personas reales trae responsabilidades:

**1. La IA asiste, no decide**

En el Triage, la IA solo *sugiere* tipo y prioridad. La decisión final es del funcionario humano. El sistema lo refuerza en múltiples niveles:
- En el backend: el endpoint devuelve una "sugerencia", no una clasificación definitiva.
- En el frontend: los campos del formulario son editables incluso después de aceptar la sugerencia.
- En la UI: hay un disclaimer visible que dice que la clasificación final la determina el funcionario.

**2. Sesgos del modelo**

Los LLMs pueden tener sesgos en sus clasificaciones. Por ejemplo, pueden darle menos prioridad a ciertos tipos de solicitudes basándose en patrones estadísticos de su entrenamiento. Monitorea si hay patrones inusuales en las sugerencias.

**3. Alucinaciones**

Los LLMs pueden generar información que suena plausible pero es incorrecta. Por eso:
- El nivel de confianza es auto-reportado por el modelo (no es una medida técnica real).
- Siempre hay un humano que valida.
- El resumen del historial no debe usarse como fuente de verdad; el historial real en la base de datos sí.

**4. Accesibilidad**

El sistema debe funcionar igual para todos los estudiantes, tengan o no acceso a la funcionalidad de IA. RF-11 garantiza esto técnicamente.

---

## 15. Errores comunes y troubleshooting

### Error 1: `No qualifying bean of type 'ChatClient.Builder'`

**Síntoma:** El servidor no arranca.

**Causa:** La dependencia de Spring AI no está en el classpath, pero `AiConfig` intenta inyectar `ChatClient.Builder` sin `required = false`.

**Solución:** Asegúrate de usar `@Autowired(required = false)` en el parámetro de `AiConfig`:

```java
@Autowired(required = false) ChatClient.Builder chatClientBuilder
```

---

### Error 2: `401 Unauthorized` desde OpenAI

**Síntoma:** El log muestra un error de OpenAI con status 401.

**Causa:** La API key está mal configurada.

**Diagnóstico:**

```bash
# Verificar que la variable de entorno está configurada:
echo $OPENAI_API_KEY

# Probar directamente desde la terminal:
curl https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY"
```

**Solución:** Asegúrate de que `OPENAI_API_KEY` esté definida como variable de entorno antes de arrancar el servidor.

---

### Error 3: El LLM no devuelve JSON válido

**Síntoma:** `com.fasterxml.jackson.core.JsonParseException` en los logs.

**Causa:** El modelo ignoró las instrucciones y devolvió texto libre en lugar de JSON.

**Diagnóstico:** Activa logging DEBUG para ver la respuesta cruda del LLM:

```yaml
# application.yml
logging:
  level:
    co.uniquindio.triage.ai: DEBUG
```

**Solución:**

1. Mejorar el prompt para ser más explícito (agregar ejemplos — few-shot).
2. Bajar la temperatura (más cercana a 0).
3. El método `limpiarJson()` ya maneja los bloques markdown; si el problema persiste, agrega más limpieza.

```java
// Limpieza más agresiva si el modelo es inconsistente:
private String limpiarJson(String respuesta) {
    if (respuesta == null) return "{}";
    var limpio = respuesta.strip();
    // Extraer el primer bloque JSON que encuentre entre { y }
    int inicio = limpio.indexOf('{');
    int fin = limpio.lastIndexOf('}');
    if (inicio >= 0 && fin > inicio) {
        return limpio.substring(inicio, fin + 1);
    }
    return limpio;
}
```

---

### Error 4: Timeout frecuente

**Síntoma:** Los logs muestran `SocketTimeoutException` frecuentemente.

**Causa:** GPT-4o-mini suele responder en 1-3 segundos, pero a veces OpenAI tiene latencia alta.

**Soluciones:**
1. Aumentar `read-timeout` a 45s o 60s.
2. Reducir `max-tokens` para obtener respuestas más cortas.
3. Considera mostrar un loading state en el frontend mientras espera.

---

### Error 5: Enums no reconocidos

**Síntoma:** `InvalidDefinitionException: Cannot construct instance of TipoSolicitud`

**Causa:** El LLM devolvió un valor de enum que no existe en tu código Java. Por ejemplo `"CANCELACION_DE_MATERIAS"` en lugar de `"CANCELACION_ASIGNATURAS"`.

**Solución:** Usar `READ_UNKNOWN_ENUM_VALUES_AS_NULL` en el ObjectMapper (ya documentado en la sección 12.3) y validar que los campos no sean null después del parseo.

---

### Error 6: CORS al llamar al endpoint de IA desde Angular

**Síntoma:** Error de CORS solo en los endpoints `/api/v1/asistente/*`.

**Causa:** Estos endpoints son nuevos y pueden no estar en la configuración de CORS.

**Solución:** Verificar que tu configuración de CORS en Spring Security permite todas las rutas `/api/v1/**`:

```java
// En SecurityConfig:
.requestMatchers("/api/v1/**").authenticated()
```

Y que `allowedOrigins` incluye la URL de Angular (`http://localhost:4200` en dev).

---

## 16. Resumen y cheat sheet

### Flujo completo de RF-10 (clasificación con IA)

```
BACKEND:
  AsistenteController.sugerirClasificacion(descripcion)
    → AsistenteService.sugerirClasificacion(descripcion)
      → AsistenteAcademicoPort.sugerirClasificacion(descripcion)
        → [SpringAiAsistenteAdapter] ChatClient → OpenAI API → JSON
            → parseado a SugerenciaClasificacionResponse
        O → [NoOpAsistenteAdapter] Optional.empty()
    → Optional<SugerenciaClasificacionResponse>
  → HTTP 200 con la sugerencia | HTTP 503 si vacío

FRONTEND:
  Usuario escribe descripción
  → clic en "Sugerir con IA"
  → AsistenteService.sugerirClasificacion(descripcion)
  → HTTP POST /api/v1/asistente/clasificacion
  → mostrar SugerenciaIaComponent con los resultados
  → usuario hace clic en "Aceptar" → form.patchValue()
  → usuario edita si quiere → envía el formulario
```

### Checklist de implementación

- [ ] Dependencia `spring-ai-openai-spring-boot-starter` en `pom.xml`
- [ ] `OPENAI_API_KEY` configurada como variable de entorno
- [ ] `triage.ai.enabled` configurado en `application.yml`
- [ ] `AiProperties` record con `@ConfigurationProperties`
- [ ] `AsistenteAcademicoPort` interfaz definida
- [ ] `NoOpAsistenteAdapter` implementado (RF-11)
- [ ] `SpringAiAsistenteAdapter` implementado con manejo de errores
- [ ] `AiConfig` registra el bean correcto según la configuración
- [ ] Templates de prompts en `resources/prompts/`
- [ ] `AsistenteService` implementado
- [ ] `AsistenteController` con endpoints documentados en Swagger
- [ ] Interfaces TypeScript en Angular (`ia.models.ts`)
- [ ] `AsistenteService` en Angular con manejo de 503
- [ ] `SugerenciaIaComponent` integrado en el formulario
- [ ] `ResumenIaComponent` integrado en la vista de detalle de solicitud
- [ ] Logging configurado para trazabilidad de llamadas a IA
- [ ] Probado el flujo sin API key (debe funcionar con NoOp)
- [ ] Probado el flujo con API key válida

### Comparativa: con IA vs sin IA

| Aspecto | Con IA habilitada | Sin IA (NoOp) |
|---|---|---|
| `estaDisponible()` | `true` | `false` |
| `sugerirClasificacion()` | `Optional<SugerenciaClasificacion>` | `Optional.empty()` |
| `resumirHistorial()` | `Optional<ResumenSolicitud>` | `Optional.empty()` |
| Endpoint `/asistente/disponible` | `{"disponible": true}` | `{"disponible": false}` |
| Endpoint `/asistente/clasificacion` | HTTP 200 con sugerencia | HTTP 503 |
| Botón "Sugerir con IA" en Angular | Visible y activo | Oculto |
| Sistema funciona | ✅ Sí | ✅ Sí |

---

## 17. Ejercicios prácticos

### Ejercicio 1 — Configurar y probar con Ollama (sin costo)

Instala Ollama en tu máquina local (https://ollama.com), descarga el modelo `llama3.2`:

```bash
ollama pull llama3.2
ollama run llama3.2  # Prueba que funciona
```

Luego cambia la dependencia en `pom.xml` a `spring-ai-ollama-spring-boot-starter` y configura:

```yaml
spring:
  ai:
    ollama:
      base-url: http://localhost:11434
      chat:
        options:
          model: llama3.2
          temperature: 0.2
```

Crea un segundo adaptador `OllamaAsistenteAdapter` que use esta configuración y modifica `AiConfig` para seleccionarlo cuando `triage.ai.provider=ollama`. Esto demostrará el poder del patrón Strategy: cero cambios en el service, cero cambios en el controller.

---

### Ejercicio 2 — Few-shot prompting para mejorar la precisión

El template actual de clasificación no tiene ejemplos. Agrega 3-4 ejemplos de pocas palabras (few-shot) al prompt y mide si la precisión mejora:

```
EJEMPLOS DE CLASIFICACIÓN:
- Descripción: "necesito inscribir la materia de Redes ya que me quedé sin cupo"
  → {"tipoSolicitudSugerido": "SOLICITUD_CUPOS", "prioridadSugerida": "ALTA", ...}

- Descripción: "quiero saber cuántas materias me faltan para graduarme"
  → {"tipoSolicitudSugerido": "CONSULTA_ACADEMICA", "prioridadSugerida": "BAJA", ...}
```

Prueba con al menos 10 descripciones distintas antes y después de agregar los ejemplos. ¿Mejora la consistencia del JSON? ¿Mejora la selección de tipo?

---

### Ejercicio 3 — Auditoría de sugerencias de IA

Toda sugerencia de IA que el usuario acepte debería quedar registrada. Implementa lo siguiente:

1. Agrega a la entidad `HistorialSolicitud` un campo `fueAsistidoPorIa: Boolean` y `confianzaIa: Integer`.
2. Cuando el estudiante registra una solicitud con la sugerencia de IA aceptada, el frontend envía un flag `asistidoPorIa: true` y el porcentaje de confianza en el request body.
3. El backend guarda esta información en el historial.
4. En el panel de administración, agrega una estadística: "Porcentaje de solicitudes donde se usó IA".

---

### Ejercicio 4 — Caché de sugerencias

Si dos estudiantes describen el mismo problema con palabras similares, ¿tiene sentido llamar a OpenAI dos veces? Implementa un caché simple usando Spring Cache:

```java
@Cacheable(value = "sugerencias-ia", key = "#descripcion.toLowerCase().replaceAll('\\s+', ' ').trim()")
public Optional<SugerenciaClasificacionResponse> sugerirClasificacion(String descripcion) {
    // ...
}
```

Configura el caché con un TTL de 1 hora (usando Caffeine o el caché en memoria de Spring). ¿Qué ventajas y desventajas tiene esto?

---

### Ejercicio 5 — Streaming de respuestas

Para la funcionalidad de resumen (RF-09), el historial puede ser largo y el LLM puede tardar varios segundos. Implementa streaming usando la API de Spring AI y Server-Sent Events en Spring Boot, para que el texto del resumen aparezca progresivamente en el frontend en lugar de que el usuario espere con una pantalla en blanco.

Pista: Spring AI tiene `chatClient.prompt().user(prompt).stream().content()` que retorna un `Flux<String>`.

---

## 18. Referencias y recursos adicionales

- **Spring AI — Documentación oficial:** https://docs.spring.io/spring-ai/reference/
- **Spring AI GitHub:** https://github.com/spring-projects/spring-ai
- **OpenAI API Reference:** https://platform.openai.com/docs/api-reference
- **Anthropic API Reference:** https://docs.anthropic.com/en/api/getting-started
- **Ollama — Modelos locales:** https://ollama.com
- **OpenAI Tokenizer (estimar costos):** https://platform.openai.com/tokenizer
- **Prompt Engineering Guide:** https://www.promptingguide.ai/es
- **RF-09, RF-10, RF-11** — Requisitos funcionales del proyecto de Triage (ver Guía Maestra).
- **Guía 06** — API REST con Spring Boot (para entender los controladores base).
- **Guía 07** — Spring Security (para los roles FUNCIONARIO/ADMIN en los endpoints de IA).
- **Guía 09** — Angular Avanzado (para los servicios HTTP e interceptores en el frontend).

---

> **Autor:** José Alfredo Ramírez Espinosa  
> **Materia:** Programación Avanzada  
> **Programa:** Ingeniería de Sistemas y Computación  
> **Universidad del Quindío** | Armenia, Colombia  
> **Año:** 2026  
> *Material de uso académico. Todos los derechos reservados al autor.*
