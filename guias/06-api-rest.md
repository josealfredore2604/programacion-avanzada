# Guía 06 — API REST con Spring Boot: Diseño, Implementación y Buenas Prácticas

> **Materia:** Programación Avanzada  
> **Programa:** Ingeniería de Sistemas y Computación — Universidad del Quindío  
> **Núcleo temático:** 3 — Construcción de Servicios de Negocio  
> **Guía:** 06 de 12  
> **Versiones:** Java 21 · Spring Boot 3.4+ · SpringDoc OpenAPI 2.x

---

## Tabla de Contenidos

1. [Prerrequisitos](#1-prerrequisitos)
2. [Objetivos de Aprendizaje](#2-objetivos-de-aprendizaje)
3. [¿Qué es REST?](#3-qué-es-rest)
   - 3.1 Los 6 principios de REST
   - 3.2 Recursos, verbos HTTP y códigos de estado
   - 3.3 ¿Qué es RESTful y qué NO lo es?
4. [Diseño de API RESTful: Convenciones y Buenas Prácticas](#4-diseño-de-api-restful-convenciones-y-buenas-prácticas)
   - 4.1 Naming de URLs
   - 4.2 Versionado de la API
   - 4.3 Diseño de los endpoints del Triage
5. [`@RestController` vs `@Controller`](#5-restcontroller-vs-controller)
6. [Anotaciones de Mapeo HTTP](#6-anotaciones-de-mapeo-http)
   - 6.1 `@GetMapping`
   - 6.2 `@PostMapping`
   - 6.3 `@PutMapping` y `@PatchMapping`
   - 6.4 `@DeleteMapping`
7. [Parámetros de Entrada](#7-parámetros-de-entrada)
   - 7.1 `@PathVariable`
   - 7.2 `@RequestParam`
   - 7.3 `@RequestBody`
   - 7.4 `@RequestHeader`
8. [`ResponseEntity<T>`: Control Total de la Respuesta](#8-responseentityt-control-total-de-la-respuesta)
9. [DTOs con Records de Java 21](#9-dtos-con-records-de-java-21)
   - 9.1 Por qué NUNCA exponer entidades JPA
   - 9.2 Request DTOs vs Response DTOs
   - 9.3 Mapeo manual
   - 9.4 Mapeo con MapStruct
   - 9.5 Antes vs Ahora: clases vs Records
10. [Validación de Entrada con `@Valid`](#10-validación-de-entrada-con-valid)
    - 10.1 Anotaciones de Bean Validation
    - 10.2 Manejo de errores de validación
11. [Manejo Global de Errores](#11-manejo-global-de-errores)
    - 11.1 `@ControllerAdvice` y `@ExceptionHandler`
    - 11.2 Excepciones personalizadas
    - 11.3 Estructura RFC 7807 — Problem Details
12. [Documentación con OpenAPI 3 / Swagger](#12-documentación-con-openapi-3--swagger)
    - 12.1 Configurar SpringDoc
    - 12.2 Anotaciones de documentación
    - 12.3 Acceder a Swagger UI
13. [CORS: Qué Es, Por Qué Ocurre y Cómo Solucionarlo](#13-cors-qué-es-por-qué-ocurre-y-cómo-solucionarlo)
14. [Paginación en la API](#14-paginación-en-la-api)
15. [Implementación Completa: Endpoints del Triage](#15-implementación-completa-endpoints-del-triage)
    - 15.1 Estructura de carpetas
    - 15.2 Entidades y enums de referencia
    - 15.3 DTOs completos
    - 15.4 Repositorios
    - 15.5 Capa de servicio
    - 15.6 Controladores completos
16. [Antes vs Ahora: `RestTemplate` vs `WebClient`](#16-antes-vs-ahora-resttemplate-vs-webclient)
17. [Probar la API con Swagger UI y Postman](#17-probar-la-api-con-swagger-ui-y-postman)
18. [Ejercicios Prácticos](#18-ejercicios-prácticos)
19. [Errores Comunes y Troubleshooting](#19-errores-comunes-y-troubleshooting)
20. [Resumen y Cheat Sheet](#20-resumen-y-cheat-sheet)
21. [Referencias y Recursos Adicionales](#21-referencias-y-recursos-adicionales)

---

## 1. Prerrequisitos

Antes de comenzar esta guía asegúrate de haber revisado o tener conocimiento de:

- **Guía 04** — Spring Boot desde Cero: IoC, DI, anotaciones fundamentales, ciclo de vida de beans
- **Guía 05** — JPA, Hibernate y Persistencia: entidades, relaciones, Spring Data JPA, Flyway
- Java 21 con Records, Optional y Streams (Guía 03)
- Patrones DTO y Repository (Guía 02)
- Conceptos básicos de HTTP: verbos, códigos de estado, headers

Herramientas que necesitas instaladas:
- JDK 21+
- Maven 3.9+
- PostgreSQL 16+ corriendo localmente
- IntelliJ IDEA o VS Code con extensión Spring Boot
- Postman o Bruno (cliente HTTP para pruebas)

---

## 2. Objetivos de Aprendizaje

Al finalizar esta guía serás capaz de:

1. Explicar los principios REST y distinguir una API RESTful de una que no lo es
2. Diseñar URLs limpias y versionadas siguiendo convenciones profesionales
3. Implementar controladores REST completos con todos los verbos HTTP
4. Usar `ResponseEntity<T>` para controlar con precisión el status code y el cuerpo de cada respuesta
5. Crear DTOs con Records de Java 21 y mapearlos desde/hacia entidades JPA
6. Validar el cuerpo de las peticiones con Bean Validation y manejar los errores de forma centralizada
7. Implementar un manejador global de errores con la estructura RFC 7807 Problem Details
8. Documentar la API automáticamente con OpenAPI 3 y Swagger UI
9. Configurar CORS correctamente para que Angular pueda consumir el backend
10. Implementar paginación del lado del servidor
11. Construir todos los endpoints del Sistema de Triage con sus DTOs, validaciones y documentación

---

## 3. ¿Qué es REST?

### Analogía inicial

Imagina que vas a una biblioteca. Tú (el cliente) llegas y le pides al bibliotecario (el servidor) un libro específico. El bibliotecario no necesita saber para qué lo quieres, ni recordar quién eres de visitas anteriores — cada vez que llegas, te identifica por tu carné. Si el libro existe, te lo da con un recibo (respuesta exitosa); si no existe, te avisa (error 404); si no tienes permiso para llevarlo (error 403). Eso es, en esencia, cómo funciona una API REST.

**REST** (Representational State Transfer) es un estilo arquitectónico para diseñar APIs de comunicación entre sistemas sobre HTTP. No es un protocolo ni un estándar, sino un conjunto de restricciones y principios que, cuando se siguen, producen APIs consistentes, escalables y fáciles de entender.

### 3.1 Los 6 Principios de REST

| # | Principio | Qué significa en la práctica |
|---|-----------|------------------------------|
| 1 | **Cliente-Servidor** | El frontend (Angular) y el backend (Spring Boot) son independientes. Cada uno puede evolucionar sin romper al otro, siempre que respeten el contrato de la API. |
| 2 | **Sin Estado (Stateless)** | Cada petición HTTP debe contener **toda** la información necesaria para procesarla. El servidor no recuerda peticiones anteriores. El estado de sesión, si se necesita, viaja en el cliente (JWT). |
| 3 | **Cacheable** | Las respuestas deben indicar si pueden almacenarse en caché. Esto mejora el rendimiento. En Spring Boot se controla con headers `Cache-Control`. |
| 4 | **Interfaz Uniforme** | Todos los recursos se acceden de la misma manera: con URLs consistentes y verbos HTTP estándar. Esto hace la API predecible. |
| 5 | **Sistema en Capas** | El cliente no sabe si se está comunicando directamente con el servidor final o con un intermediario (proxy, balanceador de carga, gateway de API). |
| 6 | **Código bajo Demanda** *(opcional)* | El servidor puede enviar código ejecutable al cliente (por ejemplo, JavaScript). Pocas APIs usan esto. |

El principio más importante para nuestro proyecto es el **#2 — Sin Estado**. Esto es lo que nos lleva a usar JWT en la Guía 07 en lugar de sesiones guardadas en el servidor.

### 3.2 Recursos, Verbos HTTP y Códigos de Estado

En REST, todo es un **recurso** — una "cosa" del dominio. Una solicitud académica es un recurso. Un usuario es un recurso. El historial de una solicitud es un recurso.

Los recursos se identifican con **URLs** y se manipulan con **verbos HTTP**:

| Verbo HTTP | Semántica | Idempotente | Body en request | Ejemplo en el Triage |
|------------|-----------|-------------|-----------------|----------------------|
| `GET` | Obtener un recurso o colección | ✅ Sí | ❌ No | `GET /api/v1/solicitudes` — listar solicitudes |
| `POST` | Crear un nuevo recurso | ❌ No | ✅ Sí | `POST /api/v1/solicitudes` — crear solicitud |
| `PUT` | Reemplazar un recurso completo | ✅ Sí | ✅ Sí | `PUT /api/v1/solicitudes/5` — reemplazar solicitud |
| `PATCH` | Modificar parcialmente un recurso | ❌ No* | ✅ Sí | `PATCH /api/v1/solicitudes/5/estado` — cambiar estado |
| `DELETE` | Eliminar un recurso | ✅ Sí | ❌ Rara vez | `DELETE /api/v1/solicitudes/5` — eliminar (admin) |

> **¿Qué es idempotente?** Significa que hacer la misma operación N veces produce el mismo resultado que hacerla una sola vez. Si haces `GET /solicitudes/5` diez veces, siempre recibes la misma solicitud. Si haces `POST /solicitudes` diez veces, creas diez solicitudes distintas.

**Códigos de estado HTTP más importantes:**

```
2xx — Éxito
  200 OK          → Petición exitosa (GET, PUT, PATCH)
  201 Created     → Recurso creado exitosamente (POST)
  204 No Content  → Éxito sin cuerpo en la respuesta (DELETE)

3xx — Redirección
  304 Not Modified → El recurso no cambió (caché válido)

4xx — Error del cliente
  400 Bad Request  → Datos de entrada inválidos
  401 Unauthorized → No estás autenticado (falta el JWT)
  403 Forbidden    → Estás autenticado pero no tienes permiso
  404 Not Found    → El recurso no existe
  409 Conflict     → Conflicto con el estado actual (ej: ya existe)
  422 Unprocessable Entity → Validación semántica falló

5xx — Error del servidor
  500 Internal Server Error → Error inesperado del servidor
  503 Service Unavailable   → Servidor no disponible temporalmente
```

### 3.3 ¿Qué es RESTful y qué NO lo es?

Veamos ejemplos concretos con el Triage:

```
✅ RESTful — siguiendo las convenciones correctamente:
  GET    /api/v1/solicitudes              → listar solicitudes
  GET    /api/v1/solicitudes/5            → ver solicitud #5
  POST   /api/v1/solicitudes              → crear solicitud
  PATCH  /api/v1/solicitudes/5/estado     → cambiar estado de solicitud #5
  GET    /api/v1/solicitudes/5/historial  → ver historial de solicitud #5
  POST   /api/v1/solicitudes/5/asignar    → asignar responsable (acción)

❌ NO RESTful — errores comunes de principiantes:
  POST   /api/v1/crearSolicitud           → el verbo ya está en el método HTTP
  GET    /api/v1/obtenerSolicitudPorId/5  → "obtener" es redundante
  POST   /api/v1/eliminarSolicitud        → usar DELETE para eliminar
  GET    /api/v1/solicitud                → usar plural para colecciones
  POST   /api/v1/solicitudes/cambiarEstadoAEnAtencion → demasiado específico en la URL
```

---

## 4. Diseño de API RESTful: Convenciones y Buenas Prácticas

### 4.1 Naming de URLs

**Reglas de oro para nombrar tus endpoints:**

```
1. Sustantivos en plural, nunca verbos
   ✅ /solicitudes    ❌ /crearSolicitud

2. Letras minúsculas, separadas por guiones
   ✅ /tipos-solicitud    ❌ /TiposSolicitud    ❌ /tiposSolicitud

3. Jerarquía clara con sub-recursos
   ✅ /solicitudes/5/historial    (historial pertenece a solicitud)
   ✅ /usuarios/3/solicitudes     (solicitudes de un usuario)

4. Verbos solo para acciones que no caben en CRUD
   ✅ POST /solicitudes/5/clasificar
   ✅ POST /solicitudes/5/asignar
   ✅ POST /auth/login
   ✅ POST /auth/logout

5. Filtros como query params, no en la URL
   ✅ GET /solicitudes?estado=EN_ATENCION&prioridad=ALTA
   ❌ GET /solicitudes/estado/EN_ATENCION/prioridad/ALTA
```

### 4.2 Versionado de la API

¿Por qué versionar? Porque tu API va a cambiar. Si un día decides modificar la estructura de una respuesta, todos los clientes que ya la consumen (Angular, apps móviles, otros sistemas) se rompen. El versionado permite evolucionar sin romper lo que ya existe.

```
Estrategia 1: Versión en la URL (la más común y simple)
  /api/v1/solicitudes
  /api/v2/solicitudes

Estrategia 2: Versión en el header
  GET /solicitudes
  Accept: application/vnd.triage.v1+json

Estrategia 3: Versión como query param
  GET /solicitudes?version=1
```

En este proyecto usaremos **versión en la URL** con `/api/v1/` por su simplicidad y claridad.

### 4.3 Diseño de los Endpoints del Triage

Aquí está el contrato completo de la API que vamos a construir:

```
=== SOLICITUDES ===
GET    /api/v1/solicitudes                          → Listar con filtros y paginación (RF-07)
GET    /api/v1/solicitudes/{id}                     → Ver detalle de una solicitud
POST   /api/v1/solicitudes                          → Registrar nueva solicitud (RF-01)
PATCH  /api/v1/solicitudes/{id}/clasificar          → Clasificar y priorizar (RF-02, RF-03)
PATCH  /api/v1/solicitudes/{id}/asignar             → Asignar responsable (RF-05)
PATCH  /api/v1/solicitudes/{id}/estado              → Cambiar estado (RF-04)
POST   /api/v1/solicitudes/{id}/cerrar              → Cerrar solicitud (RF-08)
GET    /api/v1/solicitudes/{id}/historial           → Ver historial auditable (RF-06)

=== USUARIOS ===
GET    /api/v1/usuarios                             → Listar usuarios (ADMIN)
GET    /api/v1/usuarios/{id}                        → Ver perfil de usuario
POST   /api/v1/usuarios                             → Crear usuario (ADMIN)

=== AUTENTICACIÓN (RF-13) ===
POST   /api/v1/auth/login                           → Iniciar sesión
POST   /api/v1/auth/registro                        → Registrar nuevo estudiante

=== CATÁLOGOS ===
GET    /api/v1/tipos-solicitud                      → Listar tipos de solicitud
GET    /api/v1/canales-origen                       → Listar canales de origen
```

---

## 5. `@RestController` vs `@Controller`

Antes de escribir código, debes entender cuándo usar cada uno:

```java
// @Controller — para aplicaciones web MVC tradicionales
// Devuelve el NOMBRE de una vista (HTML, Thymeleaf, etc.)
@Controller
public class VistaSolicitudController {
    
    @GetMapping("/solicitudes")
    public String listar(Model model) {
        model.addAttribute("solicitudes", servicio.listar());
        return "solicitudes/lista"; // nombre de la plantilla HTML
    }
}

// @RestController — para APIs REST
// Es equivalente a @Controller + @ResponseBody en cada método
// Devuelve los datos DIRECTAMENTE como JSON (o XML)
@RestController
public class SolicitudController {
    
    @GetMapping("/api/v1/solicitudes")
    public List<SolicitudResponse> listar() {
        return servicio.listar(); // Spring lo convierte a JSON automáticamente
    }
}
```

**Tabla comparativa:**

| Característica | `@Controller` | `@RestController` |
|---------------|--------------|-------------------|
| Uso principal | Aplicaciones MVC con vistas | APIs REST (JSON/XML) |
| Serialización | Manual, retorna nombre de vista | Automática a JSON |
| Incluye | Solo `@Controller` | `@Controller` + `@ResponseBody` |
| Retorna | String con nombre de plantilla | Objetos Java → JSON |
| Proyecto Triage | ❌ No usamos | ✅ Usamos siempre |

**Para nuestro proyecto:** siempre usaremos `@RestController`.

---

## 6. Anotaciones de Mapeo HTTP

### 6.1 `@GetMapping`

Úsalo para **obtener** recursos. Nunca debe modificar datos.

```java
// Obtener colección
@GetMapping("/api/v1/solicitudes")
public ResponseEntity<Page<SolicitudResponse>> listar(Pageable pageable) { ... }

// Obtener un recurso por ID
@GetMapping("/api/v1/solicitudes/{id}")
public ResponseEntity<SolicitudDetalleResponse> obtenerPorId(@PathVariable Long id) { ... }

// Obtener sub-recurso
@GetMapping("/api/v1/solicitudes/{id}/historial")
public ResponseEntity<List<HistorialResponse>> obtenerHistorial(@PathVariable Long id) { ... }
```

### 6.2 `@PostMapping`

Úsalo para **crear** nuevos recursos o ejecutar acciones.

```java
// Crear nueva solicitud
@PostMapping("/api/v1/solicitudes")
public ResponseEntity<SolicitudResponse> crear(@Valid @RequestBody CrearSolicitudRequest request) { ... }
// ↑ Devuelve 201 Created con la solicitud creada y su ID

// Acción: cerrar solicitud
@PostMapping("/api/v1/solicitudes/{id}/cerrar")
public ResponseEntity<SolicitudResponse> cerrar(
    @PathVariable Long id,
    @Valid @RequestBody CerrarSolicitudRequest request) { ... }
```

### 6.3 `@PutMapping` y `@PatchMapping`

```java
// PUT — Reemplaza el recurso COMPLETO
// Si mandas solo algunos campos, los demás quedan en null
@PutMapping("/api/v1/solicitudes/{id}")
public ResponseEntity<SolicitudResponse> reemplazar(
    @PathVariable Long id,
    @Valid @RequestBody SolicitudRequest request) { ... }

// PATCH — Modifica PARCIALMENTE el recurso
// Solo actualizas los campos que mandas
@PatchMapping("/api/v1/solicitudes/{id}/estado")
public ResponseEntity<SolicitudResponse> cambiarEstado(
    @PathVariable Long id,
    @Valid @RequestBody CambiarEstadoRequest request) { ... }
```

> **Cuándo usar PUT vs PATCH en el Triage:** Usaremos `PATCH` para operaciones parciales como cambiar el estado, asignar un responsable o clasificar una solicitud. Si quisiéramos editar todos los datos de un perfil de usuario de una vez, usaríamos `PUT`.

### 6.4 `@DeleteMapping`

```java
// Eliminar recurso (solo ADMIN en nuestro sistema)
@DeleteMapping("/api/v1/solicitudes/{id}")
public ResponseEntity<Void> eliminar(@PathVariable Long id) {
    servicio.eliminar(id);
    return ResponseEntity.noContent().build(); // 204 No Content
}
```

---

## 7. Parámetros de Entrada

Hay cuatro formas principales de recibir datos en un controlador REST:

### 7.1 `@PathVariable`

Para datos que **identifican** el recurso y van en la URL.

```java
// URL: GET /api/v1/solicitudes/42
@GetMapping("/solicitudes/{id}")
public ResponseEntity<SolicitudDetalleResponse> obtener(@PathVariable Long id) {
    // id = 42
    return ResponseEntity.ok(servicio.obtenerPorId(id));
}

// Múltiples variables de ruta
// URL: GET /api/v1/usuarios/5/solicitudes/42
@GetMapping("/usuarios/{usuarioId}/solicitudes/{solicitudId}")
public ResponseEntity<SolicitudDetalleResponse> obtenerSolicitudDeUsuario(
    @PathVariable Long usuarioId,
    @PathVariable Long solicitudId) { ... }

// El nombre de la variable puede diferir del parámetro
@GetMapping("/solicitudes/{solicitudId}")
public ResponseEntity<SolicitudDetalleResponse> obtener(
    @PathVariable("solicitudId") Long id) { ... }
```

### 7.2 `@RequestParam`

Para **filtros, búsquedas y paginación** — datos opcionales que van en la query string.

```java
// URL: GET /api/v1/solicitudes?estado=EN_ATENCION&prioridad=ALTA&page=0&size=10
@GetMapping("/solicitudes")
public ResponseEntity<Page<SolicitudResponse>> listar(
    @RequestParam(required = false) EstadoSolicitud estado,
    @RequestParam(required = false) Prioridad prioridad,
    @RequestParam(required = false) String tipo,
    Pageable pageable) { // Pageable se configura automáticamente con page, size, sort
    
    return ResponseEntity.ok(servicio.listar(estado, prioridad, tipo, pageable));
}

// Con valor por defecto
@GetMapping("/solicitudes")
public ResponseEntity<Page<SolicitudResponse>> listar(
    @RequestParam(defaultValue = "REGISTRADA") EstadoSolicitud estado,
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size) { ... }
```

### 7.3 `@RequestBody`

Para datos en el **cuerpo** de la petición (JSON). Usado con POST, PUT, PATCH.

```java
// El JSON del cuerpo se deserializa automáticamente al DTO
@PostMapping("/solicitudes")
public ResponseEntity<SolicitudResponse> crear(
    @Valid @RequestBody CrearSolicitudRequest request) {
    // Spring convierte el JSON a un objeto CrearSolicitudRequest automáticamente
    // @Valid activa las validaciones definidas en el Record/clase
    var respuesta = servicio.crear(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(respuesta);
}
```

### 7.4 `@RequestHeader`

Para datos en los **headers** HTTP. Rara vez lo usarás directamente (Spring Security maneja el JWT automáticamente), pero es útil saberlo.

```java
// Extraer un header específico
@GetMapping("/solicitudes")
public ResponseEntity<List<SolicitudResponse>> listar(
    @RequestHeader("X-Correlation-ID") String correlationId,
    @RequestHeader(value = "Accept-Language", required = false, defaultValue = "es") String lang) {
    // correlationId y lang disponibles aquí
    return ResponseEntity.ok(servicio.listar());
}
```

---

## 8. `ResponseEntity<T>`: Control Total de la Respuesta

`ResponseEntity<T>` es el wrapper que te permite controlar **exactamente** qué devuelve tu endpoint: el cuerpo, el código de estado y los headers.

```java
// Forma 1 — Respuesta simple (Spring pone 200 automáticamente)
@GetMapping("/{id}")
public SolicitudResponse obtener(@PathVariable Long id) {
    return servicio.obtenerPorId(id);
    // ⬆️ Funciona, pero no tienes control del código de estado
}

// Forma 2 — Con ResponseEntity, control total
@GetMapping("/{id}")
public ResponseEntity<SolicitudResponse> obtener(@PathVariable Long id) {
    var solicitud = servicio.obtenerPorId(id);
    return ResponseEntity.ok(solicitud); // 200 OK con el cuerpo
}

// Distintos códigos de estado
ResponseEntity.ok(body)                          // 200
ResponseEntity.created(URI.create("/solicitudes/5")).body(body)  // 201 con Location header
ResponseEntity.accepted().body(body)             // 202
ResponseEntity.noContent().build()               // 204 (sin cuerpo)
ResponseEntity.badRequest().body(error)          // 400
ResponseEntity.notFound().build()                // 404 (sin cuerpo)
ResponseEntity.status(HttpStatus.CONFLICT).body(error) // 409

// Con headers personalizados
@PostMapping("/solicitudes")
public ResponseEntity<SolicitudResponse> crear(@RequestBody CrearSolicitudRequest req) {
    var creada = servicio.crear(req);
    
    var location = URI.create("/api/v1/solicitudes/" + creada.id());
    
    return ResponseEntity
        .created(location)          // 201 Created + header Location
        .header("X-Solicitud-Id", creada.id().toString())
        .body(creada);
}
```

**Tabla de métodos estáticos de `ResponseEntity`:**

| Método | Código | Tiene Body | Cuándo usarlo |
|--------|--------|-----------|---------------|
| `ok(body)` | 200 | ✅ | GET exitoso, PUT/PATCH exitoso |
| `created(uri).body(body)` | 201 | ✅ | POST que crea recurso |
| `accepted().body(body)` | 202 | ✅ | Operación asíncrona aceptada |
| `noContent().build()` | 204 | ❌ | DELETE exitoso |
| `badRequest().body(error)` | 400 | ✅ | Datos inválidos |
| `notFound().build()` | 404 | ❌ | Recurso no encontrado |
| `status(HttpStatus.X).body(y)` | Cualquiera | ✅ | Cualquier caso específico |

---

## 9. DTOs con Records de Java 21

### 9.1 Por qué NUNCA exponer entidades JPA directamente

Imagina que tienes la entidad `Usuario` con campos como `password`, `salt`, `tokenReset`, `activo`, `creadoEn`, etc. Si expones esa entidad directamente en la respuesta de la API, **estás filtrando datos sensibles** a quien consuma el endpoint.

Además:
- Las entidades tienen relaciones JPA (`@OneToMany`, `@ManyToOne`) que al serializarse a JSON pueden causar loops infinitos o cargar datos que no necesitas
- Si cambias la estructura de la base de datos (renombras un campo, añades una columna), rompes automáticamente la API
- No puedes dar formas distintas de la misma entidad para distintos contextos (resumen en lista, detalle en vista individual)

**La solución:** DTOs (Data Transfer Objects) — clases o Records que representan exactamente lo que la API necesita recibir o enviar, sin más.

### 9.2 Request DTOs vs Response DTOs

```
Request DTOs  → lo que el cliente envía al servidor (cuerpo de POST, PUT, PATCH)
Response DTOs → lo que el servidor devuelve al cliente
```

### 9.3 Antes vs Ahora: Clases vs Records

#### Antes — DTO como clase (Java 8+)

```java
// Requería: campos privados, constructor, getters, equals, hashCode, toString
// Generalmente resuelto con Lombok
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class SolicitudResponseDTO {
    private Long id;
    private String descripcion;
    private String tipo;
    private String prioridad;
    private String estado;
    private LocalDateTime creadoEn;
    // ... más campos
}
```

#### Ahora — DTO como Record (Java 16+ estable, Java 21 recomendado)

```java
// Record: inmutable, compacto, con equals/hashCode/toString automáticos
// Todo lo que hacía Lombok, Java lo hace nativamente
public record SolicitudResponse(
    Long id,
    String descripcion,
    String tipo,
    String prioridad,
    String estado,
    LocalDateTime creadoEn
) {}
// ↑ ¡Eso es todo! Java genera automáticamente:
// - Constructor canónico con todos los parámetros
// - Getters (sin prefijo "get"): solicitud.id(), solicitud.estado(), etc.
// - equals() y hashCode() basados en todos los campos
// - toString() legible
// - Los campos son final (inmutables)
```

**Tabla comparativa exhaustiva:**

| Característica | Clase DTO (Lombok) | Record (Java 21) |
|----------------|-------------------|-----------------|
| Inmutabilidad | Manual (sin setters) o con `@Value` | Garantizada por el compilador |
| Código boilerplate | Anotaciones Lombok | Cero |
| Serialización JSON | ✅ Jackson | ✅ Jackson (soporte nativo desde Jackson 2.12) |
| Herencia | ✅ Puede extender | ❌ No puede extender clases (solo interfaces) |
| Validaciones `@Valid` | ✅ En campos | ✅ En parámetros del constructor canónico |
| Uso en Spring Boot 3+ | ✅ | ✅ Recomendado |
| Lombok requerido | Sí (para ser práctico) | No |
| Cuándo usar clase | Cuando necesitas mutabilidad o herencia | Rara vez para DTOs |

### 9.4 Mapeo Manual

El mapeo convierte entidades a DTOs y viceversa. El mapeo manual es explícito y fácil de entender:

```java
// Mapper manual como componente de Spring
@Component
public class SolicitudMapper {
    
    // Entidad → Response DTO (para enviar al cliente)
    public SolicitudResponse toResponse(Solicitud solicitud) {
        return new SolicitudResponse(
            solicitud.getId(),
            solicitud.getDescripcion(),
            solicitud.getTipo() != null ? solicitud.getTipo().getNombre() : null,
            solicitud.getPrioridad() != null ? solicitud.getPrioridad().name() : null,
            solicitud.getEstado().name(),
            solicitud.getCreadoEn()
        );
    }
    
    // Request DTO → Entidad (para guardar en BD)
    public Solicitud toEntity(CrearSolicitudRequest request, Usuario solicitante) {
        var solicitud = new Solicitud();
        solicitud.setDescripcion(request.descripcion());
        solicitud.setCanalOrigen(request.canalOrigen());
        solicitud.setSolicitante(solicitante);
        solicitud.setEstado(EstadoSolicitud.REGISTRADA); // estado inicial siempre
        solicitud.setFechaRegistro(LocalDateTime.now());
        return solicitud;
    }
    
    // Colección de entidades → colección de DTOs
    public List<SolicitudResponse> toResponseList(List<Solicitud> solicitudes) {
        return solicitudes.stream()
            .map(this::toResponse)
            .toList();
    }
    
    // Page<Entidad> → Page<DTO> (para paginación)
    public Page<SolicitudResponse> toResponsePage(Page<Solicitud> page) {
        return page.map(this::toResponse);
    }
}
```

### 9.5 Mapeo con MapStruct

MapStruct genera el código de mapeo en tiempo de compilación (más rápido que reflection). Úsalo cuando tengas muchos campos o muchos mappers.

Agregar en `pom.xml`:

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.6.3</version>
</dependency>
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>1.6.3</version>
    <scope>provided</scope>
</dependency>
```

```java
@Mapper(componentModel = "spring", // hace el mapper un bean de Spring
        nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
public interface SolicitudMapper {
    
    // MapStruct infiere el mapeo si los nombres de campos coinciden
    @Mapping(source = "tipo.nombre", target = "tipo")        // tipo.nombre → tipo
    @Mapping(source = "solicitante.nombre", target = "nombreSolicitante")
    @Mapping(source = "estado", target = "estado")           // enum → String automático
    SolicitudResponse toResponse(Solicitud solicitud);
    
    // Para el mapeo inverso, ignoramos campos que se setean manualmente
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "estado", ignore = true)
    @Mapping(target = "creadoEn", ignore = true)
    @Mapping(target = "historial", ignore = true)
    Solicitud toEntity(CrearSolicitudRequest request);
    
    // Mapeo de lista — MapStruct lo genera automáticamente
    List<SolicitudResponse> toResponseList(List<Solicitud> solicitudes);
}
```

---

## 10. Validación de Entrada con `@Valid`

No confíes nunca en los datos que el cliente te envía. Siempre valida antes de procesarlos.

### 10.1 Anotaciones de Bean Validation

Primero, asegúrate de tener la dependencia (ya incluida con `spring-boot-starter-web`):

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Las validaciones van en el Record del DTO:

```java
public record CrearSolicitudRequest(

    @NotBlank(message = "La descripción no puede estar vacía")
    @Size(min = 20, max = 1000, message = "La descripción debe tener entre 20 y 1000 caracteres")
    String descripcion,

    @NotNull(message = "El tipo de solicitud es obligatorio")
    Long tipoSolicitudId,

    @NotNull(message = "El canal de origen es obligatorio")
    CanalOrigen canalOrigen,

    @NotBlank(message = "La identificación del solicitante es obligatoria")
    @Pattern(regexp = "^[0-9]{7,12}$", message = "La identificación debe tener entre 7 y 12 dígitos")
    String identificacionSolicitante

) {}
```

```java
// Más ejemplos de anotaciones de validación
public record UsuarioRequest(

    @NotBlank(message = "El nombre es obligatorio")
    @Size(min = 2, max = 100, message = "El nombre debe tener entre 2 y 100 caracteres")
    String nombre,

    @NotBlank(message = "El email es obligatorio")
    @Email(message = "El email no tiene formato válido")
    String email,

    @NotBlank(message = "La contraseña es obligatoria")
    @Size(min = 8, message = "La contraseña debe tener al menos 8 caracteres")
    @Pattern(regexp = "^(?=.*[A-Za-z])(?=.*\\d).+$",
             message = "La contraseña debe tener al menos una letra y un número")
    String password,

    @NotNull(message = "El rol es obligatorio")
    Rol rol,

    @Past(message = "La fecha de nacimiento debe ser en el pasado")
    LocalDate fechaNacimiento,

    @Positive(message = "El número debe ser positivo")
    Integer codigoEstudiante
) {}
```

**Tabla de anotaciones de validación más usadas:**

| Anotación | Aplica a | Descripción |
|-----------|----------|-------------|
| `@NotNull` | Cualquier objeto | No puede ser `null` |
| `@NotBlank` | `String` | No puede ser `null`, vacío ni solo espacios |
| `@NotEmpty` | String, colecciones, arrays | No puede ser `null` ni vacío |
| `@Size(min, max)` | String, colecciones | Tamaño entre min y max |
| `@Min(value)` | Números | Valor mínimo |
| `@Max(value)` | Números | Valor máximo |
| `@Positive` | Números | Debe ser positivo (> 0) |
| `@PositiveOrZero` | Números | Debe ser ≥ 0 |
| `@Email` | String | Formato de email válido |
| `@Pattern(regexp)` | String | Debe coincidir con la expresión regular |
| `@Past` | Fechas | Debe ser en el pasado |
| `@Future` | Fechas | Debe ser en el futuro |
| `@Valid` | Objetos anidados | Activa validación en cascada |

Para activar la validación en el controlador, agrega `@Valid` antes del `@RequestBody`:

```java
@PostMapping("/solicitudes")
public ResponseEntity<SolicitudResponse> crear(@Valid @RequestBody CrearSolicitudRequest request) {
    // Si alguna validación falla, Spring lanza MethodArgumentNotValidException
    // antes de que este código se ejecute
    return ResponseEntity.status(HttpStatus.CREATED).body(servicio.crear(request));
}
```

### 10.2 Validación Personalizada

Cuando las anotaciones estándar no son suficientes, puedes crear las tuyas:

```java
// 1. Definir la anotación
@Documented
@Constraint(validatedBy = DescripcionUnicaValidator.class)
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface DescripcionUnica {
    String message() default "Ya existe una solicitud con esa descripción para este usuario";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// 2. Implementar el validador
@Component
public class DescripcionUnicaValidator implements ConstraintValidator<DescripcionUnica, String> {
    
    private final SolicitudRepository solicitudRepository;
    
    public DescripcionUnicaValidator(SolicitudRepository solicitudRepository) {
        this.solicitudRepository = solicitudRepository;
    }
    
    @Override
    public boolean isValid(String descripcion, ConstraintValidatorContext context) {
        if (descripcion == null || descripcion.isBlank()) return true; // deja que @NotBlank maneje esto
        return !solicitudRepository.existsByDescripcionIgnoreCase(descripcion);
    }
}
```

---

## 11. Manejo Global de Errores

Sin un manejo centralizado de errores, cada excepción no controlada devuelve un `500 Internal Server Error` con el stack trace completo — una fuga de información grave en producción. La solución es `@ControllerAdvice`.

### 11.1 `@ControllerAdvice` y `@ExceptionHandler`

`@ControllerAdvice` es una clase que intercepta excepciones lanzadas en cualquier controlador y les da una respuesta HTTP apropiada.

```java
@ControllerAdvice es como un "red de seguridad" global:
cuando cualquier controlador lanza una excepción sin capturar,
@ControllerAdvice la atrapa y devuelve una respuesta limpia al cliente.
```

### 11.2 Excepciones Personalizadas

Primero, definamos las excepciones del dominio del Triage:

```java
// Excepción base del dominio
public class TiageException extends RuntimeException {
    private final HttpStatus httpStatus;
    
    public TiageException(String message, HttpStatus httpStatus) {
        super(message);
        this.httpStatus = httpStatus;
    }
    
    public HttpStatus getHttpStatus() {
        return httpStatus;
    }
}

// Recurso no encontrado (404)
public class RecursoNoEncontradoException extends TiageException {
    public RecursoNoEncontradoException(String recurso, Long id) {
        super("No se encontró %s con id %d".formatted(recurso, id), HttpStatus.NOT_FOUND);
    }
    
    public RecursoNoEncontradoException(String mensaje) {
        super(mensaje, HttpStatus.NOT_FOUND);
    }
}

// Transición de estado inválida (409 Conflict)
public class TransicionEstadoInvalidaException extends TiageException {
    public TransicionEstadoInvalidaException(EstadoSolicitud estadoActual, EstadoSolicitud estadoNuevo) {
        super(
            "No es posible cambiar el estado de %s a %s".formatted(estadoActual, estadoNuevo),
            HttpStatus.CONFLICT
        );
    }
}

// Operación no permitida (403)
public class OperacionNoPermitidaException extends TiageException {
    public OperacionNoPermitidaException(String operacion) {
        super("Operación no permitida: " + operacion, HttpStatus.FORBIDDEN);
    }
}

// Solicitud ya cerrada (409)
public class SolicitudCerradaException extends TiageException {
    public SolicitudCerradaException(Long solicitudId) {
        super(
            "La solicitud %d ya está cerrada y no puede modificarse".formatted(solicitudId),
            HttpStatus.CONFLICT
        );
    }
}
```

### 11.3 Estructura RFC 7807 — Problem Details

La RFC 7807 es un estándar internacional para representar errores de APIs HTTP. En lugar de inventar tu propio formato de error, usa este estándar que el ecosistema ya conoce.

**Spring Boot 3+ soporta Problem Details nativamente.** Solo debes habilitarlo:

```yaml
# application.yml
spring:
  mvc:
    problemdetails:
      enabled: true
```

El formato RFC 7807 en JSON se ve así:

```json
{
  "type": "https://triage.uniquindio.edu.co/errors/recurso-no-encontrado",
  "title": "Recurso no encontrado",
  "status": 404,
  "detail": "No se encontró Solicitud con id 999",
  "instance": "/api/v1/solicitudes/999",
  "timestamp": "2026-03-15T10:30:00Z"
}
```

Ahora el `@ControllerAdvice` completo:

```java
@RestControllerAdvice                // = @ControllerAdvice + @ResponseBody
@Slf4j
public class GlobalExceptionHandler {
    
    // Manejo de nuestra excepción base del dominio
    @ExceptionHandler(TiageException.class)
    public ResponseEntity<ProblemDetail> handleTiageException(
            TiageException ex, HttpServletRequest request) {
        
        log.warn("Error de dominio: {} — URI: {}", ex.getMessage(), request.getRequestURI());
        
        var problem = ProblemDetail.forStatusAndDetail(ex.getHttpStatus(), ex.getMessage());
        problem.setTitle(ex.getHttpStatus().getReasonPhrase());
        problem.setInstance(URI.create(request.getRequestURI()));
        problem.setProperty("timestamp", Instant.now());
        
        return ResponseEntity.status(ex.getHttpStatus()).body(problem);
    }
    
    // Manejo de errores de validación (@Valid falló)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ProblemDetail> handleValidationException(
            MethodArgumentNotValidException ex, HttpServletRequest request) {
        
        // Recolectamos todos los errores de validación
        var errores = ex.getBindingResult().getFieldErrors().stream()
            .collect(Collectors.toMap(
                FieldError::getField,
                FieldError::getDefaultMessage,
                (existing, replacement) -> existing + "; " + replacement
            ));
        
        log.warn("Error de validación: {} errores — URI: {}", errores.size(), request.getRequestURI());
        
        var problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.BAD_REQUEST,
            "Los datos de entrada no son válidos. Revisa los errores de campo."
        );
        problem.setTitle("Error de validación");
        problem.setInstance(URI.create(request.getRequestURI()));
        problem.setProperty("timestamp", Instant.now());
        problem.setProperty("errores", errores); // campo extra con el detalle
        
        return ResponseEntity.badRequest().body(problem);
    }
    
    // JSON malformado en el cuerpo de la petición
    @ExceptionHandler(HttpMessageNotReadableException.class)
    public ResponseEntity<ProblemDetail> handleJsonMalformado(
            HttpMessageNotReadableException ex, HttpServletRequest request) {
        
        var problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.BAD_REQUEST,
            "El cuerpo de la petición tiene formato JSON inválido"
        );
        problem.setTitle("JSON inválido");
        problem.setInstance(URI.create(request.getRequestURI()));
        problem.setProperty("timestamp", Instant.now());
        
        return ResponseEntity.badRequest().body(problem);
    }
    
    // Método HTTP no soportado (ej: DELETE cuando solo existe GET y POST)
    @ExceptionHandler(HttpRequestMethodNotSupportedException.class)
    public ResponseEntity<ProblemDetail> handleMetodoNoSoportado(
            HttpRequestMethodNotSupportedException ex, HttpServletRequest request) {
        
        var problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.METHOD_NOT_ALLOWED,
            "El método %s no está soportado para esta URL".formatted(ex.getMethod())
        );
        problem.setTitle("Método no soportado");
        problem.setInstance(URI.create(request.getRequestURI()));
        problem.setProperty("metodosPermitidos", ex.getSupportedHttpMethods());
        problem.setProperty("timestamp", Instant.now());
        
        return ResponseEntity.status(HttpStatus.METHOD_NOT_ALLOWED).body(problem);
    }
    
    // Errores inesperados del servidor (loguear el stack trace, no enviarlo al cliente)
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ProblemDetail> handleGeneral(
            Exception ex, HttpServletRequest request) {
        
        // Logueamos el error completo para el equipo de desarrollo
        log.error("Error inesperado en {}: ", request.getRequestURI(), ex);
        
        // Al cliente solo le decimos que hubo un error (sin detalles internos)
        var problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.INTERNAL_SERVER_ERROR,
            "Ocurrió un error inesperado. Por favor intenta de nuevo más tarde."
        );
        problem.setTitle("Error interno del servidor");
        problem.setInstance(URI.create(request.getRequestURI()));
        problem.setProperty("timestamp", Instant.now());
        
        return ResponseEntity.internalServerError().body(problem);
    }
}
```

**Respuesta de error de validación que recibirá Angular:**

```json
{
  "type": "about:blank",
  "title": "Error de validación",
  "status": 400,
  "detail": "Los datos de entrada no son válidos. Revisa los errores de campo.",
  "instance": "/api/v1/solicitudes",
  "timestamp": "2026-03-15T10:30:00Z",
  "errores": {
    "descripcion": "La descripción debe tener entre 20 y 1000 caracteres",
    "tipoSolicitudId": "El tipo de solicitud es obligatorio"
  }
}
```

---

## 12. Documentación con OpenAPI 3 / Swagger

Una API sin documentación es como un producto sin manual. Swagger UI genera automáticamente una interfaz web interactiva donde puedes ver y probar todos los endpoints.

### 12.1 Configurar SpringDoc

Agregar dependencia en `pom.xml`:

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.7.0</version>
</dependency>
```

Configuración en `application.yml`:

```yaml
springdoc:
  api-docs:
    path: /api-docs          # endpoint JSON del contrato OpenAPI
  swagger-ui:
    path: /swagger-ui.html   # URL de la interfaz visual
    tags-sorter: alpha        # ordena los grupos alfabéticamente
    operations-sorter: alpha  # ordena los endpoints alfabéticamente
  info:
    title: API del Sistema de Triage
    description: >
      API REST para la gestión de solicitudes académicas del
      Programa de Ingeniería de Sistemas — Universidad del Quindío
    version: v1.0.0
    contact:
      name: Equipo de Desarrollo
      email: dev@uniquindio.edu.co
```

Configuración adicional con Bean:

```java
@Configuration
public class OpenApiConfig {
    
    @Bean
    public OpenAPI openApiTriage() {
        return new OpenAPI()
            .info(new Info()
                .title("API — Sistema de Triage Académico")
                .description("Gestión de solicitudes académicas: homologaciones, cupos, cancelaciones y más.")
                .version("v1.0.0")
                .contact(new Contact()
                    .name("Programación Avanzada — UniQuindío")
                    .email("prog.avanzada@uniquindio.edu.co"))
                .license(new License().name("MIT").url("https://opensource.org/licenses/MIT"))
            )
            // Configuración de autenticación JWT en Swagger
            .addSecurityItem(new SecurityRequirement().addList("bearerAuth"))
            .components(new Components()
                .addSecuritySchemes("bearerAuth", new SecurityScheme()
                    .type(SecurityScheme.Type.HTTP)
                    .scheme("bearer")
                    .bearerFormat("JWT")
                    .description("Ingresa el token JWT obtenido en /api/v1/auth/login")
                )
            );
    }
}
```

### 12.2 Anotaciones de Documentación

```java
@RestController
@RequestMapping("/api/v1/solicitudes")
@Tag(name = "Solicitudes", description = "Gestión del ciclo de vida de solicitudes académicas")
public class SolicitudController {
    
    @Operation(
        summary = "Listar solicitudes",
        description = "Retorna una lista paginada de solicitudes filtradas por estado, tipo y prioridad"
    )
    @ApiResponse(responseCode = "200", description = "Lista obtenida correctamente",
        content = @Content(schema = @Schema(implementation = Page.class)))
    @ApiResponse(responseCode = "401", description = "Token JWT no proporcionado o inválido")
    @ApiResponse(responseCode = "403", description = "No tiene permisos para esta operación")
    @GetMapping
    public ResponseEntity<Page<SolicitudResponse>> listar(
            @Parameter(description = "Estado de la solicitud") 
            @RequestParam(required = false) EstadoSolicitud estado,
            @Parameter(description = "Paginación: page=0&size=10&sort=creadoEn,desc")
            Pageable pageable) {
        return ResponseEntity.ok(servicio.listar(estado, pageable));
    }
    
    @Operation(summary = "Registrar nueva solicitud", description = "RF-01: Crea una nueva solicitud académica")
    @ApiResponse(responseCode = "201", description = "Solicitud creada correctamente")
    @ApiResponse(responseCode = "400", description = "Datos de entrada inválidos")
    @PostMapping
    public ResponseEntity<SolicitudResponse> crear(
            @io.swagger.v3.oas.annotations.parameters.RequestBody(
                description = "Datos de la nueva solicitud",
                required = true
            )
            @Valid @RequestBody CrearSolicitudRequest request) {
        var creada = servicio.crear(request);
        var location = URI.create("/api/v1/solicitudes/" + creada.id());
        return ResponseEntity.created(location).body(creada);
    }
}
```

### 12.3 Acceder a Swagger UI

Cuando ejecutes el proyecto, navega a:

```
http://localhost:8080/swagger-ui.html
```

Desde ahí puedes:
- Ver todos los endpoints documentados
- Ver los schemas de Request y Response
- Probar los endpoints directamente (con autenticación JWT si la configuras)
- Descargar el contrato OpenAPI en JSON: `http://localhost:8080/api-docs`

---

## 13. CORS: Qué Es, Por Qué Ocurre y Cómo Solucionarlo

### ¿Qué es CORS?

**CORS** (Cross-Origin Resource Sharing) es un mecanismo de seguridad del navegador que bloquea peticiones HTTP a un dominio diferente al de la página web.

**El problema concreto:** Tu frontend Angular corre en `http://localhost:4200` y tu backend Spring Boot en `http://localhost:8080`. Son **orígenes distintos** (diferente puerto). El navegador bloqueará las peticiones de Angular al backend a menos que el backend le diga explícitamente "está bien, acepto peticiones de `localhost:4200`".

```
Sin CORS configurado:
Angular (4200) → HTTP Request → Spring Boot (8080) → Respuesta correcta → Navegador LA BLOQUEA ❌

Con CORS configurado:
Angular (4200) → HTTP Request → Spring Boot (8080) → Respuesta con header CORS → Navegador la acepta ✅
```

### Solución en Spring Boot

```java
@Configuration
public class CorsConfig {
    
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        var config = new CorsConfiguration();
        
        // Orígenes permitidos — en desarrollo, Angular corre en 4200
        config.setAllowedOrigins(List.of(
            "http://localhost:4200",     // Angular en desarrollo
            "http://localhost:3000",     // Si también usas React o Vite
            "https://triage.uniquindio.edu.co" // URL de producción
        ));
        
        // Verbos HTTP permitidos
        config.setAllowedMethods(List.of("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"));
        
        // Headers que el cliente puede enviar
        config.setAllowedHeaders(List.of(
            "Authorization",   // Para el JWT
            "Content-Type",
            "Accept",
            "X-Requested-With",
            "Origin"
        ));
        
        // Headers que el cliente puede leer de la respuesta
        config.setExposedHeaders(List.of(
            "Location",        // Para las respuestas 201 Created
            "X-Solicitud-Id"
        ));
        
        // Permite enviar cookies y headers de autenticación
        config.setAllowCredentials(true);
        
        // Tiempo que el navegador puede cachear la respuesta al preflight OPTIONS
        config.setMaxAge(3600L); // 1 hora
        
        var source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", config);
        
        return source;
    }
}
```

> **Importante:** En la Guía 07 (Spring Security) verás que CORS se integra con Security. Por ahora, esta configuración funciona para el proyecto sin seguridad activa.

---

## 14. Paginación en la API

Nunca devuelvas `List<T>` sin límite para colecciones grandes. Imagina que el sistema tiene 50.000 solicitudes — devolver todas en una sola respuesta sería desastroso para el rendimiento.

Spring Data JPA + Spring MVC tienen soporte nativo para paginación:

```java
// En el repositorio
public interface SolicitudRepository extends JpaRepository<Solicitud, Long> {
    
    Page<Solicitud> findByEstado(EstadoSolicitud estado, Pageable pageable);
    
    // Con múltiples filtros usando Specification
    Page<Solicitud> findAll(Specification<Solicitud> spec, Pageable pageable);
}

// En el controlador
@GetMapping
public ResponseEntity<Page<SolicitudResponse>> listar(
        @RequestParam(required = false) EstadoSolicitud estado,
        @RequestParam(required = false) Prioridad prioridad,
        Pageable pageable) { // Spring extrae automáticamente ?page=0&size=10&sort=creadoEn,desc
    
    return ResponseEntity.ok(servicio.listar(estado, prioridad, pageable));
}
```

**Ejemplo de respuesta paginada:**

```json
{
  "content": [
    { "id": 1, "descripcion": "...", "estado": "REGISTRADA" },
    { "id": 2, "descripcion": "...", "estado": "EN_ATENCION" }
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 10,
    "sort": { "sorted": true, "unsorted": false }
  },
  "totalElements": 157,
  "totalPages": 16,
  "first": true,
  "last": false,
  "numberOfElements": 10
}
```

**URLs de ejemplo para el cliente:**

```
GET /api/v1/solicitudes                          → página 0, tamaño 20 (defaults)
GET /api/v1/solicitudes?page=1&size=10           → página 1, 10 por página
GET /api/v1/solicitudes?sort=creadoEn,desc       → ordenadas por fecha, más recientes primero
GET /api/v1/solicitudes?page=0&size=10&sort=prioridad,asc&estado=EN_ATENCION
```

---

## 15. Implementación Completa: Endpoints del Triage

Aquí construimos todo el proyecto de forma real y funcional.

### 15.1 Estructura de Carpetas

```
triage-backend/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── co/
│   │   │       └── uniquindio/
│   │   │           └── triage/
│   │   │               ├── TriageApplication.java
│   │   │               ├── config/
│   │   │               │   ├── OpenApiConfig.java
│   │   │               │   └── CorsConfig.java
│   │   │               ├── controller/
│   │   │               │   ├── SolicitudController.java
│   │   │               │   └── AuthController.java
│   │   │               ├── dto/
│   │   │               │   ├── request/
│   │   │               │   │   ├── CrearSolicitudRequest.java
│   │   │               │   │   ├── ClasificarSolicitudRequest.java
│   │   │               │   │   ├── AsignarResponsableRequest.java
│   │   │               │   │   ├── CambiarEstadoRequest.java
│   │   │               │   │   └── CerrarSolicitudRequest.java
│   │   │               │   └── response/
│   │   │               │       ├── SolicitudResponse.java
│   │   │               │       ├── SolicitudDetalleResponse.java
│   │   │               │       └── HistorialResponse.java
│   │   │               ├── exception/
│   │   │               │   ├── TiageException.java
│   │   │               │   ├── RecursoNoEncontradoException.java
│   │   │               │   ├── TransicionEstadoInvalidaException.java
│   │   │               │   ├── SolicitudCerradaException.java
│   │   │               │   └── GlobalExceptionHandler.java
│   │   │               ├── mapper/
│   │   │               │   └── SolicitudMapper.java
│   │   │               ├── model/
│   │   │               │   ├── Solicitud.java
│   │   │               │   ├── HistorialSolicitud.java
│   │   │               │   ├── TipoSolicitud.java
│   │   │               │   ├── Usuario.java
│   │   │               │   └── enums/
│   │   │               │       ├── EstadoSolicitud.java
│   │   │               │       ├── Prioridad.java
│   │   │               │       ├── CanalOrigen.java
│   │   │               │       └── Rol.java
│   │   │               ├── repository/
│   │   │               │   ├── SolicitudRepository.java
│   │   │               │   ├── HistorialSolicitudRepository.java
│   │   │               │   ├── TipoSolicitudRepository.java
│   │   │               │   └── UsuarioRepository.java
│   │   │               └── service/
│   │   │                   ├── SolicitudService.java
│   │   │                   └── impl/
│   │   │                       └── SolicitudServiceImpl.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── db/
│   │           └── migration/
│   │               ├── V1__crear_tablas_base.sql
│   │               └── V2__datos_iniciales.sql
│   └── test/
│       └── java/
│           └── co/uniquindio/triage/
│               └── service/
│                   └── SolicitudServiceTest.java
└── pom.xml
```

### 15.2 Enums y Entidades de Referencia

```java
// co/uniquindio/triage/model/enums/EstadoSolicitud.java
public enum EstadoSolicitud {
    REGISTRADA,
    CLASIFICADA,
    EN_ATENCION,
    ATENDIDA,
    CERRADA;
    
    // Valida si una transición de estado es permitida (RF-04)
    public boolean puedeTransicionarA(EstadoSolicitud nuevoEstado) {
        return switch (this) {
            case REGISTRADA  -> nuevoEstado == CLASIFICADA;
            case CLASIFICADA -> nuevoEstado == EN_ATENCION;
            case EN_ATENCION -> nuevoEstado == ATENDIDA;
            case ATENDIDA    -> nuevoEstado == CERRADA;
            case CERRADA     -> false; // estado terminal
        };
    }
}
```

```java
// co/uniquindio/triage/model/enums/Prioridad.java
public enum Prioridad {
    BAJA(1, "Impacto académico bajo"),
    MEDIA(2, "Impacto académico moderado"),
    ALTA(3, "Impacto académico significativo"),
    CRITICA(4, "Bloquea la continuación académica del estudiante");
    
    private final int nivel;
    private final String descripcion;
    
    Prioridad(int nivel, String descripcion) {
        this.nivel = nivel;
        this.descripcion = descripcion;
    }
    
    public int getNivel() { return nivel; }
    public String getDescripcion() { return descripcion; }
}
```

```java
// co/uniquindio/triage/model/enums/CanalOrigen.java
public enum CanalOrigen {
    PRESENCIAL,
    CORREO_ELECTRONICO,
    SAC,
    TELEFONICO,
    PLATAFORMA_WEB
}
```

```java
// co/uniquindio/triage/model/enums/Rol.java
public enum Rol {
    ESTUDIANTE,
    FUNCIONARIO,
    ADMIN
}
```

```java
// co/uniquindio/triage/model/TipoSolicitud.java
@Entity
@Table(name = "tipos_solicitud")
public class TipoSolicitud {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "nombre", nullable = false, unique = true, length = 100)
    private String nombre;
    
    @Column(name = "descripcion", length = 500)
    private String descripcion;
    
    @Column(name = "activo", nullable = false)
    private boolean activo = true;
    
    // getters y setters
}
```

```java
// co/uniquindio/triage/model/Usuario.java
@Entity
@Table(name = "usuarios")
@EntityListeners(AuditingEntityListener.class)
public class Usuario {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "nombre", nullable = false, length = 150)
    private String nombre;
    
    @Column(name = "email", nullable = false, unique = true, length = 200)
    private String email;
    
    @Column(name = "identificacion", nullable = false, unique = true, length = 20)
    private String identificacion;
    
    @Column(name = "password_hash", nullable = false)
    private String passwordHash;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "rol", nullable = false, length = 20)
    private Rol rol;
    
    @Column(name = "activo", nullable = false)
    private boolean activo = true;
    
    @CreatedDate
    @Column(name = "creado_en", nullable = false, updatable = false)
    private LocalDateTime creadoEn;
    
    // getters y setters
}
```

```java
// co/uniquindio/triage/model/Solicitud.java
@Entity
@Table(name = "solicitudes")
@EntityListeners(AuditingEntityListener.class)
public class Solicitud {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "descripcion", nullable = false, length = 1000)
    private String descripcion;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "tipo_solicitud_id")
    private TipoSolicitud tipo;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "prioridad", length = 20)
    private Prioridad prioridad;
    
    @Column(name = "justificacion_prioridad", length = 500)
    private String justificacionPrioridad;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "estado", nullable = false, length = 20)
    private EstadoSolicitud estado = EstadoSolicitud.REGISTRADA;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "canal_origen", nullable = false, length = 30)
    private CanalOrigen canalOrigen;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "solicitante_id", nullable = false)
    private Usuario solicitante;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "responsable_id")
    private Usuario responsable;
    
    @OneToMany(mappedBy = "solicitud", cascade = CascadeType.ALL, orphanRemoval = true)
    @OrderBy("realizadoEn ASC")
    private List<HistorialSolicitud> historial = new ArrayList<>();
    
    @CreatedDate
    @Column(name = "creado_en", nullable = false, updatable = false)
    private LocalDateTime creadoEn;
    
    @LastModifiedDate
    @Column(name = "actualizado_en")
    private LocalDateTime actualizadoEn;
    
    // getters y setters
}
```

```java
// co/uniquindio/triage/model/HistorialSolicitud.java
@Entity
@Table(name = "historial_solicitudes")
public class HistorialSolicitud {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "solicitud_id", nullable = false)
    private Solicitud solicitud;
    
    @Column(name = "accion", nullable = false, length = 200)
    private String accion;
    
    @Column(name = "observaciones", length = 1000)
    private String observaciones;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "usuario_id", nullable = false)
    private Usuario usuario;
    
    @Column(name = "realizado_en", nullable = false)
    private LocalDateTime realizadoEn = LocalDateTime.now();
    
    @Enumerated(EnumType.STRING)
    @Column(name = "estado_anterior", length = 20)
    private EstadoSolicitud estadoAnterior;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "estado_nuevo", length = 20)
    private EstadoSolicitud estadoNuevo;
    
    // getters y setters
}
```

### 15.3 DTOs Completos

```java
// === REQUEST DTOs ===

// co/uniquindio/triage/dto/request/CrearSolicitudRequest.java
// RF-01: Registrar solicitud académica
public record CrearSolicitudRequest(

    @NotBlank(message = "La descripción no puede estar vacía")
    @Size(min = 20, max = 1000, message = "La descripción debe tener entre 20 y 1000 caracteres")
    String descripcion,

    @NotNull(message = "El tipo de solicitud es obligatorio")
    Long tipoSolicitudId,

    @NotNull(message = "El canal de origen es obligatorio")
    CanalOrigen canalOrigen
    
    // El solicitante se toma del JWT — no lo pide el cliente
) {}
```

```java
// co/uniquindio/triage/dto/request/ClasificarSolicitudRequest.java
// RF-02, RF-03: Clasificar y priorizar solicitud
public record ClasificarSolicitudRequest(

    @NotNull(message = "El tipo de solicitud es obligatorio para clasificar")
    Long tipoSolicitudId,

    @NotNull(message = "La prioridad es obligatoria")
    Prioridad prioridad,

    @NotBlank(message = "Debe proporcionar una justificación para la prioridad asignada")
    @Size(max = 500, message = "La justificación no puede superar los 500 caracteres")
    String justificacionPrioridad
    
) {}
```

```java
// co/uniquindio/triage/dto/request/AsignarResponsableRequest.java
// RF-05: Asignar responsable
public record AsignarResponsableRequest(

    @NotNull(message = "El ID del responsable es obligatorio")
    Long responsableId,

    @Size(max = 500, message = "Las observaciones no pueden superar los 500 caracteres")
    String observaciones
    
) {}
```

```java
// co/uniquindio/triage/dto/request/CambiarEstadoRequest.java
// RF-04: Gestión del ciclo de vida
public record CambiarEstadoRequest(

    @NotNull(message = "El nuevo estado es obligatorio")
    EstadoSolicitud nuevoEstado,

    @Size(max = 500)
    String observaciones
    
) {}
```

```java
// co/uniquindio/triage/dto/request/CerrarSolicitudRequest.java
// RF-08: Cerrar solicitud
public record CerrarSolicitudRequest(

    @NotBlank(message = "Debe registrar una observación de cierre")
    @Size(min = 10, max = 1000, message = "La observación de cierre debe tener entre 10 y 1000 caracteres")
    String observacionCierre
    
) {}
```

```java
// === RESPONSE DTOs ===

// co/uniquindio/triage/dto/response/SolicitudResponse.java
// Vista resumida para listas
public record SolicitudResponse(
    Long id,
    String descripcion,
    String tipo,
    String prioridad,
    String estado,
    String canalOrigen,
    String nombreSolicitante,
    String nombreResponsable,   // puede ser null
    LocalDateTime creadoEn,
    LocalDateTime actualizadoEn
) {}
```

```java
// co/uniquindio/triage/dto/response/SolicitudDetalleResponse.java
// Vista completa con historial incluido
public record SolicitudDetalleResponse(
    Long id,
    String descripcion,
    Long tipoSolicitudId,
    String tipo,
    String prioridad,
    String justificacionPrioridad,
    String estado,
    String canalOrigen,
    Long solicitanteId,
    String nombreSolicitante,
    String emailSolicitante,
    Long responsableId,         // puede ser null
    String nombreResponsable,   // puede ser null
    LocalDateTime creadoEn,
    LocalDateTime actualizadoEn,
    List<HistorialResponse> historial
) {}
```

```java
// co/uniquindio/triage/dto/response/HistorialResponse.java
public record HistorialResponse(
    Long id,
    String accion,
    String observaciones,
    String estadoAnterior,
    String estadoNuevo,
    String nombreUsuario,
    LocalDateTime realizadoEn
) {}
```

### 15.4 Repositorios

```java
// co/uniquindio/triage/repository/SolicitudRepository.java
@Repository
public interface SolicitudRepository extends JpaRepository<Solicitud, Long> {
    
    // Búsqueda por estado (RF-07)
    Page<Solicitud> findByEstado(EstadoSolicitud estado, Pageable pageable);
    
    // Búsqueda por solicitante
    Page<Solicitud> findBySolicitanteId(Long solicitanteId, Pageable pageable);
    
    // Búsqueda por responsable
    Page<Solicitud> findByResponsableId(Long responsableId, Pageable pageable);
    
    // Búsqueda combinada con JPQL
    @Query("""
        SELECT s FROM Solicitud s
        LEFT JOIN FETCH s.tipo t
        LEFT JOIN FETCH s.solicitante sol
        LEFT JOIN FETCH s.responsable resp
        WHERE (:estado IS NULL OR s.estado = :estado)
          AND (:prioridad IS NULL OR s.prioridad = :prioridad)
          AND (:tipoId IS NULL OR t.id = :tipoId)
        ORDER BY s.creadoEn DESC
        """)
    Page<Solicitud> buscarConFiltros(
        @Param("estado") EstadoSolicitud estado,
        @Param("prioridad") Prioridad prioridad,
        @Param("tipoId") Long tipoId,
        Pageable pageable
    );
    
    // Con EntityGraph para evitar el problema N+1
    @EntityGraph(attributePaths = {"tipo", "solicitante", "responsable"})
    @Query("SELECT s FROM Solicitud s WHERE s.id = :id")
    Optional<Solicitud> findByIdConRelaciones(@Param("id") Long id);
    
    // Estadísticas útiles
    long countByEstado(EstadoSolicitud estado);
    boolean existsByDescripcionIgnoreCaseAndSolicitanteId(String descripcion, Long solicitanteId);
}
```

```java
// co/uniquindio/triage/repository/HistorialSolicitudRepository.java
@Repository
public interface HistorialSolicitudRepository extends JpaRepository<HistorialSolicitud, Long> {
    
    @Query("""
        SELECT h FROM HistorialSolicitud h
        LEFT JOIN FETCH h.usuario u
        WHERE h.solicitud.id = :solicitudId
        ORDER BY h.realizadoEn ASC
        """)
    List<HistorialSolicitud> findBySolicitudIdOrdenado(@Param("solicitudId") Long solicitudId);
}
```

```java
// co/uniquindio/triage/repository/UsuarioRepository.java
@Repository
public interface UsuarioRepository extends JpaRepository<Usuario, Long> {
    Optional<Usuario> findByEmail(String email);
    Optional<Usuario> findByIdentificacion(String identificacion);
    boolean existsByEmail(String email);
    List<Usuario> findByRolAndActivoTrue(Rol rol);
}
```

### 15.5 Capa de Servicio

```java
// co/uniquindio/triage/service/SolicitudService.java
public interface SolicitudService {
    SolicitudResponse crear(CrearSolicitudRequest request, Long solicitanteId);
    SolicitudDetalleResponse obtenerPorId(Long id);
    Page<SolicitudResponse> listar(EstadoSolicitud estado, Prioridad prioridad, Long tipoId, Pageable pageable);
    SolicitudResponse clasificar(Long id, ClasificarSolicitudRequest request, Long funcionarioId);
    SolicitudResponse asignarResponsable(Long id, AsignarResponsableRequest request, Long funcionarioId);
    SolicitudResponse cambiarEstado(Long id, CambiarEstadoRequest request, Long usuarioId);
    SolicitudResponse cerrar(Long id, CerrarSolicitudRequest request, Long funcionarioId);
    List<HistorialResponse> obtenerHistorial(Long solicitudId);
}
```

```java
// co/uniquindio/triage/service/impl/SolicitudServiceImpl.java
@Service
@Transactional
@Slf4j
public class SolicitudServiceImpl implements SolicitudService {
    
    private final SolicitudRepository solicitudRepository;
    private final HistorialSolicitudRepository historialRepository;
    private final TipoSolicitudRepository tipoSolicitudRepository;
    private final UsuarioRepository usuarioRepository;
    private final SolicitudMapper mapper;
    
    // Inyección por constructor — la forma recomendada en Spring Boot 3+
    public SolicitudServiceImpl(
            SolicitudRepository solicitudRepository,
            HistorialSolicitudRepository historialRepository,
            TipoSolicitudRepository tipoSolicitudRepository,
            UsuarioRepository usuarioRepository,
            SolicitudMapper mapper) {
        this.solicitudRepository = solicitudRepository;
        this.historialRepository = historialRepository;
        this.tipoSolicitudRepository = tipoSolicitudRepository;
        this.usuarioRepository = usuarioRepository;
        this.mapper = mapper;
    }
    
    // RF-01: Registrar solicitud
    @Override
    public SolicitudResponse crear(CrearSolicitudRequest request, Long solicitanteId) {
        log.info("Creando solicitud para usuario {} via canal {}", solicitanteId, request.canalOrigen());
        
        var solicitante = usuarioRepository.findById(solicitanteId)
            .orElseThrow(() -> new RecursoNoEncontradoException("Usuario", solicitanteId));
        
        var tipo = tipoSolicitudRepository.findById(request.tipoSolicitudId())
            .orElseThrow(() -> new RecursoNoEncontradoException("TipoSolicitud", request.tipoSolicitudId()));
        
        var solicitud = new Solicitud();
        solicitud.setDescripcion(request.descripcion());
        solicitud.setTipo(tipo);
        solicitud.setCanalOrigen(request.canalOrigen());
        solicitud.setSolicitante(solicitante);
        solicitud.setEstado(EstadoSolicitud.REGISTRADA);
        
        var guardada = solicitudRepository.save(solicitud);
        
        // Registrar en historial
        registrarHistorial(guardada, "REGISTRO", "Solicitud registrada por el estudiante", 
                           null, EstadoSolicitud.REGISTRADA, solicitante);
        
        log.info("Solicitud creada con id {} para usuario {}", guardada.getId(), solicitanteId);
        return mapper.toResponse(guardada);
    }
    
    // Obtener detalle con historial
    @Override
    @Transactional(readOnly = true)
    public SolicitudDetalleResponse obtenerPorId(Long id) {
        var solicitud = solicitudRepository.findByIdConRelaciones(id)
            .orElseThrow(() -> new RecursoNoEncontradoException("Solicitud", id));
        
        var historial = historialRepository.findBySolicitudIdOrdenado(id);
        
        return mapper.toDetalleResponse(solicitud, historial);
    }
    
    // RF-07: Listar con filtros y paginación
    @Override
    @Transactional(readOnly = true)
    public Page<SolicitudResponse> listar(EstadoSolicitud estado, Prioridad prioridad, 
                                           Long tipoId, Pageable pageable) {
        return solicitudRepository.buscarConFiltros(estado, prioridad, tipoId, pageable)
            .map(mapper::toResponse);
    }
    
    // RF-02, RF-03: Clasificar y priorizar
    @Override
    public SolicitudResponse clasificar(Long id, ClasificarSolicitudRequest request, Long funcionarioId) {
        var solicitud = obtenerSolicitudParaModificar(id);
        var funcionario = obtenerUsuarioActivo(funcionarioId);
        var tipo = tipoSolicitudRepository.findById(request.tipoSolicitudId())
            .orElseThrow(() -> new RecursoNoEncontradoException("TipoSolicitud", request.tipoSolicitudId()));
        
        // Validar que pueda clasificarse (solo REGISTRADA puede pasar a CLASIFICADA)
        if (!solicitud.getEstado().puedeTransicionarA(EstadoSolicitud.CLASIFICADA)) {
            throw new TransicionEstadoInvalidaException(solicitud.getEstado(), EstadoSolicitud.CLASIFICADA);
        }
        
        var estadoAnterior = solicitud.getEstado();
        solicitud.setTipo(tipo);
        solicitud.setPrioridad(request.prioridad());
        solicitud.setJustificacionPrioridad(request.justificacionPrioridad());
        solicitud.setEstado(EstadoSolicitud.CLASIFICADA);
        
        var actualizada = solicitudRepository.save(solicitud);
        
        registrarHistorial(actualizada,
            "CLASIFICACION",
            "Clasificada como '%s' con prioridad %s. Justificación: %s"
                .formatted(tipo.getNombre(), request.prioridad(), request.justificacionPrioridad()),
            estadoAnterior,
            EstadoSolicitud.CLASIFICADA,
            funcionario
        );
        
        return mapper.toResponse(actualizada);
    }
    
    // RF-05: Asignar responsable
    @Override
    public SolicitudResponse asignarResponsable(Long id, AsignarResponsableRequest request, Long funcionarioId) {
        var solicitud = obtenerSolicitudParaModificar(id);
        var funcionario = obtenerUsuarioActivo(funcionarioId);
        var responsable = usuarioRepository.findById(request.responsableId())
            .filter(Usuario::isActivo)
            .orElseThrow(() -> new RecursoNoEncontradoException(
                "No se encontró un responsable activo con id " + request.responsableId()));
        
        // La solicitud debe estar CLASIFICADA para asignar responsable
        if (solicitud.getEstado() != EstadoSolicitud.CLASIFICADA) {
            throw new OperacionNoPermitidaException(
                "Solo se puede asignar responsable a solicitudes en estado CLASIFICADA. " +
                "Estado actual: " + solicitud.getEstado());
        }
        
        var estadoAnterior = solicitud.getEstado();
        solicitud.setResponsable(responsable);
        solicitud.setEstado(EstadoSolicitud.EN_ATENCION);
        
        var actualizada = solicitudRepository.save(solicitud);
        
        registrarHistorial(actualizada,
            "ASIGNACION",
            "Solicitud asignada a %s. %s".formatted(
                responsable.getNombre(),
                Optional.ofNullable(request.observaciones()).orElse("")),
            estadoAnterior,
            EstadoSolicitud.EN_ATENCION,
            funcionario
        );
        
        return mapper.toResponse(actualizada);
    }
    
    // RF-04: Cambiar estado
    @Override
    public SolicitudResponse cambiarEstado(Long id, CambiarEstadoRequest request, Long usuarioId) {
        var solicitud = obtenerSolicitudParaModificar(id);
        var usuario = obtenerUsuarioActivo(usuarioId);
        
        if (!solicitud.getEstado().puedeTransicionarA(request.nuevoEstado())) {
            throw new TransicionEstadoInvalidaException(solicitud.getEstado(), request.nuevoEstado());
        }
        
        var estadoAnterior = solicitud.getEstado();
        solicitud.setEstado(request.nuevoEstado());
        
        var actualizada = solicitudRepository.save(solicitud);
        
        registrarHistorial(actualizada,
            "CAMBIO_ESTADO",
            Optional.ofNullable(request.observaciones()).orElse("Estado actualizado"),
            estadoAnterior,
            request.nuevoEstado(),
            usuario
        );
        
        return mapper.toResponse(actualizada);
    }
    
    // RF-08: Cerrar solicitud
    @Override
    public SolicitudResponse cerrar(Long id, CerrarSolicitudRequest request, Long funcionarioId) {
        var solicitud = obtenerSolicitudParaModificar(id);
        var funcionario = obtenerUsuarioActivo(funcionarioId);
        
        // Solo ATENDIDA puede cerrarse
        if (solicitud.getEstado() != EstadoSolicitud.ATENDIDA) {
            throw new OperacionNoPermitidaException(
                "Solo se pueden cerrar solicitudes en estado ATENDIDA. Estado actual: " + solicitud.getEstado());
        }
        
        var estadoAnterior = solicitud.getEstado();
        solicitud.setEstado(EstadoSolicitud.CERRADA);
        
        var actualizada = solicitudRepository.save(solicitud);
        
        registrarHistorial(actualizada,
            "CIERRE",
            request.observacionCierre(),
            estadoAnterior,
            EstadoSolicitud.CERRADA,
            funcionario
        );
        
        log.info("Solicitud {} cerrada por funcionario {}", id, funcionarioId);
        return mapper.toResponse(actualizada);
    }
    
    // RF-06: Historial auditable
    @Override
    @Transactional(readOnly = true)
    public List<HistorialResponse> obtenerHistorial(Long solicitudId) {
        if (!solicitudRepository.existsById(solicitudId)) {
            throw new RecursoNoEncontradoException("Solicitud", solicitudId);
        }
        return historialRepository.findBySolicitudIdOrdenado(solicitudId)
            .stream()
            .map(mapper::toHistorialResponse)
            .toList();
    }
    
    // ========== Métodos privados de apoyo ==========
    
    private Solicitud obtenerSolicitudParaModificar(Long id) {
        var solicitud = solicitudRepository.findByIdConRelaciones(id)
            .orElseThrow(() -> new RecursoNoEncontradoException("Solicitud", id));
        
        // Una solicitud cerrada no se puede modificar (RF-08)
        if (solicitud.getEstado() == EstadoSolicitud.CERRADA) {
            throw new SolicitudCerradaException(id);
        }
        
        return solicitud;
    }
    
    private Usuario obtenerUsuarioActivo(Long usuarioId) {
        return usuarioRepository.findById(usuarioId)
            .filter(Usuario::isActivo)
            .orElseThrow(() -> new RecursoNoEncontradoException("Usuario activo", usuarioId));
    }
    
    private void registrarHistorial(Solicitud solicitud, String accion, String observaciones,
                                     EstadoSolicitud estadoAnterior, EstadoSolicitud estadoNuevo,
                                     Usuario usuario) {
        var registro = new HistorialSolicitud();
        registro.setSolicitud(solicitud);
        registro.setAccion(accion);
        registro.setObservaciones(observaciones);
        registro.setEstadoAnterior(estadoAnterior);
        registro.setEstadoNuevo(estadoNuevo);
        registro.setUsuario(usuario);
        registro.setRealizadoEn(LocalDateTime.now());
        
        historialRepository.save(registro);
    }
}
```

### 15.6 Controladores Completos

```java
// co/uniquindio/triage/controller/SolicitudController.java
@RestController
@RequestMapping("/api/v1/solicitudes")
@Tag(name = "Solicitudes", description = "RF-01 al RF-08: Gestión del ciclo de vida de solicitudes académicas")
@Slf4j
public class SolicitudController {
    
    private final SolicitudService solicitudService;
    
    public SolicitudController(SolicitudService solicitudService) {
        this.solicitudService = solicitudService;
    }
    
    // =====================================================
    // GET — Consultas
    // =====================================================
    
    /**
     * RF-07: Consultar solicitudes con filtros y paginación
     * Ejemplos:
     *   GET /api/v1/solicitudes
     *   GET /api/v1/solicitudes?estado=EN_ATENCION&page=0&size=10
     *   GET /api/v1/solicitudes?prioridad=ALTA&sort=creadoEn,desc
     */
    @Operation(
        summary = "Listar solicitudes",
        description = "Retorna solicitudes paginadas con filtros opcionales por estado, prioridad y tipo"
    )
    @ApiResponse(responseCode = "200", description = "Lista obtenida correctamente")
    @ApiResponse(responseCode = "401", description = "No autenticado")
    @GetMapping
    public ResponseEntity<Page<SolicitudResponse>> listar(
            @Parameter(description = "Filtrar por estado")
            @RequestParam(required = false) EstadoSolicitud estado,
            
            @Parameter(description = "Filtrar por prioridad")
            @RequestParam(required = false) Prioridad prioridad,
            
            @Parameter(description = "Filtrar por tipo de solicitud (ID)")
            @RequestParam(required = false) Long tipoId,
            
            @Parameter(description = "Paginación: ?page=0&size=10&sort=creadoEn,desc")
            Pageable pageable) {
        
        // TODO Guía 07: extraer solicitanteId del JWT para filtrar por usuario si rol = ESTUDIANTE
        log.debug("Listando solicitudes — estado: {}, prioridad: {}, tipoId: {}", estado, prioridad, tipoId);
        
        return ResponseEntity.ok(solicitudService.listar(estado, prioridad, tipoId, pageable));
    }
    
    /**
     * Obtener el detalle completo de una solicitud con su historial
     */
    @Operation(summary = "Obtener detalle de solicitud", description = "Retorna todos los datos de la solicitud incluyendo historial completo")
    @ApiResponse(responseCode = "200", description = "Solicitud encontrada")
    @ApiResponse(responseCode = "404", description = "Solicitud no encontrada")
    @GetMapping("/{id}")
    public ResponseEntity<SolicitudDetalleResponse> obtenerPorId(
            @Parameter(description = "ID de la solicitud", required = true, example = "42")
            @PathVariable Long id) {
        
        return ResponseEntity.ok(solicitudService.obtenerPorId(id));
    }
    
    /**
     * RF-06: Historial auditable de una solicitud
     */
    @Operation(summary = "Ver historial de solicitud", description = "RF-06: Retorna el historial auditable completo de acciones realizadas sobre la solicitud")
    @ApiResponse(responseCode = "200", description = "Historial obtenido")
    @ApiResponse(responseCode = "404", description = "Solicitud no encontrada")
    @GetMapping("/{id}/historial")
    public ResponseEntity<List<HistorialResponse>> obtenerHistorial(@PathVariable Long id) {
        return ResponseEntity.ok(solicitudService.obtenerHistorial(id));
    }
    
    // =====================================================
    // POST — Crear y acciones
    // =====================================================
    
    /**
     * RF-01: Registrar nueva solicitud académica
     */
    @Operation(summary = "Registrar solicitud", description = "RF-01: Crea una nueva solicitud académica. El solicitante se toma del token JWT.")
    @ApiResponse(responseCode = "201", description = "Solicitud creada. El header Location indica la URL del nuevo recurso.")
    @ApiResponse(responseCode = "400", description = "Datos de entrada inválidos")
    @PostMapping
    public ResponseEntity<SolicitudResponse> crear(
            @Valid @RequestBody CrearSolicitudRequest request) {
        
        // TODO Guía 07: extraer solicitanteId del SecurityContextHolder (JWT)
        Long solicitanteId = 1L; // temporal hasta implementar seguridad
        
        log.info("Creando solicitud para usuario {}", solicitanteId);
        
        var creada = solicitudService.crear(request, solicitanteId);
        var location = URI.create("/api/v1/solicitudes/" + creada.id());
        
        return ResponseEntity.created(location).body(creada);
    }
    
    // =====================================================
    // PATCH — Modificaciones parciales
    // =====================================================
    
    /**
     * RF-02, RF-03: Clasificar solicitud y asignar prioridad
     */
    @Operation(
        summary = "Clasificar y priorizar solicitud",
        description = "RF-02/RF-03: Clasifica una solicitud asignándole tipo y prioridad con justificación. Solo FUNCIONARIO y ADMIN."
    )
    @ApiResponse(responseCode = "200", description = "Solicitud clasificada correctamente")
    @ApiResponse(responseCode = "404", description = "Solicitud no encontrada")
    @ApiResponse(responseCode = "409", description = "La transición de estado no es válida")
    @PatchMapping("/{id}/clasificar")
    public ResponseEntity<SolicitudResponse> clasificar(
            @PathVariable Long id,
            @Valid @RequestBody ClasificarSolicitudRequest request) {
        
        Long funcionarioId = 2L; // TODO: extraer del JWT
        
        return ResponseEntity.ok(solicitudService.clasificar(id, request, funcionarioId));
    }
    
    /**
     * RF-05: Asignar responsable a una solicitud
     */
    @Operation(summary = "Asignar responsable", description = "RF-05: Asigna un funcionario responsable de atender la solicitud. Cambia el estado a EN_ATENCION.")
    @ApiResponse(responseCode = "200", description = "Responsable asignado correctamente")
    @ApiResponse(responseCode = "404", description = "Solicitud o responsable no encontrado")
    @PatchMapping("/{id}/asignar")
    public ResponseEntity<SolicitudResponse> asignarResponsable(
            @PathVariable Long id,
            @Valid @RequestBody AsignarResponsableRequest request) {
        
        Long funcionarioId = 2L; // TODO: extraer del JWT
        
        return ResponseEntity.ok(solicitudService.asignarResponsable(id, request, funcionarioId));
    }
    
    /**
     * RF-04: Cambiar el estado de una solicitud
     */
    @Operation(
        summary = "Cambiar estado de solicitud",
        description = "RF-04: Permite transitar entre estados válidos del ciclo de vida. El sistema valida que la transición sea coherente."
    )
    @ApiResponse(responseCode = "200", description = "Estado actualizado correctamente")
    @ApiResponse(responseCode = "409", description = "Transición de estado no permitida. Revisa el flujo: REGISTRADA→CLASIFICADA→EN_ATENCION→ATENDIDA→CERRADA")
    @PatchMapping("/{id}/estado")
    public ResponseEntity<SolicitudResponse> cambiarEstado(
            @PathVariable Long id,
            @Valid @RequestBody CambiarEstadoRequest request) {
        
        Long usuarioId = 2L; // TODO: extraer del JWT
        
        return ResponseEntity.ok(solicitudService.cambiarEstado(id, request, usuarioId));
    }
    
    // =====================================================
    // POST especial — Cerrar (acción con semántica propia)
    // =====================================================
    
    /**
     * RF-08: Cerrar solicitud
     */
    @Operation(
        summary = "Cerrar solicitud",
        description = "RF-08: Cierra definitivamente una solicitud ATENDIDA. Una solicitud cerrada no puede modificarse."
    )
    @ApiResponse(responseCode = "200", description = "Solicitud cerrada correctamente")
    @ApiResponse(responseCode = "409", description = "La solicitud no está en estado ATENDIDA o ya está cerrada")
    @PostMapping("/{id}/cerrar")
    public ResponseEntity<SolicitudResponse> cerrar(
            @PathVariable Long id,
            @Valid @RequestBody CerrarSolicitudRequest request) {
        
        Long funcionarioId = 2L; // TODO: extraer del JWT
        
        return ResponseEntity.ok(solicitudService.cerrar(id, request, funcionarioId));
    }
}
```

### Script de Migración Flyway

```sql
-- src/main/resources/db/migration/V1__crear_tablas_base.sql

CREATE TABLE IF NOT EXISTS tipos_solicitud (
    id          BIGSERIAL PRIMARY KEY,
    nombre      VARCHAR(100) NOT NULL UNIQUE,
    descripcion VARCHAR(500),
    activo      BOOLEAN NOT NULL DEFAULT true
);

CREATE TABLE IF NOT EXISTS usuarios (
    id              BIGSERIAL PRIMARY KEY,
    nombre          VARCHAR(150) NOT NULL,
    email           VARCHAR(200) NOT NULL UNIQUE,
    identificacion  VARCHAR(20) NOT NULL UNIQUE,
    password_hash   VARCHAR(255) NOT NULL,
    rol             VARCHAR(20) NOT NULL CHECK (rol IN ('ESTUDIANTE', 'FUNCIONARIO', 'ADMIN')),
    activo          BOOLEAN NOT NULL DEFAULT true,
    creado_en       TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS solicitudes (
    id                      BIGSERIAL PRIMARY KEY,
    descripcion             VARCHAR(1000) NOT NULL,
    tipo_solicitud_id       BIGINT REFERENCES tipos_solicitud(id),
    prioridad               VARCHAR(20) CHECK (prioridad IN ('BAJA', 'MEDIA', 'ALTA', 'CRITICA')),
    justificacion_prioridad VARCHAR(500),
    estado                  VARCHAR(20) NOT NULL DEFAULT 'REGISTRADA'
                                CHECK (estado IN ('REGISTRADA','CLASIFICADA','EN_ATENCION','ATENDIDA','CERRADA')),
    canal_origen            VARCHAR(30) NOT NULL
                                CHECK (canal_origen IN ('PRESENCIAL','CORREO_ELECTRONICO','SAC','TELEFONICO','PLATAFORMA_WEB')),
    solicitante_id          BIGINT NOT NULL REFERENCES usuarios(id),
    responsable_id          BIGINT REFERENCES usuarios(id),
    creado_en               TIMESTAMP NOT NULL DEFAULT NOW(),
    actualizado_en          TIMESTAMP
);

CREATE TABLE IF NOT EXISTS historial_solicitudes (
    id              BIGSERIAL PRIMARY KEY,
    solicitud_id    BIGINT NOT NULL REFERENCES solicitudes(id) ON DELETE CASCADE,
    accion          VARCHAR(200) NOT NULL,
    observaciones   VARCHAR(1000),
    estado_anterior VARCHAR(20),
    estado_nuevo    VARCHAR(20),
    usuario_id      BIGINT NOT NULL REFERENCES usuarios(id),
    realizado_en    TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Índices para búsquedas frecuentes
CREATE INDEX idx_solicitudes_estado       ON solicitudes(estado);
CREATE INDEX idx_solicitudes_prioridad    ON solicitudes(prioridad);
CREATE INDEX idx_solicitudes_solicitante  ON solicitudes(solicitante_id);
CREATE INDEX idx_solicitudes_responsable  ON solicitudes(responsable_id);
CREATE INDEX idx_historial_solicitud      ON historial_solicitudes(solicitud_id);
```

```sql
-- src/main/resources/db/migration/V2__datos_iniciales.sql

-- Tipos de solicitud del sistema académico
INSERT INTO tipos_solicitud (nombre, descripcion) VALUES
    ('Registro de Asignaturas',   'Solicitud para registrar o modificar asignaturas del semestre'),
    ('Homologación',              'Solicitud para homologar asignaturas de otras instituciones'),
    ('Cancelación de Asignaturas','Solicitud para cancelar una o más asignaturas del semestre'),
    ('Solicitud de Cupos',        'Solicitud para obtener cupo en una asignatura con cupos agotados'),
    ('Consulta Académica',        'Consulta general sobre procedimientos o normativa académica'),
    ('Paz y Salvo',               'Solicitud de certificado de paz y salvo académico y financiero'),
    ('Transferencia',             'Solicitud de transferencia interna o externa');

-- Usuario admin del sistema
INSERT INTO usuarios (nombre, email, identificacion, password_hash, rol) VALUES
    ('Admin Sistema', 'admin@uniquindio.edu.co', '00000000',
     '$2a$10$placeholder.hash.del.admin', 'ADMIN'),
    ('Secretaria Académica', 'secretaria@uniquindio.edu.co', '11111111',
     '$2a$10$placeholder.hash.secretaria', 'FUNCIONARIO'),
    ('Estudiante Demo', 'demo@uniquindio.edu.co', '1094123456',
     '$2a$10$placeholder.hash.demo', 'ESTUDIANTE');
```

### Configuración `application.yml`

```yaml
# src/main/resources/application.yml
spring:
  application:
    name: triage-backend

  datasource:
    url: jdbc:postgresql://localhost:5432/triage_db
    username: triage_user
    password: triage_pass
    driver-class-name: org.postgresql.Driver

  jpa:
    hibernate:
      ddl-auto: validate          # Flyway gestiona el schema — JPA solo valida
    show-sql: false               # true en desarrollo para ver las queries
    open-in-view: false           # IMPORTANTE: desactivar para evitar el anti-patrón OSIV
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.PostgreSQLDialect
        default_schema: public

  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true

  mvc:
    problemdetails:
      enabled: true               # Habilita RFC 7807 nativo

  data:
    web:
      pageable:
        default-page-size: 10     # Tamaño de página por defecto
        max-page-size: 100        # Máximo permitido
        one-indexed-parameters: false # Las páginas empiezan en 0

server:
  port: 8080
  error:
    include-message: never        # No filtrar mensajes de error en producción
    include-stacktrace: never

# SpringDoc / Swagger
springdoc:
  api-docs:
    path: /api-docs
  swagger-ui:
    path: /swagger-ui.html
    tags-sorter: alpha

logging:
  level:
    co.uniquindio.triage: DEBUG
    org.springframework.web: INFO
    org.hibernate.SQL: DEBUG       # Muestra las queries SQL en consola
```

---

## 16. Antes vs Ahora: `RestTemplate` vs `WebClient`

Cuando tu backend necesita **consumir otra API** (por ejemplo, un servicio externo de notificaciones, o la API de la IA de la Guía 12), tienes dos opciones:

| Característica | `RestTemplate` | `WebClient` |
|---------------|----------------|-------------|
| Estado | ⚠️ Deprecado en Spring 6 | ✅ Activo, recomendado |
| Modelo | Síncrono (blocking) | Asíncrono y reactivo |
| Requiere | Solo `spring-web` | `spring-webflux` |
| API | Imperativa, fácil de leer | Funcional, fluent API |
| Errores | Excepciones simples | `onStatus()`, `bodyToMono()` |
| Cuándo usar | ❌ No usar en proyectos nuevos | ✅ Siempre para proyectos nuevos |

**Antes — RestTemplate (obsoleto):**

```java
// Forma antigua — síncrona y deprecada
@Autowired
private RestTemplate restTemplate;

public SolicitudExternaResponse consultar(Long id) {
    String url = "https://api.externa.edu.co/solicitudes/" + id;
    return restTemplate.getForObject(url, SolicitudExternaResponse.class);
}
```

**Ahora — WebClient (moderno):**

```java
// Agregar dependencia en pom.xml:
// <dependency>
//   <groupId>org.springframework.boot</groupId>
//   <artifactId>spring-boot-starter-webflux</artifactId>
// </dependency>

@Service
public class NotificacionService {
    
    private final WebClient webClient;
    
    public NotificacionService(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder
            .baseUrl("https://api.notificaciones.uniquindio.edu.co")
            .defaultHeader("Content-Type", "application/json")
            .build();
    }
    
    // Llamada asíncrona — no bloquea el hilo
    public Mono<Void> notificarCambioEstado(Long solicitudId, String nuevoEstado) {
        return webClient.post()
            .uri("/notificaciones")
            .bodyValue(new NotificacionRequest(solicitudId, nuevoEstado))
            .retrieve()
            .onStatus(HttpStatusCode::is4xxClientError, response ->
                Mono.error(new RuntimeException("Error al enviar notificación: " + response.statusCode())))
            .onStatus(HttpStatusCode::is5xxServerError, response ->
                Mono.error(new RuntimeException("Servicio de notificaciones no disponible")))
            .bodyToMono(Void.class)
            .timeout(Duration.ofSeconds(5)) // timeout de seguridad
            .onErrorResume(ex -> {
                log.warn("No se pudo enviar notificación para solicitud {}: {}", solicitudId, ex.getMessage());
                return Mono.empty(); // la falla de notificación no bloquea la operación principal
            });
    }
}
```

---

## 17. Probar la API con Swagger UI y Postman

### Probar con Swagger UI

1. Ejecuta el proyecto: `mvn spring-boot:run`
2. Abre el navegador en `http://localhost:8080/swagger-ui.html`
3. Expande el grupo "Solicitudes"
4. Haz clic en `POST /api/v1/solicitudes → Try it out`
5. Ingresa el body:

```json
{
  "descripcion": "Solicito homologación de la asignatura Programación I cursada en la UNAL en el período 2024-1",
  "tipoSolicitudId": 2,
  "canalOrigen": "PLATAFORMA_WEB"
}
```

6. Haz clic en "Execute"
7. Verás el response con status 201 y el objeto creado

### Colección Postman del Triage

```
POST   http://localhost:8080/api/v1/solicitudes
Content-Type: application/json

{
  "descripcion": "Solicito homologación de Programación I cursada en UNAL 2024-1",
  "tipoSolicitudId": 2,
  "canalOrigen": "CORREO_ELECTRONICO"
}

---

GET    http://localhost:8080/api/v1/solicitudes?estado=REGISTRADA&page=0&size=5

---

GET    http://localhost:8080/api/v1/solicitudes/1

---

PATCH  http://localhost:8080/api/v1/solicitudes/1/clasificar
Content-Type: application/json

{
  "tipoSolicitudId": 2,
  "prioridad": "ALTA",
  "justificacionPrioridad": "Impacta directamente la carga académica del semestre en curso"
}

---

PATCH  http://localhost:8080/api/v1/solicitudes/1/asignar
Content-Type: application/json

{
  "responsableId": 2,
  "observaciones": "Asignado a secretaria para revisión de documentos"
}

---

PATCH  http://localhost:8080/api/v1/solicitudes/1/estado
Content-Type: application/json

{
  "nuevoEstado": "ATENDIDA",
  "observaciones": "Homologación aprobada. Se generó la resolución 2026-045."
}

---

POST   http://localhost:8080/api/v1/solicitudes/1/cerrar
Content-Type: application/json

{
  "observacionCierre": "Proceso completado. El estudiante fue notificado sobre la aprobación."
}

---

GET    http://localhost:8080/api/v1/solicitudes/1/historial
```

---

## 18. Ejercicios Prácticos

### Reto 1 — Endpoint de estadísticas (Básico)

Implementa el endpoint `GET /api/v1/solicitudes/estadisticas` que devuelva:

```json
{
  "totalSolicitudes": 157,
  "porEstado": {
    "REGISTRADA": 23,
    "CLASIFICADA": 14,
    "EN_ATENCION": 45,
    "ATENDIDA": 38,
    "CERRADA": 37
  },
  "porPrioridad": {
    "BAJA": 40,
    "MEDIA": 67,
    "ALTA": 38,
    "CRITICA": 12
  }
}
```

Crea el Record `EstadisticasResponse`, el método en el servicio y el repositorio. Usa `@Query` con GROUP BY o múltiples llamadas a `countByEstado()`.

### Reto 2 — Búsqueda por texto (Intermedio)

Agrega a `GET /api/v1/solicitudes` un parámetro `q` para búsqueda de texto libre en la descripción:

```
GET /api/v1/solicitudes?q=homologaci%C3%B3n&estado=REGISTRADA
```

Implementa la búsqueda con `ILIKE` (case-insensitive en PostgreSQL) usando `@Query` con JPQL o Specification.

### Reto 3 — Endpoint de reasignación (Intermedio)

El FUNCIONARIO debe poder reasignar una solicitud que ya tiene responsable asignado:

```
PATCH /api/v1/solicitudes/{id}/reasignar
```

El nuevo responsable reemplaza al anterior. El historial debe registrar tanto el responsable anterior como el nuevo. Lanza `OperacionNoPermitidaException` si la solicitud está CERRADA o no tiene responsable previo.

### Reto 4 — Validación cruzada (Avanzado)

Implementa una validación personalizada `@TipoSolicitudActivo` que verifique en la base de datos que el `tipoSolicitudId` enviado en `CrearSolicitudRequest` corresponde a un tipo que existe Y está activo (`activo = true`). Si no existe o está inactivo, debe devolver un error 400 con el mensaje "El tipo de solicitud seleccionado no está disponible".

### Reto 5 — Paginación avanzada (Avanzado)

Implementa `GET /api/v1/solicitudes/mis-solicitudes` para que un estudiante vea solo sus propias solicitudes. Por ahora usa un `@RequestParam Long solicitanteId` (en la Guía 07 esto se extraerá del JWT). Ordena por prioridad descendente (CRITICA primero) y luego por fecha ascendente.

---

## 19. Errores Comunes y Troubleshooting

### Error: `org.hibernate.LazyInitializationException`

**Síntoma:** Al serializar a JSON, se lanza esta excepción sobre una colección o relación lazy.

**Causa:** Estás accediendo a una relación `FetchType.LAZY` fuera de la transacción activa (después del `@Transactional`).

**Solución:**
```java
// ❌ Mal: acceder al lazy fuera de la transacción
Solicitud solicitud = repository.findById(1L).get();
// ... transacción termina aquí ...
solicitud.getHistorial().size(); // LazyInitializationException!

// ✅ Bien 1: usar @EntityGraph para cargar todo en la transacción
@EntityGraph(attributePaths = {"historial", "tipo", "solicitante"})
Optional<Solicitud> findByIdConRelaciones(Long id);

// ✅ Bien 2: convertir a DTO dentro de la transacción (en el @Service)
// El mapper accede a las relaciones DENTRO de la transacción activa
```

**Nunca** uses `spring.jpa.open-in-view=true` como solución — este antipatrón mantiene la conexión a BD abierta durante toda la petición HTTP.

### Error: `HttpMessageNotWritableException: No converter found for return value`

**Síntoma:** Al retornar una entidad JPA con relaciones bidireccionales, Jackson entra en un loop infinito.

**Causa:** Estás exponiendo la entidad directamente, no un DTO.

**Solución:** Nunca retornes entidades JPA desde el controlador. Siempre usa DTOs (Records).

### Error: `400 Bad Request` sin mensaje claro

**Síntoma:** Recibes 400 pero no sabes qué campo falló.

**Causa:** No tienes configurado el `@ControllerAdvice` para `MethodArgumentNotValidException`, o Spring Boot 3 con `problemdetails.enabled=true` no está activo.

**Solución:** Implementa el `GlobalExceptionHandler` de la sección 11 y asegúrate de tener en `application.yml`:
```yaml
spring:
  mvc:
    problemdetails:
      enabled: true
```

### Error: CORS bloqueado desde Angular

**Síntoma:** En la consola del navegador: `Access to XMLHttpRequest at 'http://localhost:8080/...' from origin 'http://localhost:4200' has been blocked by CORS policy`.

**Causa 1:** No tienes configurado CORS en Spring Boot.
**Solución 1:** Implementar el `CorsConfig` de la sección 13.

**Causa 2:** Tienes Spring Security activo pero no integraste CORS con el `SecurityFilterChain`.
**Solución 2:** Ver Guía 07 — la configuración de CORS debe hacerse a través del `SecurityFilterChain`, no como bean separado, cuando Security está activo.

### Error: `No primary or single unique constructor found for interface Pageable`

**Síntoma:** Al tener `Pageable pageable` en el controlador, Spring no puede crear el objeto.

**Causa:** Falta la configuración de Spring Data Web.

**Solución:** Asegúrate de tener en `pom.xml`:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```
Spring Data JPA incluye automáticamente el soporte de `Pageable` en los controladores.

### Error: `Record cannot be deserialized` con Jackson

**Síntoma:** Jackson no puede deserializar un Record desde JSON.

**Causa:** Versión de Jackson anterior a 2.12, o falta la configuración.

**Solución:** Spring Boot 3.4+ incluye Jackson 2.17+, que soporta Records nativamente. Si tienes problemas:
```java
// Agregar explícitamente en el Record si es necesario
public record CrearSolicitudRequest(
    @JsonProperty("descripcion") String descripcion,
    @JsonProperty("tipoSolicitudId") Long tipoSolicitudId
) {}
```

---

## 20. Resumen y Cheat Sheet

```
=== DISEÑO DE API ===
✓ Sustantivos en plural: /solicitudes, /usuarios, /tipos-solicitud
✓ Verbos HTTP correctos: GET leer, POST crear, PUT reemplazar, PATCH parcial, DELETE eliminar
✓ Versionado en URL: /api/v1/...
✓ Filtros como query params: ?estado=REGISTRADA&prioridad=ALTA
✓ Sub-recursos: /solicitudes/{id}/historial
✓ Acciones: POST /solicitudes/{id}/cerrar, POST /solicitudes/{id}/asignar

=== CÓDIGOS DE ESTADO ===
200 OK          → GET, PUT, PATCH exitosos
201 Created     → POST exitoso (incluir header Location)
204 No Content  → DELETE exitoso
400 Bad Request → Validación fallida
401 Unauthorized → No autenticado (falta JWT)
403 Forbidden   → Autenticado pero sin permiso
404 Not Found   → Recurso inexistente
409 Conflict    → Conflicto de estado (ej: transición inválida)
500 Internal Server Error → Error inesperado del servidor

=== ANOTACIONES CLAVE ===
@RestController        → clase es controlador REST
@RequestMapping("/x")  → prefijo de URL para la clase
@GetMapping("/y")      → método HTTP GET
@PostMapping("/y")     → método HTTP POST
@PatchMapping("/{id}") → método HTTP PATCH
@DeleteMapping("/{id}")→ método HTTP DELETE
@PathVariable Long id  → variable de la URL /solicitudes/{id}
@RequestParam String q → query param ?q=valor
@RequestBody @Valid X  → deserializa JSON + valida
ResponseEntity.ok(x)   → 200 con cuerpo
ResponseEntity.created(uri).body(x) → 201 con Location

=== DTOs CON RECORDS (Java 21) ===
public record SolicitudResponse(Long id, String estado, ...) {}
// Jackson lo serializa/deserializa automáticamente
// Campos accesibles: solicitud.id(), solicitud.estado()
// Inmutable, equals/hashCode/toString automáticos

=== VALIDACIONES ===
@NotBlank    → no null, no vacío, no solo espacios
@NotNull     → no null
@Size(min,max) → tamaño entre min y max
@Email       → formato email válido
@Pattern     → expresión regular
@Valid en el método → activa la validación del @RequestBody

=== MANEJO DE ERRORES ===
@RestControllerAdvice  → clase interceptora de excepciones
@ExceptionHandler(X.class) → método que maneja excepción X
ProblemDetail.forStatusAndDetail(status, mensaje) → RFC 7807

=== PAGINACIÓN ===
Pageable pageable       → recibe ?page=0&size=10&sort=campo,asc
Page<T> resultado       → incluye content, totalElements, totalPages
repository.findAll(pageable) → paginación automática

=== CORS ===
Config en CorsConfig.java
Allowed origins: localhost:4200 (Angular dev)
Allowed methods: GET, POST, PUT, PATCH, DELETE, OPTIONS
Allowed headers: Authorization, Content-Type
Allow credentials: true (para JWT)
```

---

## 21. Referencias y Recursos Adicionales

- [Spring Boot REST Documentation — spring.io](https://spring.io/guides/tutorials/rest/)
- [SpringDoc OpenAPI — springdoc.org](https://springdoc.org/)
- [RFC 7807 Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc7807)
- [Bean Validation — jakarta.ee](https://jakarta.ee/specifications/bean-validation/3.0/jakarta-bean-validation-spec-3.0.html)
- [Java Records — OpenJDK](https://openjdk.org/jeps/395)
- **Libro recomendado:** Sharma, Sourabh (2021). *Modern API Development with Spring and Spring Boot*. Packt Publishing.
- **Libro recomendado:** Richardson, Leonard (2014). *RESTful Web APIs: Services for a Changing World*. O'Reilly Media.

---

> **Autor:** José Alfredo Ramírez Espinosa  
> **Materia:** Programación Avanzada  
> **Programa:** Ingeniería de Sistemas y Computación  
> **Universidad del Quindío** | Armenia, Colombia  
> **Año:** 2026  
> *Material de uso académico. Todos los derechos reservados al autor.*
