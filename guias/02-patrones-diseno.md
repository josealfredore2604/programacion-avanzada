# Guía 02 — Patrones de Diseño en Java: Los Esenciales para Desarrollo Empresarial

> **Materia:** Programación Avanzada | **Guía:** 02 de 12  
> **Prerrequisito:** Guía 01 — Arquitecturas de Software  
> **Tiempo estimado de estudio:** 6–8 horas  
> **Stack:** Java 21+, Spring Boot 3.4+

---

## Tabla de Contenidos

1. [Objetivos de Aprendizaje](#1-objetivos-de-aprendizaje)
2. [¿Qué es un Patrón de Diseño?](#2-qué-es-un-patrón-de-diseño)
3. [Historia y Clasificación GoF](#3-historia-y-clasificación-gof)
4. [Principios SOLID](#4-principios-solid)
5. [Patrón Singleton](#5-patrón-singleton)
6. [Patrón Factory / Abstract Factory](#6-patrón-factory--abstract-factory)
7. [Patrón Strategy](#7-patrón-strategy)
8. [Patrón Observer](#8-patrón-observer)
9. [Patrón Builder](#9-patrón-builder)
10. [Patrón DTO (Data Transfer Object)](#10-patrón-dto-data-transfer-object)
11. [Patrón Repository](#11-patrón-repository)
12. [Patrón Service Layer](#12-patrón-service-layer)
13. [MVC vs Arquitectura en Capas](#13-mvc-vs-arquitectura-en-capas)
14. [Cómo se Articulan Todos los Patrones en el Proyecto de Triage](#14-cómo-se-articulan-todos-los-patrones-en-el-proyecto-de-triage)
15. [Ejercicios Prácticos](#15-ejercicios-prácticos)
16. [Errores Comunes y Troubleshooting](#16-errores-comunes-y-troubleshooting)
17. [Resumen y Cheat Sheet](#17-resumen-y-cheat-sheet)
18. [Referencias y Recursos Adicionales](#18-referencias-y-recursos-adicionales)

---

## 1. Objetivos de Aprendizaje

Al terminar esta guía vas a poder:

- Explicar qué es un patrón de diseño y por qué existe
- Aplicar los cinco principios SOLID en código Java real
- Implementar los patrones Singleton, Factory, Strategy, Observer, Builder, DTO, Repository y Service Layer
- Comparar la implementación tradicional vs la moderna (Java 21 + Spring Boot) de cada patrón
- Identificar qué patrón aplica en qué parte del proyecto "Sistema de Triage y Gestión de Solicitudes Académicas"
- Entender cómo Spring Boot usa estos patrones internamente

---

## 2. ¿Qué es un Patrón de Diseño?

### La analogía del plano arquitectónico

Imagina que eres un arquitecto y tienes que diseñar la entrada de un edificio de oficinas. No inventas desde cero cómo hacer que la puerta sea accesible para personas en silla de ruedas: existe un estándar, una solución probada, que dice exactamente qué rampa usar, qué inclinación, qué ancho mínimo. Ese estándar es el resultado de que miles de arquitectos resolvieron el mismo problema antes que tú.

Los **patrones de diseño** son exactamente eso en programación: **soluciones probadas y reutilizables para problemas recurrentes en el diseño de software**. No son código que puedes copiar y pegar directamente, sino plantillas conceptuales que describes cómo estructurar clases e interacciones para resolver un problema específico.

### Definición formal

> Un patrón de diseño es una solución general y reutilizable a un problema que ocurre frecuentemente dentro de un contexto dado en el diseño de software.

### ¿Por qué importan?

1. **Vocabulario común:** Cuando dices "vamos a usar un Observer aquí", tu equipo entiende de qué hablas sin que tengas que dibujar un diagrama completo.
2. **Soluciones probadas:** Alguien ya encontró los problemas de esa solución y los documentó.
3. **Mantenibilidad:** El código organizado con patrones es más fácil de modificar meses después.
4. **Separación de responsabilidades:** Los patrones guían hacia código donde cada clase hace una sola cosa bien.

### ¿Qué NO son los patrones?

- **No son algoritmos:** Un algoritmo de ordenamiento resuelve un paso específico. Un patrón resuelve un problema de diseño a mayor nivel de abstracción.
- **No son librerías:** No los importas, los implementas según el problema.
- **No son obligatorios:** Aplicar un patrón donde no hace falta produce complejidad innecesaria — el famoso "over-engineering".

---

## 3. Historia y Clasificación GoF

### El Libro de los Patrones

En 1994, cuatro ingenieros — Erich Gamma, Richard Helm, Ralph Johnson y John Vlissides — publicaron el libro **"Design Patterns: Elements of Reusable Object-Oriented Software"**. Son conocidos colectivamente como el **Gang of Four (GoF)**. En ese libro catalogaron 23 patrones clásicos divididos en tres categorías.

Veintinueve años después ese catálogo sigue siendo la referencia fundamental. Spring Boot, Hibernate, Angular — todos están construidos sobre estos patrones.

### Las Tres Categorías

```
PATRONES GoF
│
├── CREACIONALES (¿Cómo crear objetos?)
│   ├── Singleton       → Una única instancia
│   ├── Factory Method  → Delegar la creación a subclases
│   ├── Abstract Factory→ Familias de objetos relacionados
│   ├── Builder         → Construir objetos complejos paso a paso
│   └── Prototype       → Clonar objetos existentes
│
├── ESTRUCTURALES (¿Cómo componer clases y objetos?)
│   ├── Adapter         → Interfaz compatible entre clases
│   ├── Bridge          → Separar abstracción de implementación
│   ├── Composite       → Tratar objetos individuales y compuestos igual
│   ├── Decorator       → Añadir responsabilidades dinámicamente
│   ├── Facade          → Interfaz simplificada a un sistema complejo
│   ├── Flyweight       → Compartir estado entre muchos objetos
│   └── Proxy           → Sustituto o marcador de otro objeto
│
└── DE COMPORTAMIENTO (¿Cómo se comunican los objetos?)
    ├── Observer        → Notificar cambios a múltiples objetos
    ├── Strategy        → Intercambiar algoritmos en tiempo de ejecución
    ├── Command         → Encapsular acciones como objetos
    ├── Iterator        → Recorrer colecciones sin exponer su estructura
    ├── Template Method → Esqueleto de un algoritmo con pasos flexibles
    ├── State           → Cambiar comportamiento según estado interno
    ├── Chain of Resp.  → Pasar peticiones por cadena de manejadores
    ├── Interpreter     → Gramática e intérprete para un lenguaje
    ├── Mediator        → Reduce acoplamiento entre objetos
    ├── Memento         → Guardar y restaurar estado de un objeto
    └── Visitor         → Operaciones sobre elementos de una estructura
```

En esta guía nos enfocamos en los que vas a usar directamente en el proyecto de Triage: **Singleton, Factory, Strategy, Observer, Builder, DTO, Repository y Service Layer** (los últimos dos son patrones empresariales, no GoF puros, pero fundamentales).

---

## 4. Principios SOLID

Antes de ver cada patrón, necesitas entender los principios SOLID. Son la filosofía detrás de casi todos los patrones. Los patrones GoF son implementaciones concretas de estos principios.

SOLID es un acrónimo creado por Robert C. Martin ("Uncle Bob"):

```
S → Single Responsibility Principle (Responsabilidad Única)
O → Open/Closed Principle (Abierto/Cerrado)
L → Liskov Substitution Principle (Sustitución de Liskov)
I → Interface Segregation Principle (Segregación de Interfaces)
D → Dependency Inversion Principle (Inversión de Dependencias)
```

Vamos a ver cada uno con ejemplos directamente del proyecto de Triage.

---

### 4.1 S — Single Responsibility Principle (SRP)

> **"Una clase debe tener una única razón para cambiar."**

#### El problema

```java
// ❌ VIOLACIÓN DE SRP — esta clase hace demasiadas cosas
public class SolicitudService {

    // Lógica de negocio → OK
    public void registrarSolicitud(Solicitud s) { /* ... */ }

    // Acceso a base de datos → NO debería estar aquí
    public void guardarEnBaseDeDatos(Solicitud s) {
        // Conexión JDBC directa
        Connection conn = DriverManager.getConnection("jdbc:postgresql://...");
        // ...
    }

    // Envío de correos → NO debería estar aquí
    public void enviarCorreoConfirmacion(String email) {
        // Lógica de SMTP
    }

    // Generación de PDF → NO debería estar aquí
    public byte[] generarPDF(Solicitud s) {
        // ...
    }
}
```

¿Cuántas razones tiene esta clase para cambiar? Cuatro:
1. Si cambia la lógica de negocio de solicitudes
2. Si cambias de PostgreSQL a MySQL
3. Si cambias de SMTP a SendGrid
4. Si cambias de iText a Apache PDFBox

#### La solución

```java
// ✅ APLICANDO SRP — cada clase tiene UNA responsabilidad

// Responsabilidad: Lógica de negocio
public class SolicitudService {
    private final SolicitudRepository repository;
    private final NotificacionService notificaciones;

    public void registrarSolicitud(Solicitud s) {
        validar(s);
        repository.save(s);
        notificaciones.notificarRegistro(s);
    }
}

// Responsabilidad: Acceso a datos
public interface SolicitudRepository {
    Solicitud save(Solicitud s);
    Optional<Solicitud> findById(Long id);
}

// Responsabilidad: Envío de notificaciones
public class NotificacionService {
    public void notificarRegistro(Solicitud s) { /* ... */ }
}

// Responsabilidad: Generación de documentos
public class DocumentoService {
    public byte[] generarPDF(Solicitud s) { /* ... */ }
}
```

---

### 4.2 O — Open/Closed Principle (OCP)

> **"Las entidades de software deben estar abiertas para extensión pero cerradas para modificación."**

Esto significa: cuando necesites agregar nueva funcionalidad, agrégala en código nuevo, no modifiques el código existente que ya funciona.

#### El problema

```java
// ❌ VIOLACIÓN DE OCP — cada nuevo tipo de prioridad requiere modificar este método
public class PriorizadorSolicitud {

    public int calcularPrioridad(Solicitud solicitud) {
        if (solicitud.getTipo() == TipoSolicitud.HOMOLOGACION) {
            return 1; // Alta prioridad
        } else if (solicitud.getTipo() == TipoSolicitud.CANCELACION) {
            return 3; // Media prioridad
        } else if (solicitud.getTipo() == TipoSolicitud.CUPO) {
            return 2;
        }
        // Si mañana agrego RECURSO_APELACION, TENGO que modificar esta clase ←— problema
        return 5;
    }
}
```

#### La solución

```java
// ✅ APLICANDO OCP — nuevas estrategias se agregan sin modificar el priorizador base

public interface EstrategiaPriorizacion {
    int calcularPrioridad(Solicitud solicitud);
    boolean aplica(TipoSolicitud tipo);
}

// Estrategia existente
public class PrioridadHomologacion implements EstrategiaPriorizacion {
    @Override
    public int calcularPrioridad(Solicitud solicitud) { return 1; }

    @Override
    public boolean aplica(TipoSolicitud tipo) {
        return tipo == TipoSolicitud.HOMOLOGACION;
    }
}

// Nueva estrategia — no toco ningún código existente
public class PrioridadRecursoApelacion implements EstrategiaPriorizacion {
    @Override
    public int calcularPrioridad(Solicitud solicitud) { return 1; }

    @Override
    public boolean aplica(TipoSolicitud tipo) {
        return tipo == TipoSolicitud.RECURSO_APELACION;
    }
}

// El priorizador usa las estrategias — nunca lo modificas
public class PriorizadorSolicitud {
    private final List<EstrategiaPriorizacion> estrategias;

    public int calcularPrioridad(Solicitud solicitud) {
        return estrategias.stream()
            .filter(e -> e.aplica(solicitud.getTipo()))
            .findFirst()
            .map(e -> e.calcularPrioridad(solicitud))
            .orElse(5); // prioridad por defecto
    }
}
```

Este ejemplo también es el patrón **Strategy**, que veremos en detalle más adelante.

---

### 4.3 L — Liskov Substitution Principle (LSP)

> **"Los objetos de una subclase deben poder reemplazar a los objetos de la superclase sin alterar el comportamiento correcto del programa."**

En términos simples: si tienes una variable de tipo `SolicitudRepository`, y en algún momento la reemplazas por `SolicitudRepositoryPostgres` o `SolicitudRepositoryOracle`, el programa debe seguir funcionando exactamente igual.

```java
// ✅ Aplicando LSP correctamente

public interface SolicitudRepository {
    Solicitud save(Solicitud s);
    Optional<Solicitud> findById(Long id);
    List<Solicitud> findAll();
}

// Esta implementación cumple el contrato de la interfaz
public class SolicitudRepositoryPostgres implements SolicitudRepository {
    @Override
    public Solicitud save(Solicitud s) {
        // Guarda en PostgreSQL
        return s;
    }

    @Override
    public Optional<Solicitud> findById(Long id) {
        // Busca en PostgreSQL
        return Optional.empty();
    }

    @Override
    public List<Solicitud> findAll() {
        // Lista de PostgreSQL
        return List.of();
    }
}

// Esta implementación también cumple — se puede usar como reemplazo
public class SolicitudRepositoryInMemory implements SolicitudRepository {
    private final Map<Long, Solicitud> store = new HashMap<>();

    @Override
    public Solicitud save(Solicitud s) {
        store.put(s.getId(), s);
        return s;
    }

    @Override
    public Optional<Solicitud> findById(Long id) {
        return Optional.ofNullable(store.get(id));
    }

    @Override
    public List<Solicitud> findAll() {
        return new ArrayList<>(store.values());
    }
}

// El servicio no sabe (ni le importa) qué implementación se usa
public class SolicitudService {
    private final SolicitudRepository repository; // ← Interfaz, no implementación

    public SolicitudService(SolicitudRepository repository) {
        this.repository = repository;
    }
}
```

---

### 4.4 I — Interface Segregation Principle (ISP)

> **"Los clientes no deben verse obligados a depender de interfaces que no usan."**

Mejor muchas interfaces pequeñas y específicas que una interfaz grande y genérica.

```java
// ❌ VIOLACIÓN DE ISP — interfaz demasiado grande
public interface GestorSolicitudes {
    void registrar(Solicitud s);
    void clasificar(Solicitud s, TipoSolicitud tipo);
    void priorizar(Solicitud s, int prioridad);
    void asignarResponsable(Solicitud s, Usuario responsable);
    void cerrar(Solicitud s, String observacion);
    void generarReporte(List<Solicitud> solicitudes);
    void enviarNotificacion(Solicitud s);
    void exportarExcel(List<Solicitud> solicitudes);
}

// El ESTUDIANTE solo debería poder registrar, pero si implementa esta interfaz
// se ve forzado a implementar clasificar(), cerrar(), generarReporte()...

// ✅ APLICANDO ISP — interfaces específicas por rol/responsabilidad
public interface RegistradorSolicitudes {
    void registrar(Solicitud s);
}

public interface ClasificadorSolicitudes {
    void clasificar(Solicitud s, TipoSolicitud tipo);
    void priorizar(Solicitud s, int prioridad);
}

public interface GestorAsignaciones {
    void asignarResponsable(Solicitud s, Usuario responsable);
}

public interface CierreSolicitudes {
    void cerrar(Solicitud s, String observacion);
}

public interface GeneradorReportes {
    void generarReporte(List<Solicitud> solicitudes);
    void exportarExcel(List<Solicitud> solicitudes);
}

// Cada clase implementa solo lo que necesita
public class SolicitudServiceEstudiante implements RegistradorSolicitudes {
    @Override
    public void registrar(Solicitud s) { /* ... */ }
}

public class SolicitudServiceFuncionario
    implements ClasificadorSolicitudes, GestorAsignaciones, CierreSolicitudes {
    @Override public void clasificar(Solicitud s, TipoSolicitud tipo) { /* ... */ }
    @Override public void priorizar(Solicitud s, int prioridad) { /* ... */ }
    @Override public void asignarResponsable(Solicitud s, Usuario u) { /* ... */ }
    @Override public void cerrar(Solicitud s, String obs) { /* ... */ }
}
```

---

### 4.5 D — Dependency Inversion Principle (DIP)

> **"Los módulos de alto nivel no deben depender de módulos de bajo nivel. Ambos deben depender de abstracciones. Las abstracciones no deben depender de detalles. Los detalles deben depender de abstracciones."**

Este es el principio que hace posible la **Inyección de Dependencias** (DI) de Spring.

```java
// ❌ VIOLACIÓN DE DIP — módulo de alto nivel depende de módulo de bajo nivel
public class SolicitudService {
    // SolicitudService (alto nivel) crea directamente SolicitudRepositoryPostgres (bajo nivel)
    private final SolicitudRepositoryPostgres repository = new SolicitudRepositoryPostgres();

    public void registrar(Solicitud s) {
        repository.save(s);
    }
}
// Problema: si cambias la BD, tienes que modificar SolicitudService.

// ✅ APLICANDO DIP — ambos dependen de la abstracción (interfaz)
public interface SolicitudRepository { // ← La abstracción
    Solicitud save(Solicitud s);
}

public class SolicitudRepositoryPostgres implements SolicitudRepository { // ← Detalle
    @Override
    public Solicitud save(Solicitud s) { /* PostgreSQL */ return s; }
}

public class SolicitudService { // ← Alto nivel
    private final SolicitudRepository repository; // ← Depende de abstracción

    // Spring inyecta la implementación concreta aquí
    public SolicitudService(SolicitudRepository repository) {
        this.repository = repository;
    }

    public void registrar(Solicitud s) {
        repository.save(s); // No sabe si es Postgres, MySQL, o In-Memory
    }
}
```

Así es exactamente como Spring Boot maneja la inyección de dependencias. Defines interfaces, Spring decide qué implementación inyectar.

---

### Tabla Resumen SOLID

| Principio | Pregunta clave | Si lo violas... |
|-----------|---------------|-----------------|
| **S**RP | ¿Esta clase tiene una sola razón para cambiar? | Un cambio rompe cosas no relacionadas |
| **O**CP | ¿Puedo extender sin modificar? | Agregar funciones requiere editar clases existentes |
| **L**SP | ¿La subclase se puede usar donde uso la superclase? | El polimorfismo produce comportamientos inesperados |
| **I**SP | ¿Cada cliente usa todo lo que implementa? | Clases con métodos vacíos o que lanzan `UnsupportedOperationException` |
| **D**IP | ¿Dependo de interfaces, no de implementaciones? | El código es difícil de testear y de cambiar |

---

## 5. Patrón Singleton

### Problema que resuelve

A veces necesitas que exista **exactamente una instancia** de una clase durante toda la vida de la aplicación. Ejemplos típicos:
- La conexión al pool de base de datos
- Un registro de logs
- La configuración de la aplicación
- Un caché en memoria

Si crearas múltiples instancias de la conexión a la BD, tendrías múltiples pools, desperdicio de recursos y posibles inconsistencias.

### Diagrama

```
┌─────────────────────────────────────┐
│           Singleton                 │
├─────────────────────────────────────┤
│ - instancia: Singleton (estática)   │
├─────────────────────────────────────┤
│ - Singleton() (privado)             │
│ + getInstance(): Singleton (static) │
│ + operacion()                       │
└─────────────────────────────────────┘
            │ retorna
            ▼
    [ La misma instancia siempre ]
```

### Implementación clásica (Java pre-11) — conocerla pero no usarla

```java
// ❌ Implementación clásica — problemática en concurrencia
public class ConfiguracionApp {
    private static ConfiguracionApp instancia; // Sin volatile → problema en hilos

    private final String bdUrl;

    private ConfiguracionApp() {
        this.bdUrl = System.getenv("DATABASE_URL");
    }

    // Problema: si dos hilos llegan simultáneamente cuando instancia == null
    // pueden crear DOS instancias
    public static ConfiguracionApp getInstance() {
        if (instancia == null) {
            instancia = new ConfiguracionApp(); // Race condition aquí
        }
        return instancia;
    }
}
```

### Implementación correcta en Java moderno (sin Spring)

```java
// ✅ Implementación thread-safe con Initialization-on-demand holder
public class ConfiguracionApp {

    private final String bdUrl;
    private final int maxConexiones;

    private ConfiguracionApp() {
        this.bdUrl = System.getenv("DATABASE_URL");
        this.maxConexiones = Integer.parseInt(
            System.getenv().getOrDefault("MAX_CONEXIONES", "10")
        );
    }

    // El ClassLoader de Java garantiza que este holder se inicializa una sola vez
    private static final class Holder {
        private static final ConfiguracionApp INSTANCIA = new ConfiguracionApp();
    }

    public static ConfiguracionApp getInstance() {
        return Holder.INSTANCIA;
    }

    public String getBdUrl() { return bdUrl; }
    public int getMaxConexiones() { return maxConexiones; }
}

// Uso:
ConfiguracionApp config = ConfiguracionApp.getInstance();
System.out.println(config.getBdUrl());
```

### Implementación con enum (la más robusta en Java puro)

```java
// ✅ Singleton con enum — protegido incluso contra serialización y reflexión
public enum ConfiguracionSingleton {
    INSTANCIA;

    private final String bdUrl = System.getenv("DATABASE_URL");
    private final int maxConexiones = 10;

    public String getBdUrl() { return bdUrl; }
    public int getMaxConexiones() { return maxConexiones; }
}

// Uso:
String url = ConfiguracionSingleton.INSTANCIA.getBdUrl();
```

### Cómo Spring Boot implementa Singleton automáticamente

Esta es la parte más importante para ti. **En Spring Boot, todos los beans son Singleton por defecto**. No tienes que implementar el patrón manualmente; Spring lo maneja solo.

```java
// En Spring, esta clase es automáticamente un Singleton
@Service  // ← Esta anotación le dice a Spring: "adminístrala como bean"
public class SolicitudService {

    // Spring inyecta automáticamente UNA SOLA instancia de SolicitudRepository
    private final SolicitudRepository repository;

    public SolicitudService(SolicitudRepository repository) {
        this.repository = repository;
    }

    // ...
}
```

```java
// Si en otro lugar del sistema pides SolicitudService, Spring te da LA MISMA instancia
@RestController
public class SolicitudController {

    // Spring inyecta aquí → mismo objeto que en cualquier otro lugar
    private final SolicitudService service;

    public SolicitudController(SolicitudService service) {
        this.service = service;
    }
}
```

**Spring tiene 5 scopes de beans:**

| Scope | Descripción | Cuándo usarlo |
|-------|-------------|---------------|
| `singleton` | **Una instancia por ApplicationContext (por defecto)** | La mayoría de los casos |
| `prototype` | Nueva instancia cada vez que se pide | Cuando el estado no debe compartirse |
| `request` | Una por petición HTTP | Solo en aplicaciones web |
| `session` | Una por sesión HTTP | Datos de sesión de usuario |
| `application` | Una por ServletContext | Datos globales de la app web |

```java
// Cambiar el scope cuando necesites
@Service
@Scope("prototype") // ← Nueva instancia cada vez
public class GeneradorReporteTemporal {
    // Estado que no debe compartirse entre peticiones
}
```

---

## 6. Patrón Factory / Abstract Factory

### Problema que resuelve

Imagina que en el sistema de Triage necesitas crear diferentes tipos de notificaciones: por correo, por SMS, por notificación push. La lógica de *cómo crearlas* es diferente para cada tipo, pero *cuándo y por qué crearlas* es la misma.

**El patrón Factory** centraliza la lógica de creación de objetos. En lugar de usar `new TipoConcreto()` directamente en tu lógica de negocio, delegas esa creación a una clase fábrica.

### Diagrama

```
           ┌──────────────────┐
           │  NotificacionFactory │
           ├──────────────────┤
           │ + crear(tipo): Notificacion │
           └────────┬─────────┘
                    │ crea
         ┌──────────┼──────────┐
         ▼          ▼          ▼
  ┌──────────┐ ┌─────────┐ ┌──────────┐
  │Notif.    │ │Notif.   │ │Notif.    │
  │Correo    │ │SMS      │ │Push      │
  └──────────┘ └─────────┘ └──────────┘
         │          │          │
         └──────────┴──────────┘
                    │
            implementan
                    │
           ┌────────▼─────────┐
           │  <<interface>>   │
           │  Notificacion    │
           ├──────────────────┤
           │ + enviar()       │
           │ + getTipo()      │
           └──────────────────┘
```

### Implementación en el Proyecto de Triage

Vas a crear un sistema de notificaciones donde, según el tipo de solicitud o el canal de origen, se envía una notificación diferente.

```java
// 1. La interfaz común que todas las notificaciones implementan
public interface Notificacion {
    void enviar(String destinatario, String mensaje);
    String getTipo();
}
```

```java
// 2. Implementación concreta: notificación por correo
public class NotificacionCorreo implements Notificacion {

    @Override
    public void enviar(String destinatario, String mensaje) {
        System.out.printf("📧 Enviando correo a %s: %s%n", destinatario, mensaje);
        // Aquí iría la integración real con JavaMail o SendGrid
    }

    @Override
    public String getTipo() { return "CORREO"; }
}
```

```java
// 3. Implementación concreta: notificación por SMS
public class NotificacionSMS implements Notificacion {

    @Override
    public void enviar(String destinatario, String mensaje) {
        System.out.printf("📱 Enviando SMS a %s: %s%n", destinatario, mensaje);
        // Aquí iría la integración con Twilio o similar
    }

    @Override
    public String getTipo() { return "SMS"; }
}
```

```java
// 4. Implementación concreta: notificación push
public class NotificacionPush implements Notificacion {

    @Override
    public void enviar(String destinatario, String mensaje) {
        System.out.printf("🔔 Enviando push a %s: %s%n", destinatario, mensaje);
        // Aquí iría Firebase Cloud Messaging
    }

    @Override
    public String getTipo() { return "PUSH"; }
}
```

```java
// 5. El enum de tipos de canal (lo usarás en RF-01)
public enum CanalOrigen {
    CORREO, SMS, PRESENCIAL, TELEFONO, SAC
}
```

```java
// 6. La Factory — aquí está la magia
public class NotificacionFactory {

    /**
     * Crea el tipo de notificación adecuado según el canal de origen de la solicitud.
     * Así el servicio de negocio no necesita saber cómo se construye cada notificación.
     */
    public static Notificacion crear(CanalOrigen canal) {
        return switch (canal) {           // Java 14+ switch expression
            case CORREO   -> new NotificacionCorreo();
            case SMS      -> new NotificacionSMS();
            case PRESENCIAL, TELEFONO, SAC -> new NotificacionCorreo(); // fallback a correo
        };
    }

    // Sobrecarga útil: crear por tipo explícito
    public static Notificacion crear(String tipo) {
        return switch (tipo.toUpperCase()) {
            case "CORREO" -> new NotificacionCorreo();
            case "SMS"    -> new NotificacionSMS();
            case "PUSH"   -> new NotificacionPush();
            default -> throw new IllegalArgumentException(
                "Tipo de notificación no reconocido: " + tipo
            );
        };
    }
}
```

```java
// 7. Cómo lo usas en el servicio — sin saber qué tipo concreto se crea
public class SolicitudService {

    public void registrarSolicitud(Solicitud solicitud) {
        // ... lógica de persistencia ...

        // La factory decide QUÉ notificación crear
        // El servicio solo sabe que EXISTE una notificación
        Notificacion notificacion = NotificacionFactory.crear(solicitud.getCanalOrigen());
        notificacion.enviar(
            solicitud.getSolicitante().getEmail(),
            "Tu solicitud #" + solicitud.getId() + " fue registrada exitosamente."
        );
    }
}
```

### Versión Spring Boot — cómo la factory se convierte en un @Component

En Spring Boot, la factory generalmente se implementa como un servicio:

```java
// ✅ Factory como Spring Bean — permite inyección de dependencias
@Component
public class NotificacionFactory {

    // Spring puede inyectar en cada notificación lo que necesite
    // Por ejemplo, el servicio de correo ya configurado
    private final JavaMailSender mailSender;
    private final SmsClient smsClient;

    public NotificacionFactory(JavaMailSender mailSender, SmsClient smsClient) {
        this.mailSender = mailSender;
        this.smsClient = smsClient;
    }

    public Notificacion crear(CanalOrigen canal) {
        return switch (canal) {
            case CORREO -> new NotificacionCorreo(mailSender);    // ← ya inicializada
            case SMS    -> new NotificacionSMS(smsClient);        // ← ya inicializada
            default     -> new NotificacionCorreo(mailSender);
        };
    }
}

// El servicio recibe la factory inyectada por Spring
@Service
public class SolicitudService {

    private final SolicitudRepository repository;
    private final NotificacionFactory notificacionFactory;

    public SolicitudService(SolicitudRepository repository,
                            NotificacionFactory notificacionFactory) {
        this.repository = repository;
        this.notificacionFactory = notificacionFactory;
    }

    public void registrarSolicitud(Solicitud solicitud) {
        repository.save(solicitud);
        var notificacion = notificacionFactory.crear(solicitud.getCanalOrigen());
        notificacion.enviar(solicitud.getSolicitante().getEmail(), "Registrada.");
    }
}
```

### Abstract Factory — cuando hay familias de objetos relacionados

El **Abstract Factory** es una evolución del Factory. En lugar de crear un solo objeto, crea *familias* de objetos que deben usarse juntos.

```java
// Escenario: el sistema de Triage tiene dos "temas" de notificación:
// - Notificaciones formales (para trámites oficiales)
// - Notificaciones informales (para recordatorios)
// Cada tema tiene su propia versión de correo, SMS y push.

public interface NotificacionAbstractFactory {
    Notificacion crearNotificacionCorreo();
    Notificacion crearNotificacionSMS();
    Notificacion crearNotificacionPush();
}

// Familia 1: Notificaciones formales
public class NotificacionFormalFactory implements NotificacionAbstractFactory {
    @Override
    public Notificacion crearNotificacionCorreo() {
        return new NotificacionCorreoFormal();
    }
    @Override
    public Notificacion crearNotificacionSMS() {
        return new NotificacionSMSFormal();
    }
    @Override
    public Notificacion crearNotificacionPush() {
        return new NotificacionPushFormal();
    }
}

// Familia 2: Notificaciones informales
public class NotificacionInformalFactory implements NotificacionAbstractFactory {
    @Override
    public Notificacion crearNotificacionCorreo() {
        return new NotificacionCorreoInformal();
    }
    @Override
    public Notificacion crearNotificacionSMS() {
        return new NotificacionSMSInformal();
    }
    @Override
    public Notificacion crearNotificacionPush() {
        return new NotificacionPushInformal();
    }
}
```

**¿Cuándo usar Factory vs Abstract Factory?**

| | Factory Method | Abstract Factory |
|---|---|---|
| **Crea** | Un tipo de objeto | Familias de objetos relacionados |
| **Cuándo** | Un solo producto variable | Múltiples productos que se usan juntos |
| **Ejemplo Triage** | Crear el tipo de notificación correcta | Crear tema completo de UI (formal/informal) |

---

## 7. Patrón Strategy

### Problema que resuelve

En el proyecto de Triage, el RF-03 dice: *"El sistema debe asignar una prioridad a cada solicitud con base en reglas definidas, tales como: tipo de solicitud, impacto académico, fecha límite asociada."*

Las reglas de priorización pueden cambiar: hoy priorizamos por tipo, mañana por fecha límite, la siguiente semana por algún algoritmo nuevo. Si hardcodeas la lógica en el servicio, cada cambio implica modificar y volver a desplegar el servicio principal.

El **patrón Strategy** define una familia de algoritmos, los encapsula en clases separadas y los hace intercambiables en tiempo de ejecución.

### Diagrama

```
┌─────────────────────┐       usa          ┌─────────────────────┐
│   SolicitudService  │──────────────────▶ │ <<interface>>       │
│ (Contexto)          │                    │ EstrategiaPrio...   │
├─────────────────────┤                    ├─────────────────────┤
│ -estrategia         │                    │ +calcularPrioridad()│
│ +setEstrategia()    │                    └──────────┬──────────┘
│ +procesarSolicitud()│                               │
└─────────────────────┘                    implementan│
                                    ┌──────────────────┼──────────────────┐
                                    ▼                  ▼                  ▼
                          ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
                          │ PrioridadPor │  │ PrioridadPor │  │ PrioridadPor │
                          │    Tipo      │  │ FechaLimite  │  │   Impacto    │
                          └──────────────┘  └──────────────┘  └──────────────┘
```

### Implementación Completa en el Triage

Primero definimos el modelo de solicitud simplificado para los ejemplos:

```java
// Modelo simplificado para este ejemplo
public enum TipoSolicitud {
    HOMOLOGACION,
    CANCELACION_ASIGNATURA,
    SOLICITUD_CUPO,
    CONSULTA_ACADEMICA,
    RECURSO_APELACION
}

public class Solicitud {
    private Long id;
    private TipoSolicitud tipo;
    private String descripcion;
    private LocalDate fechaLimite;
    private int impactoAcademico; // 1-10
    private int prioridad;

    // getters y setters (o usar Record — ver Guía 03)
    public Long getId() { return id; }
    public TipoSolicitud getTipo() { return tipo; }
    public LocalDate getFechaLimite() { return fechaLimite; }
    public int getImpactoAcademico() { return impactoAcademico; }
    public void setPrioridad(int prioridad) { this.prioridad = prioridad; }
    public int getPrioridad() { return prioridad; }
}
```

```java
// 1. La interfaz Strategy
public interface EstrategiaPriorizacion {
    /**
     * Calcula la prioridad de una solicitud.
     * @return número entero: 1 = prioridad máxima, 5 = mínima prioridad
     */
    int calcularPrioridad(Solicitud solicitud);

    /**
     * Descripción de la estrategia para el historial auditable (RF-06)
     */
    String getDescripcion();
}
```

```java
// 2. Estrategia concreta: priorizar por tipo de solicitud
public class EstrategiaPorTipo implements EstrategiaPriorizacion {

    private static final Map<TipoSolicitud, Integer> PRIORIDADES = Map.of(
        TipoSolicitud.RECURSO_APELACION,      1, // Urgente
        TipoSolicitud.HOMOLOGACION,            2, // Alta
        TipoSolicitud.SOLICITUD_CUPO,          2, // Alta
        TipoSolicitud.CANCELACION_ASIGNATURA,  3, // Media
        TipoSolicitud.CONSULTA_ACADEMICA,      4  // Baja
    );

    @Override
    public int calcularPrioridad(Solicitud solicitud) {
        return PRIORIDADES.getOrDefault(solicitud.getTipo(), 5);
    }

    @Override
    public String getDescripcion() {
        return "Priorización por tipo de solicitud";
    }
}
```

```java
// 3. Estrategia concreta: priorizar por fecha límite
public class EstrategiaPorFechaLimite implements EstrategiaPriorizacion {

    @Override
    public int calcularPrioridad(Solicitud solicitud) {
        if (solicitud.getFechaLimite() == null) {
            return 3; // Sin fecha límite → prioridad media
        }

        long diasRestantes = ChronoUnit.DAYS.between(
            LocalDate.now(),
            solicitud.getFechaLimite()
        );

        if (diasRestantes <= 1)  return 1; // Urgentísimo
        if (diasRestantes <= 3)  return 2; // Urgente
        if (diasRestantes <= 7)  return 3; // Esta semana
        if (diasRestantes <= 15) return 4; // Próximas dos semanas
        return 5;                          // Sin urgencia
    }

    @Override
    public String getDescripcion() {
        return "Priorización por fecha límite";
    }
}
```

```java
// 4. Estrategia concreta: priorizar por impacto académico
public class EstrategiaPorImpacto implements EstrategiaPriorizacion {

    @Override
    public int calcularPrioridad(Solicitud solicitud) {
        int impacto = solicitud.getImpactoAcademico(); // 1-10

        if (impacto >= 9)  return 1;
        if (impacto >= 7)  return 2;
        if (impacto >= 5)  return 3;
        if (impacto >= 3)  return 4;
        return 5;
    }

    @Override
    public String getDescripcion() {
        return "Priorización por impacto académico";
    }
}
```

```java
// 5. Estrategia compuesta: combina múltiples estrategias con pesos
public class EstrategiaCompuesta implements EstrategiaPriorizacion {

    private final List<EstrategiaPriorizacion> estrategias;

    public EstrategiaCompuesta(List<EstrategiaPriorizacion> estrategias) {
        this.estrategias = List.copyOf(estrategias);
    }

    @Override
    public int calcularPrioridad(Solicitud solicitud) {
        // Promedio de todas las estrategias, redondeado
        double promedio = estrategias.stream()
            .mapToInt(e -> e.calcularPrioridad(solicitud))
            .average()
            .orElse(3.0);

        return (int) Math.round(promedio);
    }

    @Override
    public String getDescripcion() {
        return "Priorización compuesta: " + estrategias.stream()
            .map(EstrategiaPriorizacion::getDescripcion)
            .collect(Collectors.joining(", "));
    }
}
```

```java
// 6. El contexto que usa la estrategia — el servicio de priorización
@Service
public class PriorizacionService {

    // La estrategia activa se puede cambiar en tiempo de ejecución
    private EstrategiaPriorizacion estrategiaActiva;

    public PriorizacionService() {
        // Por defecto, usamos la estrategia por tipo
        this.estrategiaActiva = new EstrategiaPorTipo();
    }

    /**
     * Permite cambiar la estrategia en tiempo de ejecución.
     * Por ejemplo, durante período de inscripciones, activar EstrategiaPorFechaLimite.
     */
    public void setEstrategia(EstrategiaPriorizacion estrategia) {
        this.estrategiaActiva = estrategia;
    }

    /**
     * Aplica la estrategia activa a una solicitud y registra en el historial.
     */
    public void priorizarSolicitud(Solicitud solicitud) {
        int prioridad = estrategiaActiva.calcularPrioridad(solicitud);
        solicitud.setPrioridad(prioridad);

        System.out.printf(
            "Solicitud #%d priorizada con valor %d usando: %s%n",
            solicitud.getId(),
            prioridad,
            estrategiaActiva.getDescripcion()
        );
    }
}
```

```java
// 7. Demostración del patrón — cómo se usa
public class DemoStrategy {

    public static void main(String[] args) {
        var service = new PriorizacionService();
        var solicitud = new Solicitud();
        solicitud.setTipo(TipoSolicitud.HOMOLOGACION);
        solicitud.setFechaLimite(LocalDate.now().plusDays(2));
        solicitud.setImpactoAcademico(8);

        // Estrategia 1: por tipo
        service.priorizarSolicitud(solicitud);
        // Salida: Solicitud #null priorizada con valor 2 usando: Priorización por tipo

        // Cambio de estrategia en tiempo de ejecución — mismo código, distinto resultado
        service.setEstrategia(new EstrategiaPorFechaLimite());
        service.priorizarSolicitud(solicitud);
        // Salida: Solicitud #null priorizada con valor 2 usando: Priorización por fecha límite

        // Estrategia compuesta
        service.setEstrategia(new EstrategiaCompuesta(List.of(
            new EstrategiaPorTipo(),
            new EstrategiaPorFechaLimite(),
            new EstrategiaPorImpacto()
        )));
        service.priorizarSolicitud(solicitud);
        // Salida: promedio de los tres
    }
}
```

### Strategy en Spring Boot con @Qualifier

En Spring Boot puedes registrar todas las estrategias como beans y seleccionar la correcta dinámicamente:

```java
// Todas las estrategias como beans de Spring
@Component("estrategiaPorTipo")
public class EstrategiaPorTipo implements EstrategiaPriorizacion { /* ... */ }

@Component("estrategiaPorFecha")
public class EstrategiaPorFechaLimite implements EstrategiaPriorizacion { /* ... */ }

@Component("estrategiaPorImpacto")
public class EstrategiaPorImpacto implements EstrategiaPriorizacion { /* ... */ }

// El servicio recibe TODAS las estrategias y las indexa
@Service
public class PriorizacionService {

    // Spring inyecta automáticamente todos los beans que implementen EstrategiaPriorizacion
    private final Map<String, EstrategiaPriorizacion> estrategias;

    public PriorizacionService(Map<String, EstrategiaPriorizacion> estrategias) {
        this.estrategias = estrategias;
    }

    public void priorizar(Solicitud solicitud, String nombreEstrategia) {
        EstrategiaPriorizacion estrategia = estrategias.get(nombreEstrategia);
        if (estrategia == null) {
            throw new IllegalArgumentException("Estrategia no encontrada: " + nombreEstrategia);
        }
        solicitud.setPrioridad(estrategia.calcularPrioridad(solicitud));
    }
}
```

---

## 8. Patrón Observer

### Problema que resuelve

En el Triage, cuando una solicitud cambia de estado (por ejemplo, de `REGISTRADA` a `CLASIFICADA`), varias cosas deben ocurrir:
1. Enviar un correo al solicitante
2. Notificar al responsable asignado
3. Registrar en el historial auditable (RF-06)
4. Actualizar métricas o estadísticas
5. Posiblemente integrar con un sistema externo

Si pones toda esa lógica directamente en el método `cambiarEstado()`, ese método tendrá 50 líneas mezclando responsabilidades. Además, si mañana quieres agregar "enviar SMS", tienes que modificar ese método.

El **patrón Observer** define una relación uno-a-muchos entre objetos: cuando el *sujeto* (la solicitud) cambia de estado, todos sus *observadores* son notificados automáticamente.

### Diagrama

```
┌──────────────────────┐    notifica     ┌─────────────────────┐
│      Solicitud       │────────────────▶│ <<interface>>       │
│      (Sujeto)        │                 │ ObservadorSolicitud │
├──────────────────────┤                 ├─────────────────────┤
│ -observadores: List  │                 │ +actualizar()       │
├──────────────────────┤                 └──────────┬──────────┘
│ +agregarObservador() │                            │
│ +cambiarEstado()     │                 implementan│
│ -notificar()         │       ┌─────────────────────┼──────────────────┐
└──────────────────────┘       ▼                     ▼                  ▼
                      ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
                      │ ObsHistorial │  │ ObsCorreo    │  │ ObsMetricas  │
                      └──────────────┘  └──────────────┘  └──────────────┘
```

### Implementación en el Triage

```java
// 1. El enum de estados del ciclo de vida (RF-04)
public enum EstadoSolicitud {
    REGISTRADA,
    CLASIFICADA,
    EN_ATENCION,
    ATENDIDA,
    CERRADA
}
```

```java
// 2. El evento que encapsula qué cambió — buena práctica no pasar el objeto directamente
public record EventoCambioEstado(
    Long solicitudId,
    EstadoSolicitud estadoAnterior,
    EstadoSolicitud estadoNuevo,
    String usuarioResponsable,
    LocalDateTime timestamp,
    String observacion
) {}
```

```java
// 3. La interfaz Observer
public interface ObservadorSolicitud {
    void onCambioEstado(EventoCambioEstado evento);
    String getNombre(); // Para logging
}
```

```java
// 4. Observer 1: registrar en historial (RF-06)
@Component
public class ObservadorHistorial implements ObservadorSolicitud {

    private final HistorialRepository historialRepository;

    public ObservadorHistorial(HistorialRepository historialRepository) {
        this.historialRepository = historialRepository;
    }

    @Override
    public void onCambioEstado(EventoCambioEstado evento) {
        var entrada = new HistorialSolicitud(
            evento.solicitudId(),
            "CAMBIO_ESTADO",
            String.format("Estado cambió de %s a %s. Obs: %s",
                evento.estadoAnterior(),
                evento.estadoNuevo(),
                evento.observacion()),
            evento.usuarioResponsable(),
            evento.timestamp()
        );
        historialRepository.save(entrada);
        System.out.printf("[HISTORIAL] Guardada entrada para solicitud #%d%n",
            evento.solicitudId());
    }

    @Override
    public String getNombre() { return "ObservadorHistorial"; }
}
```

```java
// 5. Observer 2: enviar notificación al solicitante
@Component
public class ObservadorNotificacionCorreo implements ObservadorSolicitud {

    private final NotificacionService notificacionService;
    private final UsuarioRepository usuarioRepository;

    public ObservadorNotificacionCorreo(NotificacionService notificacionService,
                                        UsuarioRepository usuarioRepository) {
        this.notificacionService = notificacionService;
        this.usuarioRepository = usuarioRepository;
    }

    @Override
    public void onCambioEstado(EventoCambioEstado evento) {
        // Solo notificamos en ciertos cambios de estado — no en todos
        if (evento.estadoNuevo() == EstadoSolicitud.CERRADA ||
            evento.estadoNuevo() == EstadoSolicitud.EN_ATENCION) {

            usuarioRepository.findBySolicitudId(evento.solicitudId())
                .ifPresent(usuario ->
                    notificacionService.enviar(
                        usuario.getEmail(),
                        String.format("Tu solicitud #%d ahora está en estado: %s",
                            evento.solicitudId(),
                            evento.estadoNuevo())
                    )
                );
        }
    }

    @Override
    public String getNombre() { return "ObservadorNotificacionCorreo"; }
}
```

```java
// 6. Observer 3: actualizar métricas
@Component
public class ObservadorMetricas implements ObservadorSolicitud {

    // En producción sería Micrometer + Prometheus, aquí simplificado
    private final Map<EstadoSolicitud, AtomicLong> contadores = new EnumMap<>(EstadoSolicitud.class);

    @Override
    public void onCambioEstado(EventoCambioEstado evento) {
        contadores
            .computeIfAbsent(evento.estadoNuevo(), k -> new AtomicLong(0))
            .incrementAndGet();

        System.out.printf("[MÉTRICAS] Solicitudes en estado %s: %d%n",
            evento.estadoNuevo(),
            contadores.get(evento.estadoNuevo()).get());
    }

    @Override
    public String getNombre() { return "ObservadorMetricas"; }

    public long getContador(EstadoSolicitud estado) {
        return contadores.getOrDefault(estado, new AtomicLong(0)).get();
    }
}
```

```java
// 7. El gestor de eventos — el "sujeto" que notifica a todos los observadores
@Service
public class GestorEventosSolicitud {

    // Spring inyecta automáticamente TODOS los beans que implementen ObservadorSolicitud
    private final List<ObservadorSolicitud> observadores;

    public GestorEventosSolicitud(List<ObservadorSolicitud> observadores) {
        this.observadores = List.copyOf(observadores);
        System.out.printf("[OBSERVER] %d observadores registrados: %s%n",
            observadores.size(),
            observadores.stream()
                .map(ObservadorSolicitud::getNombre)
                .collect(Collectors.joining(", ")));
    }

    /**
     * Dispara el evento a todos los observadores registrados.
     * Cada observador decide qué hacer con el evento.
     */
    public void publicar(EventoCambioEstado evento) {
        observadores.forEach(obs -> {
            try {
                obs.onCambioEstado(evento);
            } catch (Exception e) {
                // Un observador que falla NO debe bloquear a los demás
                System.err.printf("[OBSERVER ERROR] %s falló: %s%n",
                    obs.getNombre(), e.getMessage());
            }
        });
    }
}
```

```java
// 8. El servicio de negocio que usa el gestor de eventos
@Service
public class SolicitudService {

    private final SolicitudRepository repository;
    private final GestorEventosSolicitud gestorEventos;

    public SolicitudService(SolicitudRepository repository,
                            GestorEventosSolicitud gestorEventos) {
        this.repository = repository;
        this.gestorEventos = gestorEventos;
    }

    public void cambiarEstado(Long solicitudId,
                               EstadoSolicitud nuevoEstado,
                               String usuarioResponsable,
                               String observacion) {

        Solicitud solicitud = repository.findById(solicitudId)
            .orElseThrow(() -> new RuntimeException("Solicitud no encontrada: " + solicitudId));

        // Validar transición (RF-04)
        validarTransicion(solicitud.getEstado(), nuevoEstado);

        EstadoSolicitud estadoAnterior = solicitud.getEstado();
        solicitud.setEstado(nuevoEstado);
        repository.save(solicitud);

        // Notificar a todos los observadores — ellos se encargan de lo demás
        gestorEventos.publicar(new EventoCambioEstado(
            solicitudId,
            estadoAnterior,
            nuevoEstado,
            usuarioResponsable,
            LocalDateTime.now(),
            observacion
        ));
    }

    private void validarTransicion(EstadoSolicitud actual, EstadoSolicitud nuevo) {
        // Mapa de transiciones válidas (RF-04)
        Map<EstadoSolicitud, Set<EstadoSolicitud>> transicionesValidas = Map.of(
            EstadoSolicitud.REGISTRADA,   Set.of(EstadoSolicitud.CLASIFICADA),
            EstadoSolicitud.CLASIFICADA,  Set.of(EstadoSolicitud.EN_ATENCION),
            EstadoSolicitud.EN_ATENCION,  Set.of(EstadoSolicitud.ATENDIDA),
            EstadoSolicitud.ATENDIDA,     Set.of(EstadoSolicitud.CERRADA),
            EstadoSolicitud.CERRADA,      Set.of() // Estado final
        );

        if (!transicionesValidas.getOrDefault(actual, Set.of()).contains(nuevo)) {
            throw new IllegalStateException(
                String.format("Transición inválida: %s → %s", actual, nuevo)
            );
        }
    }
}
```

### Observer vs Spring Events

Spring Boot trae su propio mecanismo de eventos que es una implementación del Observer:

```java
// Alternativa con Spring Events — más integrado con el ecosistema Spring
@Component
public class ObservadorHistorialSpring {

    @EventListener  // ← Spring detecta y llama automáticamente
    public void onCambioEstado(EventoCambioEstado evento) {
        // mismo código que antes
    }
}

// Para publicar el evento:
@Service
public class SolicitudService {
    private final ApplicationEventPublisher eventPublisher;

    public void cambiarEstado(/* ... */) {
        // ...
        eventPublisher.publishEvent(new EventoCambioEstado(/* ... */));
    }
}
```

| | Observer manual | Spring Events |
|---|---|---|
| **Acoplamiento** | Bajo (interfaz) | Muy bajo (solo el tipo del evento) |
| **Configuración** | Manual | Automática con `@EventListener` |
| **Asincronía** | Manual | `@Async` + `@EventListener` |
| **Transaccional** | Manual | `@TransactionalEventListener` |
| **Recomendado en Spring Boot** | No | Sí |

---

## 9. Patrón Builder

### Problema que resuelve

Imagina que tienes que crear una `Solicitud` con 12 campos. Algunos son obligatorios, otros opcionales. Si crearas constructores para cada combinación posible, tendrías:

```java
// El "constructor telescópico" — pesadilla de mantenimiento
new Solicitud(tipo, descripcion, canal);
new Solicitud(tipo, descripcion, canal, prioridad);
new Solicitud(tipo, descripcion, canal, prioridad, fechaLimite);
new Solicitud(tipo, descripcion, canal, prioridad, fechaLimite, solicitante);
// ... y así hasta 15 constructores
```

¿Qué significa el quinto argumento que es `null`? ¿Es `prioridad` o `fechaLimite`? Imposible saberlo sin mirar el constructor.

El **patrón Builder** proporciona una forma fluida y legible de construir objetos complejos paso a paso.

### Implementación Manual (sin Lombok)

```java
// La entidad Solicitud — con Builder interno estático
public class Solicitud {

    // Campos obligatorios
    private final TipoSolicitud tipo;
    private final String descripcion;
    private final CanalOrigen canalOrigen;
    private final Long solicitanteId;

    // Campos opcionales
    private final Integer prioridad;
    private final LocalDate fechaLimite;
    private final String observacionInicial;
    private final EstadoSolicitud estado;

    // Constructor PRIVADO — solo el Builder puede crear instancias
    private Solicitud(Builder builder) {
        this.tipo               = builder.tipo;
        this.descripcion        = builder.descripcion;
        this.canalOrigen        = builder.canalOrigen;
        this.solicitanteId      = builder.solicitanteId;
        this.prioridad          = builder.prioridad;
        this.fechaLimite        = builder.fechaLimite;
        this.observacionInicial = builder.observacionInicial;
        this.estado             = builder.estado;
    }

    // Getters
    public TipoSolicitud getTipo()            { return tipo; }
    public String getDescripcion()            { return descripcion; }
    public CanalOrigen getCanalOrigen()       { return canalOrigen; }
    public Long getSolicitanteId()            { return solicitanteId; }
    public Optional<Integer> getPrioridad()   { return Optional.ofNullable(prioridad); }
    public Optional<LocalDate> getFechaLimite() { return Optional.ofNullable(fechaLimite); }
    public EstadoSolicitud getEstado()        { return estado; }

    @Override
    public String toString() {
        return "Solicitud{tipo=%s, canal=%s, estado=%s, prioridad=%s}"
            .formatted(tipo, canalOrigen, estado, prioridad);
    }

    // ─────────────────────────────────────────
    // El Builder — clase estática interna
    // ─────────────────────────────────────────
    public static class Builder {

        // Campos obligatorios — se pasan en el constructor del Builder
        private final TipoSolicitud tipo;
        private final String descripcion;
        private final CanalOrigen canalOrigen;
        private final Long solicitanteId;

        // Campos opcionales — con valores por defecto
        private Integer prioridad = null;
        private LocalDate fechaLimite = null;
        private String observacionInicial = "";
        private EstadoSolicitud estado = EstadoSolicitud.REGISTRADA;

        // Constructor del Builder con los campos obligatorios
        public Builder(TipoSolicitud tipo, String descripcion,
                       CanalOrigen canalOrigen, Long solicitanteId) {
            // Validaciones tempranas — mejor fallar aquí que en el servicio
            if (tipo == null)           throw new IllegalArgumentException("El tipo es obligatorio");
            if (descripcion == null || descripcion.isBlank())
                throw new IllegalArgumentException("La descripción es obligatoria");
            if (canalOrigen == null)    throw new IllegalArgumentException("El canal es obligatorio");
            if (solicitanteId == null)  throw new IllegalArgumentException("El solicitante es obligatorio");

            this.tipo          = tipo;
            this.descripcion   = descripcion;
            this.canalOrigen   = canalOrigen;
            this.solicitanteId = solicitanteId;
        }

        // Métodos fluidos — cada uno devuelve this para encadenamiento
        public Builder prioridad(int prioridad) {
            if (prioridad < 1 || prioridad > 5)
                throw new IllegalArgumentException("Prioridad debe estar entre 1 y 5");
            this.prioridad = prioridad;
            return this;
        }

        public Builder fechaLimite(LocalDate fechaLimite) {
            if (fechaLimite != null && fechaLimite.isBefore(LocalDate.now()))
                throw new IllegalArgumentException("La fecha límite no puede ser en el pasado");
            this.fechaLimite = fechaLimite;
            return this;
        }

        public Builder observacionInicial(String observacion) {
            this.observacionInicial = observacion;
            return this;
        }

        public Builder estado(EstadoSolicitud estado) {
            this.estado = estado;
            return this;
        }

        // El método terminal — construye el objeto final
        public Solicitud build() {
            return new Solicitud(this);
        }
    }
}
```

```java
// Uso del Builder — legible y sin ambigüedades
public class EjemploUsoBuilder {

    public static void main(String[] args) {

        // Solicitud mínima
        Solicitud solicitudSimple = new Solicitud.Builder(
                TipoSolicitud.CONSULTA_ACADEMICA,
                "Necesito información sobre el proceso de grado",
                CanalOrigen.CORREO,
                1001L
        ).build();

        System.out.println(solicitudSimple);

        // Solicitud completa — cada campo identificado por nombre de método
        Solicitud solicitudCompleta = new Solicitud.Builder(
                TipoSolicitud.HOMOLOGACION,
                "Solicito homologación de Programación I por Fundamentos de Software cursada en otra universidad",
                CanalOrigen.PRESENCIAL,
                1042L
        )
            .prioridad(2)
            .fechaLimite(LocalDate.now().plusDays(10))
            .observacionInicial("El estudiante adjunta certificado de notas")
            .build();

        System.out.println(solicitudCompleta);

        // ¿Qué pasa si faltas un campo obligatorio?
        try {
            new Solicitud.Builder(null, "Descripción", CanalOrigen.CORREO, 1L).build();
        } catch (IllegalArgumentException e) {
            System.out.println("Error capturado: " + e.getMessage());
            // Salida: Error capturado: El tipo es obligatorio
        }
    }
}
```

### Builder con Lombok (la forma más usada en proyectos reales)

En la práctica, la biblioteca **Lombok** genera todo el Builder automáticamente con una sola anotación:

```java
// Con Lombok — el Builder se genera automáticamente en tiempo de compilación
import lombok.Builder;
import lombok.Getter;
import lombok.NonNull;

@Getter
@Builder(toBuilder = true) // toBuilder permite clonar y modificar
public class SolicitudLombok {

    @NonNull
    private final TipoSolicitud tipo;

    @NonNull
    private final String descripcion;

    @NonNull
    private final CanalOrigen canalOrigen;

    @NonNull
    private final Long solicitanteId;

    @Builder.Default
    private final EstadoSolicitud estado = EstadoSolicitud.REGISTRADA;

    private final Integer prioridad;
    private final LocalDate fechaLimite;
    private final String observacionInicial;
}

// Uso — exactamente igual, el código lo genera Lombok
SolicitudLombok s = SolicitudLombok.builder()
    .tipo(TipoSolicitud.HOMOLOGACION)
    .descripcion("Homologación de Programación I")
    .canalOrigen(CanalOrigen.CORREO)
    .solicitanteId(1042L)
    .prioridad(2)
    .fechaLimite(LocalDate.now().plusDays(10))
    .build();

// Clonar con modificación — útil para crear solicitudes similares
SolicitudLombok copia = s.toBuilder()
    .descripcion("Descripción actualizada")
    .build();
```

**Cuándo usar la versión manual vs Lombok:**

| | Builder Manual | Lombok @Builder |
|---|---|---|
| **Validaciones** | Puedes agregar en el Builder | Necesitas agregar en `@NonNull` o método custom |
| **Código** | Verbose pero explícito | Mínimo, generado automáticamente |
| **Legibilidad** | Total control | Transparente (debes confiar en Lombok) |
| **Proyectos reales** | Para casos con mucha lógica de validación | La mayoría de los casos |

---

## 10. Patrón DTO (Data Transfer Object)

### El problema sin DTOs

Este es uno de los patrones más importantes para el desarrollo web. Sin DTOs, la API REST expondría directamente las entidades JPA, lo que crea problemas graves:

```java
// ❌ MAL — exponer la entidad directamente en el controlador
@GetMapping("/solicitudes/{id}")
public Solicitud getSolicitud(@PathVariable Long id) {
    return repository.findById(id).orElseThrow();
    // Problemas:
    // 1. Expones campos internos (versión, campos de auditoría)
    // 2. Posibles bucles infinitos por relaciones @OneToMany/@ManyToOne
    // 3. Lazy loading exceptions (LazyInitializationException)
    // 4. Expones información de seguridad (¡contraseñas!)
    // 5. La API queda acoplada al modelo de datos interno
}
```

### DTO con clase tradicional (antes de Java 16)

```java
// ✅ Antes — DTO como clase con getters y setters
public class SolicitudResponseDTO {

    private Long id;
    private String tipo;
    private String descripcion;
    private String estado;
    private String canalOrigen;
    private Integer prioridad;
    private LocalDateTime fechaRegistro;
    private String nombreSolicitante; // Campo aplanado — no la entidad Usuario completa

    // Constructor vacío (necesario para Jackson)
    public SolicitudResponseDTO() {}

    // Constructor completo
    public SolicitudResponseDTO(Long id, String tipo, String descripcion,
                                 String estado, String canalOrigen,
                                 Integer prioridad, LocalDateTime fechaRegistro,
                                 String nombreSolicitante) {
        this.id = id;
        this.tipo = tipo;
        // ...
    }

    // Getters y setters para TODOS los campos...
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    // ... y así hasta el final. Son 40+ líneas de código boilerplate.
}
```

### DTO con Record de Java 16+ (la forma moderna)

Un **Record** es un tipo especial de clase en Java que es inmutable y genera automáticamente constructor, getters, `equals()`, `hashCode()` y `toString()`.

```java
// ✅ Ahora — DTO como Record de Java 21
// Una línea reemplaza 40+ líneas de código boilerplate

// DTO de respuesta para una solicitud (lo que el backend devuelve al frontend)
public record SolicitudResponseDTO(
    Long id,
    String tipo,
    String descripcion,
    String estado,
    String canalOrigen,
    Integer prioridad,
    String prioridadJustificacion,
    LocalDateTime fechaRegistro,
    LocalDateTime ultimaActualizacion,
    String nombreSolicitante,    // Aplanado — no el objeto Usuario completo
    String emailSolicitante,
    String responsableAsignado   // Puede ser null
) {}

// DTO de creación — lo que el frontend envía al crear una solicitud (RF-01)
public record CrearSolicitudRequest(
    @NotBlank(message = "El tipo de solicitud es obligatorio")
    String tipo,

    @NotBlank(message = "La descripción es obligatoria")
    @Size(min = 10, max = 500, message = "La descripción debe tener entre 10 y 500 caracteres")
    String descripcion,

    @NotBlank(message = "El canal de origen es obligatorio")
    String canalOrigen,

    String observacionInicial // Opcional
) {}

// DTO para clasificar una solicitud (RF-02, RF-03)
public record ClasificarSolicitudRequest(
    @NotNull(message = "El tipo es obligatorio")
    TipoSolicitud tipo,

    @NotNull(message = "La prioridad es obligatoria")
    @Min(value = 1, message = "La prioridad mínima es 1")
    @Max(value = 5, message = "La prioridad máxima es 5")
    Integer prioridad,

    @NotBlank(message = "La justificación de prioridad es obligatoria")
    String justificacionPrioridad
) {}

// DTO para cambiar estado (RF-04)
public record CambiarEstadoRequest(
    @NotNull(message = "El nuevo estado es obligatorio")
    EstadoSolicitud nuevoEstado,

    @NotBlank(message = "La observación es obligatoria para el historial")
    String observacion
) {}

// DTO de respuesta paginada — lo que devuelve el endpoint de listado (RF-07)
public record SolicitudPageResponse(
    List<SolicitudResponseDTO> solicitudes,
    int paginaActual,
    int totalPaginas,
    long totalElementos,
    boolean esUltimaPagina
) {}
```

### Comparación: clase tradicional vs Record

```java
// ─────────────────────────────────────────
// ANTES: clase tradicional (Java 8-15)
// ─────────────────────────────────────────
public class SolicitudResponseDTOTradicional {
    private Long id;
    private String tipo;
    private String estado;

    public SolicitudResponseDTOTradicional() {}

    public SolicitudResponseDTOTradicional(Long id, String tipo, String estado) {
        this.id = id;
        this.tipo = tipo;
        this.estado = estado;
    }

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getTipo() { return tipo; }
    public void setTipo(String tipo) { this.tipo = tipo; }
    public String getEstado() { return estado; }
    public void setEstado(String estado) { this.estado = estado; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        SolicitudResponseDTOTradicional that = (SolicitudResponseDTOTradicional) o;
        return Objects.equals(id, that.id) &&
               Objects.equals(tipo, that.tipo) &&
               Objects.equals(estado, that.estado);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, tipo, estado);
    }

    @Override
    public String toString() {
        return "SolicitudResponseDTO{id=" + id + ", tipo='" + tipo + "', estado='" + estado + "'}";
    }
}
// Total: ~40 líneas

// ─────────────────────────────────────────
// AHORA: Record (Java 16+)
// ─────────────────────────────────────────
public record SolicitudResponseDTORecord(Long id, String tipo, String estado) {}
// Total: 1 línea — equals, hashCode, toString, getters: TODO incluido y es INMUTABLE
```

### Mapper: convertir Entidad ↔ DTO

El **mapper** es la clase responsable de convertir entre entidades JPA y DTOs. Dos formas:

#### Mapper manual

```java
// Mapper manual — más verboso pero sin magia
@Component
public class SolicitudMapper {

    /**
     * Convierte una Entidad Solicitud al DTO de respuesta.
     * Aplana las relaciones — en lugar de pasar toda la entidad Usuario,
     * solo pasamos el nombre y email.
     */
    public SolicitudResponseDTO toResponseDTO(Solicitud solicitud) {
        return new SolicitudResponseDTO(
            solicitud.getId(),
            solicitud.getTipo().name(),
            solicitud.getDescripcion(),
            solicitud.getEstado().name(),
            solicitud.getCanalOrigen().name(),
            solicitud.getPrioridad(),
            solicitud.getPrioridadJustificacion(),
            solicitud.getFechaRegistro(),
            solicitud.getUltimaActualizacion(),
            solicitud.getSolicitante() != null
                ? solicitud.getSolicitante().getNombreCompleto()
                : null,
            solicitud.getSolicitante() != null
                ? solicitud.getSolicitante().getEmail()
                : null,
            solicitud.getResponsable() != null
                ? solicitud.getResponsable().getNombreCompleto()
                : null
        );
    }

    /**
     * Convierte el DTO de creación a una entidad nueva.
     * El ID, estado y fechas los asigna el sistema, no el usuario.
     */
    public Solicitud toEntity(CrearSolicitudRequest dto, Long solicitanteId) {
        return new Solicitud.Builder(
                TipoSolicitud.valueOf(dto.tipo().toUpperCase()),
                dto.descripcion(),
                CanalOrigen.valueOf(dto.canalOrigen().toUpperCase()),
                solicitanteId
        )
            .observacionInicial(dto.observacionInicial())
            .build();
    }

    /**
     * Convierte una lista de entidades a lista de DTOs.
     */
    public List<SolicitudResponseDTO> toResponseDTOList(List<Solicitud> solicitudes) {
        return solicitudes.stream()
            .map(this::toResponseDTO)
            .collect(Collectors.toList());
    }
}
```

#### MapStruct — generación automática de mappers

**MapStruct** es una librería que genera el código del mapper en tiempo de compilación, basándose en anotaciones:

```java
// Con MapStruct — define la interfaz, el código se genera solo
@Mapper(componentModel = "spring") // ← Lo hace un @Component de Spring
public interface SolicitudMapper {

    // MapStruct intenta mapear por nombre de campo
    // Cuando los nombres no coinciden, usas @Mapping
    @Mapping(source = "solicitante.nombreCompleto", target = "nombreSolicitante")
    @Mapping(source = "solicitante.email", target = "emailSolicitante")
    @Mapping(source = "responsable.nombreCompleto", target = "responsableAsignado")
    @Mapping(source = "tipo", target = "tipo", qualifiedByName = "enumToString")
    SolicitudResponseDTO toResponseDTO(Solicitud solicitud);

    // Mapeo inverso — del DTO a la entidad
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "estado", constant = "REGISTRADA")
    @Mapping(target = "fechaRegistro", ignore = true)
    Solicitud toEntity(CrearSolicitudRequest dto);

    List<SolicitudResponseDTO> toResponseDTOList(List<Solicitud> solicitudes);

    @Named("enumToString")
    default String enumToString(Enum<?> e) {
        return e != null ? e.name() : null;
    }
}
```

**Tabla comparativa: Mapper manual vs MapStruct**

| Aspecto | Mapper manual | MapStruct |
|---------|---------------|-----------|
| **Configuración** | Ninguna | Agregar dependencia en pom.xml |
| **Código a escribir** | Mucho (todos los campos) | Solo las diferencias con `@Mapping` |
| **Rendimiento** | Igual | Igual (código generado en compile time) |
| **Depuración** | Fácil — ves exactamente qué hace | Debes ver el código generado en target/ |
| **Flexibilidad** | Total | Alta con `qualifiedByName` y `@AfterMapping` |
| **Proyecto Triage** | OK para empezar | Recomendado cuando tengas muchos DTOs |

---

## 11. Patrón Repository

### Problema que resuelve

El **patrón Repository** es uno de los más importantes en desarrollo empresarial. Proporciona una **abstracción sobre la capa de acceso a datos**, de forma que el servicio de negocio no sepa si los datos vienen de PostgreSQL, MongoDB, un caché en memoria o un archivo CSV.

Sin Repository:
```java
// ❌ Sin Repository — el servicio habla directamente con la BD
@Service
public class SolicitudService {

    // El servicio sabe de SQL, de JDBC, de conexiones...
    public Solicitud buscarPorId(Long id) {
        try (Connection conn = DriverManager.getConnection("jdbc:postgresql://...")) {
            PreparedStatement stmt = conn.prepareStatement("SELECT * FROM solicitudes WHERE id = ?");
            stmt.setLong(1, id);
            ResultSet rs = stmt.executeQuery();
            // ... mapeo manual del ResultSet...
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }
}
```

### El Repository como contrato (interfaz)

```java
// 1. La interfaz del Repository — el contrato que el servicio conoce
public interface SolicitudRepository {

    // CRUD básico
    Solicitud save(Solicitud solicitud);
    Optional<Solicitud> findById(Long id);
    List<Solicitud> findAll();
    void delete(Long id);
    boolean existsById(Long id);

    // Consultas específicas del dominio (RF-07)
    List<Solicitud> findByEstado(EstadoSolicitud estado);
    List<Solicitud> findByTipo(TipoSolicitud tipo);
    List<Solicitud> findByPrioridad(int prioridad);
    List<Solicitud> findBySolicitanteId(Long solicitanteId);
    List<Solicitud> findByResponsableId(Long responsableId);

    // Consulta con múltiples filtros (RF-07)
    List<Solicitud> findByFiltros(EstadoSolicitud estado,
                                   TipoSolicitud tipo,
                                   Integer prioridad,
                                   Long responsableId);

    // Consultas de conteo para reportes
    long countByEstado(EstadoSolicitud estado);
    long countByTipo(TipoSolicitud tipo);
}
```

```java
// 2. Implementación en memoria — útil para tests y demostraciones
public class SolicitudRepositoryInMemory implements SolicitudRepository {

    private final Map<Long, Solicitud> store = new ConcurrentHashMap<>();
    private final AtomicLong idGenerator = new AtomicLong(1);

    @Override
    public Solicitud save(Solicitud solicitud) {
        if (solicitud.getId() == null) {
            solicitud.setId(idGenerator.getAndIncrement());
        }
        store.put(solicitud.getId(), solicitud);
        return solicitud;
    }

    @Override
    public Optional<Solicitud> findById(Long id) {
        return Optional.ofNullable(store.get(id));
    }

    @Override
    public List<Solicitud> findAll() {
        return new ArrayList<>(store.values());
    }

    @Override
    public void delete(Long id) {
        store.remove(id);
    }

    @Override
    public boolean existsById(Long id) {
        return store.containsKey(id);
    }

    @Override
    public List<Solicitud> findByEstado(EstadoSolicitud estado) {
        return store.values().stream()
            .filter(s -> s.getEstado() == estado)
            .collect(Collectors.toList());
    }

    @Override
    public List<Solicitud> findByTipo(TipoSolicitud tipo) {
        return store.values().stream()
            .filter(s -> s.getTipo() == tipo)
            .collect(Collectors.toList());
    }

    @Override
    public List<Solicitud> findByPrioridad(int prioridad) {
        return store.values().stream()
            .filter(s -> s.getPrioridad() != null && s.getPrioridad() == prioridad)
            .collect(Collectors.toList());
    }

    @Override
    public List<Solicitud> findBySolicitanteId(Long solicitanteId) {
        return store.values().stream()
            .filter(s -> Objects.equals(s.getSolicitanteId(), solicitanteId))
            .collect(Collectors.toList());
    }

    @Override
    public List<Solicitud> findByResponsableId(Long responsableId) {
        return store.values().stream()
            .filter(s -> Objects.equals(s.getResponsableId(), responsableId))
            .collect(Collectors.toList());
    }

    @Override
    public List<Solicitud> findByFiltros(EstadoSolicitud estado,
                                          TipoSolicitud tipo,
                                          Integer prioridad,
                                          Long responsableId) {
        return store.values().stream()
            .filter(s -> estado == null || s.getEstado() == estado)
            .filter(s -> tipo == null || s.getTipo() == tipo)
            .filter(s -> prioridad == null || Objects.equals(s.getPrioridad(), prioridad))
            .filter(s -> responsableId == null || Objects.equals(s.getResponsableId(), responsableId))
            .collect(Collectors.toList());
    }

    @Override
    public long countByEstado(EstadoSolicitud estado) {
        return store.values().stream()
            .filter(s -> s.getEstado() == estado)
            .count();
    }

    @Override
    public long countByTipo(TipoSolicitud tipo) {
        return store.values().stream()
            .filter(s -> s.getTipo() == tipo)
            .count();
    }
}
```

```java
// 3. Implementación con Spring Data JPA — en el proyecto real con base de datos
// (La implementación se hace automáticamente por Spring Data JPA)
// Verás esto en detalle en la Guía 05
@Repository
public interface SolicitudJpaRepository extends JpaRepository<SolicitudEntity, Long> {

    // Spring Data genera la consulta SQL automáticamente a partir del nombre del método
    List<SolicitudEntity> findByEstado(EstadoSolicitud estado);
    List<SolicitudEntity> findByTipoAndEstado(TipoSolicitud tipo, EstadoSolicitud estado);
    List<SolicitudEntity> findByPrioridad(int prioridad);
    long countByEstado(EstadoSolicitud estado);

    // Para consultas complejas, JPQL personalizado
    @Query("""
        SELECT s FROM SolicitudEntity s
        WHERE (:estado IS NULL OR s.estado = :estado)
          AND (:tipo IS NULL OR s.tipo = :tipo)
          AND (:prioridad IS NULL OR s.prioridad = :prioridad)
          AND (:responsableId IS NULL OR s.responsable.id = :responsableId)
        ORDER BY s.prioridad ASC, s.fechaRegistro DESC
        """)
    List<SolicitudEntity> findByFiltros(
        @Param("estado") EstadoSolicitud estado,
        @Param("tipo") TipoSolicitud tipo,
        @Param("prioridad") Integer prioridad,
        @Param("responsableId") Long responsableId
    );
}
```

---

## 12. Patrón Service Layer

### Qué es y por qué existe

El **patrón Service Layer** (capa de servicio) define una capa de aplicación que coordina las respuestas de la aplicación en cada caso de uso del sistema. Actúa como un límite claro entre:

- La capa de presentación (controladores REST, que solo deberían recibir y devolver datos)
- La capa de dominio/negocio (donde viven las reglas de negocio)
- La capa de persistencia (repositories)

**La regla de oro:** Los controladores no deben tener lógica de negocio. Los repositories no deben tener lógica de negocio. La lógica de negocio vive **exclusivamente** en los servicios.

### Implementación del Service Layer para el Triage

```java
// El servicio principal del Triage — implementa todos los casos de uso
@Service
@Transactional  // ← Toda operación que modifique datos debe ser transaccional
public class SolicitudService {

    private final SolicitudRepository repository;
    private final HistorialRepository historialRepository;
    private final UsuarioRepository usuarioRepository;
    private final PriorizacionService priorizacionService;
    private final GestorEventosSolicitud gestorEventos;
    private final SolicitudMapper mapper;

    // Inyección por constructor — la forma recomendada (más testeable)
    public SolicitudService(SolicitudRepository repository,
                             HistorialRepository historialRepository,
                             UsuarioRepository usuarioRepository,
                             PriorizacionService priorizacionService,
                             GestorEventosSolicitud gestorEventos,
                             SolicitudMapper mapper) {
        this.repository         = repository;
        this.historialRepository = historialRepository;
        this.usuarioRepository   = usuarioRepository;
        this.priorizacionService = priorizacionService;
        this.gestorEventos       = gestorEventos;
        this.mapper              = mapper;
    }

    // ────────────────────────────────────────────────────
    // RF-01: Registro de solicitudes
    // ────────────────────────────────────────────────────
    public SolicitudResponseDTO registrar(CrearSolicitudRequest request, Long solicitanteId) {
        // 1. Validar que el solicitante existe
        var solicitante = usuarioRepository.findById(solicitanteId)
            .orElseThrow(() -> new SolicitanteNoEncontradoException(solicitanteId));

        // 2. Crear la entidad usando el Builder
        var solicitud = mapper.toEntity(request, solicitanteId);

        // 3. Persistir
        var solicitudGuardada = repository.save(solicitud);

        // 4. Registrar en historial
        registrarEnHistorial(solicitudGuardada.getId(),
            "CREACION",
            "Solicitud registrada por " + solicitante.getNombreCompleto(),
            solicitanteId);

        // 5. Publicar evento (dispara notificaciones, métricas, etc.)
        gestorEventos.publicar(new EventoCambioEstado(
            solicitudGuardada.getId(),
            null,
            EstadoSolicitud.REGISTRADA,
            solicitante.getNombreCompleto(),
            LocalDateTime.now(),
            "Solicitud creada"
        ));

        return mapper.toResponseDTO(solicitudGuardada);
    }

    // ────────────────────────────────────────────────────
    // RF-02 + RF-03: Clasificar y priorizar
    // ────────────────────────────────────────────────────
    public SolicitudResponseDTO clasificarYPriorizar(Long solicitudId,
                                                      ClasificarSolicitudRequest request,
                                                      Long funcionarioId) {
        var solicitud = obtenerSolicitudOFallar(solicitudId);

        // Solo se puede clasificar si está en estado REGISTRADA
        if (solicitud.getEstado() != EstadoSolicitud.REGISTRADA) {
            throw new TransicionEstadoInvalidaException(
                solicitudId, solicitud.getEstado(), EstadoSolicitud.CLASIFICADA
            );
        }

        solicitud.setTipo(request.tipo());
        solicitud.setPrioridad(request.prioridad());
        solicitud.setPrioridadJustificacion(request.justificacionPrioridad());
        solicitud.setEstado(EstadoSolicitud.CLASIFICADA);

        var guardada = repository.save(solicitud);

        registrarEnHistorial(solicitudId,
            "CLASIFICACION",
            String.format("Clasificada como %s con prioridad %d. Justificación: %s",
                request.tipo(), request.prioridad(), request.justificacionPrioridad()),
            funcionarioId);

        gestorEventos.publicar(new EventoCambioEstado(
            solicitudId, EstadoSolicitud.REGISTRADA, EstadoSolicitud.CLASIFICADA,
            obtenerNombreUsuario(funcionarioId), LocalDateTime.now(),
            "Clasificada y priorizada"
        ));

        return mapper.toResponseDTO(guardada);
    }

    // ────────────────────────────────────────────────────
    // RF-04: Cambiar estado
    // ────────────────────────────────────────────────────
    public SolicitudResponseDTO cambiarEstado(Long solicitudId,
                                              CambiarEstadoRequest request,
                                              Long usuarioId) {
        var solicitud = obtenerSolicitudOFallar(solicitudId);

        // Validar la transición
        validarTransicion(solicitud.getEstado(), request.nuevoEstado());

        // Una solicitud cerrada NO puede modificarse (RF-08)
        if (solicitud.getEstado() == EstadoSolicitud.CERRADA) {
            throw new SolicitudCerradaException(solicitudId);
        }

        EstadoSolicitud estadoAnterior = solicitud.getEstado();
        solicitud.setEstado(request.nuevoEstado());
        var guardada = repository.save(solicitud);

        registrarEnHistorial(solicitudId, "CAMBIO_ESTADO", request.observacion(), usuarioId);

        gestorEventos.publicar(new EventoCambioEstado(
            solicitudId, estadoAnterior, request.nuevoEstado(),
            obtenerNombreUsuario(usuarioId), LocalDateTime.now(),
            request.observacion()
        ));

        return mapper.toResponseDTO(guardada);
    }

    // ────────────────────────────────────────────────────
    // RF-05: Asignar responsable
    // ────────────────────────────────────────────────────
    public SolicitudResponseDTO asignarResponsable(Long solicitudId,
                                                    Long responsableId,
                                                    Long adminId) {
        var solicitud = obtenerSolicitudOFallar(solicitudId);
        var responsable = usuarioRepository.findById(responsableId)
            .orElseThrow(() -> new UsuarioNoEncontradoException(responsableId));

        // El responsable debe estar activo (RF-05)
        if (!responsable.isActivo()) {
            throw new UsuarioInactivoException(responsableId);
        }

        solicitud.setResponsableId(responsableId);
        var guardada = repository.save(solicitud);

        registrarEnHistorial(solicitudId, "ASIGNACION_RESPONSABLE",
            "Responsable asignado: " + responsable.getNombreCompleto(), adminId);

        return mapper.toResponseDTO(guardada);
    }

    // ────────────────────────────────────────────────────
    // RF-07: Consultas con filtros
    // ────────────────────────────────────────────────────
    @Transactional(readOnly = true) // ← Para solo-lectura: optimización de rendimiento
    public List<SolicitudResponseDTO> buscarConFiltros(EstadoSolicitud estado,
                                                       TipoSolicitud tipo,
                                                       Integer prioridad,
                                                       Long responsableId) {
        return repository.findByFiltros(estado, tipo, prioridad, responsableId)
            .stream()
            .map(mapper::toResponseDTO)
            .collect(Collectors.toList());
    }

    @Transactional(readOnly = true)
    public SolicitudResponseDTO buscarPorId(Long id) {
        return mapper.toResponseDTO(obtenerSolicitudOFallar(id));
    }

    // ────────────────────────────────────────────────────
    // RF-08: Cierre de solicitudes
    // ────────────────────────────────────────────────────
    public SolicitudResponseDTO cerrar(Long solicitudId, String observacion, Long funcionarioId) {
        var solicitud = obtenerSolicitudOFallar(solicitudId);

        // Solo se puede cerrar si está ATENDIDA (RF-08)
        if (solicitud.getEstado() != EstadoSolicitud.ATENDIDA) {
            throw new IllegalStateException(
                "Solo se puede cerrar una solicitud en estado ATENDIDA. Estado actual: "
                + solicitud.getEstado()
            );
        }

        solicitud.setEstado(EstadoSolicitud.CERRADA);
        solicitud.setObservacionCierre(observacion);
        var guardada = repository.save(solicitud);

        registrarEnHistorial(solicitudId, "CIERRE", observacion, funcionarioId);

        gestorEventos.publicar(new EventoCambioEstado(
            solicitudId, EstadoSolicitud.ATENDIDA, EstadoSolicitud.CERRADA,
            obtenerNombreUsuario(funcionarioId), LocalDateTime.now(), observacion
        ));

        return mapper.toResponseDTO(guardada);
    }

    // ────────────────────────────────────────────────────
    // Métodos privados de soporte
    // ────────────────────────────────────────────────────

    private Solicitud obtenerSolicitudOFallar(Long id) {
        return repository.findById(id)
            .orElseThrow(() -> new SolicitudNoEncontradaException(id));
    }

    private String obtenerNombreUsuario(Long usuarioId) {
        return usuarioRepository.findById(usuarioId)
            .map(u -> u.getNombreCompleto())
            .orElse("Usuario desconocido");
    }

    private void registrarEnHistorial(Long solicitudId, String accion,
                                       String observacion, Long usuarioId) {
        var entrada = new HistorialSolicitud(
            solicitudId, accion, observacion, usuarioId, LocalDateTime.now()
        );
        historialRepository.save(entrada);
    }

    private void validarTransicion(EstadoSolicitud actual, EstadoSolicitud nuevo) {
        var transicionesValidas = Map.of(
            EstadoSolicitud.REGISTRADA,  Set.of(EstadoSolicitud.CLASIFICADA),
            EstadoSolicitud.CLASIFICADA, Set.of(EstadoSolicitud.EN_ATENCION),
            EstadoSolicitud.EN_ATENCION, Set.of(EstadoSolicitud.ATENDIDA),
            EstadoSolicitud.ATENDIDA,    Set.of(EstadoSolicitud.CERRADA),
            EstadoSolicitud.CERRADA,     Set.<EstadoSolicitud>of()
        );

        if (!transicionesValidas.getOrDefault(actual, Set.of()).contains(nuevo)) {
            throw new TransicionEstadoInvalidaException(null, actual, nuevo);
        }
    }
}
```

---

## 13. MVC vs Arquitectura en Capas

### MVC (Model-View-Controller)

MVC es un patrón de presentación que divide la UI en tres roles:

```
┌─────────────────────────────────────────────────────┐
│                       MVC                          │
│                                                     │
│   ┌───────────┐    actualiza    ┌───────────────┐  │
│   │   MODEL   │◀───────────────│  CONTROLLER   │  │
│   │ (Datos y  │                 │  (Maneja      │  │
│   │  Lógica)  │                 │  peticiones)  │  │
│   └─────┬─────┘                 └───────┬───────┘  │
│         │ notifica                      │ actualiza │
│         ▼                               ▼           │
│   ┌───────────┐    interactúa    ┌─────────────┐   │
│   │   VIEW    │◀────────────────│   USUARIO   │   │
│   │ (Lo que   │                  └─────────────┘   │
│   │ se ve)    │                                     │
│   └───────────┘                                     │
└─────────────────────────────────────────────────────┘
```

### Arquitectura en Capas para API REST (lo que usaremos)

En aplicaciones REST + Angular, el patrón que realmente se usa es arquitectura en capas:

```
PETICIÓN HTTP (desde Angular)
        │
        ▼
┌───────────────────┐
│   CONTROLLER      │  ← Solo recibe/valida entrada, llama al servicio, devuelve respuesta
│   (@RestController│    No tiene lógica de negocio
│   @GetMapping...) │
└────────┬──────────┘
         │ llama
         ▼
┌───────────────────┐
│   SERVICE LAYER   │  ← Toda la lógica de negocio vive aquí
│   (@Service)      │    Coordina repositories, valida reglas, publica eventos
│                   │
└────────┬──────────┘
         │ usa
         ▼
┌───────────────────┐
│   REPOSITORY      │  ← Solo acceso a datos, sin lógica
│   (@Repository    │    Traduce objetos de dominio ↔ base de datos
│   JpaRepository)  │
└────────┬──────────┘
         │ accede
         ▼
┌───────────────────┐
│   BASE DE DATOS   │
│   (PostgreSQL)    │
└───────────────────┘
```

### Comparación completa

| Aspecto | MVC clásico | Capas (REST API) |
|---------|------------|-----------------|
| **Vista** | HTML generado en servidor (Thymeleaf) | Angular (cliente separado) |
| **Controlador** | Devuelve Vista + Modelo | Devuelve JSON (ResponseEntity) |
| **Modelo** | Objeto pasado a la vista | DTO serializado como JSON |
| **Cuándo usar** | Apps web tradicionales (sin SPA) | APIs REST consumidas por Angular/React |
| **Separación** | Frontend y backend en el mismo proyecto | Proyectos separados |
| **Nuestro Triage** | No | ✅ Sí |

---

## 14. Cómo se Articulan Todos los Patrones en el Proyecto de Triage

Esta es la vista panorámica de cómo todos los patrones de esta guía conviven en el mismo proyecto:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    PROYECTO DE TRIAGE — MAPA DE PATRONES                │
│                                                                         │
│  CAPA DE PRESENTACIÓN (Angular)                                         │
│  ┌──────────────────────────────────────────────────────┐               │
│  │   Formulario → SolicitudService(TS) → HttpClient     │               │
│  └──────────────────────────────────────────────────────┘               │
│                              │ HTTP REST                                │
│  CAPA CONTROLADOR (Spring Boot)                                         │
│  ┌──────────────────────────────────────────────────────┐               │
│  │   SolicitudController                                │               │
│  │   @RestController, @GetMapping, @PostMapping         │               │
│  │   DTO ← [PATRÓN DTO con Records]                    │               │
│  │   Mapper ← [SolicitudMapper con MapStruct]           │               │
│  └──────────────────────────────────────────────────────┘               │
│                              │ llama                                    │
│  CAPA DE SERVICIO (Lógica de Negocio)                                   │
│  ┌──────────────────────────────────────────────────────┐               │
│  │   SolicitudService ← [PATRÓN SERVICE LAYER]          │               │
│  │   PriorizacionService ← [PATRÓN STRATEGY]            │               │
│  │   NotificacionFactory ← [PATRÓN FACTORY]             │               │
│  │   GestorEventos ← [PATRÓN OBSERVER]                  │               │
│  │   SolicitudBuilder ← [PATRÓN BUILDER]                │               │
│  └──────────────────────────────────────────────────────┘               │
│                              │ usa                                      │
│  CAPA DE REPOSITORIO (Acceso a Datos)                                   │
│  ┌──────────────────────────────────────────────────────┐               │
│  │   SolicitudRepository ← [PATRÓN REPOSITORY]          │               │
│  │   HistorialRepository                                │               │
│  │   UsuarioRepository                                  │               │
│  └──────────────────────────────────────────────────────┘               │
│                              │ accede                                   │
│  BASE DE DATOS                                                          │
│  ┌──────────────────────────────────────────────────────┐               │
│  │   PostgreSQL                                         │               │
│  └──────────────────────────────────────────────────────┘               │
│                                                                         │
│  TRANSVERSAL                                                            │
│  ┌──────────────────────────────────────────────────────┐               │
│  │   Spring IoC Container ← [PATRÓN SINGLETON para beans│               │
│  │   Spring Security ← [DIP + Strategy para auth]       │               │
│  │   SOLID principles ← en todo el código               │               │
│  └──────────────────────────────────────────────────────┘               │
└─────────────────────────────────────────────────────────────────────────┘
```

### Estructura de carpetas del proyecto (ya aplicando los patrones)

```
triage-backend/
│
├── src/main/java/co/uniquindio/triage/
│   │
│   ├── TiageApplication.java                    ← @SpringBootApplication
│   │
│   ├── config/                                   ← Configuraciones de Spring
│   │   ├── SecurityConfig.java
│   │   ├── OpenApiConfig.java
│   │   └── CorsConfig.java
│   │
│   ├── domain/                                   ← Entidades y enums del dominio
│   │   ├── Solicitud.java                        ← Con Builder interno
│   │   ├── HistorialSolicitud.java
│   │   ├── Usuario.java
│   │   ├── enums/
│   │   │   ├── TipoSolicitud.java
│   │   │   ├── EstadoSolicitud.java
│   │   │   ├── CanalOrigen.java
│   │   │   └── RolUsuario.java
│   │   └── events/
│   │       └── EventoCambioEstado.java           ← Record
│   │
│   ├── dto/                                      ← Patrón DTO
│   │   ├── request/
│   │   │   ├── CrearSolicitudRequest.java        ← Record
│   │   │   ├── ClasificarSolicitudRequest.java   ← Record
│   │   │   ├── CambiarEstadoRequest.java         ← Record
│   │   │   └── AsignarResponsableRequest.java    ← Record
│   │   └── response/
│   │       ├── SolicitudResponseDTO.java         ← Record
│   │       ├── HistorialResponseDTO.java         ← Record
│   │       ├── UsuarioResponseDTO.java           ← Record
│   │       └── SolicitudPageResponse.java        ← Record
│   │
│   ├── mapper/                                   ← Patrón DTO (mappers)
│   │   ├── SolicitudMapper.java                  ← MapStruct o manual
│   │   └── HistorialMapper.java
│   │
│   ├── repository/                               ← Patrón Repository
│   │   ├── SolicitudRepository.java              ← Interfaz
│   │   ├── HistorialRepository.java              ← Interfaz
│   │   └── UsuarioRepository.java                ← Interfaz
│   │
│   ├── service/                                  ← Patrón Service Layer
│   │   ├── SolicitudService.java                 ← Lógica principal
│   │   ├── PriorizacionService.java              ← Usa Strategy
│   │   ├── HistorialService.java
│   │   ├── UsuarioService.java
│   │   └── strategy/                             ← Patrón Strategy
│   │       ├── EstrategiaPriorizacion.java       ← Interfaz
│   │       ├── EstrategiaPorTipo.java
│   │       ├── EstrategiaPorFechaLimite.java
│   │       ├── EstrategiaPorImpacto.java
│   │       └── EstrategiaCompuesta.java
│   │
│   ├── factory/                                  ← Patrón Factory
│   │   └── NotificacionFactory.java
│   │
│   ├── observer/                                 ← Patrón Observer
│   │   ├── ObservadorSolicitud.java              ← Interfaz
│   │   ├── ObservadorHistorial.java
│   │   ├── ObservadorNotificacionCorreo.java
│   │   ├── ObservadorMetricas.java
│   │   └── GestorEventosSolicitud.java
│   │
│   ├── controller/                               ← Capa REST
│   │   ├── SolicitudController.java
│   │   ├── HistorialController.java
│   │   ├── UsuarioController.java
│   │   └── AuthController.java
│   │
│   └── exception/                                ← Excepciones personalizadas
│       ├── SolicitudNoEncontradaException.java
│       ├── TransicionEstadoInvalidaException.java
│       ├── SolicitudCerradaException.java
│       ├── UsuarioNoEncontradoException.java
│       ├── UsuarioInactivoException.java
│       └── GlobalExceptionHandler.java           ← @ControllerAdvice
│
└── src/test/java/co/uniquindio/triage/
    ├── service/
    │   └── SolicitudServiceTest.java
    └── repository/
        └── SolicitudRepositoryTest.java
```

---

## 15. Ejercicios Prácticos

### Ejercicio 1 — SOLID en práctica

Crea una clase `ValidadorSolicitud` que verifique si una solicitud puede hacer una transición de estado. Aplica el principio **OCP**: añade una nueva regla de validación sin modificar el validador principal.

```
Pista: Define una interfaz ReglValidacion con un método boolean aplica(Solicitud, EstadoSolicitud).
Implementa al menos tres reglas: que no esté cerrada, que tenga responsable asignado antes de pasar
a EN_ATENCION, y que tenga observación de cierre antes de pasar a CERRADA.
```

### Ejercicio 2 — Patrón Strategy

Implementa una nueva estrategia de priorización llamada `EstrategiaPorUrgencia` que combine:
- Si el tipo es `RECURSO_APELACION` → prioridad 1 automáticamente
- Si la fecha límite es en menos de 24 horas → prioridad 1 automáticamente
- Si el impacto > 8 y la fecha límite < 7 días → prioridad 2
- Otros casos → usa `EstrategiaPorTipo`

Después, agrega esta estrategia a la `EstrategiaCompuesta` y demuestra que el `PriorizacionService` no cambió.

### Ejercicio 3 — Patrón Observer

Implementa un nuevo observador `ObservadorAlertaFuncionario` que:
- Se dispara SOLO cuando el estado cambia a `EN_ATENCION`
- Verifica que existe un responsable asignado
- Si existe, "envía" una alerta al responsable (puede ser un `System.out.println`)
- Si no existe, lanza una advertencia en el log

Verifica que al agregar este observador a la lista, no tuviste que modificar `GestorEventosSolicitud` ni `SolicitudService`.

### Ejercicio 4 — Builder con validaciones

Extiende el `Builder` de `Solicitud` para que valide:
- La descripción debe tener al menos 10 caracteres
- La fecha límite, si se provee, no puede ser en el pasado
- La prioridad, si se provee, debe estar entre 1 y 5
- Si el tipo es `RECURSO_APELACION`, la prioridad debe ser 1 o 2 obligatoriamente

Escribe pruebas unitarias que verifiquen cada validación.

### Ejercicio 5 — DTO con Records

Crea los Records DTO para:
- `AsignarResponsableRequest` con el id del responsable y una observación opcional
- `HistorialEntradaResponseDTO` con: id, solicitudId, accion, descripcion, usuarioResponsable, timestamp
- `ResumenEstadisticasDTO` con: totalSolicitudes, porEstado (Map), porTipo (Map), promedioTiempoAtencion

---

## 16. Errores Comunes y Troubleshooting

### Error 1: NullPointerException en Optional

```java
// ❌ MAL — puede lanzar NoSuchElementException si no hay valor
Solicitud s = repository.findById(id).get();

// ❌ TAMBIÉN MAL — no es idiomático
if (repository.findById(id).isPresent()) {
    Solicitud s = repository.findById(id).get(); // Llamas findById DOS veces
}

// ✅ CORRECTO
Solicitud s = repository.findById(id)
    .orElseThrow(() -> new SolicitudNoEncontradaException(id));

// ✅ TAMBIÉN CORRECTO — cuando puede no existir y está bien
Optional<Solicitud> opt = repository.findById(id);
opt.ifPresent(solicitud -> System.out.println("Encontrada: " + solicitud));
```

### Error 2: Singleton con estado mutable

```java
// ❌ PELIGROSO — bean Singleton con estado mutable
@Service
public class SolicitudService {
    // Esta variable es compartida entre TODOS los hilos
    // Si dos peticiones llegan al mismo tiempo, race condition
    private Solicitud ultimaSolicitudProcesada; // ← MAL en un Singleton

    public void procesar(Solicitud s) {
        this.ultimaSolicitudProcesada = s; // ← Modificar estado en Singleton = problemas
    }
}

// ✅ CORRECTO — los beans Singleton deben ser stateless
@Service
public class SolicitudService {
    // Todo el estado se pasa por parámetro, no se guarda en la clase
    public SolicitudResponseDTO procesar(Solicitud s) {
        // Trabaja con s como variable local, no como campo de instancia
        return mapper.toResponseDTO(repository.save(s));
    }
}
```

### Error 3: Observer que falla bloquea a los demás

```java
// ❌ MAL — si ObservadorCorreo falla, ObservadorHistorial nunca se ejecuta
observadores.forEach(obs -> obs.onCambioEstado(evento)); // Sin manejo de errores

// ✅ CORRECTO — cada observador es independiente
observadores.forEach(obs -> {
    try {
        obs.onCambioEstado(evento);
    } catch (Exception e) {
        log.error("Observador {} falló, continuando con los demás: {}",
            obs.getNombre(), e.getMessage());
        // No relanzamos — los demás observadores siguen ejecutándose
    }
});
```

### Error 4: Exponer entidades JPA directamente desde el controlador

```java
// ❌ MAL — expone la entidad directamente
@GetMapping("/solicitudes/{id}")
public Solicitud getSolicitud(@PathVariable Long id) {
    return repository.findById(id).orElseThrow(); // LazyInitializationException probable
}

// ✅ CORRECTO — pasa siempre por DTO
@GetMapping("/solicitudes/{id}")
public ResponseEntity<SolicitudResponseDTO> getSolicitud(@PathVariable Long id) {
    return ResponseEntity.ok(service.buscarPorId(id));
}
```

### Error 5: Lógica de negocio en el controlador

```java
// ❌ MAL — el controlador no debería tener lógica de negocio
@PostMapping("/solicitudes/{id}/clasificar")
public ResponseEntity<SolicitudResponseDTO> clasificar(@PathVariable Long id,
                                                        @RequestBody ClasificarRequest req) {
    Solicitud s = repository.findById(id).orElseThrow();
    if (s.getEstado() != EstadoSolicitud.REGISTRADA) { // ← Lógica aquí es incorrecto
        return ResponseEntity.badRequest().build();
    }
    s.setTipo(req.tipo());
    // ...
}

// ✅ CORRECTO — el controlador delega toda la lógica al servicio
@PostMapping("/solicitudes/{id}/clasificar")
public ResponseEntity<SolicitudResponseDTO> clasificar(@PathVariable Long id,
                                                        @Valid @RequestBody ClasificarRequest req,
                                                        @AuthenticationPrincipal UserDetails user) {
    // Solo orquesta: valida entrada, llama al servicio, devuelve respuesta
    return ResponseEntity.ok(service.clasificarYPriorizar(id, req, user.getId()));
}
```

### Error 6: Violar LSP con métodos que lanzan `UnsupportedOperationException`

```java
// ❌ MAL — viola LSP
public class SolicitudRepositoryReadOnly implements SolicitudRepository {
    @Override
    public Solicitud save(Solicitud s) {
        throw new UnsupportedOperationException("Este repositorio es de solo lectura");
    }
    // Quien use SolicitudRepository no espera que save() falle así
}

// ✅ CORRECTO — crea una interfaz separada para solo lectura
public interface SolicitudRepositoryReadOnly {
    Optional<Solicitud> findById(Long id);
    List<Solicitud> findAll();
    List<Solicitud> findByEstado(EstadoSolicitud estado);
}

// Ahora la implementación de solo lectura no tiene que implementar save()
public class SolicitudRepositoryCache implements SolicitudRepositoryReadOnly {
    @Override
    public Optional<Solicitud> findById(Long id) { /* desde caché */ return Optional.empty(); }
    // ...
}
```

---

## 17. Resumen y Cheat Sheet

### Tabla de Patrones — cuándo y dónde usarlo en el Triage

| Patrón | Problema que resuelve | Dónde en el Triage | Clase/Interface principal |
|--------|----------------------|-------------------|--------------------------|
| **Singleton** | Una instancia única | Todos los `@Service`, `@Repository`, `@Component` | Gestionado por Spring IoC |
| **Factory** | Centralizar creación de objetos | Crear `Notificacion` según canal | `NotificacionFactory` |
| **Abstract Factory** | Familias de objetos | Temas de notificaciones (formal/informal) | `NotificacionAbstractFactory` |
| **Strategy** | Intercambiar algoritmos | Reglas de priorización (RF-03) | `EstrategiaPriorizacion` |
| **Observer** | Notificar cambios a múltiples receptores | Cambios de estado de solicitud | `ObservadorSolicitud` |
| **Builder** | Construir objetos complejos legiblemente | Crear `Solicitud` con muchos campos | `Solicitud.Builder` |
| **DTO** | Transferir datos entre capas | Request/Response de la API REST | Records Java 21 |
| **Repository** | Abstraer acceso a datos | Persistencia de todas las entidades | `SolicitudRepository` |
| **Service Layer** | Encapsular lógica de negocio | Todos los casos de uso del Triage | `SolicitudService` |

### Cheat Sheet de SOLID en una tabla

| Principio | Señal de que lo estás violando | Cómo corregirlo |
|-----------|-------------------------------|-----------------|
| **SRP** | La clase cambia por múltiples razones | Extrae responsabilidades a clases separadas |
| **OCP** | Agregar función = editar clase existente | Usa interfaces + nuevas implementaciones |
| **LSP** | La subclase lanza `UnsupportedOperationException` | Revisa la jerarquía de herencia |
| **ISP** | La clase implementa métodos que deja vacíos | Divide la interfaz grande en interfaces pequeñas |
| **DIP** | `new ConcreteClass()` en lógica de negocio | Depende de interfaces, usa inyección de dependencias |

### Decisiones de diseño del Triage

```
¿Necesito crear objetos de distintos tipos en tiempo de ejecución?
└─▶ FACTORY

¿Necesito construir un objeto con muchos campos opcionales?
└─▶ BUILDER

¿Necesito intercambiar algoritmos sin cambiar el código que los usa?
└─▶ STRATEGY

¿Necesito notificar a N objetos cuando algo cambia?
└─▶ OBSERVER

¿Necesito transferir datos entre capas sin exponer la entidad?
└─▶ DTO (Record de Java 21)

¿Necesito abstraer el acceso a datos?
└─▶ REPOSITORY

¿Necesito encapsular la lógica de negocio de un caso de uso?
└─▶ SERVICE LAYER

¿Necesito garantizar una única instancia?
└─▶ SINGLETON (en Spring: solo anota con @Service/@Component)
```

---

## 18. Referencias y Recursos Adicionales

### Libros fundamentales

- **"Design Patterns: Elements of Reusable Object-Oriented Software"** — Gamma, Helm, Johnson, Vlissides (Gang of Four), 1994. El libro original. Denso pero esencial.
- **"Head First Design Patterns"** — Freeman, Robson. Más didáctico y visual. Excelente para empezar.
- **"Clean Code"** — Robert C. Martin. SOLID y buenas prácticas en Java.
- **"Effective Java"** — Joshua Bloch. Java idiomático moderno, incluyendo el patrón Builder con Enum Singleton.
- **"Patterns of Enterprise Application Architecture"** — Martin Fowler. Service Layer, Repository y más.
- **"Introducción a los patrones de diseño: Un enfoque práctico"** — Oscar J. Blancarte (en español, mencionado en el sílabo).

### Recursos online

- [Refactoring.Guru](https://refactoring.guru/es/design-patterns) — La mejor referencia visual de patrones, en español.
- [Spring Framework Documentation](https://docs.spring.io/spring-framework/reference/) — Documentación oficial de Spring.
- [Java 21 Records](https://openjdk.org/jeps/395) — JEP oficial de Records.
- [Baeldung — Spring Design Patterns](https://www.baeldung.com/spring-framework-design-patterns) — Artículos prácticos.

### Herramientas que usarás

- [Spring Initializr](https://start.spring.io) — Generador de proyectos Spring Boot.
- [MapStruct](https://mapstruct.org) — Generación automática de mappers DTO ↔ Entidad.
- [Lombok](https://projectlombok.org) — @Builder, @Getter, @Setter y más.

---

> **Autor:** José Alfredo Ramírez Espinosa  
> **Materia:** Programación Avanzada  
> **Programa:** Ingeniería de Sistemas y Computación  
> **Universidad del Quindío** | Armenia, Colombia  
> **Año:** 2026  
> *Material de uso académico. Todos los derechos reservados al autor.*
