# Guía 01 — Arquitecturas de Software: De Monolito a Microservicios

> **Materia:** Programación Avanzada  
> **Programa:** Ingeniería de Sistemas y Computación — Universidad del Quindío  
> **Semestre:** 6to | **Núcleo Temático:** 1. Introducción  
> **Versión:** 2026-1

---

## Tabla de Contenidos

1. [Objetivos de Aprendizaje](#1-objetivos-de-aprendizaje)
2. [Prerrequisitos](#2-prerrequisitos)
3. [¿Qué es una Arquitectura de Software?](#3-qué-es-una-arquitectura-de-software)
4. [Arquitectura Monolítica](#4-arquitectura-monolítica)
5. [Arquitectura Cliente-Servidor](#5-arquitectura-cliente-servidor)
6. [Arquitectura en Capas (N-Tier)](#6-arquitectura-en-capas-n-tier)
7. [Arquitectura Hexagonal (Puertos y Adaptadores)](#7-arquitectura-hexagonal-puertos-y-adaptadores)
8. [Arquitectura Limpia (Clean Architecture)](#8-arquitectura-limpia-clean-architecture)
9. [Microservicios](#9-microservicios)
10. [Tabla Comparativa de Arquitecturas](#10-tabla-comparativa-de-arquitecturas)
11. [La Arquitectura que Usaremos en el Proyecto de Triage](#11-la-arquitectura-que-usaremos-en-el-proyecto-de-triage)
12. [Estructura de Carpetas Completa del Proyecto de Triage](#12-estructura-de-carpetas-completa-del-proyecto-de-triage)
13. [Ejercicios Prácticos](#13-ejercicios-prácticos)
14. [Errores Comunes y Troubleshooting](#14-errores-comunes-y-troubleshooting)
15. [Resumen y Cheat Sheet](#15-resumen-y-cheat-sheet)
16. [Referencias y Recursos Adicionales](#16-referencias-y-recursos-adicionales)

---

## 1. Objetivos de Aprendizaje

Al finalizar esta guía, serás capaz de:

- Explicar qué es una arquitectura de software y por qué es la decisión más importante en un proyecto
- Identificar las características, ventajas y desventajas de las arquitecturas monolítica, cliente-servidor, en capas, hexagonal, limpia y de microservicios
- Leer y entender un diagrama de arquitectura
- Trazar el flujo completo de datos en una arquitectura en capas
- Justificar por qué el proyecto del semestre (Sistema de Triage) usará una arquitectura específica
- Crear la estructura de carpetas profesional del proyecto de Triage para backend y frontend

---

## 2. Prerrequisitos

Antes de empezar, asegúrate de tener claro lo siguiente:

- ✅ Programación orientada a objetos en Java: clases, herencia, polimorfismo, interfaces
- ✅ Colecciones de Java: `List`, `Map`, `Set`
- ✅ Qué es una base de datos relacional (aunque no sepas usar una todavía)
- ✅ Qué es HTTP de manera general (lo que pasa cuando abres una página web)

> **No necesitas** saber Spring Boot, Angular, ni ningún framework. Esta guía es el punto de partida.

---

## 3. ¿Qué es una Arquitectura de Software?

### 3.1 La analogía del edificio

Imagina que vas a construir una casa. Antes de poner un solo ladrillo, necesitas un **plano arquitectónico** que defina:

- Cuántos pisos va a tener
- Cómo se distribuyen los espacios (sala, cocina, habitaciones)
- Por dónde van a pasar los cables eléctricos y las tuberías
- Cómo se comunican los espacios entre sí (pasillos, puertas, escaleras)

Si construyes sin plano, al principio puede parecer que avanzas rápido. Pero cuando quieras agregar una habitación o cambiar la cocina de lugar, todo se va a complicar enormemente.

**En software pasa exactamente lo mismo.**

Una **arquitectura de software** es el conjunto de decisiones fundamentales sobre cómo se va a organizar y estructurar un sistema de software. Estas decisiones definen:

- Cómo se dividen las responsabilidades entre los distintos componentes del sistema
- Cómo se comunican esos componentes entre sí
- Qué reglas deben seguirse para que el sistema sea mantenible, escalable y comprensible

### 3.2 ¿Por qué importa tanto?

Aquí un dato que te va a quedar grabado: **el 70% del costo de un software es el mantenimiento**, no el desarrollo inicial. Esto significa que la arquitectura no se evalúa cuando terminas el proyecto, sino 2, 3 o 5 años después, cuando alguien tiene que agregar funcionalidades o corregir errores.

Una mala arquitectura genera lo que se conoce como **deuda técnica**: es como pedir prestado tiempo ahora a costa de pagar intereses después. Cuanto más tiempo pasa sin pagar esa deuda, más caro se vuelve.

### 3.3 Atributos de calidad que una buena arquitectura debe garantizar

| Atributo | Definición | Ejemplo en el Triage |
|---|---|---|
| **Mantenibilidad** | Qué tan fácil es cambiar el sistema sin romper otras partes | Agregar un nuevo tipo de solicitud sin modificar la capa de presentación |
| **Escalabilidad** | Qué tan fácil es crecer cuando hay más usuarios o datos | Soportar 1.400 estudiantes enviando solicitudes al mismo tiempo |
| **Testeabilidad** | Qué tan fácil es probar cada parte de forma aislada | Probar la lógica de priorización sin necesidad de la base de datos |
| **Seguridad** | Qué tan protegido está el sistema de accesos no autorizados | Solo un FUNCIONARIO puede cambiar el estado de una solicitud |
| **Rendimiento** | Qué tan rápido responde el sistema | Devolver la lista de solicitudes filtradas en menos de 500ms |
| **Desacoplamiento** | Qué tan independientes son los componentes entre sí | Cambiar de PostgreSQL a MySQL sin tocar la lógica de negocio |

### 3.4 Arquitectura vs. diseño vs. implementación

Es común confundir estos tres niveles. Aquí la diferencia:

```
ARQUITECTURA  →  Las decisiones grandes: ¿monolito o microservicios? ¿qué tecnologías?
                 ¿cómo se comunican las capas? ¿dónde vive la lógica de negocio?

DISEÑO        →  Las decisiones medianas: ¿qué patrones de diseño usar?
                 ¿cómo modelar las clases? ¿qué interfaces necesitamos?

IMPLEMENTACIÓN → Las decisiones pequeñas: ¿usar un for o un stream?
                 ¿cuál es el mejor nombre para esta variable?
```

Esta guía se ocupa del **primer nivel**. Las siguientes guías irán bajando al diseño y luego a la implementación.

---

## 4. Arquitectura Monolítica

### 4.1 ¿Qué es?

Una arquitectura monolítica es aquella donde **toda la aplicación se despliega como una sola unidad**. Todo el código — la interfaz de usuario, la lógica de negocio, el acceso a datos — vive en un único proceso ejecutable.

Piensa en ello como un edificio donde la cocina, la sala y las habitaciones están todas en un cuarto gigante sin paredes divisorias. Funciona, pero si quieres arreglar la cocina, terminas afectando todo lo demás.

### 4.2 Diagrama de un monolito

```
┌─────────────────────────────────────────────────────┐
│                   APLICACIÓN MONOLÍTICA              │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │           INTERFAZ DE USUARIO               │   │
│  │  (HTML, CSS, JS mezclado con Java en JSP)   │   │
│  └─────────────────────────────────────────────┘   │
│                         │                           │
│  ┌─────────────────────────────────────────────┐   │
│  │           LÓGICA DE NEGOCIO                 │   │
│  │  (Reglas de priorización, validaciones,      │   │
│  │   gestión de estados de solicitudes)        │   │
│  └─────────────────────────────────────────────┘   │
│                         │                           │
│  ┌─────────────────────────────────────────────┐   │
│  │           ACCESO A DATOS                    │   │
│  │  (SQL directo, JDBC, todo mezclado)         │   │
│  └─────────────────────────────────────────────┘   │
│                         │                           │
│              ┌──────────┴──────────┐               │
│              │    BASE DE DATOS    │               │
│              └────────────────────┘               │
└─────────────────────────────────────────────────────┘
                  Un solo proceso, un solo despliegue
```

### 4.3 Ejemplo de cómo se ve el código en un monolito Java

En un monolito desorganizado, es habitual encontrar cosas como esta:

```java
// ⚠️ EJEMPLO DE LO QUE NO SE DEBE HACER - Código de un monolito desorganizado
// Un Servlet que hace TODO: validar, acceder a la BD, construir HTML
public class SolicitudServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) 
            throws ServletException, IOException {
        
        // Leer datos del formulario
        String descripcion = req.getParameter("descripcion");
        String tipoSolicitud = req.getParameter("tipo");
        String estudianteId = req.getParameter("estudianteId");
        
        // Validar (lógica de negocio mezclada con presentación)
        if (descripcion == null || descripcion.isBlank()) {
            resp.getWriter().println("<html><body>Error: descripción vacía</body></html>");
            return;
        }
        
        // Determinar prioridad (lógica de negocio mezclada con acceso a datos)
        String prioridad = "NORMAL";
        if ("CANCELACION_ASIGNATURA".equals(tipoSolicitud)) {
            prioridad = "ALTA"; // ¿Por qué? Nadie lo sabe, está hardcodeado aquí
        }
        
        // Insertar directamente en la base de datos (acceso a datos en el servlet)
        Connection conn = null;
        try {
            conn = DriverManager.getConnection("jdbc:postgresql://localhost/triage", "admin", "1234");
            PreparedStatement ps = conn.prepareStatement(
                "INSERT INTO solicitud (descripcion, tipo, prioridad, estado, estudiante_id) " +
                "VALUES (?, ?, ?, 'REGISTRADA', ?)"
            );
            ps.setString(1, descripcion);
            ps.setString(2, tipoSolicitud);
            ps.setString(3, prioridad);
            ps.setString(4, estudianteId);
            ps.executeUpdate();
            
        } catch (SQLException e) {
            e.printStackTrace(); // El stacktrace se imprime en la consola del servidor
        } finally {
            if (conn != null) try { conn.close(); } catch (SQLException e) {}
        }
        
        // Construir HTML directamente (presentación mezclada con todo)
        resp.getWriter().println("<html><body>Solicitud registrada con prioridad: " + prioridad + "</body></html>");
    }
}
```

¿Puedes ver los problemas? Validación, lógica de negocio, acceso a datos y presentación, **todo en el mismo lugar**. Si mañana quieres cambiar la regla de priorización, tienes que buscarla aquí dentro. Si quieres reutilizar la lógica de inserción desde otro servlet, no puedes sin copiar y pegar.

### 4.4 ¿Cuándo usar un monolito?

A pesar de su mala reputación, el monolito no siempre es malo. Es apropiado cuando:

- El equipo es pequeño (1-5 personas)
- El proyecto es nuevo y los requisitos cambian frecuentemente
- La complejidad del dominio es baja o media
- No hay necesidad de escalar partes independientes del sistema
- Recursos de infraestructura son limitados

> **La industria ha aprendido a la fuerza:** muchos proyectos que empezaron como microservicios desde el día uno fracasaron por la complejidad prematura. El consejo hoy es: *"Start with a monolith, extract services when needed."*

### 4.5 Ventajas y desventajas del monolito

**Ventajas:**
- Simplicidad en el desarrollo inicial
- Un solo proyecto para configurar, compilar y desplegar
- Debugging más fácil: todo el código está en el mismo proceso
- Rendimiento interno alto (llamadas entre módulos son llamadas a métodos, no peticiones HTTP)
- Sin problemas de latencia de red entre componentes

**Desventajas:**
- Con el tiempo, se vuelve un *"big ball of mud"* — nadie entiende dónde va cada cosa
- Un bug en un módulo puede derribar toda la aplicación
- Escalar significa replicar TODO el sistema, aunque solo una parte esté saturada
- El equipo crece y todos tocan el mismo código — conflictos constantes
- Los tiempos de compilación y despliegue crecen con el proyecto
- Difícil adoptar nuevas tecnologías (estás atado al stack original)

---

## 5. Arquitectura Cliente-Servidor

### 5.1 ¿Qué es?

La arquitectura cliente-servidor divide el sistema en dos partes con roles claramente diferenciados:

- **El servidor** provee servicios, recursos o datos. Está siempre disponible y espera solicitudes.
- **El cliente** consume esos servicios. Puede ser una aplicación web, móvil, de escritorio, u otro servidor.

Esta es la arquitectura que **usaremos todo el semestre**. Nuestro backend en Spring Boot será el servidor, y nuestro frontend en Angular será el cliente.

### 5.2 Diagrama cliente-servidor

```
                    ┌─────────────────────────┐
                    │       CLIENTE           │
                    │   (Angular en el        │
                    │    navegador del        │
                    │    estudiante)          │
                    └────────────┬────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │  Petición HTTP/HTTPS    │
                    │  GET /api/v1/solicitudes│
                    │  Authorization: Bearer  │
                    │  eyJhbGciOiJIUzI1...    │
                    └────────────┬────────────┘
                                 │   Internet / Red local
                    ┌────────────┴────────────┐
                    │  Respuesta HTTP         │
                    │  200 OK                 │
                    │  Content-Type: JSON     │
                    │  [{"id":1, "tipo":...}] │
                    └────────────┬────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │       SERVIDOR          │
                    │  (Spring Boot escucha   │
                    │   en el puerto 8080)    │
                    │                         │
                    │  - Autentica al usuario │
                    │  - Aplica reglas de     │
                    │    negocio              │
                    │  - Consulta la BD       │
                    │  - Devuelve JSON        │
                    └────────────┬────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │    BASE DE DATOS        │
                    │    (PostgreSQL          │
                    │     puerto 5432)        │
                    └─────────────────────────┘
```

### 5.3 El protocolo que los une: HTTP

HTTP (HyperText Transfer Protocol) es el idioma que hablan cliente y servidor. Cada conversación tiene la estructura:

**Petición (del cliente al servidor):**
```
POST /api/v1/solicitudes HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...

{
  "tipo": "HOMOLOGACION",
  "descripcion": "Solicito homologar la materia Cálculo I cursada en otra universidad",
  "canalOrigen": "WEB"
}
```

**Respuesta (del servidor al cliente):**
```
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/v1/solicitudes/47

{
  "id": 47,
  "tipo": "HOMOLOGACION",
  "descripcion": "Solicito homologar la materia Cálculo I...",
  "estado": "REGISTRADA",
  "prioridad": "NORMAL",
  "fechaRegistro": "2026-03-15T10:30:00",
  "solicitante": {
    "id": 123,
    "nombre": "Ana García"
  }
}
```

### 5.4 Los verbos HTTP y su significado

En el Sistema de Triage usaremos todos estos verbos:

| Verbo HTTP | Significado | Ejemplo en el Triage |
|---|---|---|
| `GET` | Obtener un recurso o lista | Consultar todas las solicitudes pendientes |
| `POST` | Crear un nuevo recurso | Registrar una nueva solicitud académica |
| `PUT` | Reemplazar completamente un recurso | Actualizar todos los datos de una solicitud |
| `PATCH` | Modificar parcialmente un recurso | Cambiar solo el estado de una solicitud |
| `DELETE` | Eliminar un recurso | Eliminar una solicitud (solo ADMIN) |

### 5.5 Los códigos de estado HTTP más importantes

```
2xx → Éxito
  200 OK          → Todo salió bien, aquí está el resultado
  201 Created     → El recurso fue creado exitosamente
  204 No Content  → La operación fue exitosa, sin cuerpo de respuesta

4xx → Error del cliente (tú enviaste algo mal)
  400 Bad Request       → La petición tiene errores de formato o validación
  401 Unauthorized      → No enviaste credenciales o son inválidas
  403 Forbidden         → Tienes credenciales pero no tienes permiso para esto
  404 Not Found         → El recurso que buscas no existe
  409 Conflict          → Hay un conflicto con el estado actual del recurso
  422 Unprocessable     → Datos bien formados pero semánticamente incorrectos

5xx → Error del servidor (algo falló en nuestro código)
  500 Internal Server Error → Error inesperado en el servidor
  503 Service Unavailable   → El servidor está caído o sobrecargado
```

---

## 6. Arquitectura en Capas (N-Tier)

### 6.1 El problema que resuelve

Volvamos al código monolítico desorganizado que vimos antes. El problema fundamental era que **una misma clase tenía múltiples responsabilidades**. La solución que la industria desarrolló hace décadas es organizar el código en **capas**, donde cada capa tiene una responsabilidad única y bien definida.

Piensa en las capas como los pisos de un edificio de oficinas:
- **Piso 1 (Presentación):** La recepción. Atiende al público, recibe documentos, entrega resultados. No toma decisiones de fondo.
- **Piso 2 (Negocio):** Los profesionales. Aquí se analizan los casos, se aplican las reglas, se toman las decisiones.
- **Piso 3 (Persistencia):** El archivo. Guarda y recupera documentos. No decide qué hacer con ellos.
- **Sótano (Base de datos):** El almacén físico.

### 6.2 Las capas del sistema de Triage

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   FRONTEND (Angular - en el navegador del usuario)         │
│   ► Componentes de interfaz                                 │
│   ► Formularios reactivos                                   │
│   ► Servicios HTTP que consumen la API                      │
│                                                             │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP/HTTPS (JSON)
                           ▼
┌─────────────────────────────────────────────────────────────┐
│              CAPA DE PRESENTACIÓN (Controllers)             │
│                                                             │
│   @RestController                                           │
│   ► SolicitudController.java                                │
│   ► UsuarioController.java                                  │
│   ► AuthController.java                                     │
│                                                             │
│   Responsabilidades:                                        │
│   ✓ Recibir peticiones HTTP                                 │
│   ✓ Deserializar el JSON a objetos Java                     │
│   ✓ Validar el formato de la entrada (@Valid)               │
│   ✓ Delegar al servicio correspondiente                     │
│   ✓ Serializar la respuesta a JSON                          │
│   ✓ Retornar el código de estado HTTP correcto              │
│                                                             │
│   ✗ NO contiene lógica de negocio                           │
│   ✗ NO accede a la base de datos directamente              │
└──────────────────────────┬──────────────────────────────────┘
                           │ Llamada a método Java
                           ▼
┌─────────────────────────────────────────────────────────────┐
│              CAPA DE NEGOCIO (Services)                     │
│                                                             │
│   @Service                                                  │
│   ► SolicitudService.java                                   │
│   ► PriorizacionService.java                                │
│   ► HistorialService.java                                   │
│   ► NotificacionService.java                                │
│                                                             │
│   Responsabilidades:                                        │
│   ✓ Implementar las reglas de negocio del dominio           │
│   ✓ Coordinar operaciones entre repositorios                │
│   ✓ Gestionar transacciones (@Transactional)                │
│   ✓ Aplicar lógica de priorización de solicitudes           │
│   ✓ Validar transiciones de estado (Registrada → Clasificada)│
│   ✓ Registrar entradas en el historial auditable            │
│                                                             │
│   ✗ NO recibe HttpServletRequest/Response                   │
│   ✗ NO sabe nada de HTTP ni de JSON                         │
└──────────────────────────┬──────────────────────────────────┘
                           │ Llamada a método Java
                           ▼
┌─────────────────────────────────────────────────────────────┐
│              CAPA DE PERSISTENCIA (Repositories)            │
│                                                             │
│   @Repository / JpaRepository                               │
│   ► SolicitudRepository.java                                │
│   ► UsuarioRepository.java                                  │
│   ► HistorialRepository.java                                │
│                                                             │
│   Responsabilidades:                                        │
│   ✓ Operaciones CRUD contra la base de datos                │
│   ✓ Consultas personalizadas con JPQL o SQL nativo          │
│   ✓ Paginación y ordenamiento                               │
│   ✓ Abstraer el motor de base de datos específico           │
│                                                             │
│   ✗ NO contiene lógica de negocio                           │
│   ✗ NO sabe cuándo o por qué se llaman sus métodos          │
└──────────────────────────┬──────────────────────────────────┘
                           │ SQL / JPA
                           ▼
┌─────────────────────────────────────────────────────────────┐
│              BASE DE DATOS (PostgreSQL)                     │
│                                                             │
│   ► Tabla: solicitud                                        │
│   ► Tabla: historial_solicitud                              │
│   ► Tabla: usuario                                          │
│   ► Tabla: rol                                              │
│   ► Tabla: tipo_solicitud                                   │
│   ► Tabla: usuario_rol (tabla intermedia)                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 6.3 Flujo de datos completo: Un estudiante crea una solicitud

Vamos a rastrear exactamente qué pasa cuando un estudiante hace clic en "Enviar solicitud" en el sistema de Triage. Este es el flujo completo, paso a paso:

```
PASO 1: El estudiante llena el formulario en Angular y hace clic en "Enviar"
────────────────────────────────────────────────────────────────────────────
Angular construye y envía esta petición HTTP:

POST http://localhost:8080/api/v1/solicitudes
Headers:
  Content-Type: application/json
  Authorization: Bearer eyJhbGci...  ← Token JWT del estudiante logueado
Body:
  {
    "tipo": "HOMOLOGACION",
    "descripcion": "Solicito homologar Cálculo I de la Univ. Nacional",
    "canalOrigen": "WEB"
  }

PASO 2: Spring Security intercepta la petición (antes de llegar al Controller)
────────────────────────────────────────────────────────────────────────────
► Spring Security lee el header Authorization
► Extrae y valida el token JWT
► Identifica al usuario: Ana García (rol: ESTUDIANTE)
► Verifica que ESTUDIANTE tiene permiso para POST /api/v1/solicitudes
► Si todo está bien, deja pasar la petición

PASO 3: El Controller recibe la petición
────────────────────────────────────────────────────────────────────────────
@RestController SolicitudController recibe la llamada al método crearSolicitud()
► Deserializa el JSON a un objeto CreateSolicitudRequest (DTO)
► Ejecuta @Valid → verifica que "descripcion" no esté vacía, que "tipo" sea válido
► Si la validación falla → devuelve 400 Bad Request inmediatamente
► Si pasa → llama a solicitudService.crearSolicitud(dto, usuarioLogueado)

PASO 4: El Service aplica la lógica de negocio
────────────────────────────────────────────────────────────────────────────
@Service SolicitudService.crearSolicitud() ejecuta:
► Verifica que el estudiante no tenga ya una solicitud del mismo tipo pendiente
  (regla de negocio: no se permite duplicar solicitudes activas)
► Determina la prioridad según el tipo:
  - HOMOLOGACION → NORMAL (tiene 30 días para resolverse)
  - CANCELACION_EXTEMPORANEA → ALTA (tiene fecha límite institucional)
  - SOLICITUD_CUPO → ALTA si quedan menos de 5 días para inicio de semestre
► Crea el objeto Solicitud con estado = REGISTRADA
► Llama a solicitudRepository.save(solicitud)
► Crea la primera entrada en el historial: "Solicitud registrada por el estudiante"
► Llama a historialRepository.save(historialEntry)

PASO 5: El Repository persiste en la base de datos
────────────────────────────────────────────────────────────────────────────
@Repository SolicitudRepository.save() ejecuta:
► JPA/Hibernate genera el SQL:
  INSERT INTO solicitud 
  (tipo, descripcion, canal_origen, estado, prioridad, fecha_registro, solicitante_id)
  VALUES ('HOMOLOGACION', 'Solicito...', 'WEB', 'REGISTRADA', 'NORMAL', NOW(), 123)
  RETURNING id;  -- PostgreSQL devuelve el ID generado: 47

► La BD guarda el registro y devuelve el objeto con id = 47

PASO 6: La respuesta sube por las capas de vuelta
────────────────────────────────────────────────────────────────────────────
Repository → devuelve Solicitud con id=47 al Service
Service → devuelve el mismo objeto al Controller
Controller → convierte la Solicitud a SolicitudResponse (DTO)
           → devuelve ResponseEntity.status(201).body(solicitudResponse)
Spring → serializa el DTO a JSON y envía la respuesta HTTP

PASO 7: Angular recibe la respuesta
────────────────────────────────────────────────────────────────────────────
HTTP 201 Created
{
  "id": 47,
  "tipo": "HOMOLOGACION",
  "estado": "REGISTRADA",
  "prioridad": "NORMAL",
  "fechaRegistro": "2026-03-15T10:30:00",
  ...
}

Angular muestra un mensaje: "✅ Tu solicitud #47 fue registrada exitosamente."
```

### 6.4 La regla fundamental de las capas: la dependencia solo va hacia abajo

Esta es la regla más importante de la arquitectura en capas:

```
Controller  →  puede llamar a  →  Service
Service     →  puede llamar a  →  Repository
Repository  →  puede llamar a  →  Base de Datos

❌ NUNCA al revés:
Repository  ✗  NO llama a  ✗  Service
Service     ✗  NO llama a  ✗  Controller
```

Si un Repository necesita llamar a un Service, es una señal de que algo está mal en el diseño. En la práctica, esto significa que no debes tener una instancia de `SolicitudService` dentro de `SolicitudRepository`.

### 6.5 El modelo de dominio: las entidades del Triage en Java

Antes de continuar con otras arquitecturas, veamos cómo se ven las entidades principales del dominio del Triage. Esto es **puro Java** por ahora, sin anotaciones de Spring ni de JPA:

```java
// Las siguientes clases representan el dominio de negocio del Triage.
// Por ahora son Plain Old Java Objects (POJOs).
// En guías posteriores agregaremos anotaciones JPA para persistirlas.

// ── Enumeraciones del dominio ──────────────────────────────────────────────

public enum EstadoSolicitud {
    REGISTRADA,      // El estudiante la acaba de crear
    CLASIFICADA,     // Un funcionario le asignó tipo y prioridad
    EN_ATENCION,     // Un responsable la está procesando
    ATENDIDA,        // El responsable la resolvió
    CERRADA          // Se formalizó el cierre con observación
}

public enum TipoSolicitud {
    REGISTRO_ASIGNATURA,
    HOMOLOGACION,
    CANCELACION_ASIGNATURA,
    SOLICITUD_CUPO,
    CONSULTA_ACADEMICA
}

public enum Prioridad {
    BAJA,
    NORMAL,
    ALTA,
    CRITICA
}

public enum CanalOrigen {
    WEB,           // A través del sistema que estamos construyendo
    CORREO,        // Enviada por email
    PRESENCIAL,    // El estudiante fue al CSU
    TELEFONICO,    // Por llamada
    SAC            // Sistema de Atención al Cliente
}

public enum NombreRol {
    ESTUDIANTE,
    FUNCIONARIO,
    ADMIN
}

// ── Entidades del dominio ──────────────────────────────────────────────────

/**
 * Representa a un usuario del sistema.
 * Puede ser estudiante, funcionario o administrador.
 */
public class Usuario {
    private Long id;
    private String nombre;
    private String apellido;
    private String correo;
    private String contrasena;  // Siempre hasheada con BCrypt, NUNCA en texto plano
    private boolean activo;
    private List<Rol> roles;
    
    // Constructores, getters y setters omitidos por brevedad.
    // En guías posteriores usaremos Records para los DTOs
    // y Lombok (o Java puro) para las entidades JPA.
}

/**
 * Representa el rol de un usuario en el sistema.
 */
public class Rol {
    private Long id;
    private NombreRol nombre;
}

/**
 * La entidad central del dominio.
 * Representa un trámite académico que un estudiante ingresa al sistema.
 */
public class Solicitud {
    private Long id;
    private TipoSolicitud tipo;
    private String descripcion;
    private CanalOrigen canalOrigen;
    private EstadoSolicitud estado;
    private Prioridad prioridad;
    private String justificacionPrioridad;
    private java.time.LocalDateTime fechaRegistro;
    private java.time.LocalDateTime fechaUltimaActualizacion;
    private Usuario solicitante;       // El estudiante que la creó
    private Usuario responsableActual; // El funcionario asignado
    private List<HistorialSolicitud> historial;
}

/**
 * Registra cada acción realizada sobre una solicitud.
 * Es la "bitácora" que garantiza trazabilidad y auditoría (RF-06).
 */
public class HistorialSolicitud {
    private Long id;
    private Solicitud solicitud;
    private LocalDateTime fechaAccion;
    private String accionRealizada;          // Ej: "Estado cambiado a EN_ATENCION"
    private EstadoSolicitud estadoAnterior;
    private EstadoSolicitud estadoNuevo;
    private Usuario usuarioResponsable;      // Quién realizó la acción
    private String observaciones;
}
```

### 6.6 El paquete de la capa de negocio: las reglas del Triage

```java
/**
 * Contiene las reglas de negocio para gestionar el ciclo de vida de solicitudes.
 * Este es el corazón del sistema de Triage.
 *
 * Reglas implementadas:
 * - RF-03: Las solicitudes se priorizan según tipo, impacto y fecha límite
 * - RF-04: Las transiciones de estado son validadas (no se puede cerrar una solicitud no atendida)
 * - RF-05: Solo usuarios activos pueden ser asignados como responsables
 * - RF-08: Una solicitud cerrada no puede ser modificada
 */
public class SolicitudService {

    // Esta clase tendrá su implementación completa en la Guía 06.
    // Por ahora queremos entender QUÉ hace, no cómo lo hace técnicamente.

    /**
     * Registra una nueva solicitud académica (RF-01).
     * Determina la prioridad inicial basada en el tipo (RF-03).
     * Registra la primera entrada en el historial (RF-06).
     */
    public Solicitud crearSolicitud(CrearSolicitudRequest request, Usuario solicitante) {
        // Lógica pendiente para Guía 06
        return null;
    }

    /**
     * Valida y ejecuta la transición de estado de una solicitud (RF-04).
     * Solo permite transiciones coherentes:
     *   REGISTRADA → CLASIFICADA
     *   CLASIFICADA → EN_ATENCION
     *   EN_ATENCION → ATENDIDA
     *   ATENDIDA → CERRADA
     * Registra la transición en el historial (RF-06).
     */
    public Solicitud cambiarEstado(Long solicitudId, EstadoSolicitud nuevoEstado,
                                   String observacion, Usuario ejecutadoPor) {
        // Lógica pendiente para Guía 06
        return null;
    }

    /**
     * Asigna un funcionario como responsable de atender la solicitud (RF-05).
     * Valida que el funcionario exista y esté activo.
     */
    public Solicitud asignarResponsable(Long solicitudId, Long responsableId,
                                        Usuario asignadoPor) {
        // Lógica pendiente para Guía 06
        return null;
    }
}
```

---

## 7. Arquitectura Hexagonal (Puertos y Adaptadores)

### 7.1 El problema que resuelve

La arquitectura en capas es una gran mejora sobre el monolito desorganizado. Pero tiene un problema sutil: **las capas internas (negocio) dependen de las capas externas (persistencia)**.

¿Qué significa esto en la práctica? Que tu `SolicitudService` (lógica de negocio) necesita importar `SolicitudRepository` (que es JPA, que es Hibernate, que habla con PostgreSQL). Si mañana quieres:
- Cambiar de PostgreSQL a MongoDB
- Probar el servicio sin base de datos
- Cambiar el REST Controller por un CLI (command-line interface)

...tienes que tocar el código de negocio. **Y el código de negocio es el más valioso de tu sistema.** Es donde está todo el conocimiento del dominio.

La **Arquitectura Hexagonal**, propuesta por Alistair Cockburn en 2005, invierte esta relación.

### 7.2 La idea central: el dominio en el centro

```
                    ┌──────────────────────────────────────────┐
                    │          EXTERIOR DEL HEXÁGONO          │
                    │                                          │
         ┌──────────┤ Adaptadores de ENTRADA (Driving)         │
         │  HTTP    │ ► REST Controller                        │
         │  Request ├──────────────────┐                       │
         │          │ ► CLI Command    │                       │
         │  Angular ├──────────────────┤                       │
         │          │ ► Mensajería     │                       │
         └──────────┤   (Kafka, RMQ)  │                       │
                    │                 │                       │
                    │        ┌────────▼──────────────────┐    │
                    │        │   PUERTO DE ENTRADA        │    │
                    │        │  (Interface Java)          │    │
                    │        │  SolicitudUseCase          │    │
                    │        └────────┬──────────────────┘    │
                    │                 │                       │
                    │        ┌────────▼──────────────────┐    │
                    │        │      DOMINIO               │    │
                    │        │   (El corazón del          │    │
                    │        │    sistema de Triage)      │    │
                    │        │                            │    │
                    │        │  ► Entidades: Solicitud,   │    │
                    │        │    Historial, Usuario      │    │
                    │        │  ► Lógica de priorización  │    │
                    │        │  ► Reglas de transición    │    │
                    │        │    de estados              │    │
                    │        │  ► SolicitudServiceImpl    │    │
                    │        └────────┬──────────────────┘    │
                    │                 │                       │
                    │        ┌────────▼──────────────────┐    │
                    │        │   PUERTO DE SALIDA         │    │
                    │        │  (Interface Java)          │    │
                    │        │  SolicitudRepository       │    │
                    │        │  CorreoNotificador         │    │
                    │        └────────┬──────────────────┘    │
                    │                 │                       │
         ┌──────────┤ Adaptadores de SALIDA (Driven)          │
         │  JPA /   │ ► JpaSolicitudRepository               │
         │  Hibernate├──────────────────┐                     │
         │          │ ► MongoRepository │                     │
         │  BD      ├──────────────────┤                     │
         │          │ ► EmailService   │                     │
         │  Correo  ├──────────────────┤                     │
         └──────────┤ ► LLM Adapter    │                     │
                    │   (OpenAI, Claude)│                     │
                    └──────────────────┴─────────────────────┘
```

### 7.3 Los puertos: contratos definidos como interfaces Java

La pieza clave de la arquitectura hexagonal es que el dominio define **interfaces** (puertos) que dicen **qué necesita** sin especificar **cómo se hace**:

```java
// PUERTO DE ENTRADA: Define las operaciones que el dominio expone
// Esta interfaz vive en el paquete del dominio.
// Los Controllers la implementan desde afuera.
public interface SolicitudUseCase {
    
    /**
     * RF-01: Registra una nueva solicitud en el sistema.
     */
    Solicitud crearSolicitud(CrearSolicitudCommand command);
    
    /**
     * RF-04: Cambia el estado de una solicitud validando la transición.
     */
    Solicitud cambiarEstado(Long solicitudId, EstadoSolicitud nuevoEstado,
                             String observacion, Long ejecutadoPorId);
    
    /**
     * RF-07: Consulta solicitudes con filtros y paginación.
     */
    Page<Solicitud> consultarSolicitudes(FiltroBusquedaCommand filtro);
}

// PUERTO DE SALIDA: Define lo que el dominio necesita de la infraestructura
// Esta interfaz también vive en el dominio.
// JPA la implementa como adaptador de salida.
public interface SolicitudPersistencePort {
    
    Solicitud guardar(Solicitud solicitud);
    
    Optional<Solicitud> buscarPorId(Long id);
    
    Page<Solicitud> buscarConFiltros(FiltroBusqueda filtro);
    
    boolean existeSolicitudActivaDelTipo(Long solicitanteId, TipoSolicitud tipo);
}

// ADAPTADOR DE SALIDA: Implementa el puerto usando JPA
// Esta clase vive en la capa de infraestructura, FUERA del dominio.
// El dominio no sabe que esta clase existe.
public class JpaSolicitudAdapter implements SolicitudPersistencePort {
    
    private final JpaSolicitudRepository jpaRepo;  // Repository de Spring Data
    private final SolicitudMapper mapper;
    
    @Override
    public Solicitud guardar(Solicitud solicitud) {
        SolicitudEntity entity = mapper.toEntity(solicitud);
        SolicitudEntity savedEntity = jpaRepo.save(entity);
        return mapper.toDomain(savedEntity);
    }
    
    // Resto de implementaciones...
}
```

### 7.4 Beneficio clave: testeabilidad

Gracias a los puertos, puedes probar toda la lógica de negocio **sin base de datos**:

```java
// Test del dominio sin Spring, sin JPA, sin base de datos
// Solo Java puro y una implementación falsa del repositorio
class SolicitudServiceTest {
    
    // Un "mock" manual del puerto de persistencia
    // No hay base de datos. Guardamos en memoria.
    SolicitudPersistencePort repositorioFalso = new SolicitudPersistencePort() {
        private Map<Long, Solicitud> almacenEnMemoria = new HashMap<>();
        private long contadorId = 1;
        
        @Override
        public Solicitud guardar(Solicitud solicitud) {
            solicitud.setId(contadorId++);
            almacenEnMemoria.put(solicitud.getId(), solicitud);
            return solicitud;
        }
        
        @Override
        public Optional<Solicitud> buscarPorId(Long id) {
            return Optional.ofNullable(almacenEnMemoria.get(id));
        }
        
        // etc.
    };
    
    @Test
    void unaHomologacionDebeCrearseConPrioridadNormal() {
        // ARRANGE
        var service = new SolicitudServiceImpl(repositorioFalso);
        var command = new CrearSolicitudCommand(
            TipoSolicitud.HOMOLOGACION,
            "Solicito homologar Cálculo I",
            CanalOrigen.WEB,
            usuarioEstudiante
        );
        
        // ACT
        Solicitud resultado = service.crearSolicitud(command);
        
        // ASSERT
        assertThat(resultado.getPrioridad()).isEqualTo(Prioridad.NORMAL);
        assertThat(resultado.getEstado()).isEqualTo(EstadoSolicitud.REGISTRADA);
    }
    
    @Test
    void noPuedeCrearseSolicitudSiYaHayUnaActivaDelMismoTipo() {
        // ARRANGE
        // Simular que ya existe una solicitud activa
        // ...
        // Esta prueba valida RF-03 sin tocar la base de datos
    }
}
```

Esto no sería posible si el `SolicitudService` dependiera directamente de `JpaSolicitudRepository` (la implementación concreta de JPA).

### 7.5 Arquitectura Hexagonal vs. Arquitectura en Capas

| Aspecto | En Capas | Hexagonal |
|---|---|---|
| **Dirección de dependencias** | Capas superiores dependen de inferiores | Todo depende del dominio; dominio no depende de nadie |
| **Acoplamiento** | Service acoplado al Repository concreto | Service acoplado solo a una interfaz (puerto) |
| **Testeabilidad** | Requiere BD o mocks de Spring | Pruebas de dominio con Java puro |
| **Cambiar la BD** | Afecta la capa de negocio | Solo cambia el adaptador; dominio no se toca |
| **Cambiar el framework web** | Gran impacto | Solo cambia el adaptador de entrada |
| **Complejidad inicial** | Baja-Media | Media-Alta |
| **Curva de aprendizaje** | Baja | Media |
| **Proyectos recomendados** | Proyectos simples/medianos | Proyectos complejos con larga vida útil |

---

## 8. Arquitectura Limpia (Clean Architecture)

### 8.1 ¿Qué es?

La **Clean Architecture**, propuesta por Robert C. Martin ("Uncle Bob") en 2012, es una evolución y formalización de ideas similares a la hexagonal. La visualiza como **anillos concéntricos** (como las capas de una cebolla), donde el código más valioso e importante está en el centro.

### 8.2 Los anillos concéntricos

```
                ╔═══════════════════════════════════════╗
                ║                                       ║
                ║    ┌─────────────────────────────┐   ║
                ║    │   ┌────────────────────┐    │   ║
                ║    │   │  ┌─────────────┐   │    │   ║
                ║    │   │  │   ENTITIES  │   │    │   ║
                ║    │   │  │   (Domain)  │   │    │   ║
                ║    │   │  │             │   │    │   ║
                ║    │   │  │ ► Solicitud │   │    │   ║
                ║    │   │  │ ► Usuario   │   │    │   ║
                ║    │   │  │ ► Historial │   │    │   ║
                ║    │   │  └─────────────┘   │    │   ║
                ║    │   │                    │    │   ║
                ║    │   │    USE CASES       │    │   ║
                ║    │   │    (Application)   │    │   ║
                ║    │   │                    │    │   ║
                ║    │   │  ► CrearSolicitud  │    │   ║
                ║    │   │  ► CambiarEstado   │    │   ║
                ║    │   │  ► Priorizar       │    │   ║
                ║    │   └────────────────────┘    │   ║
                ║    │                             │   ║
                ║    │    INTERFACE ADAPTERS       │   ║
                ║    │                             │   ║
                ║    │  ► Controllers (REST)       │   ║
                ║    │  ► Presenters (DTOs)        │   ║
                ║    │  ► Repository Adapters      │   ║
                ║    └─────────────────────────────┘   ║
                ║                                       ║
                ║    FRAMEWORKS & DRIVERS               ║
                ║                                       ║
                ║  ► Spring Boot    ► Angular           ║
                ║  ► PostgreSQL     ► Docker            ║
                ║  ► JPA/Hibernate  ► JWT               ║
                ║                                       ║
                ╚═══════════════════════════════════════╝
```

### 8.3 La regla de dependencia

La regla fundamental de Clean Architecture es:

> **El código que está en un anillo interior NUNCA puede depender de código que está en un anillo exterior.**

En términos prácticos para el Triage:

```
✅ CORRECTO:
La entidad Solicitud (anillo interior) no importa nada de Spring, JPA, ni Angular.
El caso de uso CrearSolicitud (segundo anillo) solo depende de entidades del dominio.
El Controller REST (tercer anillo) depende de los casos de uso (segundo anillo).
Spring Boot (anillo exterior) configura y conecta todo.

❌ INCORRECTO:
La entidad Solicitud importa javax.persistence.Entity → 
  depende del framework (anillo exterior)

El caso de uso CrearSolicitud importa JpaSolicitudRepository →
  depende de la implementación concreta de infraestructura

El Controller accede directamente a la BD →
  saltó anillos intermedios
```

### 8.4 Clean Architecture vs. Hexagonal

En la práctica, ambas arquitecturas son muy similares en sus objetivos y diferencias frente a la arquitectura en capas tradicional. Las diferencias son más de terminología y énfasis:

| Concepto | Hexagonal | Clean Architecture |
|---|---|---|
| El núcleo | Dominio | Entidades + Casos de Uso |
| Interacción con exterior | Puertos y Adaptadores | Interfaces Adapters + Frameworks |
| Regla principal | La lógica de dominio no depende de frameworks | El flujo de dependencias apunta hacia adentro |
| Cuándo usarla | Cuando quieres aislar el dominio | Cuando quieres máximo aislamiento en todos los niveles |

> **Para el semestre:** No vamos a implementar Clean Architecture pura ni Hexagonal pura. Vamos a usar una **arquitectura en capas bien estructurada** que toma prestadas las mejores ideas de ambas — en particular, la separación entre entidades, servicios, repositorios y controladores, y el uso de interfaces para desacoplar.

---

## 9. Microservicios

### 9.1 ¿Qué son?

Los microservicios son una evolución del monolito en la dirección opuesta: en lugar de tener una sola aplicación grande, tienes **muchas aplicaciones pequeñas** (servicios), cada una con una responsabilidad acotada, que se comunican entre sí a través de la red.

### 9.2 Diagrama de microservicios

```
                        ┌─────────────────┐
                        │   API Gateway   │
                        │  (Punto único   │
                        │   de entrada)   │
                        └────────┬────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
    ┌─────────▼──────┐  ┌───────▼───────┐  ┌──────▼───────┐
    │  Servicio de   │  │ Servicio de   │  │ Servicio de  │
    │  Solicitudes   │  │  Usuarios     │  │  Notificac.  │
    │                │  │               │  │              │
    │ Spring Boot    │  │ Spring Boot   │  │ Spring Boot  │
    │ Puerto: 8081   │  │ Puerto: 8082  │  │ Puerto: 8083 │
    └────────┬───────┘  └───────┬───────┘  └──────┬───────┘
             │                  │                  │
    ┌────────▼───────┐  ┌───────▼───────┐  ┌──────▼───────┐
    │  BD Solicitud  │  │  BD Usuarios  │  │  BD Notif.   │
    │  PostgreSQL    │  │  PostgreSQL   │  │  MongoDB     │
    └────────────────┘  └───────────────┘  └──────────────┘
              │
    ┌─────────▼──────────────────────────────────────┐
    │        Bus de Mensajería (Kafka / RabbitMQ)    │
    │  "Solicitud creada" → lo escucha Notificaciones│
    └────────────────────────────────────────────────┘
```

### 9.3 ¿Cuándo usar microservicios?

Los microservicios resuelven problemas reales, pero crean nuevos problemas. Solo tienen sentido cuando:

| Condición necesaria | ¿Por qué? |
|---|---|
| Equipo grande (20+ devs) | Equipos pequeños pueden trabajar de forma autónoma en servicios distintos |
| Escala masiva de usuarios | Puedes escalar solo el servicio que está saturado |
| Dominios claramente separados | Si el dominio es uno solo, dividirlo artificialmente crea más problemas que soluciones |
| Madurez en DevOps | Necesitas automatización de despliegue, monitoreo avanzado, service discovery |
| Requisitos de disponibilidad 99.99% | Un servicio caído no afecta al resto |

### 9.4 Los problemas que introducen los microservicios

```
Latencia de red:   Una llamada a método local tarda nanosegundos.
                   Una llamada HTTP entre servicios tarda milisegundos.
                   Si tienes 10 servicios en cadena → 10 llamadas HTTP = latencia acumulada.

Consistencia:      En un monolito, una transacción de BD es ACID.
                   En microservicios, una "transacción" atraviesa múltiples BDs.
                   ¿Qué pasa si el servicio de solicitudes guardó pero el de notificaciones falló?
                   Necesitas patrones como Saga o 2-Phase Commit.

Complejidad:       Un bug en monolito: ver logs de una sola app.
                   Un bug en microservicios: rastrear a través de 5 servicios con Jaeger/Zipkin.

Testing:           En monolito: correr un test local.
                   En microservicios: levantar varios contenedores Docker para el entorno de prueba.
```

### 9.5 ¿Por qué NO usaremos microservicios en este semestre?

El Sistema de Triage que construiremos tiene:
- Un dominio cohesionado (gestión de solicitudes académicas)
- Un equipo pequeño (máximo 3 personas)
- Una escala moderada (~1.400 usuarios)
- Menos de 15 semanas de desarrollo

Dividirlo en microservicios agregaría **complejidad accidental** (complejidad que no viene del dominio sino de las decisiones técnicas) sin beneficio real. Aprenderemos los conceptos de microservicios, pero la implementación será un **monolito bien estructurado con arquitectura en capas**, que es exactamente el punto de partida correcto para un sistema de este tamaño.

---

## 10. Tabla Comparativa de Arquitecturas

| Característica | Monolito | En Capas | Hexagonal / Limpia | Microservicios |
|---|---|---|---|---|
| **Complejidad inicial** | Baja | Media | Media-Alta | Muy Alta |
| **Velocidad de desarrollo inicial** | Alta | Media | Media | Baja |
| **Mantenibilidad a largo plazo** | Baja | Media | Alta | Alta |
| **Testeabilidad** | Baja | Media | Alta | Alta |
| **Desacoplamiento** | Ninguno | Parcial | Alto | Muy Alto |
| **Escalabilidad** | Todo o nada | Todo o nada | Todo o nada | Por servicio |
| **Consistencia de datos** | ACID nativa | ACID nativa | ACID nativa | Compleja (eventual) |
| **Debugging** | Fácil | Medio | Medio | Difícil |
| **Curva de aprendizaje del equipo** | Baja | Media | Alta | Muy Alta |
| **Infraestructura necesaria** | Mínima | Mínima | Mínima | Compleja (K8s, etc.) |
| **Independencia tecnológica** | No | Parcial | Alta | Total |
| **Recomendado para proyectos** | Pequeños / MVP | Medianos | Medianos-Grandes | Grandes / Empresas |
| **Ejemplo real** | Aplicaciones legacy | **El Triage** ← | Sistemas financieros | Netflix, Amazon |

---

## 11. La Arquitectura que Usaremos en el Proyecto de Triage

### 11.1 La decisión y su justificación

Usaremos una **arquitectura en capas bien estructurada (N-Tier)**, influenciada por principios de la arquitectura hexagonal, con estas características:

1. **Separación clara en tres capas en el backend:**
   - `presentation` (Controllers + DTOs)
   - `service` (lógica de negocio)
   - `repository` (persistencia con JPA)

2. **Interfaces para los servicios:** Aunque no haremos hexagonal pura, definiremos interfaces para los servicios de negocio principales. Esto facilita el testing y nos acerca a las buenas prácticas de la industria.

3. **DTOs estrictamente separados de las entidades:** La capa de presentación nunca expondrá entidades JPA directamente (explicado a fondo en la Guía 06).

4. **Frontend completamente separado:** Angular se ejecuta de forma independiente y se comunica con el backend únicamente a través de la API REST (típico de la arquitectura cliente-servidor).

### 11.2 Flujo de datos resumido del Triage

```
[Estudiante] → [Angular SPA] → HTTPS → [Spring Security Filter]
                                              ↓
                                     [REST Controller]
                                              ↓
                                     [Service (negocio)]
                                              ↓
                                     [Repository (JPA)]
                                              ↓
                                     [PostgreSQL]
```

---

## 12. Estructura de Carpetas Completa del Proyecto de Triage

Esta es la estructura que construirás durante todo el semestre. **Memoriza esta organización.** Cada carpeta tiene un propósito específico.

### 12.1 Backend: Spring Boot

```
triage-backend/
│
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── co/
│   │   │       └── uniquindio/
│   │   │           └── triage/
│   │   │               │
│   │   │               ├── TriageApplication.java          ← Punto de entrada (@SpringBootApplication)
│   │   │               │
│   │   │               ├── config/                         ← Configuraciones de Spring
│   │   │               │   ├── SecurityConfig.java         ← Spring Security + JWT
│   │   │               │   ├── CorsConfig.java             ← Configuración CORS para Angular
│   │   │               │   ├── OpenApiConfig.java          ← Documentación Swagger
│   │   │               │   └── AuditoriaConfig.java        ← Auditoría automática con Spring Data
│   │   │               │
│   │   │               ├── domain/                         ← EL CORAZÓN DEL SISTEMA
│   │   │               │   │                               ← (Inspirado en arquitectura hexagonal)
│   │   │               │   ├── model/                      ← Entidades del dominio (puro Java)
│   │   │               │   │   ├── Solicitud.java
│   │   │               │   │   ├── HistorialSolicitud.java
│   │   │               │   │   ├── Usuario.java
│   │   │               │   │   └── Rol.java
│   │   │               │   │
│   │   │               │   └── enums/                      ← Enumeraciones del dominio
│   │   │               │       ├── EstadoSolicitud.java
│   │   │               │       ├── TipoSolicitud.java
│   │   │               │       ├── Prioridad.java
│   │   │               │       ├── CanalOrigen.java
│   │   │               │       └── NombreRol.java
│   │   │               │
│   │   │               ├── persistence/                    ← Capa de persistencia (JPA)
│   │   │               │   ├── entity/                     ← Entidades JPA (@Entity)
│   │   │               │   │   ├── SolicitudEntity.java
│   │   │               │   │   ├── HistorialSolicitudEntity.java
│   │   │               │   │   ├── UsuarioEntity.java
│   │   │               │   │   └── RolEntity.java
│   │   │               │   │
│   │   │               │   ├── repository/                 ← Repositorios Spring Data JPA
│   │   │               │   │   ├── SolicitudRepository.java
│   │   │               │   │   ├── HistorialSolicitudRepository.java
│   │   │               │   │   ├── UsuarioRepository.java
│   │   │               │   │   └── RolRepository.java
│   │   │               │   │
│   │   │               │   └── mapper/                     ← Conversión entidad JPA ↔ modelo dominio
│   │   │               │       ├── SolicitudMapper.java
│   │   │               │       └── UsuarioMapper.java
│   │   │               │
│   │   │               ├── service/                        ← Capa de negocio
│   │   │               │   ├── SolicitudService.java       ← Interface (contrato)
│   │   │               │   ├── UsuarioService.java         ← Interface
│   │   │               │   ├── AuthService.java            ← Interface
│   │   │               │   ├── HistorialService.java       ← Interface
│   │   │               │   └── impl/                       ← Implementaciones
│   │   │               │       ├── SolicitudServiceImpl.java
│   │   │               │       ├── UsuarioServiceImpl.java
│   │   │               │       ├── AuthServiceImpl.java
│   │   │               │       ├── HistorialServiceImpl.java
│   │   │               │       └── PriorizacionServiceImpl.java
│   │   │               │
│   │   │               ├── web/                            ← Capa de presentación (REST API)
│   │   │               │   ├── controller/                 ← REST Controllers
│   │   │               │   │   ├── SolicitudController.java
│   │   │               │   │   ├── UsuarioController.java
│   │   │               │   │   └── AuthController.java
│   │   │               │   │
│   │   │               │   ├── dto/                        ← Data Transfer Objects
│   │   │               │   │   ├── request/                ← Lo que recibimos del cliente
│   │   │               │   │   │   ├── CreateSolicitudRequest.java   ← Record Java 21
│   │   │               │   │   │   ├── UpdateEstadoRequest.java
│   │   │               │   │   │   ├── AsignarResponsableRequest.java
│   │   │               │   │   │   ├── LoginRequest.java
│   │   │               │   │   │   └── RegistroUsuarioRequest.java
│   │   │               │   │   │
│   │   │               │   │   └── response/               ← Lo que enviamos al cliente
│   │   │               │   │       ├── SolicitudResponse.java        ← Record Java 21
│   │   │               │   │       ├── HistorialResponse.java
│   │   │               │   │       ├── UsuarioResponse.java
│   │   │               │   │       ├── AuthResponse.java             ← Contiene el JWT
│   │   │               │   │       └── PageResponse.java             ← Paginación genérica
│   │   │               │   │
│   │   │               │   └── exception/                  ← Manejo global de errores
│   │   │               │       ├── GlobalExceptionHandler.java       ← @ControllerAdvice
│   │   │               │       ├── SolicitudNotFoundException.java
│   │   │               │       ├── EstadoInvalidoException.java
│   │   │               │       ├── UsuarioNoActivoException.java
│   │   │               │       └── SolicitudDuplicadaException.java
│   │   │               │
│   │   │               └── security/                       ← Seguridad JWT
│   │   │                   ├── JwtService.java             ← Generar y validar tokens
│   │   │                   ├── JwtAuthenticationFilter.java← Interceptar peticiones
│   │   │                   └── UserDetailsServiceImpl.java ← Cargar usuario de la BD
│   │   │
│   │   └── resources/
│   │       ├── application.yml                             ← Configuración principal
│   │       ├── application-dev.yml                         ← Config. desarrollo (BD local)
│   │       ├── application-prod.yml                        ← Config. producción (BD en nube)
│   │       └── db/
│   │           └── migration/                              ← Scripts Flyway (versionado de BD)
│   │               ├── V1__crear_tablas_base.sql
│   │               ├── V2__insertar_tipos_solicitud.sql
│   │               ├── V3__insertar_roles.sql
│   │               └── V4__insertar_usuario_admin.sql
│   │
│   └── test/
│       └── java/
│           └── co/uniquindio/triage/
│               ├── service/
│               │   ├── SolicitudServiceTest.java
│               │   └── PriorizacionServiceTest.java
│               └── web/
│                   └── controller/
│                       └── SolicitudControllerTest.java    ← Tests de integración con MockMvc
│
├── pom.xml                                                 ← Dependencias Maven
├── Dockerfile                                              ← Para construir la imagen Docker
├── .env.example                                            ← Variables de entorno de ejemplo
└── README.md
```

### 12.2 Frontend: Angular

```
triage-frontend/
│
├── src/
│   ├── app/
│   │   │
│   │   ├── core/                               ← Servicios singleton y guards (una sola instancia)
│   │   │   ├── services/
│   │   │   │   ├── auth.service.ts             ← Login, logout, manejo del JWT
│   │   │   │   ├── solicitud.service.ts        ← CRUD de solicitudes
│   │   │   │   └── usuario.service.ts          ← Gestión de usuarios
│   │   │   │
│   │   │   ├── interceptors/
│   │   │   │   ├── jwt.interceptor.ts          ← Adjunta el token a cada petición HTTP
│   │   │   │   └── error.interceptor.ts        ← Maneja 401, 403, 500 globalmente
│   │   │   │
│   │   │   ├── guards/
│   │   │   │   ├── auth.guard.ts               ← Protege rutas que requieren login
│   │   │   │   └── role.guard.ts               ← Protege rutas por rol (ADMIN, FUNCIONARIO)
│   │   │   │
│   │   │   └── models/                         ← Interfaces TypeScript = DTOs del backend
│   │   │       ├── solicitud.model.ts
│   │   │       ├── usuario.model.ts
│   │   │       ├── historial.model.ts
│   │   │       └── auth.model.ts
│   │   │
│   │   ├── features/                           ← Módulos de funcionalidad (lazy loading)
│   │   │   │
│   │   │   ├── auth/                           ← Autenticación
│   │   │   │   ├── login/
│   │   │   │   │   ├── login.component.ts
│   │   │   │   │   ├── login.component.html
│   │   │   │   │   └── login.component.css
│   │   │   │   └── registro/
│   │   │   │       ├── registro.component.ts
│   │   │   │       └── registro.component.html
│   │   │   │
│   │   │   ├── solicitudes/                    ← Gestión de solicitudes
│   │   │   │   ├── lista-solicitudes/          ← Vista de tabla con filtros y paginación
│   │   │   │   │   ├── lista-solicitudes.component.ts
│   │   │   │   │   └── lista-solicitudes.component.html
│   │   │   │   ├── crear-solicitud/            ← Formulario RF-01
│   │   │   │   │   ├── crear-solicitud.component.ts
│   │   │   │   │   └── crear-solicitud.component.html
│   │   │   │   ├── detalle-solicitud/          ← Ver detalle + historial
│   │   │   │   │   ├── detalle-solicitud.component.ts
│   │   │   │   │   └── detalle-solicitud.component.html
│   │   │   │   └── clasificar-solicitud/       ← Clasificar y priorizar (solo FUNCIONARIO)
│   │   │   │       ├── clasificar-solicitud.component.ts
│   │   │   │       └── clasificar-solicitud.component.html
│   │   │   │
│   │   │   └── dashboard/                      ← Panel principal con métricas (solo ADMIN)
│   │   │       ├── dashboard.component.ts
│   │   │       └── dashboard.component.html
│   │   │
│   │   ├── shared/                             ← Componentes reutilizables
│   │   │   ├── components/
│   │   │   │   ├── header/
│   │   │   │   ├── footer/
│   │   │   │   ├── loading-spinner/
│   │   │   │   ├── badge-estado/               ← Badge de color según estado de solicitud
│   │   │   │   └── timeline-historial/         ← Línea de tiempo del historial
│   │   │   │
│   │   │   └── pipes/
│   │   │       ├── estado.pipe.ts              ← Transforma 'EN_ATENCION' a 'En Atención'
│   │   │       └── prioridad-color.pipe.ts     ← Devuelve clase CSS según prioridad
│   │   │
│   │   ├── app.component.ts                    ← Componente raíz
│   │   ├── app.component.html
│   │   └── app.routes.ts                       ← Configuración de rutas (standalone)
│   │
│   ├── environments/
│   │   ├── environment.ts                      ← Dev: apiUrl = 'http://localhost:8080'
│   │   └── environment.prod.ts                 ← Prod: apiUrl = 'https://api.triage.uniquindio.co'
│   │
│   ├── index.html
│   ├── main.ts                                 ← Punto de entrada de Angular
│   └── styles.css                              ← Estilos globales
│
├── angular.json
├── package.json
├── tsconfig.json
├── Dockerfile                                  ← Para construir con Nginx
└── nginx.conf                                  ← Configuración de Nginx para producción
```

### 12.3 El proyecto completo como un todo

```
triage/                              ← Carpeta raíz del proyecto
├── triage-backend/                  ← Proyecto Spring Boot
├── triage-frontend/                 ← Proyecto Angular
├── docker-compose.yml               ← Levanta backend + frontend + BD con un comando
├── .env                             ← Variables de entorno (NO subir a Git)
├── .env.example                     ← Plantilla de variables (sí subir a Git)
└── README.md                        ← Documentación general del proyecto
```

---

## 13. Ejercicios Prácticos

### Ejercicio 1 — Analizar arquitecturas existentes

Investiga y determina qué tipo de arquitectura usan los siguientes sistemas reales. Justifica tu respuesta con al menos 3 argumentos:

1. El sistema de registro académico (ACADEMUSOFT) de la Universidad del Quindío
2. Netflix
3. Una aplicación de lista de tareas básica (ToDo app)
4. El sistema bancario de un banco regional pequeño

Crea un documento con tus conclusiones, incluyendo:
- Tipo de arquitectura identificada
- Evidencias que sustentan tu elección
- Ventajas que ese tipo de arquitectura le ofrece al sistema
- Desventajas o riesgos que enfrenta

---

### Ejercicio 2 — Crear la estructura de carpetas del proyecto de Triage

Crea la estructura de carpetas del backend del Triage en tu equipo local siguiendo exactamente el árbol mostrado en la sección 12.1. Dentro de cada carpeta, crea un archivo `.gitkeep` vacío para que Git registre la carpeta (Git no versiona carpetas vacías).

**Pasos sugeridos:**

```bash
# 1. Crear el directorio raíz del proyecto
mkdir -p triage/triage-backend/src/main/java/co/uniquindio/triage

# 2. Navegar al proyecto
cd triage/triage-backend

# 3. Crear las carpetas principales del código
mkdir -p src/main/java/co/uniquindio/triage/{config,domain/{model,enums},persistence/{entity,repository,mapper},service/impl,web/{controller,dto/{request,response},exception},security}

# 4. Crear las carpetas de recursos
mkdir -p src/main/resources/db/migration

# 5. Crear las carpetas de tests
mkdir -p src/test/java/co/uniquindio/triage/{service,web/controller}

# 6. Verificar la estructura creada
find src -type d | sort
```

Luego, dentro de cada carpeta, crea los archivos Java vacíos correspondientes (solo la declaración del paquete y el nombre de la clase, sin implementación):

```java
// Ejemplo: src/main/java/co/uniquindio/triage/domain/enums/EstadoSolicitud.java
package co.uniquindio.triage.domain.enums;

public enum EstadoSolicitud {
    REGISTRADA,
    CLASIFICADA,
    EN_ATENCION,
    ATENDIDA,
    CERRADA
}
```

Crea **todos** los enums del dominio: `EstadoSolicitud`, `TipoSolicitud`, `Prioridad`, `CanalOrigen`, `NombreRol`.

---

### Ejercicio 3 — Diagrama de arquitectura del Triage

Dibuja (puede ser en papel, en draw.io, Lucidchart o cualquier herramienta de diagramas) la arquitectura completa del Sistema de Triage que construirás este semestre. El diagrama debe incluir:

- El navegador del usuario (Angular)
- El servidor de backend (Spring Boot)
- La base de datos (PostgreSQL)
- Las capas internas del backend (Controller, Service, Repository)
- Los flujos de comunicación con los protocolos usados (HTTP/HTTPS, SQL/JPA)
- Los roles de usuario y a qué partes del sistema tienen acceso

---

### Ejercicio 4 — Trazar flujos de datos

Para cada uno de los siguientes casos de uso del Sistema de Triage, escribe en texto la secuencia completa de pasos que ocurrirían en la arquitectura en capas, desde que el usuario hace una acción en Angular hasta que la respuesta vuelve al navegador:

1. **Un funcionario consulta todas las solicitudes con estado "REGISTRADA"** (RF-07)
2. **Un admin asigna un funcionario como responsable de una solicitud** (RF-05)
3. **Un estudiante intenta cerrar su propia solicitud** (acción no permitida — RF-13)

Para el caso 3, especifica en qué capa se detecta el intento no autorizado y qué respuesta HTTP se devuelve.

---

### Ejercicio 5 — Comparar con lo conocido

Piensa en un proyecto Java que hayas desarrollado en semestres anteriores (cualquier proyecto). Responde:

1. ¿Qué tipo de arquitectura usaba ese proyecto (aunque no lo sabías en ese momento)?
2. ¿Qué problemas tenías cuando querías cambiar algo?
3. ¿Cómo habría sido diferente si hubieras usado arquitectura en capas desde el principio?

---

## 14. Errores Comunes y Troubleshooting

### ❌ Error 1: Poner lógica de negocio en el Controller

```java
// ❌ MAL: El Controller decide la prioridad
@PostMapping
public ResponseEntity<SolicitudResponse> crear(@RequestBody CreateSolicitudRequest req) {
    // La lógica de priorización NO va aquí
    String prioridad = "NORMAL";
    if (req.tipo().equals("CANCELACION_ASIGNATURA")) {
        prioridad = "ALTA";
    }
    solicitudRepository.save(new Solicitud(req.tipo(), prioridad));
    // ...
}

// ✅ BIEN: El Controller delega al Service
@PostMapping
public ResponseEntity<SolicitudResponse> crear(@RequestBody CreateSolicitudRequest req) {
    var solicitud = solicitudService.crearSolicitud(req, usuarioActual());
    return ResponseEntity.status(201).body(SolicitudResponse.from(solicitud));
}
```

**Por qué es un problema:** Si la regla de priorización cambia (y cambiará), tendrás que buscarla en todos los controllers donde la copiaste. Además, no puedes probarla sin levantar el servidor web completo.

---

### ❌ Error 2: Exponer entidades JPA directamente en la API

```java
// ❌ MAL: Devolver la entidad JPA directamente
@GetMapping("/{id}")
public Solicitud obtenerPorId(@PathVariable Long id) {
    return solicitudRepository.findById(id).orElseThrow();
    // Esto expone campos internos de la BD, referencias circulares (Solicitud → Historial → Solicitud),
    // y datos sensibles como contraseñas del solicitante.
}

// ✅ BIEN: Convertir a DTO antes de devolver
@GetMapping("/{id}")
public ResponseEntity<SolicitudResponse> obtenerPorId(@PathVariable Long id) {
    var solicitud = solicitudService.buscarPorId(id);
    return ResponseEntity.ok(SolicitudResponse.from(solicitud));
}
```

**Por qué es un problema:** La entidad JPA tiene anotaciones de persistencia, puede causar referencias circulares que Jackson no puede serializar (StackOverflowError), y expone datos internos que el cliente no debería ver.

---

### ❌ Error 3: Crear un solo "SolicitudService" que lo haga todo

```java
// ❌ MAL: Un servicio con 50 métodos que maneja TODO
public class SolicitudService {
    public void crearSolicitud() { ... }
    public void cambiarEstado() { ... }
    public void asignarResponsable() { ... }
    public void enviarCorreo() { ... }
    public void calcularEstadisticas() { ... }
    public void generarReporte() { ... }
    public void autenticarUsuario() { ... } // ← ¿Qué hace aquí la autenticación?
    // ... 47 métodos más
}

// ✅ BIEN: Responsabilidades distribuidas
public class SolicitudService { /* Solo CRUD y ciclo de vida de solicitudes */ }
public class PriorizacionService { /* Solo reglas de priorización */ }
public class HistorialService { /* Solo registro de eventos en historial */ }
public class NotificacionService { /* Solo envío de notificaciones */ }
public class AuthService { /* Solo autenticación y JWT */ }
```

**Por qué es un problema:** Una clase con muchas responsabilidades viola el Principio de Responsabilidad Única (SRP, explicado en la Guía 02). Con el tiempo se vuelve imposible de entender y modificar.

---

### ❌ Error 4: No definir interfaces para los servicios

```java
// ❌ MAL: El Controller inyecta la implementación concreta
@RestController
public class SolicitudController {
    private final SolicitudServiceImpl solicitudService; // ← La implementación concreta
    // Si quieres usar una implementación distinta en pruebas, no puedes
}

// ✅ BIEN: El Controller inyecta la interfaz
@RestController
public class SolicitudController {
    private final SolicitudService solicitudService; // ← La interfaz
    // Spring inyectará SolicitudServiceImpl, pero en pruebas puedo inyectar un mock
}
```

---

### ❌ Error 5: Mezclar las capas de carpetas

```
// ❌ MAL: Estructura plana sin separación
src/main/java/co/uniquindio/triage/
    ├── Solicitud.java                  ← ¿Entidad? ¿Modelo? ¿DTO?
    ├── SolicitudController.java
    ├── SolicitudService.java
    ├── SolicitudRepository.java
    ├── Usuario.java
    ├── UsuarioController.java
    ... todo en la misma carpeta

// ✅ BIEN: Estructura con separación clara por capa y luego por dominio
src/main/java/co/uniquindio/triage/
    ├── domain/model/Solicitud.java
    ├── web/controller/SolicitudController.java
    ├── service/SolicitudService.java
    ├── persistence/repository/SolicitudRepository.java
```

---

## 15. Resumen y Cheat Sheet

### 15.1 Tipos de arquitectura en resumen

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CHEAT SHEET — ARQUITECTURAS                        │
├─────────────────────┬───────────────────────────────────────────────────────┤
│ Monolítica          │ Todo en uno. Simple al inicio, caótico después.       │
│                     │ Usa para: MVPs, proyectos pequeños, prototipado rápido│
├─────────────────────┼───────────────────────────────────────────────────────┤
│ Cliente-Servidor    │ Separar quien pide (cliente) de quien sirve (servidor)│
│                     │ Angular = cliente. Spring Boot = servidor.            │
│                     │ Hablan por HTTP con JSON.                             │
├─────────────────────┼───────────────────────────────────────────────────────┤
│ En Capas (N-Tier)   │ Controller → Service → Repository → BD               │
│                     │ Cada capa tiene UNA responsabilidad.                  │
│                     │ Dependencias SOLO hacia abajo.                        │
│                     │ Usa para: la mayoría de aplicaciones empresariales    │
├─────────────────────┼───────────────────────────────────────────────────────┤
│ Hexagonal           │ El dominio en el centro, rodeado de adaptadores.      │
│                     │ Puertos = interfaces Java. Adaptadores = implementaciones│
│                     │ Beneficio clave: testeabilidad sin frameworks.         │
├─────────────────────┼───────────────────────────────────────────────────────┤
│ Clean Architecture  │ Anillos concéntricos. El flujo de dependencias apunta │
│                     │ siempre hacia el centro (las entidades y casos de uso).│
├─────────────────────┼───────────────────────────────────────────────────────┤
│ Microservicios      │ Muchas apps pequeñas comunicadas por HTTP/mensajería.  │
│                     │ Usa para: equipos grandes, escala masiva.              │
│                     │ NO para: proyectos medianos con equipos pequeños.      │
└─────────────────────┴───────────────────────────────────────────────────────┘
```

### 15.2 Las capas del Triage en una línea

```
Angular (UI) → Controller (HTTP) → Service (negocio) → Repository (SQL) → PostgreSQL
```

### 15.3 Reglas de oro

```
✅ Cada clase tiene UNA responsabilidad
✅ Las dependencias van HACIA ABAJO (Controller → Service → Repository)
✅ Los servicios exponen INTERFACES, no implementaciones concretas
✅ Los DTOs son DIFERENTES de las entidades JPA
✅ La lógica de negocio vive en el SERVICE, no en el Controller ni en el Repository
✅ Las entidades del dominio son POJO — sin dependencias de frameworks
```

### 15.4 Verbos HTTP del Triage

```
GET    /api/v1/solicitudes           → Listar solicitudes (con filtros opcionales)
GET    /api/v1/solicitudes/{id}      → Ver una solicitud específica
GET    /api/v1/solicitudes/{id}/historial → Ver historial de la solicitud
POST   /api/v1/solicitudes           → Crear nueva solicitud (RF-01)
PATCH  /api/v1/solicitudes/{id}/estado → Cambiar estado (RF-04)
PATCH  /api/v1/solicitudes/{id}/responsable → Asignar responsable (RF-05)
PATCH  /api/v1/solicitudes/{id}/clasificar → Clasificar y priorizar (RF-02, RF-03)
POST   /api/v1/auth/login            → Obtener JWT
POST   /api/v1/auth/registro         → Registrar nuevo usuario
GET    /api/v1/usuarios              → Listar usuarios (solo ADMIN)
```

---

## 16. Referencias y Recursos Adicionales

### Libros fundamentales

- **"Clean Architecture"** — Robert C. Martin (Uncle Bob), 2017. El libro de referencia para arquitecturas modernas.
- **"Building Microservices"** — Sam Newman, 2021 (2nd ed.). Si quieres profundizar en microservicios después del semestre.
- **"Designing Data-Intensive Applications"** — Martin Kleppmann. Para comprender escalabilidad y persistencia.
- **"Patterns of Enterprise Application Architecture"** — Martin Fowler. El catálogo clásico de patrones empresariales (Repository, Service Layer, DTO, etc.).

### Artículos y recursos en línea

- **The Hexagonal Architecture** — Alistair Cockburn (alistair.cockburn.us/hexagonal-architecture) — El artículo original del autor del patrón.
- **The Clean Architecture** — Robert C. Martin (blog.cleancoder.com) — El artículo original.
- **Microservices** — Martin Fowler & James Lewis (martinfowler.com/articles/microservices.html) — El artículo que popularizó el término.

### Herramientas para diagramas de arquitectura

- **draw.io** (app.diagrams.net) — Gratuito, diagramas de arquitectura y UML en el navegador
- **Lucidchart** — Alternativa con buena colección de plantillas de arquitectura
- **C4 Model** (c4model.com) — Un enfoque moderno y sistemático para documentar arquitecturas software

### Para el proyecto de Triage

En las siguientes guías construirás cada componente mostrado en la estructura de carpetas:
- **Guía 02** → Patrones de diseño que aplicarás dentro de las capas
- **Guía 03** → Características modernas de Java que usarás en las entidades y DTOs
- **Guía 04** → Configurar el proyecto Spring Boot y la primera capa (Controllers)
- **Guía 05** → Implementar la capa de persistencia con JPA
- **Guía 06** → Implementar la capa de negocio y los servicios REST completos

---

> **Autor:** José Alfredo Ramírez Espinosa  
> **Materia:** Programación Avanzada  
> **Programa:** Ingeniería de Sistemas y Computación  
> **Universidad del Quindío** | Armenia, Colombia  
> **Año:** 2026  
> *Material de uso académico. Todos los derechos reservados al autor.*
