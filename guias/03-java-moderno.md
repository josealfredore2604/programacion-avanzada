# Guía 03 — Java Moderno (Java 17–21): Features que Necesitas para Spring Boot

> **Materia:** Programación Avanzada | **Programa:** Ingeniería de Sistemas y Computación  
> **Universidad del Quindío** | Armenia, Colombia | **Año:** 2026  
> **Autor:** José Alfredo Ramírez Espinosa

---

## Tabla de Contenidos

1. [Objetivos de Aprendizaje](#1-objetivos-de-aprendizaje)
2. [Prerrequisitos](#2-prerrequisitos)
3. [Introducción: ¿Por qué Java moderno importa?](#3-introducción-por-qué-java-moderno-importa)
4. [Records: Adiós al Boilerplate](#4-records-adiós-al-boilerplate)
   - 4.1 [El problema con las clases POJO tradicionales](#41-el-problema-con-las-clases-pojo-tradicionales)
   - 4.2 [¿Qué es un Record?](#42-qué-es-un-record)
   - 4.3 [Records en la práctica: DTOs del Triage](#43-records-en-la-práctica-dtos-del-triage)
   - 4.4 [Limitaciones de los Records](#44-limitaciones-de-los-records)
   - 4.5 [Tabla comparativa: Clase POJO vs Record](#45-tabla-comparativa-clase-pojo-vs-record)
5. [Sealed Classes e Interfaces: Modelar con Precisión](#5-sealed-classes-e-interfaces-modelar-con-precisión)
   - 5.1 [El problema que resuelven](#51-el-problema-que-resuelven)
   - 5.2 [Sintaxis y reglas](#52-sintaxis-y-reglas)
   - 5.3 [Aplicación al Triage: TipoSolicitud sellada](#53-aplicación-al-triage-tiposolicitud-sellada)
6. [Pattern Matching: Switch y instanceof Modernos](#6-pattern-matching-switch-e-instanceof-modernos)
   - 6.1 [Pattern Matching con instanceof](#61-pattern-matching-con-instanceof)
   - 6.2 [Switch Expressions (Java 14+)](#62-switch-expressions-java-14)
   - 6.3 [Pattern Matching en switch (Java 21)](#63-pattern-matching-en-switch-java-21)
   - 6.4 [Aplicación al Triage](#64-aplicación-al-triage)
7. [Text Blocks: Strings Multilinea sin Sufrimiento](#7-text-blocks-strings-multilinea-sin-sufrimiento)
   - 7.1 [Antes de Text Blocks](#71-antes-de-text-blocks)
   - 7.2 [Sintaxis de Text Blocks](#72-sintaxis-de-text-blocks)
   - 7.3 [Aplicación al Triage: Queries y JSON de prueba](#73-aplicación-al-triage-queries-y-json-de-prueba)
8. [var: Inferencia de Tipos Local](#8-var-inferencia-de-tipos-local)
   - 8.1 [Qué es var y cómo funciona](#81-qué-es-var-y-cómo-funciona)
   - 8.2 [Cuándo usarlo y cuándo NO](#82-cuándo-usarlo-y-cuándo-no)
9. [Optional: El Fin de los NullPointerException](#9-optional-el-fin-de-los-nullpointerexception)
   - 9.1 [El problema del null](#91-el-problema-del-null)
   - 9.2 [Creación de Optional](#92-creación-de-optional)
   - 9.3 [Métodos esenciales](#93-métodos-esenciales)
   - 9.4 [Antipatrones comunes](#94-antipatrones-comunes)
   - 9.5 [Optional en servicios del Triage](#95-optional-en-servicios-del-triage)
10. [Stream API: Procesamiento Funcional de Colecciones](#10-stream-api-procesamiento-funcional-de-colecciones)
    - 10.1 [¿Qué es un Stream?](#101-qué-es-un-stream)
    - 10.2 [Operaciones intermedias](#102-operaciones-intermedias)
    - 10.3 [Operaciones terminales](#103-operaciones-terminales)
    - 10.4 [Collectors avanzados: groupingBy, partitioningBy, counting](#104-collectors-avanzados-groupingby-partitioningby-counting)
    - 10.5 [Streams paralelos: cuándo y por qué (no siempre)](#105-streams-paralelos-cuándo-y-por-qué-no-siempre)
    - 10.6 [Caso completo: Procesar listas de solicitudes del Triage](#106-caso-completo-procesar-listas-de-solicitudes-del-triage)
11. [Nuevas APIs de Colecciones](#11-nuevas-apis-de-colecciones)
    - 11.1 [Colecciones inmutables](#111-colecciones-inmutables)
    - 11.2 [copyOf y toList()](#112-copyof-y-tolist)
    - 11.3 [Map.entry y Map.ofEntries](#113-mapentry-y-mapofentries)
12. [Tabla Comparativa: Java 8 → Java 21](#12-tabla-comparativa-java-8--java-21)
13. [Modelando el Proyecto Triage con Java Moderno](#13-modelando-el-proyecto-triage-con-java-moderno)
    - 13.1 [Estructura de carpetas de los modelos](#131-estructura-de-carpetas-de-los-modelos)
    - 13.2 [Enums y tipos del dominio](#132-enums-y-tipos-del-dominio)
    - 13.3 [Records como DTOs del Triage](#133-records-como-dtos-del-triage)
    - 13.4 [Clase de utilidades con Streams y Optional](#134-clase-de-utilidades-con-streams-y-optional)
14. [Ejercicios Prácticos](#14-ejercicios-prácticos)
15. [Errores Comunes y Troubleshooting](#15-errores-comunes-y-troubleshooting)
16. [Resumen y Cheat Sheet](#16-resumen-y-cheat-sheet)
17. [Referencias y Recursos Adicionales](#17-referencias-y-recursos-adicionales)

---

## 1. Objetivos de Aprendizaje

Al terminar esta guía serás capaz de:

- Crear Records de Java para modelar DTOs inmutables sin código repetitivo.
- Usar Sealed Classes para representar jerarquías de tipos cerradas y seguras.
- Aplicar Pattern Matching con `instanceof` y `switch` para escribir código más expresivo.
- Trabajar con Text Blocks para strings multilinea legibles.
- Usar `var` de forma correcta y saber cuándo evitarlo.
- Manejar valores potencialmente nulos con `Optional` sin caer en antipatrones.
- Procesar colecciones de solicitudes usando el Stream API con `filter`, `map`, `flatMap`, `collect`, `groupingBy` y operadores avanzados de RxJS.
- Crear colecciones inmutables con las nuevas APIs de Java.
- Modelar todas las entidades y DTOs del proyecto de Triage usando estas características modernas.

---

## 2. Prerrequisitos

Antes de continuar, asegúrate de:

- Tener instalado **JDK 21** o superior. Puedes verificarlo con:
  ```bash
  java -version
  # Debería mostrar: openjdk version "21.x.x" o superior
  ```
- Conocer los fundamentos de Java: clases, herencia, interfaces, colecciones (`ArrayList`, `HashMap`), ciclos y métodos.
- Haber leído la **Guía 01** (Arquitecturas de Software) y la **Guía 02** (Patrones de Diseño) para tener contexto del proyecto base.
- Tener un IDE configurado: **IntelliJ IDEA** (recomendado) o VS Code con el Extension Pack for Java.

> **Nota:** Esta guía no usa Spring Boot aún. Todo el código se puede ejecutar en proyectos Java puros. La conexión con Spring Boot se hace en la Guía 04. Sin embargo, cada concepto aquí se introduce exactamente pensando en cómo lo vas a usar después.

---

## 3. Introducción: ¿Por qué Java Moderno Importa?

Cuando aprendiste Java por primera vez, probablemente escribiste cosas como esta:

```java
public class EstudianteDTO {
    private String nombre;
    private String correo;
    private String codigo;

    public EstudianteDTO(String nombre, String correo, String codigo) {
        this.nombre = nombre;
        this.correo = correo;
        this.codigo = codigo;
    }

    public String getNombre() { return nombre; }
    public String getCorreo() { return correo; }
    public String getCodigo() { return codigo; }

    @Override
    public boolean equals(Object o) { /* 15 líneas más... */ }

    @Override
    public int hashCode() { /* 5 líneas más... */ }

    @Override
    public String toString() { /* 3 líneas más... */ }
}
```

Eso son **más de 30 líneas de código** para representar un objeto con 3 campos. Y si agregabas un campo, tenías que actualizar el constructor, el getter, el `equals`, el `hashCode` y el `toString` a mano (o confiar en que el IDE lo hacía por ti, con el riesgo de olvidarlo).

Java ha evolucionado enormemente desde la versión 8. Las versiones 14, 16, 17 (LTS), y 21 (LTS) trajeron características que hacen el código más conciso, seguro, expresivo y fácil de mantener. Y lo mejor: **Spring Boot 3.4+ aprovecha estas características al máximo**.

En esta guía vas a aprender exactamente las características que necesitas para desarrollar el proyecto de Triage de forma profesional. Nada de teoría innecesaria: todo lo que aquí aparece lo vas a usar directamente en el código del sistema.

---

## 4. Records: Adiós al Boilerplate

### 4.1 El Problema con las Clases POJO Tradicionales

Un **POJO** (Plain Old Java Object) es una clase Java simple que solo almacena datos: tiene campos, constructor, getters, y opcionalmente `equals`, `hashCode` y `toString`.

El problema es que en un sistema empresarial como el Triage vas a necesitar *decenas* de estas clases: una para registrar una solicitud, otra para responder al frontend con los datos de la solicitud, otra para recibir los datos de login, otra para el historial...

Escribir ese código a mano es tedioso, propenso a errores y difícil de mantener.

### 4.2 ¿Qué es un Record?

Un **Record** de Java (introducido en Java 14 como preview, estable desde Java 16) es una clase especial diseñada para transportar datos inmutables. Cuando declaras un Record, Java genera automáticamente:

- Un constructor canónico (con todos los campos).
- Getters para cada campo (sin el prefijo `get`, se llaman igual que el campo).
- `equals()` que compara campo por campo.
- `hashCode()` coherente con `equals()`.
- `toString()` legible.

**Analogía:** Piensa en un Record como un formulario impreso. Una vez que lo llenas (lo construyes), no puedes borrar y re-escribir los campos. Es solo lectura. Su propósito es llevar información de un lugar a otro.

### 4.3 Records en la Práctica: DTOs del Triage

Imaginemos que necesitamos un DTO para recibir los datos cuando un estudiante registra una nueva solicitud académica (corresponde al **RF-01** de nuestro proyecto).

**Con clase tradicional:**

```java
// ❌ Manera antigua - demasiado código
public class RegistrarSolicitudRequest {
    private String tipoSolicitud;
    private String descripcion;
    private String canalOrigen;
    private String identificacionSolicitante;

    public RegistrarSolicitudRequest(String tipoSolicitud, String descripcion,
                                      String canalOrigen, String identificacionSolicitante) {
        this.tipoSolicitud = tipoSolicitud;
        this.descripcion = descripcion;
        this.canalOrigen = canalOrigen;
        this.identificacionSolicitante = identificacionSolicitante;
    }

    public String getTipoSolicitud() { return tipoSolicitud; }
    public String getDescripcion() { return descripcion; }
    public String getCanalOrigen() { return canalOrigen; }
    public String getIdentificacionSolicitante() { return identificacionSolicitante; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof RegistrarSolicitudRequest that)) return false;
        return Objects.equals(tipoSolicitud, that.tipoSolicitud) &&
               Objects.equals(descripcion, that.descripcion) &&
               Objects.equals(canalOrigen, that.canalOrigen) &&
               Objects.equals(identificacionSolicitante, that.identificacionSolicitante);
    }

    @Override
    public int hashCode() {
        return Objects.hash(tipoSolicitud, descripcion, canalOrigen, identificacionSolicitante);
    }

    @Override
    public String toString() {
        return "RegistrarSolicitudRequest{" +
               "tipoSolicitud='" + tipoSolicitud + '\'' +
               ", descripcion='" + descripcion + '\'' +
               ", canalOrigen='" + canalOrigen + '\'' +
               ", identificacionSolicitante='" + identificacionSolicitante + '\'' +
               '}';
    }
}
```

**Con Record:**

```java
// ✅ Manera moderna - todo en 1 línea
public record RegistrarSolicitudRequest(
    String tipoSolicitud,
    String descripcion,
    String canalOrigen,
    String identificacionSolicitante
) {}
```

Eso es todo. Esa única declaración genera el mismo código que las 50 líneas de arriba. Vamos a explorar cómo usarlo:

```java
public class EjemploRecord {
    public static void main(String[] args) {

        // Construcción: se usa el constructor canónico
        var solicitud = new RegistrarSolicitudRequest(
            "HOMOLOGACION",
            "Solicito homologar la materia Cálculo I cursada en la Universidad X",
            "CORREO",
            "1088123456"
        );

        // Getters: el nombre del campo directamente, sin "get"
        System.out.println(solicitud.tipoSolicitud());         // HOMOLOGACION
        System.out.println(solicitud.descripcion());           // Solicito homologar...
        System.out.println(solicitud.canalOrigen());           // CORREO
        System.out.println(solicitud.identificacionSolicitante()); // 1088123456

        // toString() automático
        System.out.println(solicitud);
        // RegistrarSolicitudRequest[tipoSolicitud=HOMOLOGACION, descripcion=Solicito..., ...]

        // equals() automático: compara campo por campo
        var solicitud2 = new RegistrarSolicitudRequest(
            "HOMOLOGACION",
            "Solicito homologar la materia Cálculo I cursada en la Universidad X",
            "CORREO",
            "1088123456"
        );
        System.out.println(solicitud.equals(solicitud2)); // true

        // Los Records son INMUTABLES: no puedes hacer solicitud.tipoSolicitud = "otro";
        // Eso genera error de compilación. Esto es una ventaja, no un defecto.
    }
}
```

#### Records con validación en el constructor compacto

¿Qué pasa si necesitas validar que los campos no estén vacíos? Los Records tienen lo que se llama un **constructor compacto** donde puedes agregar lógica de validación:

```java
// Ahora vamos a crear el DTO de respuesta para una solicitud registrada (RF-01)
// Este es lo que el backend devuelve al frontend después de registrar exitosamente.
public record SolicitudCreadaResponse(
    Long id,
    String tipoSolicitud,
    String descripcion,
    String canalOrigen,
    String estado,
    String fechaRegistro,
    String identificacionSolicitante
) {
    // Constructor compacto: sin paréntesis, la validación va aquí
    // Los parámetros ya fueron asignados a los campos; aquí solo los validamos.
    public SolicitudCreadaResponse {
        if (id == null || id <= 0) {
            throw new IllegalArgumentException("El ID de la solicitud debe ser positivo");
        }
        if (descripcion == null || descripcion.isBlank()) {
            throw new IllegalArgumentException("La descripción no puede estar vacía");
        }
        // Normalización: puedes modificar los valores antes de que se asignen
        tipoSolicitud = tipoSolicitud != null ? tipoSolicitud.toUpperCase() : null;
        descripcion = descripcion.trim();
    }
}
```

#### Records con métodos adicionales

Un Record puede tener métodos adicionales (pero no puede tener campos de instancia adicionales ni ser mutable):

```java
public record SolicitudResumenResponse(
    Long id,
    String tipo,
    String prioridad,
    String estado,
    String solicitante
) {
    // Método que determina si la solicitud es urgente
    public boolean esUrgente() {
        return "ALTA".equalsIgnoreCase(prioridad) || "CRITICA".equalsIgnoreCase(prioridad);
    }

    // Método de fábrica estático (útil para crear desde una entidad)
    public static SolicitudResumenResponse desdeEntidad(/* Solicitud entidad - se verá en Guía 05 */) {
        // Por ahora, ejemplo simplificado
        return new SolicitudResumenResponse(1L, "HOMOLOGACION", "ALTA", "REGISTRADA", "Juan Pérez");
    }
}
```

#### Records con anotaciones (compatibles con Jackson y Bean Validation)

Spring Boot usa Jackson para serializar/deserializar JSON. Los Records son totalmente compatibles. También puedes usar anotaciones de Bean Validation:

```java
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;

// DTO de entrada para registrar solicitud (RF-01) con validaciones
public record RegistrarSolicitudRequest(

    @NotNull(message = "El tipo de solicitud es obligatorio")
    String tipoSolicitud,

    @NotBlank(message = "La descripción no puede estar vacía")
    @Size(min = 20, max = 1000, message = "La descripción debe tener entre 20 y 1000 caracteres")
    String descripcion,

    @NotNull(message = "El canal de origen es obligatorio")
    String canalOrigen,

    @NotBlank(message = "La identificación del solicitante es obligatoria")
    String identificacionSolicitante

) {}
```

> **Nota importante:** Para que las anotaciones de validación funcionen en los parámetros del constructor de un Record, asegúrate de tener la dependencia `spring-boot-starter-validation` en tu `pom.xml`. Lo veremos en la Guía 04 y 05.

### 4.4 Limitaciones de los Records

Los Records son poderosos pero tienen limitaciones que debes conocer:

- **No pueden extender otras clases.** Un Record implícitamente extiende `java.lang.Record`. Sin embargo, sí pueden implementar interfaces.
- **Sus campos son finales.** No puedes cambiar el valor de un campo después de la construcción. Esto es intencional y es una ventaja de diseño.
- **No puedes agregar campos de instancia adicionales** más allá de los declarados en el encabezado.
- **No son buenas entidades JPA.** Las entidades de base de datos necesitan ser mutables (JPA/Hibernate necesita setters o acceso a campos). Para entidades, seguirás usando clases normales. Los Records son para DTOs.

### 4.5 Tabla Comparativa: Clase POJO vs Record

| Característica | Clase POJO | Record |
|---|---|---|
| Líneas de código | 30-50 por clase | 3-5 por clase |
| Mutabilidad | Mutable (con setters) | Inmutable |
| Constructor | Manual | Automático (canónico) |
| Getters | `getNombre()` | `nombre()` |
| `equals` y `hashCode` | Manual o IDE | Automático |
| `toString` | Manual o IDE | Automático |
| Uso ideal | Entidades JPA, lógica compleja | DTOs, respuestas de API, datos temporales |
| Compatible con Spring Boot | ✅ | ✅ (desde Spring Boot 2.5+) |
| Compatible con Jackson (JSON) | ✅ | ✅ |
| Compatible con Bean Validation | ✅ | ✅ |
| Puede ser entidad JPA | ✅ | ❌ (no es mutable) |

---

## 5. Sealed Classes e Interfaces: Modelar con Precisión

### 5.1 El Problema que Resuelven

Imagina que en el sistema de Triage tienes una interfaz `ResultadoOperacion` y cualquier clase en cualquier lugar del proyecto puede implementarla. Eso dificulta razonar sobre el código: ¿cuántos tipos de resultado existen? ¿Podría alguien agregar un `ResultadoMisterioso` sin que yo lo sepa?

Las **Sealed Classes** (introducidas en Java 17 como estable) resuelven esto: te permiten declarar exactamente qué clases o interfaces pueden extender/implementar a la tuya. Cierran la jerarquía.

**Analogía:** Una Sealed Class es como un formulario con opciones de selección múltiple. No puedes escribir una opción que no esté listada. Solo puedes marcar "Sí", "No" o "Pendiente" — nada más.

### 5.2 Sintaxis y Reglas

```java
// Declarar una sealed interface
public sealed interface ResultadoOperacion
    permits ResultadoExitoso, ResultadoFallido, ResultadoPendiente {

    String mensaje();
}

// Las clases permitidas deben usar una de tres palabras clave:
// - final: no puede ser extendida más
// - sealed: puede ser extendida pero con permits
// - non-sealed: puede ser extendida sin restricciones (abre la jerarquía)

public record ResultadoExitoso(String mensaje, Object datos) implements ResultadoOperacion {}
public record ResultadoFallido(String mensaje, String codigoError) implements ResultadoOperacion {}
public record ResultadoPendiente(String mensaje, Long solicitudId) implements ResultadoOperacion {}
```

Reglas importantes:
- Las clases permitidas (`permits`) deben estar en el mismo paquete o módulo.
- Cada clase permitida debe ser `final`, `sealed`, o `non-sealed`.
- Si todas las clases permitidas están en el mismo archivo, el compilador infiere el `permits` automáticamente.

### 5.3 Aplicación al Triage: TipoSolicitud Sellada

En nuestro proyecto, los tipos de solicitud son un conjunto cerrado y conocido (RF-02). Con una Sealed Interface podemos modelar esto con seguridad de tipos:

```java
// Jerarquía sellada para los tipos de solicitud del Triage
// Cada tipo puede tener sus propias reglas de prioridad y datos específicos

public sealed interface TipoSolicitudDetalle
    permits SolicitudHomologacion, SolicitudCupo,
            SolicitudCancelacion, SolicitudConsulta, SolicitudRegistroAsignatura {

    String codigoTipo();
    int diasMaximoAtencion();   // SLA por tipo de solicitud
    boolean requiereDocumentos();
}

// Cada tipo es un Record (inmutable, con sus datos específicos)
public record SolicitudHomologacion(
    String asignaturaOrigen,
    String universidadOrigen
) implements TipoSolicitudDetalle {

    @Override public String codigoTipo() { return "HOMOLOGACION"; }
    @Override public int diasMaximoAtencion() { return 15; }
    @Override public boolean requiereDocumentos() { return true; }
}

public record SolicitudCupo(
    String codigoAsignatura,
    String grupoSolicitado
) implements TipoSolicitudDetalle {

    @Override public String codigoTipo() { return "CUPO"; }
    @Override public int diasMaximoAtencion() { return 3; }
    @Override public boolean requiereDocumentos() { return false; }
}

public record SolicitudCancelacion(
    String codigoAsignatura,
    String motivoCancelacion
) implements TipoSolicitudDetalle {

    @Override public String codigoTipo() { return "CANCELACION"; }
    @Override public int diasMaximoAtencion() { return 5; }
    @Override public boolean requiereDocumentos() { return false; }
}

public record SolicitudConsulta(
    String temaConsulta
) implements TipoSolicitudDetalle {

    @Override public String codigoTipo() { return "CONSULTA"; }
    @Override public int diasMaximoAtencion() { return 2; }
    @Override public boolean requiereDocumentos() { return false; }
}

public record SolicitudRegistroAsignatura(
    String codigoAsignatura,
    String periodo
) implements TipoSolicitudDetalle {

    @Override public String codigoTipo() { return "REGISTRO_ASIGNATURA"; }
    @Override public int diasMaximoAtencion() { return 7; }
    @Override public boolean requiereDocumentos() { return false; }
}
```

La ventaja real se ve cuando combinas Sealed Classes con Pattern Matching (próxima sección):

```java
// El compilador SABE que estos son TODOS los tipos posibles.
// Si agregas un nuevo tipo y te olvidas manejarlo aquí, el compilador te avisa.
public String generarMensajeBienvenida(TipoSolicitudDetalle tipo) {
    return switch (tipo) {
        case SolicitudHomologacion h ->
            "Tu solicitud de homologación de " + h.asignaturaOrigen() +
            " fue recibida. Requiere documentos de soporte.";
        case SolicitudCupo c ->
            "Solicitaste un cupo en el grupo " + c.grupoSolicitado() +
            " de " + c.codigoAsignatura() + ".";
        case SolicitudCancelacion ca ->
            "Solicitud de cancelación de " + ca.codigoAsignatura() + " recibida.";
        case SolicitudConsulta con ->
            "Tu consulta sobre " + con.temaConsulta() + " fue registrada.";
        case SolicitudRegistroAsignatura r ->
            "Solicitud de registro de " + r.codigoAsignatura() +
            " para el periodo " + r.periodo() + ".";
        // No necesitas default porque el compilador sabe que cubriste todos los casos
    };
}
```

---

## 6. Pattern Matching: Switch e instanceof Modernos

### 6.1 Pattern Matching con instanceof

Antes de Java 16, cuando necesitabas verificar el tipo de un objeto y luego usarlo como ese tipo, tenías que hacer un cast manual:

```java
// ❌ Manera antigua: verbose y propensa a errores
public void procesarResultado(Object resultado) {
    if (resultado instanceof ResultadoExitoso) {
        ResultadoExitoso exitoso = (ResultadoExitoso) resultado; // cast manual
        System.out.println("Exitoso: " + exitoso.datos());
    } else if (resultado instanceof ResultadoFallido) {
        ResultadoFallido fallido = (ResultadoFallido) resultado; // otro cast
        System.out.println("Error: " + fallido.codigoError());
    }
}
```

Con **Pattern Matching para instanceof** (estable desde Java 16):

```java
// ✅ Manera moderna: la variable ya está casteada en el if
public void procesarResultado(Object resultado) {
    if (resultado instanceof ResultadoExitoso exitoso) {
        // "exitoso" es directamente de tipo ResultadoExitoso, sin cast
        System.out.println("Exitoso: " + exitoso.datos());
    } else if (resultado instanceof ResultadoFallido fallido) {
        System.out.println("Error: " + fallido.codigoError());
    } else if (resultado instanceof ResultadoPendiente pendiente) {
        System.out.println("Pendiente, solicitud ID: " + pendiente.solicitudId());
    }
}
```

Incluso puedes agregar condiciones adicionales (llamadas **guards**):

```java
// Pattern matching con guard (condición adicional)
public String clasificarSolicitudPorPrioridad(Object solicitud) {
    if (solicitud instanceof SolicitudHomologacion h && h.requiereDocumentos()) {
        return "Homologación con documentos — prioridad media";
    } else if (solicitud instanceof SolicitudCupo c && c.grupoSolicitado().startsWith("01")) {
        return "Cupo en primer grupo — alta demanda";
    }
    return "Solicitud estándar";
}
```

### 6.2 Switch Expressions (Java 14+)

El `switch` tradicional devolvía void y era propenso a olvidar el `break`. Desde Java 14 hay **switch expressions** que devuelven un valor y son más concisas:

```java
// ❌ switch tradicional: propenso a fall-through y verbose
public String obtenerDescripcionEstado(String estado) {
    String descripcion;
    switch (estado) {
        case "REGISTRADA":
            descripcion = "La solicitud fue recibida y está pendiente de clasificación";
            break;
        case "CLASIFICADA":
            descripcion = "La solicitud fue clasificada y espera ser asignada";
            break;
        case "EN_ATENCION":
            descripcion = "Un funcionario está trabajando en esta solicitud";
            break;
        case "ATENDIDA":
            descripcion = "La solicitud fue procesada exitosamente";
            break;
        case "CERRADA":
            descripcion = "El trámite fue finalizado";
            break;
        default:
            descripcion = "Estado desconocido";
    }
    return descripcion;
}

// ✅ switch expression: devuelve valor, sin break, sin fall-through
public String obtenerDescripcionEstado(String estado) {
    return switch (estado) {
        case "REGISTRADA"   -> "La solicitud fue recibida y está pendiente de clasificación";
        case "CLASIFICADA"  -> "La solicitud fue clasificada y espera ser asignada";
        case "EN_ATENCION"  -> "Un funcionario está trabajando en esta solicitud";
        case "ATENDIDA"     -> "La solicitud fue procesada exitosamente";
        case "CERRADA"      -> "El trámite fue finalizado";
        default             -> "Estado desconocido";
    };
}
```

También puedes usar `yield` para bloques más complejos dentro del switch:

```java
public int calcularDiasMaximoAtencion(String tipoSolicitud) {
    return switch (tipoSolicitud.toUpperCase()) {
        case "HOMOLOGACION" -> 15;
        case "CUPO" -> 3;
        case "CANCELACION" -> 5;
        case "CONSULTA" -> 2;
        case "REGISTRO_ASIGNATURA" -> {
            // Bloque de código → usar yield para devolver el valor
            int base = 7;
            boolean esPeriodoMatricula = verificarPeriodoMatricula(); // método hipotético
            yield esPeriodoMatricula ? base / 2 : base;
        }
        default -> throw new IllegalArgumentException("Tipo desconocido: " + tipoSolicitud);
    };
}
```

### 6.3 Pattern Matching en switch (Java 21)

Java 21 combina lo mejor: switch expressions con pattern matching de tipos. Esto es especialmente poderoso con Sealed Classes porque el compilador verifica exhaustividad:

```java
// Aplicar Pattern Matching en switch a nuestros tipos sellados
public String generarNotificacion(TipoSolicitudDetalle tipo) {
    return switch (tipo) {
        // Pattern matching: extrae el tipo Y tiene acceso a sus campos
        case SolicitudHomologacion h when h.requiereDocumentos() ->
            "Recuerda adjuntar el certificado de notas de " + h.universidadOrigen();

        case SolicitudHomologacion h ->
            "Tu homologación de " + h.asignaturaOrigen() + " está en proceso.";

        case SolicitudCupo c ->
            "Verificaremos disponibilidad en el grupo " + c.grupoSolicitado() + ".";

        case SolicitudCancelacion ca ->
            "La cancelación de " + ca.codigoAsignatura() + " será procesada.";

        case SolicitudConsulta con ->
            "Tu consulta sobre " + con.temaConsulta() + " fue asignada a un asesor.";

        case SolicitudRegistroAsignatura r ->
            "Registrando " + r.codigoAsignatura() + " para el periodo " + r.periodo() + ".";

        // No necesitas default porque el compilador sabe que Sealed Interface tiene 5 implementaciones
        // y las cubriste todas. ¡Seguridad de tipos en tiempo de compilación!
    };
}
```

### 6.4 Aplicación al Triage

Veamos un ejemplo completo que integra todo lo anterior: determinar la prioridad de una solicitud basándose en su tipo (esto corresponde al RF-03):

```java
// Clase de servicio que determina la prioridad de una solicitud
// (Lo convertiremos en un @Service de Spring Boot en la Guía 04 y 06)
public class PriorizadorDeSolicitudes {

    public enum Prioridad { BAJA, MEDIA, ALTA, CRITICA }

    /**
     * Determina la prioridad de una solicitud según su tipo y contexto.
     * Esta es la lógica de negocio del RF-03.
     */
    public Prioridad determinarPrioridad(TipoSolicitudDetalle tipo, boolean proximoAFechaLimite) {
        return switch (tipo) {
            // Una homologación siempre empieza con prioridad media
            case SolicitudHomologacion ignored -> Prioridad.MEDIA;

            // Un cupo es urgente si está próximo al cierre de matrícula
            case SolicitudCupo ignored when proximoAFechaLimite -> Prioridad.CRITICA;
            case SolicitudCupo ignored -> Prioridad.ALTA;

            // Cancelaciones son urgentes siempre (afectan el historial académico)
            case SolicitudCancelacion ignored -> Prioridad.ALTA;

            // Las consultas son de baja prioridad
            case SolicitudConsulta ignored -> Prioridad.BAJA;

            // Registro de asignatura: crítico en periodos de matrícula
            case SolicitudRegistroAsignatura ignored when proximoAFechaLimite -> Prioridad.CRITICA;
            case SolicitudRegistroAsignatura ignored -> Prioridad.MEDIA;
        };
    }
}
```

---

## 7. Text Blocks: Strings Multilinea sin Sufrimiento

### 7.1 Antes de Text Blocks

Antes de Java 15 (donde se estabilizaron), escribir un String multilinea era un calvario:

```java
// ❌ Antes: concatenación con \n y \" por todos lados
String consultaSQL = "SELECT s.id, s.tipo_solicitud, s.descripcion, u.nombre " +
                     "FROM solicitudes s " +
                     "JOIN usuarios u ON s.solicitante_id = u.id " +
                     "WHERE s.estado = 'REGISTRADA' " +
                     "AND s.prioridad = 'ALTA' " +
                     "ORDER BY s.fecha_registro ASC";

String jsonPrueba = "{\n" +
                    "    \"tipoSolicitud\": \"HOMOLOGACION\",\n" +
                    "    \"descripcion\": \"Solicito homologar Cálculo I\",\n" +
                    "    \"canalOrigen\": \"CORREO\",\n" +
                    "    \"identificacionSolicitante\": \"1088123456\"\n" +
                    "}";
```

Difícil de leer, difícil de editar y fácil de olvidar un `\n` o una `\"`.

### 7.2 Sintaxis de Text Blocks

Un Text Block comienza con `"""` seguido de un salto de línea, y termina con `"""`. Preserva el formato del código.

```java
// ✅ Con Text Blocks: limpios y legibles
String consultaSQL = """
        SELECT s.id, s.tipo_solicitud, s.descripcion, u.nombre
        FROM solicitudes s
        JOIN usuarios u ON s.solicitante_id = u.id
        WHERE s.estado = 'REGISTRADA'
        AND s.prioridad = 'ALTA'
        ORDER BY s.fecha_registro ASC
        """;

String jsonPrueba = """
        {
            "tipoSolicitud": "HOMOLOGACION",
            "descripcion": "Solicito homologar Cálculo I",
            "canalOrigen": "CORREO",
            "identificacionSolicitante": "1088123456"
        }
        """;
```

**Reglas de indentación:** Java elimina automáticamente el espacio en blanco de la izquierda común a todas las líneas. Esto significa que puedes indentar el Text Block para que se vea bien en tu código sin que esa indentación quede en el String final.

También puedes usar interpolación con `formatted()`:

```java
// Interpolación de valores en un Text Block
public String generarMensajeBienvenida(String nombre, String tipoSolicitud, Long id) {
    return """
            Estimado/a %s,
            
            Su solicitud de tipo %s ha sido registrada exitosamente.
            Número de radicado: %d
            
            Puede hacer seguimiento de su solicitud en el portal académico.
            
            Atentamente,
            Sistema de Triage Académico - Universidad del Quindío
            """.formatted(nombre, tipoSolicitud, id);
}
```

### 7.3 Aplicación al Triage: Queries y JSON de Prueba

Aquí están los usos más comunes en nuestro proyecto:

```java
// 1. Queries JPQL complejas (las veremos en la Guía 05)
public static final String QUERY_SOLICITUDES_ACTIVAS = """
        SELECT s FROM Solicitud s
        LEFT JOIN FETCH s.historial h
        WHERE s.estado IN ('REGISTRADA', 'CLASIFICADA', 'EN_ATENCION')
        AND s.solicitante.id = :usuarioId
        ORDER BY s.fechaRegistro DESC
        """;

// 2. Queries SQL nativas para migraciones (Flyway - Guía 05)
public static final String CREAR_TABLA_SOLICITUDES = """
        CREATE TABLE IF NOT EXISTS solicitudes (
            id              BIGSERIAL PRIMARY KEY,
            tipo_solicitud  VARCHAR(50) NOT NULL,
            descripcion     TEXT NOT NULL,
            canal_origen    VARCHAR(30) NOT NULL,
            prioridad       VARCHAR(20) NOT NULL DEFAULT 'MEDIA',
            estado          VARCHAR(30) NOT NULL DEFAULT 'REGISTRADA',
            solicitante_id  BIGINT NOT NULL REFERENCES usuarios(id),
            responsable_id  BIGINT REFERENCES usuarios(id),
            fecha_registro  TIMESTAMP NOT NULL DEFAULT NOW(),
            fecha_actualizacion TIMESTAMP
        );
        """;

// 3. Datos de prueba en tests de integración (los veremos en la Guía 10)
public static final String SOLICITUD_HOMOLOGACION_JSON = """
        {
            "tipoSolicitud": "HOMOLOGACION",
            "descripcion": "Solicito la homologación de la asignatura Cálculo I, cursada en el primer semestre de 2023 en la Universidad Nacional de Colombia, sede Manizales.",
            "canalOrigen": "CORREO",
            "identificacionSolicitante": "1088123456"
        }
        """;

// 4. Mensajes de error estructurados (Problem Details - Guía 06)
public static String generarErrorBody(String titulo, String detalle, int status) {
    return """
            {
                "type": "https://api.triage.uniquindio.edu.co/errores/%s",
                "title": "%s",
                "status": %d,
                "detail": "%s",
                "instance": "/api/v1/solicitudes"
            }
            """.formatted(titulo.toLowerCase().replace(" ", "-"), titulo, status, detalle);
}
```

---

## 8. var: Inferencia de Tipos Local

### 8.1 Qué es var y Cómo Funciona

`var` es una palabra clave introducida en Java 10 que permite al compilador inferir el tipo de una variable local. Es importante entender que **var NO hace a Java un lenguaje de tipado dinámico**. El tipo sigue siendo estático y se determina en tiempo de compilación.

```java
// Con var, el compilador infiere el tipo del lado derecho
var nombre = "Juan Pérez";          // String
var id = 12345L;                    // Long
var solicitudes = new ArrayList<String>(); // ArrayList<String>
var resultado = calcularPrioridad(); // El tipo es lo que devuelva el método

// Es equivalente exactamente a:
String nombre2 = "Juan Pérez";
Long id2 = 12345L;
ArrayList<String> solicitudes2 = new ArrayList<String>();
```

### 8.2 Cuándo Usarlo y Cuándo NO

`var` mejora la legibilidad cuando el tipo es obvio del lado derecho. La regla general: **usa var cuando el tipo sea claro sin necesidad de leer la declaración completa**.

```java
// ✅ Casos donde var AYUDA a la legibilidad

// El tipo es obvio: es un String
var mensaje = "Solicitud registrada exitosamente";

// Evita repetir el tipo en construcción
var solicitud = new SolicitudHomologacion("Cálculo I", "Universidad Nacional");

// En ciclos con tipos genéricos complejos
var solicitudesAgrupadas = solicitudes.stream()
    .collect(Collectors.groupingBy(SolicitudResumenResponse::estado));

// Con streams donde el tipo inferido sería muy largo
var estadisticas = solicitudes.stream()
    .collect(Collectors.summarizingInt(s -> calcularDias(s.fechaRegistro())));

// En bloques try-with-resources
try (var conexion = dataSource.getConnection()) {
    // ...
}
```

```java
// ❌ Casos donde var DIFICULTA la legibilidad

// No está claro qué devuelve el método sin ir a verlo
var resultado = servicio.procesarSolicitud(id);  // ¿Qué tipo es resultado?

// Literales numéricos ambiguos
var valor = 42; // ¿Es int? ¿long? ¿Integer?

// En declaraciones de campo de clase (var NO funciona aquí)
// public var nombre = "..."; // ERROR DE COMPILACIÓN

// En parámetros de métodos (tampoco funciona)
// public void procesar(var solicitud) {} // ERROR DE COMPILACIÓN

// En tipos de retorno de métodos (tampoco)
// public var obtenerSolicitud() {} // ERROR DE COMPILACIÓN
```

**Regla práctica:** Si tienes dudas sobre si usar `var`, no lo uses. La claridad del código siempre es más importante que la brevedad.

---

## 9. Optional: El Fin de los NullPointerException

### 9.1 El Problema del null

El `NullPointerException` (NPE) es el error más común en Java. El problema viene cuando un método puede devolver `null` pero el llamador lo olvida y accede a un método del objeto sin verificar:

```java
// ❌ Código peligroso sin Optional
public String obtenerNombreSolicitante(Long solicitudId) {
    Solicitud solicitud = repositorio.buscarPorId(solicitudId); // Podría ser null
    return solicitud.getSolicitante().getNombre(); // NPE si solicitud es null!
}
```

`Optional<T>` es un contenedor que puede tener un valor o puede estar vacío. Hace explícito en el tipo de retorno que un valor podría no existir, obligando al llamador a manejarlo.

### 9.2 Creación de Optional

```java
// Crear un Optional con valor
Optional<String> conValor = Optional.of("Juan Pérez");

// Crear un Optional vacío
Optional<String> vacio = Optional.empty();

// Cuando no sabes si el valor es null (el más común en la práctica)
String nombre = obtenerNombreDesdeDB(); // podría ser null
Optional<String> quizasNombre = Optional.ofNullable(nombre);
```

### 9.3 Métodos Esenciales

```java
Optional<SolicitudResumenResponse> optSolicitud = buscarSolicitud(id);

// isPresent() / isEmpty(): verificar si tiene valor
if (optSolicitud.isPresent()) {
    System.out.println("Existe la solicitud");
}
if (optSolicitud.isEmpty()) {
    System.out.println("No se encontró la solicitud");
}

// get(): obtiene el valor — ⚠️ lanza NoSuchElementException si está vacío
// NUNCA uses get() sin verificar isPresent() antes
SolicitudResumenResponse solicitud = optSolicitud.get(); // peligroso solo

// orElse(): devuelve un valor por defecto si está vacío
SolicitudResumenResponse solicitud2 = optSolicitud.orElse(solicitudVacia());

// orElseGet(): como orElse pero recibe un Supplier (más eficiente cuando el default es costoso)
SolicitudResumenResponse solicitud3 = optSolicitud.orElseGet(() -> crearSolicitudDefault(id));

// orElseThrow(): lanza una excepción personalizada si está vacío (muy común en servicios)
SolicitudResumenResponse solicitud4 = optSolicitud
    .orElseThrow(() -> new SolicitudNoEncontradaException("No existe solicitud con ID: " + id));

// map(): transforma el valor si existe, devuelve Optional vacío si no existe
Optional<String> nombreSolicitante = optSolicitud
    .map(SolicitudResumenResponse::solicitante);

// flatMap(): cuando el mapper devuelve otro Optional (evita Optional<Optional<T>>)
Optional<String> descripcionTipo = optSolicitud
    .flatMap(s -> buscarDescripcionTipo(s.tipo()));

// filter(): filtra el valor según una condición; si no pasa el filtro, devuelve vacío
Optional<SolicitudResumenResponse> solicitudUrgente = optSolicitud
    .filter(SolicitudResumenResponse::esUrgente);

// ifPresent(): ejecuta una acción si hay valor (para efectos secundarios)
optSolicitud.ifPresent(s -> System.out.println("Procesando: " + s.id()));

// ifPresentOrElse(): acción si existe O acción si vacío (Java 9+)
optSolicitud.ifPresentOrElse(
    s -> System.out.println("Solicitud: " + s.id()),
    () -> System.out.println("No se encontró la solicitud")
);
```

### 9.4 Antipatrones Comunes

```java
// ❌ Antipatrón 1: usar Optional como if-else glorificado
// Si vas a hacer esto, mejor usa un if normal
Optional<Solicitud> opt = buscar(id);
if (opt.isPresent()) {
    procesar(opt.get()); // Equivalente a un null-check manual
}
// ✅ Mejor:
buscar(id).ifPresent(this::procesar);

// ❌ Antipatrón 2: Optional de Optional
Optional<Optional<String>> doble = Optional.of(Optional.of("valor")); // Nunca hagas esto
// ✅ Usar flatMap()

// ❌ Antipatrón 3: Optional en campos de clase
public class Solicitud {
    private Optional<String> descripcion; // NO - usa String o maneja el null directamente
}
// Optional está diseñado para ser valor de RETORNO de métodos, no para campos.

// ❌ Antipatrón 4: Optional en parámetros de método
public void procesar(Optional<String> nombre) {} // NO
// ✅ Mejor: tener dos métodos o usar @Nullable

// ❌ Antipatrón 5: usar orElse con expresiones costosas
Optional<Solicitud> optS = buscar(id);
Solicitud s = optS.orElse(crearNuevaSolicitudDesdeAPI()); // crearNuevaSolicitudDesdeAPI() SIEMPRE se ejecuta
// ✅ Usar orElseGet() que evalúa el Supplier solo si es necesario
Solicitud s2 = optS.orElseGet(() -> crearNuevaSolicitudDesdeAPI());
```

### 9.5 Optional en Servicios del Triage

Así es como usarás Optional en el proyecto (lo implementaremos completamente en la Guía 05 y 06):

```java
// Ejemplo de cómo se verá el servicio de solicitudes
// (Aún sin Spring, para entender el patrón)
public class SolicitudService {

    private final SolicitudRepository repositorio;

    public SolicitudService(SolicitudRepository repositorio) {
        this.repositorio = repositorio;
    }

    // RF-07: Consultar una solicitud por ID
    // El método devuelve Optional porque la solicitud puede no existir
    public Optional<SolicitudResumenResponse> buscarPorId(Long id) {
        return repositorio.findById(id)        // Spring Data devuelve Optional<Solicitud>
            .map(this::convertirAResumen);      // Si existe, convertimos a DTO
    }

    // RF-08: Cerrar una solicitud
    // Lanza excepción si no existe — esto es el patrón más común en APIs REST
    public void cerrarSolicitud(Long id, String observacionCierre) {
        var solicitud = repositorio.findById(id)
            .orElseThrow(() ->
                new SolicitudNoEncontradaException("No existe solicitud con ID: " + id));

        if (!"ATENDIDA".equals(solicitud.getEstado())) {
            throw new TransicionEstadoInvalidaException(
                "Solo se pueden cerrar solicitudes en estado ATENDIDA. " +
                "Estado actual: " + solicitud.getEstado()
            );
        }

        solicitud.setEstado("CERRADA");
        solicitud.setObservacionCierre(observacionCierre);
        repositorio.save(solicitud);
    }

    // RF-05: Asignar responsable
    public SolicitudResumenResponse asignarResponsable(Long solicitudId, Long responsableId) {
        var solicitud = repositorio.findById(solicitudId)
            .orElseThrow(() -> new SolicitudNoEncontradaException(
                "Solicitud " + solicitudId + " no encontrada"));

        var responsable = usuarioRepositorio.findById(responsableId)
            .filter(u -> u.isActivo())  // RF-05: el responsable debe estar activo
            .orElseThrow(() -> new ResponsableNoDisponibleException(
                "El usuario " + responsableId + " no existe o no está activo"));

        solicitud.setResponsable(responsable);
        return convertirAResumen(repositorio.save(solicitud));
    }

    private SolicitudResumenResponse convertirAResumen(Object solicitud) {
        // Conversión a DTO — se implementará en Guías 05 y 06
        return null; // placeholder
    }
}
```

---

## 10. Stream API: Procesamiento Funcional de Colecciones

### 10.1 ¿Qué es un Stream?

Un **Stream** es una secuencia de elementos que soporta operaciones de procesamiento secuencial y paralelo. No es una estructura de datos (no almacena datos), sino un canal de procesamiento.

**Analogía:** Imagina una cadena de montaje en una fábrica. Los productos (datos) entran por un extremo, pasan por varias estaciones de transformación o filtrado (operaciones intermedias) y salen por el otro extremo ya procesados (operación terminal). La cadena de montaje en sí (el Stream) no almacena productos.

```
Colección → stream() → [filter] → [map] → [sorted] → [collect]
                       operaciones intermedias        terminal
```

### 10.2 Operaciones Intermedias

Son operaciones que transforman un Stream en otro Stream. Son **lazy**: no se ejecutan hasta que hay una operación terminal.

```java
// Supongamos que tenemos una lista de solicitudes (Records) para procesar
List<SolicitudResumenResponse> todasLasSolicitudes = obtenerTodasLasSolicitudes();

// --- filter(): conservar solo los que cumplen una condición ---
List<SolicitudResumenResponse> solicitudesUrgentes = todasLasSolicitudes.stream()
    .filter(s -> "ALTA".equals(s.prioridad()) || "CRITICA".equals(s.prioridad()))
    .toList(); // operación terminal

// --- map(): transformar cada elemento en otro ---
List<Long> idsDeSolicitudes = todasLasSolicitudes.stream()
    .map(SolicitudResumenResponse::id)   // referencia a método (más limpio que lambda)
    .toList();

List<String> estadosUnicos = todasLasSolicitudes.stream()
    .map(SolicitudResumenResponse::estado)
    .distinct() // eliminar duplicados
    .toList();

// --- flatMap(): cuando map() devuelve una colección y quieres aplanar ---
// Supongamos que cada solicitud tiene múltiples observaciones en el historial
List<String> todasLasObservaciones = todasLasSolicitudes.stream()
    .flatMap(s -> obtenerObservaciones(s.id()).stream()) // cada s produce una lista
    .toList(); // se aplana todo en una sola lista

// --- sorted(): ordenar ---
List<SolicitudResumenResponse> ordenadas = todasLasSolicitudes.stream()
    .sorted(Comparator.comparing(SolicitudResumenResponse::prioridad)
                      .thenComparing(SolicitudResumenResponse::solicitante))
    .toList();

// --- limit() y skip(): paginación manual ---
List<SolicitudResumenResponse> pagina2 = todasLasSolicitudes.stream()
    .skip(10)   // saltar los primeros 10 (página 1)
    .limit(10)  // tomar los siguientes 10 (página 2)
    .toList();

// --- peek(): para debugging, ejecuta una acción sin cambiar el Stream ---
List<SolicitudResumenResponse> con_debug = todasLasSolicitudes.stream()
    .filter(SolicitudResumenResponse::esUrgente)
    .peek(s -> System.out.println("Procesando urgente: " + s.id()))
    .map(s -> new SolicitudResumenResponse(s.id(), s.tipo(), s.prioridad(), "EN_ATENCION", s.solicitante()))
    .toList();
```

### 10.3 Operaciones Terminales

Son operaciones que consumen el Stream y producen un resultado (o un efecto secundario). Después de una operación terminal, el Stream ya no puede usarse.

```java
List<SolicitudResumenResponse> solicitudes = obtenerTodasLasSolicitudes();

// --- collect(): la más versátil — recolectar en una colección ---
List<SolicitudResumenResponse> comoLista = solicitudes.stream()
    .filter(s -> "REGISTRADA".equals(s.estado()))
    .collect(Collectors.toList()); // o .toList() que es más corto desde Java 16

Set<String> tiposUnicos = solicitudes.stream()
    .map(SolicitudResumenResponse::tipo)
    .collect(Collectors.toSet());

// --- toList(): atajo desde Java 16, devuelve lista INMUTABLE ---
List<Long> ids = solicitudes.stream()
    .map(SolicitudResumenResponse::id)
    .toList(); // inmutable, más conciso

// --- count(): contar elementos ---
long cantidadUrgentes = solicitudes.stream()
    .filter(SolicitudResumenResponse::esUrgente)
    .count();

// --- findFirst() / findAny(): buscar el primero que cumpla una condición ---
Optional<SolicitudResumenResponse> primeraCritica = solicitudes.stream()
    .filter(s -> "CRITICA".equals(s.prioridad()))
    .findFirst(); // devuelve Optional

// --- anyMatch() / allMatch() / noneMatch() ---
boolean hayRegistradas = solicitudes.stream()
    .anyMatch(s -> "REGISTRADA".equals(s.estado())); // ¿alguna es REGISTRADA?

boolean todasAtendidas = solicitudes.stream()
    .allMatch(s -> "ATENDIDA".equals(s.estado()) || "CERRADA".equals(s.estado())); // ¿todas?

boolean ningunaFallida = solicitudes.stream()
    .noneMatch(s -> "ERROR".equals(s.estado())); // ¿ninguna?

// --- forEach(): ejecutar una acción por cada elemento (efecto secundario) ---
solicitudes.stream()
    .filter(SolicitudResumenResponse::esUrgente)
    .forEach(s -> enviarAlerta(s.id())); // ⚠️ no retorna nada, solo efectos

// --- reduce(): combinar todos los elementos en uno ---
// (menos común pero útil para cálculos acumulativos)
int totalDiasAcumulados = solicitudes.stream()
    .mapToInt(s -> calcularDiasEnSistema(s))
    .sum(); // para int, también tienes .average(), .min(), .max()

// --- joining(): concatenar Strings ---
String resumenIds = solicitudes.stream()
    .map(s -> String.valueOf(s.id()))
    .collect(Collectors.joining(", ", "[", "]")); // "[1, 2, 3, 4]"
```

### 10.4 Collectors Avanzados: groupingBy, partitioningBy, counting

Estos son los Collectors más poderosos para generar reportes y estadísticas del sistema de Triage:

```java
List<SolicitudResumenResponse> solicitudes = obtenerTodasLasSolicitudes();

// --- groupingBy(): agrupar en un Map según una clave ---

// Agrupar por estado
Map<String, List<SolicitudResumenResponse>> porEstado = solicitudes.stream()
    .collect(Collectors.groupingBy(SolicitudResumenResponse::estado));

// porEstado.get("REGISTRADA") → [lista de solicitudes registradas]
// porEstado.get("EN_ATENCION") → [lista en atención]
// ...

// Agrupar por tipo y contar
Map<String, Long> cantidadPorTipo = solicitudes.stream()
    .collect(Collectors.groupingBy(
        SolicitudResumenResponse::tipo,
        Collectors.counting()         // downstream collector
    ));
// {"HOMOLOGACION": 45, "CUPO": 120, "CANCELACION": 33, ...}

// Agrupar por prioridad y obtener solo los IDs
Map<String, List<Long>> idsPorPrioridad = solicitudes.stream()
    .collect(Collectors.groupingBy(
        SolicitudResumenResponse::prioridad,
        Collectors.mapping(SolicitudResumenResponse::id, Collectors.toList())
    ));

// Agrupar por estado y ordenar por ID dentro de cada grupo
Map<String, List<SolicitudResumenResponse>> porEstadoOrdenado = solicitudes.stream()
    .collect(Collectors.groupingBy(
        SolicitudResumenResponse::estado,
        Collectors.collectingAndThen(
            Collectors.toList(),
            lista -> lista.stream()
                .sorted(Comparator.comparing(SolicitudResumenResponse::id))
                .toList()
        )
    ));

// --- partitioningBy(): dividir en dos grupos (true/false) ---
Map<Boolean, List<SolicitudResumenResponse>> urgentesVsNoUrgentes = solicitudes.stream()
    .collect(Collectors.partitioningBy(SolicitudResumenResponse::esUrgente));

List<SolicitudResumenResponse> urgentes = urgentesVsNoUrgentes.get(true);
List<SolicitudResumenResponse> noUrgentes = urgentesVsNoUrgentes.get(false);

// --- summarizingInt(): estadísticas completas ---
IntSummaryStatistics estadisticasDias = solicitudes.stream()
    .collect(Collectors.summarizingInt(s -> calcularDiasEnSistema(s)));
// estadisticasDias.getAverage(), .getMax(), .getMin(), .getSum(), .getCount()

// --- toMap(): convertir a Map con clave y valor personalizados ---
Map<Long, String> estadoPorId = solicitudes.stream()
    .collect(Collectors.toMap(
        SolicitudResumenResponse::id,       // clave: el ID
        SolicitudResumenResponse::estado,   // valor: el estado
        (v1, v2) -> v1                      // merge function: si hay claves duplicadas, toma la primera
    ));
```

### 10.5 Streams Paralelos: Cuándo y Por Qué (No Siempre)

Los Streams paralelos dividen el trabajo en múltiples threads. Pueden ser más rápidos para colecciones muy grandes, pero también pueden ser más lentos para colecciones pequeñas debido al overhead de sincronización.

```java
// Stream paralelo: agregar .parallel() o usar parallelStream()
long solicitudesUrgentesCriticas = solicitudes.parallelStream()
    .filter(s -> "CRITICA".equals(s.prioridad()))
    .count();

// ⚠️ ADVERTENCIA: Los streams paralelos solo vale la pena cuando:
// 1. La colección tiene miles o millones de elementos.
// 2. Las operaciones son computacionalmente intensas (no simples comparaciones).
// 3. Las operaciones son stateless (no modifican estado compartido).
// Para el proyecto de Triage (menos de 10,000 solicitudes activas típicamente),
// los streams secuenciales son suficientes y más predecibles.
```

### 10.6 Caso Completo: Procesar Listas de Solicitudes del Triage

Veamos un caso de uso real del proyecto: generar un reporte del estado del sistema (RF-07):

```java
// Servicio de reportes para el sistema de Triage
// (Se convertirá en @Service de Spring Boot en la Guía 04)
public class ReporteService {

    /**
     * Genera un resumen del estado actual del sistema.
     * Agrupa solicitudes por estado, calcula urgentes, y prepara datos para el dashboard.
     */
    public ResumenSistema generarResumen(List<SolicitudResumenResponse> solicitudes) {

        // 1. Agrupar por estado con conteo
        var porEstado = solicitudes.stream()
            .collect(Collectors.groupingBy(
                SolicitudResumenResponse::estado,
                Collectors.counting()
            ));

        // 2. Solicitudes urgentes que aún no están asignadas (requieren atención inmediata)
        var urgentesNoAsignadas = solicitudes.stream()
            .filter(s -> "REGISTRADA".equals(s.estado()) || "CLASIFICADA".equals(s.estado()))
            .filter(SolicitudResumenResponse::esUrgente)
            .sorted(Comparator.comparing(SolicitudResumenResponse::prioridad).reversed())
            .toList();

        // 3. Conteo total
        var total = (long) solicitudes.size();
        var atendidas = porEstado.getOrDefault("ATENDIDA", 0L)
                      + porEstado.getOrDefault("CERRADA", 0L);

        // 4. Porcentaje de resolución
        var porcentajeResolucion = total > 0 ? (atendidas * 100.0 / total) : 0.0;

        // 5. Tipos más frecuentes (top 3)
        var topTipos = solicitudes.stream()
            .collect(Collectors.groupingBy(SolicitudResumenResponse::tipo, Collectors.counting()))
            .entrySet().stream()
            .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
            .limit(3)
            .map(e -> e.getKey() + ": " + e.getValue())
            .collect(Collectors.joining(", "));

        return new ResumenSistema(
            total,
            porEstado,
            urgentesNoAsignadas.size(),
            porcentajeResolucion,
            topTipos
        );
    }

    /**
     * Filtra y pagina solicitudes según múltiples criterios (RF-07).
     */
    public List<SolicitudResumenResponse> filtrarSolicitudes(
            List<SolicitudResumenResponse> todas,
            String estado,
            String tipo,
            String prioridad,
            String responsable,
            int pagina,
            int tamano) {

        return todas.stream()
            .filter(s -> estado == null || estado.isBlank() || estado.equalsIgnoreCase(s.estado()))
            .filter(s -> tipo == null || tipo.isBlank() || tipo.equalsIgnoreCase(s.tipo()))
            .filter(s -> prioridad == null || prioridad.isBlank() || prioridad.equalsIgnoreCase(s.prioridad()))
            .filter(s -> responsable == null || responsable.isBlank()
                         || responsable.equalsIgnoreCase(s.solicitante()))
            .skip((long) pagina * tamano)
            .limit(tamano)
            .toList();
    }
}

// Record para el resumen del sistema
public record ResumenSistema(
    long totalSolicitudes,
    Map<String, Long> distribucionPorEstado,
    int urgentesNoAsignadas,
    double porcentajeResolucion,
    String topTipos
) {}
```

---

## 11. Nuevas APIs de Colecciones

### 11.1 Colecciones Inmutables

Java 9+ trajo métodos de fábrica para crear colecciones inmutables de forma concisa:

```java
// ❌ Antes: verbose y mutable
List<String> estadosMutable = new ArrayList<>();
estadosMutable.add("REGISTRADA");
estadosMutable.add("CLASIFICADA");
estadosMutable.add("EN_ATENCION");
estadosMutable.add("ATENDIDA");
estadosMutable.add("CERRADA");
List<String> estadosInmutable = Collections.unmodifiableList(estadosMutable);

// ✅ Ahora: conciso e inmutable desde el inicio
List<String> ESTADOS_VALIDOS = List.of(
    "REGISTRADA", "CLASIFICADA", "EN_ATENCION", "ATENDIDA", "CERRADA"
);

Set<String> CANALES_ORIGEN = Set.of(
    "CSU", "CORREO", "SAC", "TELEFONICO", "PRESENCIAL", "PORTAL_WEB"
);

Map<String, Integer> SLA_POR_TIPO = Map.of(
    "HOMOLOGACION", 15,
    "CUPO", 3,
    "CANCELACION", 5,
    "CONSULTA", 2,
    "REGISTRO_ASIGNATURA", 7
);
```

**Restricciones importantes de las colecciones inmutables:**

```java
// Las listas inmutables NO pueden modificarse
ESTADOS_VALIDOS.add("NUEVO_ESTADO");    // ❌ UnsupportedOperationException
ESTADOS_VALIDOS.remove("CERRADA");      // ❌ UnsupportedOperationException

// List.of() NO acepta null
List.of("uno", null, "tres");           // ❌ NullPointerException

// Para colecciones que podrían tener null o necesitas modificar:
List<String> mutable = new ArrayList<>(List.of("uno", "dos")); // copia mutable
```

### 11.2 copyOf y toList()

```java
List<SolicitudResumenResponse> original = obtenerSolicitudes();

// copyOf(): copia inmutable de una colección existente
List<SolicitudResumenResponse> copia = List.copyOf(original);
Set<String> tiposUnicos = Set.copyOf(
    original.stream().map(SolicitudResumenResponse::tipo).toList()
);

// toList() en Streams (Java 16+): alternativa concisa a collect(Collectors.toList())
// Devuelve lista INMUTABLE
List<SolicitudResumenResponse> urgentes = original.stream()
    .filter(SolicitudResumenResponse::esUrgente)
    .toList(); // ✅ limpio, conciso, inmutable

// Si necesitas lista MUTABLE desde un Stream:
List<SolicitudResumenResponse> urgentesModificable = original.stream()
    .filter(SolicitudResumenResponse::esUrgente)
    .collect(Collectors.toCollection(ArrayList::new));
```

### 11.3 Map.entry y Map.ofEntries

Cuando un `Map.of()` tiene más de 10 pares (su límite), usa `Map.ofEntries()`:

```java
// Map.of() tiene límite de 10 entradas
Map<String, String> descripcionesEstado = Map.ofEntries(
    Map.entry("REGISTRADA",   "La solicitud fue recibida y está pendiente de clasificación"),
    Map.entry("CLASIFICADA",  "La solicitud fue clasificada y espera ser asignada"),
    Map.entry("EN_ATENCION",  "Un funcionario está trabajando en esta solicitud"),
    Map.entry("ATENDIDA",     "La solicitud fue procesada exitosamente"),
    Map.entry("CERRADA",      "El trámite fue finalizado y archivado"),
    Map.entry("RECHAZADA",    "La solicitud fue rechazada por no cumplir requisitos"),
    Map.entry("EN_REVISION",  "La solicitud está siendo revisada por coordinación"),
    Map.entry("DEVUELTA",     "La solicitud fue devuelta al estudiante para correcciones"),
    Map.entry("ESCALADA",     "La solicitud fue escalada a un nivel superior"),
    Map.entry("EN_ESPERA",    "Pendiente de documentación del estudiante"),
    Map.entry("SUSPENDIDA",   "El trámite está temporalmente suspendido")
);
```

---

## 12. Tabla Comparativa: Java 8 → Java 21

Esta tabla resume la evolución de las características que usarás en el proyecto:

| Característica | Java 8 | Java 11 | Java 17 | Java 21 |
|---|---|---|---|---|
| **DTOs** | Clases con getters/setters | Igual | Igual | **Records** ✅ |
| **Strings multilinea** | Concatenación con `+` y `\n` | Igual | Igual | **Text Blocks** ✅ |
| **Tipos cerrados** | Interfaces normales | Igual | **Sealed Classes** ✅ | Estable |
| **switch** | Statement tradicional | Igual | Switch Expression (preview) | **Pattern Matching** ✅ |
| **instanceof** | `instanceof` + cast manual | Igual | **Pattern Matching** ✅ | Estable |
| **Colecciones inmutables** | `Collections.unmodifiableList()` | `List.of()` ✅ | Igual | `.toList()` en Streams ✅ |
| **null handling** | null checks manuales | Igual | Igual | **Optional** maduro ✅ |
| **Inferencia de tipos** | Tipos explícitos | **var** ✅ | Estable | Estable |
| **Stream API** | Streams básicos | Mejoras menores | Igual | `mapMulti()` ✅ |
| **HTTP Client** | `HttpURLConnection` | `HttpClient` ✅ | Estable | Estable |
| **Soporte LTS** | ✅ (hasta 2030) | ✅ (hasta 2026) | ✅ (hasta 2029) | ✅ (hasta 2031) |

---

## 13. Modelando el Proyecto Triage con Java Moderno

Ahora vamos a aplicar todo lo aprendido para modelar el dominio completo del proyecto. Este código servirá como base para las guías siguientes.

### 13.1 Estructura de Carpetas de los Modelos

```
src/main/java/com/uniquindio/triage/
│
├── domain/                          # Modelo de dominio puro (sin Spring)
│   ├── enums/
│   │   ├── EstadoSolicitud.java
│   │   ├── Prioridad.java
│   │   ├── CanalOrigen.java
│   │   └── RolUsuario.java
│   │
│   └── types/                       # Tipos sellados para el dominio
│       └── TipoSolicitudDetalle.java (y sus implementaciones)
│
├── dto/                             # Records - Data Transfer Objects
│   ├── request/
│   │   ├── RegistrarSolicitudRequest.java
│   │   ├── ClasificarSolicitudRequest.java
│   │   ├── AsignarResponsableRequest.java
│   │   ├── CambiarEstadoRequest.java
│   │   └── LoginRequest.java
│   │
│   └── response/
│       ├── SolicitudCreadaResponse.java
│       ├── SolicitudResumenResponse.java
│       ├── SolicitudDetalleResponse.java
│       ├── HistorialEntradaResponse.java
│       └── ResumenSistema.java
│
└── service/                         # Lógica de negocio (sin Spring aún)
    └── util/
        ├── PriorizadorDeSolicitudes.java
        └── ValidadorTransicionEstado.java
```

### 13.2 Enums y Tipos del Dominio

```java
// EstadoSolicitud.java
// Los estados corresponden al RF-04 del proyecto
public enum EstadoSolicitud {
    REGISTRADA("La solicitud fue recibida"),
    CLASIFICADA("La solicitud fue clasificada"),
    EN_ATENCION("Un funcionario está atendiéndola"),
    ATENDIDA("La solicitud fue procesada"),
    CERRADA("El trámite finalizó");

    private final String descripcion;

    EstadoSolicitud(String descripcion) {
        this.descripcion = descripcion;
    }

    public String getDescripcion() { return descripcion; }

    /**
     * Verifica si la transición al estado destino es válida.
     * RF-04: "El sistema debe validar que las transiciones entre estados sean coherentes."
     */
    public boolean puedeTransicionarA(EstadoSolicitud destino) {
        return switch (this) {
            case REGISTRADA  -> destino == CLASIFICADA;
            case CLASIFICADA -> destino == EN_ATENCION;
            case EN_ATENCION -> destino == ATENDIDA;
            case ATENDIDA    -> destino == CERRADA;
            case CERRADA     -> false; // RF-08: una solicitud cerrada no puede modificarse
        };
    }
}
```

```java
// Prioridad.java
// RF-03: La prioridad queda registrada junto con una justificación
public enum Prioridad {
    BAJA(1, "Atención en el horario normal"),
    MEDIA(2, "Atención prioritaria dentro del plazo estándar"),
    ALTA(3, "Requiere atención a la brevedad"),
    CRITICA(4, "Atención inmediata - impacto académico alto");

    private final int nivel;
    private final String descripcion;

    Prioridad(int nivel, String descripcion) {
        this.nivel = nivel;
        this.descripcion = descripcion;
    }

    public int getNivel() { return nivel; }
    public String getDescripcion() { return descripcion; }

    public boolean esMasAltaQue(Prioridad otra) {
        return this.nivel > otra.nivel;
    }
}
```

```java
// CanalOrigen.java
// RF-01: Canal de origen de la solicitud
public enum CanalOrigen {
    CSU("Centro de Servicios Universitarios"),
    CORREO("Correo electrónico institucional"),
    SAC("Sistema de Atención al Ciudadano"),
    TELEFONICO("Llamada telefónica"),
    PRESENCIAL("Atención presencial en ventanilla"),
    PORTAL_WEB("Portal web académico");

    private final String descripcionCompleta;

    CanalOrigen(String descripcionCompleta) {
        this.descripcionCompleta = descripcionCompleta;
    }

    public String getDescripcionCompleta() { return descripcionCompleta; }
}
```

```java
// RolUsuario.java
// RF-13: Roles para autorización de operaciones
public enum RolUsuario {
    ESTUDIANTE,     // Puede registrar y consultar sus propias solicitudes
    FUNCIONARIO,    // Puede clasificar, priorizar y atender solicitudes asignadas
    ADMIN           // Acceso completo: gestión de usuarios, reportes, cierre
}
```

### 13.3 Records como DTOs del Triage

```java
// ─── REQUEST DTOs ───────────────────────────────────────────────────────────

// RegistrarSolicitudRequest.java — RF-01
// Este Record representa los datos que el frontend envía para crear una solicitud
import jakarta.validation.constraints.*;

public record RegistrarSolicitudRequest(

    @NotNull(message = "El tipo de solicitud es obligatorio")
    String tipoSolicitud,

    @NotBlank(message = "La descripción no puede estar vacía")
    @Size(min = 20, max = 2000,
          message = "La descripción debe tener entre 20 y 2000 caracteres")
    String descripcion,

    @NotNull(message = "El canal de origen es obligatorio")
    String canalOrigen,

    @NotBlank(message = "La identificación del solicitante es obligatoria")
    String identificacionSolicitante

) {}
```

```java
// ClasificarSolicitudRequest.java — RF-02 y RF-03
// Datos para clasificar y priorizar una solicitud (acción de FUNCIONARIO)
public record ClasificarSolicitudRequest(

    @NotNull(message = "El tipo de solicitud es obligatorio para clasificar")
    String tipoSolicitud,

    @NotNull(message = "La prioridad es obligatoria")
    String prioridad,

    @NotBlank(message = "Debe indicar la justificación de la prioridad asignada")
    @Size(max = 500, message = "La justificación no debe superar 500 caracteres")
    String justificacionPrioridad

) {}
```

```java
// AsignarResponsableRequest.java — RF-05
// Para asignar un funcionario responsable a una solicitud
public record AsignarResponsableRequest(

    @NotNull(message = "El ID del responsable es obligatorio")
    @Positive(message = "El ID del responsable debe ser un número positivo")
    Long responsableId

) {}
```

```java
// CambiarEstadoRequest.java — RF-04, RF-08
// Para cambiar el estado de una solicitud (con observación obligatoria al cerrar)
public record CambiarEstadoRequest(

    @NotNull(message = "El estado destino es obligatorio")
    String estadoDestino,

    @Size(max = 1000, message = "La observación no debe superar 1000 caracteres")
    String observacion   // Obligatoria solo para CERRADA (se valida en el Service)

) {}
```

```java
// LoginRequest.java
// Para el flujo de autenticación (Guía 07)
public record LoginRequest(

    @NotBlank(message = "El correo institucional es obligatorio")
    @Email(message = "Debe ser un correo electrónico válido")
    String correo,

    @NotBlank(message = "La contraseña es obligatoria")
    @Size(min = 8, message = "La contraseña debe tener al menos 8 caracteres")
    String contrasena

) {}
```

```java
// ─── RESPONSE DTOs ──────────────────────────────────────────────────────────

// SolicitudCreadaResponse.java — RF-01 (respuesta de creación)
// Lo que el backend devuelve inmediatamente después de registrar una solicitud
public record SolicitudCreadaResponse(
    Long id,
    String tipoSolicitud,
    String descripcion,
    String canalOrigen,
    String estado,
    String prioridad,
    String fechaRegistro,
    String solicitanteNombre,
    String mensajeBienvenida
) {
    // Constructor compacto para normalizar el estado al crearse
    public SolicitudCreadaResponse {
        if (!"REGISTRADA".equals(estado)) {
            throw new IllegalStateException("Una solicitud recién creada siempre inicia en REGISTRADA");
        }
    }
}
```

```java
// SolicitudResumenResponse.java — RF-07 (para listar solicitudes)
// Versión liviana para mostrar en tablas y listados
public record SolicitudResumenResponse(
    Long id,
    String tipo,
    String prioridad,
    String estado,
    String solicitante,
    String responsable,        // null si aún no tiene responsable asignado
    String fechaRegistro,
    int diasEnSistema
) {
    // Método de negocio: determina si es urgente para mostrar alertas en la UI
    public boolean esUrgente() {
        return "ALTA".equalsIgnoreCase(prioridad) || "CRITICA".equalsIgnoreCase(prioridad);
    }

    // Método de negocio: verifica si está en un estado activo (no finalizado)
    public boolean estaActiva() {
        return !"CERRADA".equalsIgnoreCase(estado);
    }

    // Señal de alerta: lleva mucho tiempo sin cerrar
    public boolean estaVencida(int diasMaximoPorTipo) {
        return diasEnSistema > diasMaximoPorTipo && estaActiva();
    }
}
```

```java
// HistorialEntradaResponse.java — RF-06 (para mostrar el historial)
// Cada entrada en el historial auditable de una solicitud
public record HistorialEntradaResponse(
    Long id,
    String accion,
    String estadoAnterior,
    String estadoNuevo,
    String usuarioResponsable,
    String observaciones,
    String fechaHoraAccion
) {}
```

```java
// SolicitudDetalleResponse.java — RF-07 (vista detallada de una solicitud)
// Vista completa con toda la información y el historial
public record SolicitudDetalleResponse(
    Long id,
    String tipo,
    String descripcion,
    String canalOrigen,
    String prioridad,
    String justificacionPrioridad,
    String estado,
    String solicitanteNombre,
    String solicitanteIdentificacion,
    String responsableNombre,
    String fechaRegistro,
    String fechaUltimaActualizacion,
    int diasEnSistema,
    List<HistorialEntradaResponse> historial    // RF-06: historial completo
) {
    public boolean esUrgente() {
        return "ALTA".equalsIgnoreCase(prioridad) || "CRITICA".equalsIgnoreCase(prioridad);
    }
}
```

### 13.4 Clase de Utilidades con Streams y Optional

```java
// ValidadorTransicionEstado.java
// Valida las transiciones de estado usando el enum EstadoSolicitud (RF-04)
public class ValidadorTransicionEstado {

    /**
     * Verifica que la transición de estado sea válida.
     * Lanza IllegalStateException si la transición no está permitida.
     */
    public static void validar(String estadoActualStr, String estadoDestinoStr) {
        var estadoActual = parsearEstado(estadoActualStr);
        var estadoDestino = parsearEstado(estadoDestinoStr);

        if (!estadoActual.puedeTransicionarA(estadoDestino)) {
            throw new IllegalStateException(
                "Transición inválida: no se puede pasar de %s a %s. %s".formatted(
                    estadoActual.name(),
                    estadoDestino.name(),
                    estadoActual == EstadoSolicitud.CERRADA
                        ? "Una solicitud cerrada no puede modificarse (RF-08)."
                        : "Las transiciones deben seguir el flujo: REGISTRADA → CLASIFICADA → EN_ATENCION → ATENDIDA → CERRADA."
                )
            );
        }
    }

    private static EstadoSolicitud parsearEstado(String estado) {
        try {
            return EstadoSolicitud.valueOf(estado.toUpperCase());
        } catch (IllegalArgumentException e) {
            throw new IllegalArgumentException(
                "Estado desconocido: '%s'. Estados válidos: %s".formatted(
                    estado,
                    java.util.Arrays.toString(EstadoSolicitud.values())
                )
            );
        }
    }
}
```

```java
// PriorizadorDeSolicitudes.java
// Motor de reglas de prioridad (RF-03) usando pattern matching y enums modernos
public class PriorizadorDeSolicitudes {

    // Constantes: colecciones inmutables con tipos de solicitud por prioridad por defecto
    private static final Set<String> TIPOS_ALTA_PRIORIDAD = Set.of(
        "CUPO", "CANCELACION", "REGISTRO_ASIGNATURA"
    );

    private static final Set<String> TIPOS_MEDIA_PRIORIDAD = Set.of(
        "HOMOLOGACION"
    );

    /**
     * Determina la prioridad de una solicitud con base en reglas de negocio.
     * RF-03: "La prioridad debe quedar registrada junto con una justificación."
     *
     * @return Un Record con la prioridad calculada y su justificación
     */
    public ResultadoPriorizacion calcular(String tipoSolicitud, boolean proximoAFechaLimite, int diasEnSistema) {

        var tipo = tipoSolicitud.toUpperCase();

        // Regla 1: si está próximo a fecha límite, cualquier solicitud escala
        if (proximoAFechaLimite && TIPOS_ALTA_PRIORIDAD.contains(tipo)) {
            return new ResultadoPriorizacion(
                Prioridad.CRITICA,
                "Solicitud de tipo %s con fecha límite inminente. Atención inmediata requerida.".formatted(tipo)
            );
        }

        // Regla 2: solicitudes que llevan más de 10 días sin atender escalan
        if (diasEnSistema > 10 && !Set.of("ATENDIDA", "CERRADA").contains(tipo)) {
            return new ResultadoPriorizacion(
                Prioridad.ALTA,
                "La solicitud lleva %d días en el sistema sin ser atendida. Escalada automáticamente.".formatted(diasEnSistema)
            );
        }

        // Regla 3: prioridad según tipo (regla base)
        if (TIPOS_ALTA_PRIORIDAD.contains(tipo)) {
            return new ResultadoPriorizacion(
                Prioridad.ALTA,
                "El tipo %s tiene prioridad alta por impacto académico directo.".formatted(tipo)
            );
        }

        if (TIPOS_MEDIA_PRIORIDAD.contains(tipo)) {
            return new ResultadoPriorizacion(
                Prioridad.MEDIA,
                "El tipo %s tiene prioridad media. Se procesará en el plazo estándar.".formatted(tipo)
            );
        }

        // Por defecto: prioridad baja
        return new ResultadoPriorizacion(
            Prioridad.BAJA,
            "Consulta general. Se atenderá en el horario normal."
        );
    }

    // Record anidado: el resultado de la priorización incluye la justificación (RF-03)
    public record ResultadoPriorizacion(Prioridad prioridad, String justificacion) {}
}
```

---

## 14. Ejercicios Prácticos

Estos retos están diseñados para que practiques por tu cuenta los conceptos de esta guía. No hay una única solución correcta; lo importante es que el código compile y produzca el resultado esperado.

---

### Reto 1: Records y Validación

Crea un Record `UsuarioRegistroRequest` que represente los datos para registrar un nuevo usuario en el sistema de Triage. Debe incluir:
- `nombre` (obligatorio, entre 2 y 100 caracteres)
- `correo` (obligatorio, formato de email, debe terminar en `@uniquindio.edu.co`)
- `identificacion` (obligatorio, entre 8 y 12 dígitos)
- `rol` (obligatorio, debe ser uno de: ESTUDIANTE, FUNCIONARIO, ADMIN)
- `contrasena` (obligatorio, mínimo 8 caracteres, al menos un número)

Agrega un método `nombreAbreviado()` que devuelva solo el primer nombre y apellido (suponiendo que el nombre completo tiene al menos dos palabras).

---

### Reto 2: Sealed Classes y Pattern Matching

Crea una Sealed Interface `ResultadoValidacion` con tres implementaciones:
- `ValidacionExitosa(String mensaje)`
- `ValidacionFallida(String campo, String error)`
- `ValidacionParcial(List<String> advertencias)`

Luego crea un método `generarReporteValidacion(ResultadoValidacion resultado)` que use pattern matching en switch para generar un String descriptivo de cada tipo de resultado. Pruébalo con los tres tipos.

---

### Reto 3: Stream API Completo

Dado el siguiente conjunto de datos de solicitudes (créalos como Records en tu código):

```
ID  | Tipo              | Prioridad | Estado       | Días en sistema
1   | HOMOLOGACION      | MEDIA     | REGISTRADA   | 3
2   | CUPO              | ALTA      | EN_ATENCION  | 1
3   | CANCELACION       | ALTA      | ATENDIDA     | 7
4   | CUPO              | CRITICA   | REGISTRADA   | 2
5   | CONSULTA          | BAJA      | CERRADA      | 15
6   | HOMOLOGACION      | MEDIA     | CLASIFICADA  | 5
7   | REGISTRO_ASIG     | ALTA      | REGISTRADA   | 0
8   | CUPO              | ALTA      | EN_ATENCION  | 3
9   | CONSULTA          | BAJA      | REGISTRADA   | 8
10  | CANCELACION       | CRITICA   | CLASIFICADA  | 4
```

Usando Stream API, calcula:
1. La cantidad de solicitudes urgentes (prioridad ALTA o CRITICA).
2. La lista de IDs de solicitudes en estado REGISTRADA, ordenadas por prioridad descendente.
3. Un Map que agrupe las solicitudes por tipo con la cantidad de cada uno.
4. El promedio de días en sistema de todas las solicitudes activas (no CERRADAS).
5. Si hay alguna solicitud CRITICA que lleva más de 3 días sin atenderse.

---

### Reto 4: Optional y Manejo de Nulos

Crea un método `buscarYNotificar(Long id, Map<Long, SolicitudResumenResponse> base)` que:
1. Busque la solicitud en el mapa (puede no existir).
2. Si existe y es urgente, devuelva un mensaje de alerta con el ID.
3. Si existe pero no es urgente, devuelva un mensaje informativo.
4. Si no existe, devuelva un mensaje de "no encontrada".

Implementa el método usando únicamente métodos de Optional (`map`, `filter`, `orElse`, `orElseGet`, `ifPresentOrElse`). No uses `isPresent()` ni `get()` directamente.

---

### Reto 5: Java Moderno Integrado

Crea una clase `AnalizadorDeTriage` con un método `generarInforme(List<SolicitudResumenResponse> solicitudes)` que devuelva un `String` (usando Text Block) con el siguiente formato:

```
═══════════════════════════════════════════
   INFORME DEL SISTEMA DE TRIAGE ACADÉMICO
   Universidad del Quindío
═══════════════════════════════════════════

Total de solicitudes: [X]
Solicitudes activas: [X]
Solicitudes cerradas: [X]

Distribución por estado:
  • REGISTRADA:  [X] solicitudes
  • CLASIFICADA: [X] solicitudes
  • EN_ATENCION: [X] solicitudes
  • ATENDIDA:    [X] solicitudes
  • CERRADA:     [X] solicitudes

⚠️  Solicitudes urgentes sin asignar: [X]

Tipo más frecuente: [TIPO] ([X] solicitudes)
═══════════════════════════════════════════
```

Usa Streams, Optional, Text Blocks, Records y switch expressions en la implementación.

---

## 15. Errores Comunes y Troubleshooting

### Error 1: `UnsupportedOperationException` al modificar una lista inmutable

```java
// ❌ Causa
List<String> estados = List.of("REGISTRADA", "CLASIFICADA");
estados.add("NUEVO"); // UnsupportedOperationException!

// ✅ Solución: si necesitas modificar, usa ArrayList
List<String> estadosModificable = new ArrayList<>(List.of("REGISTRADA", "CLASIFICADA"));
estadosModificable.add("NUEVO"); // OK
```

### Error 2: `NoSuchElementException` al llamar `Optional.get()` sin verificar

```java
// ❌ Causa
Optional<SolicitudResumenResponse> opt = buscar(id);
SolicitudResumenResponse s = opt.get(); // NoSuchElementException si está vacío!

// ✅ Solución: usa orElseThrow() con mensaje descriptivo
SolicitudResumenResponse s = opt.orElseThrow(
    () -> new SolicitudNoEncontradaException("No existe solicitud con ID: " + id)
);
```

### Error 3: Records como entidades JPA

```java
// ❌ Causa
@Entity
public record Solicitud(Long id, String tipo) {} // Error: JPA necesita clase mutable

// ✅ Solución: usa clases normales para entidades JPA, Records solo para DTOs
@Entity
public class Solicitud {
    @Id private Long id;
    private String tipo;
    // setters necesarios para JPA
}
```

### Error 4: Stream ya consumido

```java
// ❌ Causa
var stream = solicitudes.stream().filter(SolicitudResumenResponse::esUrgente);
long count = stream.count();     // OK: primera operación terminal
var lista = stream.toList();     // ❌ IllegalStateException: stream ya consumido

// ✅ Solución: crea un nuevo stream para cada operación
var urgentes = solicitudes.stream()
    .filter(SolicitudResumenResponse::esUrgente)
    .toList(); // guarda el resultado en una colección

long count = urgentes.size(); // Ahora puedes reutilizar la lista
```

### Error 5: `NullPointerException` en `List.of()` con nulos

```java
// ❌ Causa
String responsable = null;
List<String> lista = List.of("Juan", null, "María"); // NullPointerException!

// ✅ Solución: filtra nulos antes o usa ArrayList
List<String> lista = new ArrayList<>(Arrays.asList("Juan", null, "María")); // ArrayList SÍ acepta null
List<String> sinNulos = lista.stream().filter(Objects::nonNull).toList();
```

### Error 6: Pattern Matching en switch no es exhaustivo

```java
// ❌ Error de compilación si Sealed Class tiene subtipos no cubiertos
public String procesar(TipoSolicitudDetalle tipo) {
    return switch (tipo) {
        case SolicitudHomologacion h -> "Homologación";
        // Error: falta cubrir SolicitudCupo, SolicitudCancelacion, etc.
    };
}

// ✅ Solución: cubrir todos los casos permitidos
public String procesar(TipoSolicitudDetalle tipo) {
    return switch (tipo) {
        case SolicitudHomologacion h    -> "Homologación";
        case SolicitudCupo c            -> "Cupo";
        case SolicitudCancelacion ca    -> "Cancelación";
        case SolicitudConsulta con      -> "Consulta";
        case SolicitudRegistroAsignatura r -> "Registro";
    };
}
```

### Error 7: `var` en campos de clase o parámetros

```java
// ❌ Error de compilación
public class MiServicio {
    var repositorio = new ArrayList<>(); // Error: var no funciona en campos

    public void procesar(var solicitud) { // Error: var no funciona en parámetros
    }
}

// ✅ var solo funciona en variables locales dentro de métodos
public void procesar(SolicitudResumenResponse solicitud) {
    var urgente = solicitud.esUrgente(); // OK: variable local
}
```

---

## 16. Resumen y Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════════════╗
║          CHEAT SHEET — JAVA MODERNO PARA EL PROYECTO DE TRIAGE              ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  RECORDS (DTOs)                                                              ║
║  public record MiRecord(String campo1, int campo2) {}                        ║
║  • Getters: miRecord.campo1() — sin "get"                                    ║
║  • equals, hashCode, toString: automáticos                                   ║
║  • Constructor compacto: public MiRecord { /* validar aquí */ }              ║
║  • USO: DTOs, respuestas de API. NO para entidades JPA.                     ║
║                                                                              ║
║  SEALED CLASSES                                                              ║
║  public sealed interface MiTipo permits SubtipoA, SubtipoB {}               ║
║  public record SubtipoA(...) implements MiTipo {}                            ║
║  • Garantiza jerarquías cerradas y conocidas                                 ║
║  • El compilador verifica exhaustividad en switch                            ║
║                                                                              ║
║  PATTERN MATCHING                                                            ║
║  if (obj instanceof MiTipo t) { t.metodo(); }  // con cast automático        ║
║  return switch (sealed) {                                                    ║
║      case SubtipoA a -> "A: " + a.campo();                                   ║
║      case SubtipoB b when b.condicion() -> "B especial";                    ║
║      case SubtipoB b -> "B normal";                                          ║
║  };                                                                          ║
║                                                                              ║
║  TEXT BLOCKS                                                                 ║
║  String s = """                                                              ║
║              SELECT * FROM solicitudes                                       ║
║              WHERE estado = 'REGISTRADA'                                     ║
║              """;                                                            ║
║  Con interpolación: """Hola %s""".formatted(nombre)                         ║
║                                                                              ║
║  VAR                                                                         ║
║  var x = expresión;  // solo en variables locales                            ║
║  ✅ var solicitud = new Solicitud(...);  // tipo obvio                        ║
║  ❌ var result = servicio.procesar(id); // tipo no obvio                     ║
║                                                                              ║
║  OPTIONAL                                                                    ║
║  Optional.of(valor)         // lanza NPE si valor es null                   ║
║  Optional.ofNullable(valor) // acepta null → Optional vacío                 ║
║  Optional.empty()           // sin valor                                     ║
║  opt.orElseThrow(...)       // lanza excepción si vacío ← más común         ║
║  opt.map(fn)                // transforma si existe                          ║
║  opt.filter(pred)           // filtra si existe                              ║
║  opt.ifPresentOrElse(f, g)  // acción si existe O si vacío                  ║
║  ❌ NUNCA: opt.get() sin verificar isPresent() antes                        ║
║  ❌ NUNCA: Optional en campos de clase o parámetros de método                ║
║                                                                              ║
║  STREAMS ESENCIALES                                                          ║
║  .filter(pred)      → conservar los que cumplen condición                    ║
║  .map(fn)           → transformar cada elemento                              ║
║  .flatMap(fn)       → transformar y aplanar colecciones                      ║
║  .sorted(comp)      → ordenar                                                ║
║  .distinct()        → eliminar duplicados                                    ║
║  .limit(n)          → tomar los primeros n                                   ║
║  .skip(n)           → saltar los primeros n                                  ║
║  .toList()          → lista INMUTABLE (Java 16+)                            ║
║  .collect(...)      → recolectar con Collector personalizado                 ║
║  .count()           → contar                                                 ║
║  .findFirst()       → primero como Optional                                  ║
║  .anyMatch(pred)    → ¿alguno cumple?                                        ║
║  groupingBy(fn)     → agrupar en Map<K, List<V>>                            ║
║  groupingBy(fn, counting()) → Map<K, Long> con conteo                       ║
║                                                                              ║
║  COLECCIONES INMUTABLES                                                      ║
║  List.of("a", "b")      → lista inmutable                                   ║
║  Set.of("x", "y")       → set inmutable (sin duplicados, sin null)          ║
║  Map.of("k", "v")       → map inmutable (máx 10 pares)                      ║
║  Map.ofEntries(...)      → map inmutable con más de 10 pares                ║
║  List.copyOf(colección)  → copia inmutable de colección existente           ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

---

## 17. Referencias y Recursos Adicionales

- **JEP 395 – Records:** https://openjdk.org/jeps/395
- **JEP 409 – Sealed Classes:** https://openjdk.org/jeps/409
- **JEP 441 – Pattern Matching for switch (Java 21):** https://openjdk.org/jeps/441
- **JEP 378 – Text Blocks:** https://openjdk.org/jeps/378
- **Java SE 21 API Docs – Optional:** https://docs.oracle.com/en/java/docs/api/java.base/java/util/Optional.html
- **Java SE 21 API Docs – Stream:** https://docs.oracle.com/en/java/docs/api/java.base/java/util/stream/Stream.html
- **Baeldung – Guide to Java Records:** https://www.baeldung.com/java-record-keyword
- **Baeldung – Java Optional:** https://www.baeldung.com/java-optional
- **Baeldung – Java Stream API:** https://www.baeldung.com/java-8-streams
- **Oracle – Java Language Updates:** https://docs.oracle.com/en/java/javase/21/language/

---

> **Autor:** José Alfredo Ramírez Espinosa  
> **Materia:** Programación Avanzada  
> **Programa:** Ingeniería de Sistemas y Computación  
> **Universidad del Quindío** | Armenia, Colombia  
> **Año:** 2026  
> *Material de uso académico. Todos los derechos reservados al autor.*
