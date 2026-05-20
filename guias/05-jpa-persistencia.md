# Guía 05 — JPA, Hibernate y Persistencia: Del Modelo al Código

> **Materia:** Programación Avanzada | **Semestre:** 6to | **Universidad del Quindío**  
> **Stack:** Java 21 · Spring Boot 3.4+ · PostgreSQL · JPA/Hibernate · Flyway  
> **Proyecto base:** Sistema de Triage y Gestión de Solicitudes Académicas

---

## Tabla de Contenidos

1. [Objetivos de Aprendizaje](#1-objetivos-de-aprendizaje)
2. [Prerrequisitos](#2-prerrequisitos)
3. [¿Qué es un ORM y por qué usarlo?](#3-qué-es-un-orm-y-por-qué-usarlo)
4. [JPA vs Hibernate: diferencias y relación](#4-jpa-vs-hibernate-diferencias-y-relación)
5. [Configurar la conexión a PostgreSQL](#5-configurar-la-conexión-a-postgresql)
6. [Entidades JPA: mapear clases a tablas](#6-entidades-jpa-mapear-clases-a-tablas)
7. [Estrategias de generación de ID](#7-estrategias-de-generación-de-id)
8. [Relaciones entre entidades](#8-relaciones-entre-entidades)
9. [Enums en JPA](#9-enums-en-jpa)
10. [Auditoría automática con Spring Data](#10-auditoría-automática-con-spring-data)
11. [Spring Data JPA Repositories](#11-spring-data-jpa-repositories)
12. [Paginación y ordenamiento](#12-paginación-y-ordenamiento)
13. [Validaciones con Bean Validation](#13-validaciones-con-bean-validation)
14. [Modelo completo del proyecto Triage](#14-modelo-completo-del-proyecto-triage)
15. [Migraciones con Flyway](#15-migraciones-con-flyway)
16. [Antes vs Ahora: EntityManager vs Spring Data JPA](#16-antes-vs-ahora-entitymanager-vs-spring-data-jpa)
17. [Errores comunes y troubleshooting](#17-errores-comunes-y-troubleshooting)
18. [Ejercicios prácticos](#18-ejercicios-prácticos)
19. [Resumen y Cheat Sheet](#19-resumen-y-cheat-sheet)
20. [Referencias y recursos adicionales](#20-referencias-y-recursos-adicionales)

---

## 1. Objetivos de Aprendizaje

Al terminar esta guía serás capaz de:

- Explicar qué es un ORM y en qué situaciones conviene usarlo frente a SQL directo.
- Distinguir JPA de Hibernate y entender su relación como especificación e implementación.
- Configurar una conexión a PostgreSQL en Spring Boot con `application.yml`.
- Crear entidades JPA con todas sus anotaciones, constraints e índices.
- Modelar relaciones `@OneToMany`, `@ManyToOne`, `@ManyToMany` y `@OneToOne` de forma correcta, evitando el problema N+1.
- Usar Spring Data JPA con métodos derivados, consultas JPQL y nativas.
- Implementar paginación y ordenamiento con `Pageable` y `Page<T>`.
- Aplicar validaciones con Bean Validation sobre las entidades.
- Mapear todas las entidades del proyecto de Triage con sus relaciones.
- Gestionar el esquema de base de datos con Flyway usando scripts versionados.

---

## 2. Prerrequisitos

Antes de continuar, asegúrate de:

- Haber leído la **Guía 04** (Spring Boot desde Cero). Ya debes tener un proyecto Spring Boot corriendo.
- Tener **PostgreSQL** instalado localmente (o un contenedor Docker con Postgres).
- Conocer conceptos básicos de bases de datos relacionales: tablas, columnas, llaves primarias, llaves foráneas, y relaciones.
- Saber Java básico: clases, herencia, anotaciones, genéricos.

> **¿No tienes PostgreSQL?** La forma más rápida es correr este comando si tienes Docker:
> ```bash
> docker run --name triage-db -e POSTGRES_USER=triage -e POSTGRES_PASSWORD=triage123 \
>   -e POSTGRES_DB=triage_db -p 5432:5432 -d postgres:16
> ```
> Eso levanta PostgreSQL en el puerto 5432 listo para usar.

---

## 3. ¿Qué es un ORM y por qué usarlo?

### La analogía del traductor

Imagina que tu aplicación Java habla "español" (objetos, clases, atributos) y tu base de datos habla "inglés" (tablas, columnas, filas). Para que se entiendan, necesitas un traductor. Ese traductor es un **ORM (Object-Relational Mapping)**.

Un ORM se encarga de:

- Convertir tus objetos Java en filas de una tabla (cuando guardas datos).
- Convertir filas de una tabla en objetos Java (cuando consultas datos).
- Generar el SQL por ti en la mayoría de los casos.
- Gestionar el ciclo de vida de los objetos (nuevo, persistido, modificado, eliminado).

### Sin ORM (JDBC puro)

Así luciría guardar una `Solicitud` sin ORM:

```java
// Sin ORM — todo a mano con JDBC
String sql = "INSERT INTO solicitudes (descripcion, estado, fecha_registro, usuario_id) " +
             "VALUES (?, ?, ?, ?)";

try (Connection conn = dataSource.getConnection();
     PreparedStatement stmt = conn.prepareStatement(sql)) {
    
    stmt.setString(1, solicitud.getDescripcion());
    stmt.setString(2, solicitud.getEstado().name());
    stmt.setTimestamp(3, Timestamp.valueOf(solicitud.getFechaRegistro()));
    stmt.setLong(4, solicitud.getUsuario().getId());
    stmt.executeUpdate();
    
} catch (SQLException e) {
    throw new RuntimeException("Error guardando solicitud", e);
}
```

Esto tiene varios problemas:
- Tienes que escribir SQL para cada operación (INSERT, UPDATE, SELECT, DELETE).
- Si cambias un atributo en la clase, debes cambiar el SQL manualmente.
- El manejo de excepciones es verboso.
- Las relaciones entre objetos se vuelven una pesadilla.

### Con ORM (JPA + Hibernate)

```java
// Con JPA — limpio, simple, sin SQL manual
solicitudRepository.save(solicitud);
```

Una sola línea. El ORM se encarga del resto.

### ¿Cuándo NO usar ORM?

Un ORM no siempre es la solución ideal. Hay casos donde SQL directo es mejor:

| Situación | ¿Usar ORM? |
|-----------|------------|
| CRUD simple sobre pocos registros | ✅ Sí, ORM es perfecto |
| Consultas complejas con muchos JOINs | ⚠️ Considera SQL nativo o JPQL |
| Reportes con millones de filas | ❌ Usa SQL nativo o herramientas como jOOQ |
| Lógica de base de datos muy optimizada | ❌ SQL nativo es más eficiente |
| Operaciones masivas (bulk inserts) | ⚠️ ORM puede ser lento; considera batch |

En el proyecto de Triage, el ORM es perfectamente adecuado porque nuestras operaciones son mayoritariamente CRUD con relaciones manejables.

---

## 4. JPA vs Hibernate: diferencias y relación

Esta es una confusión muy común. Aclarémosla de una vez.

### JPA — La especificación

**JPA (Jakarta Persistence API)** es una **especificación** — un conjunto de interfaces, anotaciones y contratos que definen cómo debe funcionar el ORM en Java. JPA no es un framework ni una librería; es un estándar documentado.

Piénsalo así: JPA es como el **plano arquitectónico** de una casa. Define cómo debe ser la casa, pero no es la casa en sí.

Las anotaciones que usarás (`@Entity`, `@Table`, `@Id`, `@OneToMany`, etc.) son parte de la especificación JPA, que vive en el paquete `jakarta.persistence.*`.

### Hibernate — La implementación

**Hibernate** es una **implementación** de la especificación JPA. Es el código real que ejecuta el trabajo. Traduce tus anotaciones JPA en SQL, gestiona la caché, maneja las transacciones, etc.

Siguiendo la analogía: Hibernate es la **constructora** que toma el plano (JPA) y construye la casa real.

Existen otras implementaciones de JPA (EclipseLink, OpenJPA), pero Hibernate es por lejos la más usada y es la que Spring Boot incluye por defecto.

```
Tu código Java
      ↓
  Anotaciones JPA (jakarta.persistence.*)
      ↓
  Hibernate (implementa JPA)
      ↓
  JDBC Driver (conecta con la BD)
      ↓
  PostgreSQL
```

### ¿Cuál importo en mi código?

Siempre importa desde `jakarta.persistence.*` (JPA), **no** desde `org.hibernate.*`. Así tu código es portátil y no dependiente de Hibernate específicamente.

```java
// ✅ Correcto — importar desde JPA (independiente de implementación)
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.OneToMany;

// ❌ Evitar — importar directamente de Hibernate (acoplamiento innecesario)
import org.hibernate.annotations.SQLDelete; // solo si NO hay alternativa en JPA
```

---

## 5. Configurar la conexión a PostgreSQL

### Dependencias necesarias

En tu `pom.xml` debes tener estas dependencias. Si creaste el proyecto con Spring Initializr seleccionando "Spring Data JPA" y "PostgreSQL Driver", ya las tienes:

```xml
<!-- pom.xml -->
<dependencies>
    <!-- Spring Data JPA (incluye Hibernate) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- Driver JDBC de PostgreSQL -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
    
    <!-- Flyway para migraciones -->
    <dependency>
        <groupId>org.flywaydb</groupId>
        <artifactId>flyway-core</artifactId>
    </dependency>
    <dependency>
        <groupId>org.flywaydb</groupId>
        <artifactId>flyway-database-postgresql</artifactId>
    </dependency>
    
    <!-- Bean Validation -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
</dependencies>
```

### Configuración en application.yml

Crea o modifica tu archivo `src/main/resources/application.yml`. Vamos a configurarlo con perfiles para desarrollo y producción:

```yaml
# src/main/resources/application.yml
# Configuración base (compartida por todos los perfiles)

spring:
  application:
    name: triage-backend

  # Configuración de JPA/Hibernate
  jpa:
    hibernate:
      # validate: valida que el esquema de BD coincida con tus entidades (recomendado con Flyway)
      # update: Hibernate modifica el esquema automáticamente (útil solo para pruebas rápidas)
      # create-drop: crea el esquema al iniciar y lo borra al detener (solo para tests)
      # none: no hace nada (cuando usas Flyway, aquí va "validate")
      ddl-auto: validate
    show-sql: false          # true solo para depurar consultas SQL
    open-in-view: false      # ⚠️ IMPORTANTE: desactivar siempre (ver explicación abajo)
    properties:
      hibernate:
        format_sql: true               # Formatea el SQL si show-sql está activo
        dialect: org.hibernate.dialect.PostgreSQLDialect
        jdbc:
          batch_size: 25               # Para operaciones batch
        order_inserts: true            # Optimiza el batch
        order_updates: true

  # Flyway
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true

---
# Perfil de desarrollo
spring:
  config:
    activate:
      on-profile: dev

  datasource:
    url: jdbc:postgresql://localhost:5432/triage_db
    username: triage
    password: triage123
    driver-class-name: org.postgresql.Driver
    hikari:
      maximum-pool-size: 5
      minimum-idle: 2
      connection-timeout: 30000
      idle-timeout: 600000

  jpa:
    show-sql: true   # En dev sí mostramos el SQL

---
# Perfil de producción
spring:
  config:
    activate:
      on-profile: prod

  datasource:
    url: ${DATABASE_URL}       # Variable de entorno
    username: ${DATABASE_USER}
    password: ${DATABASE_PASSWORD}
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5

  jpa:
    show-sql: false
```

> **¿Qué es `open-in-view: false`?**
>
> Por defecto, Spring Boot activa `open-in-view: true`. Esto mantiene la sesión de Hibernate abierta durante todo el procesamiento de la petición HTTP (incluyendo la serialización de la respuesta). El problema es que puede causar consultas SQL inesperadas mientras se serializa el JSON, lo que hace el comportamiento difícil de predecir. **Siempre ponlo en `false`** y maneja las consultas explícitamente en tu capa de servicio.

### Activar el perfil de desarrollo

Para correr la aplicación con el perfil `dev`:

```bash
# Con Maven
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev

# O como variable de entorno
SPRING_PROFILES_ACTIVE=dev ./mvnw spring-boot:run
```

También puedes configurarlo en el IDE (IntelliJ IDEA → Edit Configurations → Active profiles: `dev`).

---

## 6. Entidades JPA: mapear clases a tablas

Una **entidad JPA** es una clase Java que se mapea a una tabla de la base de datos. Cada instancia de la clase corresponde a una fila de la tabla.

### Anatomía de una entidad básica

Vamos a crear la entidad más sencilla del Triage: `TipoSolicitud`. En nuestro sistema, un tipo de solicitud define la categoría (Homologación, Cupo, Cancelación, etc.).

```
src/main/java/
└── co/edu/uniquindio/triage/
    ├── domain/
    │   ├── entity/
    │   │   ├── TipoSolicitud.java      ← aquí
    │   │   ├── Solicitud.java
    │   │   ├── Usuario.java
    │   │   ├── Rol.java
    │   │   └── HistorialSolicitud.java
    │   └── enums/
    │       ├── EstadoSolicitud.java
    │       ├── Prioridad.java
    │       └── CanalOrigen.java
    └── ...
```

```java
// src/main/java/co/edu/uniquindio/triage/domain/entity/TipoSolicitud.java

package co.edu.uniquindio.triage.domain.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

@Entity                             // (1) Esta clase es una entidad JPA
@Table(
    name = "tipos_solicitud",       // (2) Nombre de la tabla en la BD
    indexes = {
        @Index(name = "idx_tipo_solicitud_nombre",
               columnList = "nombre",
               unique = true)       // (3) Índice único sobre la columna nombre
    }
)
public class TipoSolicitud {

    @Id                             // (4) Esta es la llave primaria
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // (5) Autoincremental
    private Long id;

    @NotBlank(message = "El nombre del tipo de solicitud es obligatorio")
    @Size(max = 100, message = "El nombre no puede superar 100 caracteres")
    @Column(name = "nombre",
            nullable = false,       // (6) NOT NULL en BD
            length = 100,           // Longitud de la columna VARCHAR
            unique = true)          // Restricción UNIQUE en BD
    private String nombre;

    @Size(max = 500)
    @Column(name = "descripcion", length = 500)
    private String descripcion;

    @Column(name = "activo", nullable = false)
    private boolean activo = true;  // (7) Valor por defecto

    // (8) Constructor vacío requerido por JPA
    protected TipoSolicitud() {}

    // Constructor con los campos obligatorios
    public TipoSolicitud(String nombre, String descripcion) {
        this.nombre = nombre;
        this.descripcion = descripcion;
        this.activo = true;
    }

    // (9) Getters y setters
    public Long getId() { return id; }

    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }

    public String getDescripcion() { return descripcion; }
    public void setDescripcion(String descripcion) { this.descripcion = descripcion; }

    public boolean isActivo() { return activo; }
    public void setActivo(boolean activo) { this.activo = activo; }

    // (10) equals y hashCode basados en el ID
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof TipoSolicitud other)) return false;
        return id != null && id.equals(other.id);
    }

    @Override
    public int hashCode() {
        return getClass().hashCode();
    }

    @Override
    public String toString() {
        return "TipoSolicitud{id=%d, nombre='%s'}".formatted(id, nombre);
    }
}
```

### Explicación anotación por anotación

**(1) `@Entity`**
Marca la clase como entidad JPA. Sin esta anotación, Hibernate la ignora por completo. Es obligatoria.

**(2) `@Table(name = "tipos_solicitud")`**
Define el nombre de la tabla. Si omites `@Table`, JPA usa el nombre de la clase (en este caso `TipoSolicitud`, que puede o no coincidir con la convención de tu BD). **Buena práctica:** siempre especificar el nombre de tabla explícitamente con `snake_case`.

**(3) `@Index`**
Crea un índice en la base de datos. Los índices aceleran las búsquedas pero ocupan espacio y ralentizan los INSERT/UPDATE. Úsalos en columnas que consultas frecuentemente o que necesitan ser únicas.

**(4) `@Id`**
Marca el campo como llave primaria. Toda entidad JPA debe tener exactamente un campo con `@Id`.

**(5) `@GeneratedValue`**
Indica cómo se genera el ID automáticamente. Los detalles de las estrategias están en la sección 7.

**(6) `@Column`** — atributos más importantes:

| Atributo | Qué hace | Ejemplo |
|----------|----------|---------|
| `name` | Nombre de la columna en BD | `name = "fecha_registro"` |
| `nullable` | Permite NULL en la BD | `nullable = false` |
| `length` | Longitud para VARCHAR | `length = 255` |
| `unique` | Restricción UNIQUE | `unique = true` |
| `precision` | Dígitos totales para DECIMAL | `precision = 10` |
| `scale` | Decimales para DECIMAL | `scale = 2` |
| `columnDefinition` | DDL personalizado | `columnDefinition = "TEXT"` |
| `insertable` | ¿Se incluye en INSERT? | `insertable = false` |
| `updatable` | ¿Se incluye en UPDATE? | `updatable = false` |

**(7) Valores por defecto**
En Java puedes asignar valores por defecto directamente. Estos se aplican cuando creas una instancia pero no cuando Hibernate carga de BD (en ese caso ya viene el valor de la tabla).

**(8) Constructor vacío**
JPA **requiere** un constructor sin argumentos para poder instanciar la entidad al cargarla de la BD. Puedes hacerlo `protected` para que no lo use código externo, pero Hibernate sí puede accederlo.

**(9) Getters y setters**
JPA necesita acceso a los campos. Puedes usar getters/setters (acceso por propiedad) o directamente los campos (acceso por campo con `@Id` en el field). Usar getters/setters es más común y predecible.

**(10) `equals` y `hashCode`**
Este tema es delicado con JPA. La regla más segura es:
- Si el ID es `null` (entidad no persistida aún), dos entidades del mismo tipo son iguales solo si son el mismo objeto en memoria (`this == o`).
- Si el ID no es `null`, la igualdad se basa en el ID.
- El `hashCode` debe ser **constante** (no basado en el ID) para que las entidades funcionen bien en colecciones cuando su ID pasa de `null` a un valor.

---

## 7. Estrategias de generación de ID

### `GenerationType.IDENTITY` — Autoincremental de la BD

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

- La base de datos genera el ID usando `SERIAL` o `IDENTITY`.
- **Pros:** Simple, familiar, compatible con todas las BDs.
- **Contras:** Hibernate no puede hacer batch inserts optimizados porque necesita hacer el INSERT para obtener el ID.
- **Cuándo usar:** La opción más común para proyectos típicos.

La columna en PostgreSQL será `BIGSERIAL` o `GENERATED ALWAYS AS IDENTITY`.

### `GenerationType.SEQUENCE` — Secuencias de la BD

```java
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "solicitud_seq")
@SequenceGenerator(
    name = "solicitud_seq",
    sequenceName = "seq_solicitudes",
    allocationSize = 50     // Pre-asigna 50 IDs a la vez (mejor rendimiento)
)
private Long id;
```

- Usa una secuencia de la BD (`CREATE SEQUENCE`).
- **Pros:** Permite batch inserts optimizados (Hibernate pre-asigna IDs).
- **Contras:** Más configuración.
- **Cuándo usar:** Cuando necesitas alto rendimiento en inserciones masivas.

### UUID — Identificadores únicos globales

```java
import java.util.UUID;

@Id
@GeneratedValue(strategy = GenerationType.UUID)
@Column(name = "id", updatable = false, nullable = false)
private UUID id;
```

- Spring Boot 3+ / Hibernate 6+ tiene soporte nativo para UUID con `GenerationType.UUID`.
- **Pros:** Los IDs son únicos globalmente, no son secuenciales (más difícil de predecir), útil para sistemas distribuidos o APIs públicas.
- **Contras:** Ocupa más espacio (16 bytes vs 8 de Long), los índices son ligeramente más lentos, las URLs quedan más largas.
- **Cuándo usar:** Cuando expondrás el ID en URLs públicas (por seguridad, no exponer IDs secuenciales) o en sistemas distribuidos.

### Tabla comparativa de estrategias

| Estrategia | Tipo Java | Generado por | Batch inserts | Cuándo usar |
|------------|-----------|--------------|---------------|-------------|
| `IDENTITY` | `Long` | Base de datos | ❌ No | Casos generales, simplicidad |
| `SEQUENCE` | `Long` | BD (secuencia) | ✅ Sí | Alto volumen de inserciones |
| `UUID` | `UUID` | Hibernate/BD | ✅ Sí | APIs públicas, sistemas distribuidos |

> **Para el proyecto de Triage** usaremos `IDENTITY` para la mayoría de entidades por simplicidad, y `UUID` para la entidad `Usuario` ya que su ID puede exponerse en tokens JWT.

---

## 8. Relaciones entre entidades

Las relaciones son donde JPA se vuelve realmente poderoso (y también donde más errores se cometen). Vamos con calma.

### 8.1 `@ManyToOne` y `@OneToMany` — La relación más común

En el Triage, **una solicitud pertenece a un tipo** y **un tipo puede tener muchas solicitudes**. Esto es una relación muchos-a-uno (desde Solicitud) y uno-a-muchos (desde TipoSolicitud).

```
TipoSolicitud (1) ←─────────── (N) Solicitud
    "Homologación"         solicitud_1
                           solicitud_2
                           solicitud_3
```

**Lado "muchos" — la entidad que tiene la llave foránea:**

```java
// En Solicitud.java — el lado @ManyToOne tiene la columna FK
@ManyToOne(
    fetch = FetchType.LAZY,         // (A) No cargues el tipo hasta que se necesite
    optional = false                // (B) La solicitud DEBE tener un tipo
)
@JoinColumn(
    name = "tipo_solicitud_id",     // (C) Nombre de la columna FK en la tabla solicitudes
    nullable = false
)
private TipoSolicitud tipoSolicitud;
```

**Lado "uno" — la entidad que tiene la colección:**

```java
// En TipoSolicitud.java — el lado @OneToMany NO tiene columna FK propia
@OneToMany(
    mappedBy = "tipoSolicitud",     // (D) El nombre del campo en Solicitud
    fetch = FetchType.LAZY,         // SIEMPRE LAZY en @OneToMany
    cascade = CascadeType.ALL       // (E) Las operaciones se propagan
)
private List<Solicitud> solicitudes = new ArrayList<>();
```

### Explicación de los parámetros clave

**(A) `FetchType.LAZY` vs `FetchType.EAGER`** — el error más frecuente

Este es el concepto más importante de todo JPA. Presta mucha atención.

- **LAZY (perezoso):** Hibernate NO carga el objeto relacionado hasta que realmente lo accedes en tu código. Solo carga un "proxy" (objeto falso). Cuando accedes al atributo por primera vez, **en ese momento** va a la BD y lo carga.

- **EAGER (ansioso):** Hibernate carga el objeto relacionado INMEDIATAMENTE junto con la entidad principal, siempre, aunque no lo necesites.

```java
// Con EAGER:
Solicitud s = solicitudRepository.findById(1L).get();
// ↑ Este select ya trajo el TipoSolicitud, el Usuario, etc. aunque no los uses

// Con LAZY:
Solicitud s = solicitudRepository.findById(1L).get();
// ↑ Solo trajo la solicitud. El tipoSolicitud es un proxy vacío.

String nombreTipo = s.getTipoSolicitud().getNombre();
// ↑ AQUÍ Hibernate hace un SELECT adicional para cargar el TipoSolicitud
```

**Regla:** Usa `LAZY` siempre, especialmente en `@OneToMany`. Usa EAGER solo si puedes demostrar que siempre necesitas el dato relacionado.

**(B) `optional = false`**
Le dice a JPA que la relación es obligatoria. Se traduce en `INNER JOIN` en lugar de `LEFT JOIN` en las consultas, lo que puede ser más eficiente.

**(C) `@JoinColumn(name = "tipo_solicitud_id")`**
El nombre de la columna de llave foránea en la tabla `solicitudes`. Si lo omites, JPA genera un nombre automático que puede no seguir tus convenciones.

**(D) `mappedBy = "tipoSolicitud"`**
Indica que el lado `@ManyToOne` en `Solicitud` (el campo llamado `tipoSolicitud`) es el "dueño" de la relación y tiene la columna FK. Sin `mappedBy`, JPA crearía una tabla de unión intermedia innecesaria.

**(E) `cascade = CascadeType.ALL`**
Las operaciones (persist, merge, remove, refresh, detach) se propagan al lado hijo. Úsalo con cuidado en `@OneToMany`: si cascadeas `REMOVE`, al eliminar un tipo de solicitud eliminarías todas sus solicitudes.

Tipos de cascade:

| Tipo | Qué hace |
|------|----------|
| `PERSIST` | Al guardar el padre, guarda los hijos nuevos |
| `MERGE` | Al actualizar el padre, actualiza los hijos |
| `REMOVE` | Al eliminar el padre, elimina los hijos |
| `REFRESH` | Al refrescar el padre, refresca los hijos |
| `DETACH` | Al desconectar el padre, desconecta los hijos |
| `ALL` | Todos los anteriores |

---

### 8.2 El problema N+1 y cómo solucionarlo

Este es el problema de rendimiento más común con JPA y DEBES entenderlo.

**El escenario:** Quieres mostrar una lista de 100 solicitudes, cada una con su tipo de solicitud.

**¿Qué pasa con LAZY sin precaución?**

```java
// En el servicio
List<Solicitud> solicitudes = solicitudRepository.findAll(); // 1 SELECT

for (Solicitud s : solicitudes) {
    System.out.println(s.getTipoSolicitud().getNombre()); 
    // ↑ Cada acceso dispara 1 SELECT adicional para cargar el TipoSolicitud
    // Con 100 solicitudes → 100 SELECTs adicionales
}
// Total: 1 + 100 = 101 SELECTs → PROBLEMA N+1
```

**Solución 1: JOIN FETCH en JPQL**

```java
// En el Repository
@Query("SELECT s FROM Solicitud s JOIN FETCH s.tipoSolicitud WHERE s.activo = true")
List<Solicitud> findAllWithTipoSolicitud();
```

Esto genera un solo SELECT con JOIN: trae las solicitudes y sus tipos en una sola consulta.

**Solución 2: `@EntityGraph`**

```java
// En el Repository — más elegante
@EntityGraph(attributePaths = {"tipoSolicitud", "usuarioSolicitante"})
List<Solicitud> findByActivo(boolean activo);
```

`@EntityGraph` le dice a JPA qué relaciones cargar de forma "eager" solo para esa consulta específica, sin cambiar la configuración global de la entidad.

**Regla de oro:** Nunca iteres sobre una colección de entidades y accedas a relaciones LAZY dentro del bucle, a menos que hayas hecho un JOIN FETCH o EntityGraph previamente.

---

### 8.3 `@ManyToMany` — Relación muchos a muchos

En el Triage, un `Usuario` puede tener múltiples `Rol`es, y un `Rol` puede ser asignado a múltiples usuarios.

```java
// En Usuario.java
@ManyToMany(fetch = FetchType.LAZY)
@JoinTable(
    name = "usuarios_roles",                    // Tabla intermedia
    joinColumns = @JoinColumn(name = "usuario_id"),
    inverseJoinColumns = @JoinColumn(name = "rol_id")
)
private Set<Rol> roles = new HashSet<>();
```

```java
// En Rol.java
@ManyToMany(mappedBy = "roles", fetch = FetchType.LAZY)
private Set<Usuario> usuarios = new HashSet<>();
```

JPA creará automáticamente la tabla intermedia `usuarios_roles` con dos columnas FK.

> **Consejo:** Usa `Set<T>` en lugar de `List<T>` para relaciones `@ManyToMany`. Hibernate maneja los sets más eficientemente para esta relación (evita `HibernateJpaDialect` duplicates).

---

### 8.4 `@OneToOne` — Relación uno a uno

Aunque no es tan común en el Triage, la incluimos por completitud. Un ejemplo sería si cada solicitud tuviera un objeto de configuración exclusivo:

```java
// En Solicitud.java
@OneToOne(
    mappedBy = "solicitud",
    cascade = CascadeType.ALL,
    fetch = FetchType.LAZY,
    optional = true
)
private ConfiguracionSolicitud configuracion;
```

```java
// En ConfiguracionSolicitud.java
@OneToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "solicitud_id", unique = true)
private Solicitud solicitud;
```

---

## 9. Enums en JPA

En el proyecto de Triage usamos enums para `EstadoSolicitud`, `Prioridad` y `CanalOrigen`. JPA puede mapear enums de dos formas:

### `@Enumerated(EnumType.ORDINAL)` — NUNCA uses esto

```java
// ❌ MAL — guarda el índice numérico (0, 1, 2, ...)
@Enumerated(EnumType.ORDINAL)
private EstadoSolicitud estado;
```

Si tienes el enum `[REGISTRADA=0, CLASIFICADA=1, EN_ATENCION=2]` y después insertas un nuevo valor al inicio o en el medio, todos los registros quedan con el estado incorrecto. Es una bomba de tiempo.

### `@Enumerated(EnumType.STRING)` — Siempre usa esto

```java
// ✅ BIEN — guarda el nombre del enum como texto
@Enumerated(EnumType.STRING)
@Column(name = "estado", nullable = false, length = 50)
private EstadoSolicitud estado;
```

La columna almacenará el texto `"REGISTRADA"`, `"CLASIFICADA"`, etc. Puedes agregar, reordenar o renombrar valores del enum sin afectar los datos existentes (siempre que mantengas los nombres).

### Definición de los enums del Triage

```java
// src/main/java/co/edu/uniquindio/triage/domain/enums/EstadoSolicitud.java

package co.edu.uniquindio.triage.domain.enums;

public enum EstadoSolicitud {
    REGISTRADA,
    CLASIFICADA,
    EN_ATENCION,
    ATENDIDA,
    CERRADA;

    // Método de utilidad para verificar transiciones válidas
    public boolean puedeTransicionarA(EstadoSolicitud nuevoEstado) {
        return switch (this) {
            case REGISTRADA   -> nuevoEstado == CLASIFICADA;
            case CLASIFICADA  -> nuevoEstado == EN_ATENCION;
            case EN_ATENCION  -> nuevoEstado == ATENDIDA;
            case ATENDIDA     -> nuevoEstado == CERRADA;
            case CERRADA      -> false; // Estado terminal
        };
    }
}
```

```java
// src/main/java/co/edu/uniquindio/triage/domain/enums/Prioridad.java

package co.edu.uniquindio.triage.domain.enums;

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
// src/main/java/co/edu/uniquindio/triage/domain/enums/CanalOrigen.java

package co.edu.uniquindio.triage.domain.enums;

public enum CanalOrigen {
    PRESENCIAL,
    CORREO_ELECTRONICO,
    SISTEMA_ACADEMICO,
    TELEFONICO,
    PLATAFORMA_WEB
}
```

---

## 10. Auditoría automática con Spring Data

En el proyecto de Triage necesitamos saber cuándo se creó y modificó cada registro, y quién lo hizo. Spring Data puede hacer esto automáticamente.

### Configurar la auditoría

**Paso 1:** Habilitar la auditoría en la clase de configuración principal:

```java
// src/main/java/co/edu/uniquindio/triage/TriageBackendApplication.java

package co.edu.uniquindio.triage;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

@SpringBootApplication
@EnableJpaAuditing(auditorAwareRef = "auditorProvider")  // Habilita la auditoría
public class TriageBackendApplication {
    public static void main(String[] args) {
        SpringApplication.run(TriageBackendApplication.class, args);
    }
}
```

**Paso 2:** Crear el `AuditorAware` que le dice a Spring quién es el usuario actual:

```java
// src/main/java/co/edu/uniquindio/triage/config/AuditorConfig.java

package co.edu.uniquindio.triage.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.domain.AuditorAware;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;

import java.util.Optional;

@Configuration
public class AuditorConfig {

    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> {
            Authentication auth = SecurityContextHolder.getContext().getAuthentication();
            if (auth == null || !auth.isAuthenticated() || 
                auth.getPrincipal().equals("anonymousUser")) {
                return Optional.of("sistema");  // Si no hay usuario autenticado
            }
            return Optional.of(auth.getName()); // El username del usuario actual
        };
    }
}
```

**Paso 3:** Crear la clase base con los campos de auditoría que heredarán todas las entidades:

```java
// src/main/java/co/edu/uniquindio/triage/domain/entity/AuditableEntity.java

package co.edu.uniquindio.triage.domain.entity;

import jakarta.persistence.*;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.LocalDateTime;

@MappedSuperclass   // (1) Esta clase no es una entidad propia, es una clase base
@EntityListeners(AuditingEntityListener.class)  // (2) Activa los listeners de auditoría
public abstract class AuditableEntity {

    @CreatedDate                           // (3) Se rellena automáticamente al crear
    @Column(name = "fecha_creacion",
            nullable = false,
            updatable = false)             // Nunca se modifica después de creado
    private LocalDateTime fechaCreacion;

    @LastModifiedDate                      // (4) Se actualiza en cada modificación
    @Column(name = "fecha_modificacion")
    private LocalDateTime fechaModificacion;

    @CreatedBy                             // (5) Usuario que creó el registro
    @Column(name = "creado_por",
            nullable = false,
            updatable = false,
            length = 100)
    private String creadoPor;

    @LastModifiedBy                        // (6) Último usuario que modificó
    @Column(name = "modificado_por", length = 100)
    private String modificadoPor;

    // Getters (sin setters — la auditoría los gestiona Spring)
    public LocalDateTime getFechaCreacion() { return fechaCreacion; }
    public LocalDateTime getFechaModificacion() { return fechaModificacion; }
    public String getCreadoPor() { return creadoPor; }
    public String getModificadoPor() { return modificadoPor; }
}
```

### Explicación

**(1) `@MappedSuperclass`**
Le dice a JPA que esta clase no tiene su propia tabla, sino que sus columnas se agregan a las tablas de las clases que la heredan. Es diferente de `@Entity` con herencia.

**(2) `@EntityListeners(AuditingEntityListener.class)`**
Registra el listener de Spring que intercepta los eventos de JPA (prePersist, preUpdate) y rellena automáticamente los campos de auditoría.

**(3) `@CreatedDate`** — Solo se rellena cuando se hace el primer INSERT. La columna tiene `updatable = false` para garantizar que nunca cambie.

**(4) `@LastModifiedDate`** — Se actualiza en cada UPDATE automáticamente.

**(5) y (6)** — `@CreatedBy` y `@LastModifiedBy` usan el `AuditorAware<String>` que configuraste para obtener el nombre del usuario actual.

---

## 11. Spring Data JPA Repositories

Un **Repository** en Spring Data JPA es una interfaz que extiende `JpaRepository<Entidad, TipoDelId>`. Spring genera automáticamente la implementación en tiempo de ejecución — tú no escribes ningún SQL para las operaciones básicas.

### JpaRepository — lo que obtienes gratis

```java
// Al extender JpaRepository<Solicitud, Long>, ya tienes:
solicitudRepository.save(solicitud);                // INSERT o UPDATE
solicitudRepository.findById(1L);                   // SELECT por ID
solicitudRepository.findAll();                      // SELECT todos
solicitudRepository.findAll(pageable);              // SELECT con paginación
solicitudRepository.deleteById(1L);                 // DELETE por ID
solicitudRepository.existsById(1L);                 // EXISTS por ID
solicitudRepository.count();                        // COUNT(*)
solicitudRepository.saveAll(listaDeSolicitudes);    // Batch INSERT/UPDATE
```

### Métodos derivados por nombre

Spring Data puede generar consultas SQL simplemente leyendo el nombre del método. La sintaxis sigue patrones específicos:

```java
// src/main/java/co/edu/uniquindio/triage/domain/repository/SolicitudRepository.java

package co.edu.uniquindio.triage.domain.repository;

import co.edu.uniquindio.triage.domain.entity.Solicitud;
import co.edu.uniquindio.triage.domain.enums.EstadoSolicitud;
import co.edu.uniquindio.triage.domain.enums.Prioridad;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public interface SolicitudRepository extends JpaRepository<Solicitud, Long> {

    // ── Métodos derivados por nombre ─────────────────────────────────────

    // SELECT * FROM solicitudes WHERE estado = ?
    List<Solicitud> findByEstado(EstadoSolicitud estado);

    // SELECT * FROM solicitudes WHERE estado = ? AND prioridad = ?
    List<Solicitud> findByEstadoAndPrioridad(EstadoSolicitud estado, Prioridad prioridad);

    // SELECT * FROM solicitudes WHERE usuario_solicitante_id = ?
    List<Solicitud> findByUsuarioSolicitanteId(Long usuarioId);

    // SELECT * FROM solicitudes WHERE responsable_id = ?
    Page<Solicitud> findByResponsableId(Long responsableId, Pageable pageable);

    // SELECT COUNT(*) FROM solicitudes WHERE estado = ?
    long countByEstado(EstadoSolicitud estado);

    // SELECT * FROM solicitudes WHERE estado != ?
    List<Solicitud> findByEstadoNot(EstadoSolicitud estado);

    // ── Consultas JPQL con @Query ────────────────────────────────────────

    // JPQL usa nombres de entidades y campos Java, no tablas y columnas SQL
    @Query("""
            SELECT s FROM Solicitud s
            JOIN FETCH s.tipoSolicitud
            JOIN FETCH s.usuarioSolicitante
            WHERE s.estado = :estado
            ORDER BY s.fechaCreacion DESC
            """)
    List<Solicitud> findByEstadoConDetalles(@Param("estado") EstadoSolicitud estado);

    // Consulta con múltiples filtros opcionales (usando COALESCE / IS NULL)
    @Query("""
            SELECT s FROM Solicitud s
            WHERE (:estado IS NULL OR s.estado = :estado)
              AND (:prioridad IS NULL OR s.prioridad = :prioridad)
              AND (:tipoSolicitudId IS NULL OR s.tipoSolicitud.id = :tipoSolicitudId)
            """)
    Page<Solicitud> buscarConFiltros(
            @Param("estado") EstadoSolicitud estado,
            @Param("prioridad") Prioridad prioridad,
            @Param("tipoSolicitudId") Long tipoSolicitudId,
            Pageable pageable
    );

    // ── Consulta SQL nativa con @Query(nativeQuery = true) ───────────────

    // Usa cuando necesitas SQL específico de PostgreSQL que JPQL no soporta
    @Query(
        value = """
            SELECT s.*, u.nombre as nombre_solicitante
            FROM solicitudes s
            INNER JOIN usuarios u ON s.usuario_solicitante_id = u.id
            WHERE s.fecha_creacion >= NOW() - INTERVAL '30 days'
            ORDER BY s.fecha_creacion DESC
            LIMIT 10
            """,
        nativeQuery = true
    )
    List<Object[]> findUltimasSolicitudesRecientes();
}
```

### Palabras clave para métodos derivados

| Keyword | Operación SQL | Ejemplo |
|---------|---------------|---------|
| `findBy` | `SELECT WHERE` | `findByEstado` |
| `countBy` | `SELECT COUNT WHERE` | `countByEstado` |
| `existsBy` | `SELECT EXISTS WHERE` | `existsByEmail` |
| `deleteBy` | `DELETE WHERE` | `deleteByEstado` |
| `And` | `AND` | `findByEstadoAndPrioridad` |
| `Or` | `OR` | `findByEstadoOrPrioridad` |
| `Not` | `!=` | `findByEstadoNot` |
| `IsNull` | `IS NULL` | `findByResponsableIsNull` |
| `IsNotNull` | `IS NOT NULL` | `findByResponsableIsNotNull` |
| `Like` | `LIKE` | `findByDescripcionLike` |
| `Containing` | `LIKE %?%` | `findByDescripcionContaining` |
| `StartingWith` | `LIKE ?%` | `findByNombreStartingWith` |
| `IgnoreCase` | `LOWER()` | `findByNombreIgnoreCase` |
| `OrderBy` | `ORDER BY` | `findByEstadoOrderByFechaCreacionDesc` |
| `Top` / `First` | `LIMIT` | `findTop5ByEstado` |
| `In` | `IN (...)` | `findByEstadoIn(List<EstadoSolicitud>)` |

---

## 12. Paginación y ordenamiento

Devolver miles de registros de una vez es ineficiente y puede hacer colapsar tu API. Spring Data JPA tiene soporte nativo para paginación.

### Recibir paginación en el servicio

```java
// En el servicio
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;

// Crear un Pageable: página 0, 20 elementos por página, ordenado por fechaCreacion DESC
Pageable pageable = PageRequest.of(
    0,                                          // página (base 0)
    20,                                         // elementos por página
    Sort.by(Sort.Direction.DESC, "fechaCreacion")
);

Page<Solicitud> resultado = solicitudRepository.findByEstado(
    EstadoSolicitud.REGISTRADA, 
    pageable
);

// El objeto Page<T> contiene:
resultado.getContent();          // List<Solicitud> — los datos de esta página
resultado.getTotalElements();    // Long — total de registros que coinciden
resultado.getTotalPages();       // int — total de páginas
resultado.getNumber();           // int — número de página actual
resultado.getSize();             // int — tamaño de página
resultado.isFirst();             // boolean
resultado.isLast();              // boolean
resultado.hasNext();             // boolean
resultado.hasPrevious();         // boolean
```

### Exponer paginación en la API REST

Cuando expongas la paginación en tu Controller (Guía 06), recibirás los parámetros como `@RequestParam` o directamente como `Pageable`:

```java
// En el Controller
@GetMapping("/solicitudes")
public ResponseEntity<Page<SolicitudResponse>> listar(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size,
    @RequestParam(defaultValue = "fechaCreacion") String sortBy,
    @RequestParam(defaultValue = "DESC") String direction
) {
    var sort = Sort.by(Sort.Direction.fromString(direction), sortBy);
    var pageable = PageRequest.of(page, size, sort);
    // ...
}
```

---

## 13. Validaciones con Bean Validation

Bean Validation (JSR-380) es el estándar Java para validar objetos. Con la anotación `@Valid` en tu Controller, Spring activa las validaciones automáticamente.

### Anotaciones más usadas

```java
import jakarta.validation.constraints.*;

public class CrearSolicitudRequest {

    @NotNull(message = "El tipo de solicitud es obligatorio")
    private Long tipoSolicitudId;

    @NotBlank(message = "La descripción no puede estar vacía")
    @Size(min = 10, max = 2000, message = "La descripción debe tener entre 10 y 2000 caracteres")
    private String descripcion;

    @NotNull(message = "El canal de origen es obligatorio")
    private CanalOrigen canalOrigen;

    @NotBlank(message = "La identificación del solicitante es obligatoria")
    @Pattern(regexp = "^[0-9]{8,12}$", message = "La identificación debe tener entre 8 y 12 dígitos")
    private String identificacionSolicitante;

    @Email(message = "El correo electrónico no tiene formato válido")
    private String correoContacto;

    @Future(message = "La fecha límite debe ser en el futuro")
    private LocalDate fechaLimite;

    @Min(value = 1, message = "El valor mínimo es 1")
    @Max(value = 100, message = "El valor máximo es 100")
    private Integer prioridad;

    @Positive(message = "El valor debe ser positivo")
    private BigDecimal monto;
}
```

### Tabla de anotaciones de validación

| Anotación | Qué valida |
|-----------|------------|
| `@NotNull` | El valor no es `null` |
| `@NotBlank` | No es `null`, no es vacío, no solo espacios |
| `@NotEmpty` | No es `null`, no es vacío (permite espacios) |
| `@Size(min, max)` | Longitud de String, tamaño de colección |
| `@Min(value)` | Número mayor o igual al valor |
| `@Max(value)` | Número menor o igual al valor |
| `@Positive` | Número estrictamente positivo (>0) |
| `@PositiveOrZero` | Número positivo o cero (>=0) |
| `@Negative` | Número negativo (<0) |
| `@Email` | Formato de email válido |
| `@Pattern(regexp)` | Expresión regular |
| `@Past` | Fecha en el pasado |
| `@Future` | Fecha en el futuro |
| `@PastOrPresent` | Fecha en el pasado o ahora |
| `@FutureOrPresent` | Fecha en el futuro o ahora |
| `@AssertTrue` | El valor booleano es `true` |
| `@AssertFalse` | El valor booleano es `false` |

### Validación personalizada

Cuando ninguna anotación estándar se ajusta a tu caso, puedes crear la tuya:

```java
// 1. Definir la anotación
@Documented
@Constraint(validatedBy = EstadoTransicionValida.class)
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface TransicionEstadoValida {
    String message() default "La transición de estado no es válida";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// 2. Implementar el validador
public class EstadoTransicionValida 
    implements ConstraintValidator<TransicionEstadoValida, CambiarEstadoRequest> {

    @Override
    public boolean isValid(CambiarEstadoRequest request, ConstraintValidatorContext ctx) {
        if (request.estadoActual() == null || request.nuevoEstado() == null) {
            return false;
        }
        return request.estadoActual().puedeTransicionarA(request.nuevoEstado());
    }
}
```

---

## 14. Modelo completo del proyecto Triage

Ahora vamos a construir todas las entidades del sistema. Primero, veamos el diagrama de relaciones:

```
                    ┌─────────────┐
                    │     Rol     │
                    │  (ADMIN,    │
                    │ FUNCIONARIO,│
                    │ ESTUDIANTE) │
                    └──────┬──────┘
                           │ @ManyToMany
                    ┌──────┴──────┐
                    │   Usuario   │
                    │  (id UUID)  │
                    └──┬───────┬──┘
                       │       │
          solicitante  │       │  responsable
                       │       │
               ┌───────┴───────┴───────┐
               │       Solicitud       │
               │  estado, prioridad,   │
               │  descripcion, canal   │
               └──────┬───────┬────────┘
                      │       │
             ┌────────┘       └────────┐
             │                         │
    ┌────────┴──────┐    ┌─────────────┴──────┐
    │ TipoSolicitud │    │ HistorialSolicitud  │
    │ (Homologación,│    │  (auditoría de cada │
    │  Cupos, etc.) │    │   cambio de estado) │
    └───────────────┘    └────────────────────┘
```

### Estructura de carpetas completa

```
src/main/java/co/edu/uniquindio/triage/
├── TriageBackendApplication.java
├── config/
│   ├── AuditorConfig.java
│   └── (SecurityConfig.java — Guía 07)
├── domain/
│   ├── entity/
│   │   ├── AuditableEntity.java     ← clase base abstracta
│   │   ├── TipoSolicitud.java
│   │   ├── Solicitud.java
│   │   ├── HistorialSolicitud.java
│   │   ├── Usuario.java
│   │   └── Rol.java
│   ├── enums/
│   │   ├── EstadoSolicitud.java
│   │   ├── Prioridad.java
│   │   └── CanalOrigen.java
│   └── repository/
│       ├── SolicitudRepository.java
│       ├── TipoSolicitudRepository.java
│       ├── HistorialSolicitudRepository.java
│       ├── UsuarioRepository.java
│       └── RolRepository.java
├── service/
│   ├── SolicitudService.java
│   └── (otros servicios — Guía 06)
└── web/
    └── (controllers, DTOs — Guía 06)
```

### Entidad `Rol`

```java
// src/main/java/co/edu/uniquindio/triage/domain/entity/Rol.java

package co.edu.uniquindio.triage.domain.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

@Entity
@Table(name = "roles")
public class Rol {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank
    @Size(max = 50)
    @Column(name = "nombre", nullable = false, unique = true, length = 50)
    private String nombre;  // "ROLE_ADMIN", "ROLE_FUNCIONARIO", "ROLE_ESTUDIANTE"

    @Size(max = 200)
    @Column(name = "descripcion", length = 200)
    private String descripcion;

    protected Rol() {}

    public Rol(String nombre, String descripcion) {
        this.nombre = nombre;
        this.descripcion = descripcion;
    }

    public Long getId() { return id; }
    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }
    public String getDescripcion() { return descripcion; }
    public void setDescripcion(String descripcion) { this.descripcion = descripcion; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Rol other)) return false;
        return id != null && id.equals(other.id);
    }

    @Override
    public int hashCode() { return getClass().hashCode(); }
}
```

### Entidad `Usuario`

```java
// src/main/java/co/edu/uniquindio/triage/domain/entity/Usuario.java

package co.edu.uniquindio.triage.domain.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

import java.util.HashSet;
import java.util.Set;
import java.util.UUID;

@Entity
@Table(
    name = "usuarios",
    indexes = {
        @Index(name = "idx_usuario_email", columnList = "email", unique = true),
        @Index(name = "idx_usuario_numero_identificacion",
               columnList = "numero_identificacion", unique = true)
    }
)
public class Usuario extends AuditableEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.UUID)  // UUID para no exponer IDs secuenciales
    @Column(name = "id", updatable = false, nullable = false)
    private UUID id;

    @NotBlank
    @Size(max = 150)
    @Column(name = "nombre", nullable = false, length = 150)
    private String nombre;

    @NotBlank
    @Email
    @Size(max = 100)
    @Column(name = "email", nullable = false, unique = true, length = 100)
    private String email;

    @NotBlank
    @Size(max = 20)
    @Column(name = "numero_identificacion", nullable = false, unique = true, length = 20)
    private String numeroIdentificacion;

    @NotBlank
    @Column(name = "password_hash", nullable = false)
    private String passwordHash;  // Almacena el hash BCrypt, nunca la contraseña en texto plano

    @Column(name = "activo", nullable = false)
    private boolean activo = true;

    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(
        name = "usuarios_roles",
        joinColumns = @JoinColumn(name = "usuario_id"),
        inverseJoinColumns = @JoinColumn(name = "rol_id")
    )
    private Set<Rol> roles = new HashSet<>();

    protected Usuario() {}

    public Usuario(String nombre, String email, String numeroIdentificacion, String passwordHash) {
        this.nombre = nombre;
        this.email = email;
        this.numeroIdentificacion = numeroIdentificacion;
        this.passwordHash = passwordHash;
    }

    // Método de utilidad para agregar un rol
    public void agregarRol(Rol rol) {
        this.roles.add(rol);
    }

    // Método de utilidad para remover un rol
    public void removerRol(Rol rol) {
        this.roles.remove(rol);
    }

    // Getters
    public UUID getId() { return id; }
    public String getNombre() { return nombre; }
    public String getEmail() { return email; }
    public String getNumeroIdentificacion() { return numeroIdentificacion; }
    public String getPasswordHash() { return passwordHash; }
    public boolean isActivo() { return activo; }
    public Set<Rol> getRoles() { return roles; }

    // Setters
    public void setNombre(String nombre) { this.nombre = nombre; }
    public void setEmail(String email) { this.email = email; }
    public void setPasswordHash(String passwordHash) { this.passwordHash = passwordHash; }
    public void setActivo(boolean activo) { this.activo = activo; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Usuario other)) return false;
        return id != null && id.equals(other.id);
    }

    @Override
    public int hashCode() { return getClass().hashCode(); }
}
```

### Entidad `TipoSolicitud` (versión completa)

```java
// src/main/java/co/edu/uniquindio/triage/domain/entity/TipoSolicitud.java

package co.edu.uniquindio.triage.domain.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

import java.util.ArrayList;
import java.util.List;

@Entity
@Table(
    name = "tipos_solicitud",
    indexes = {
        @Index(name = "idx_tipo_solicitud_nombre", columnList = "nombre", unique = true)
    }
)
public class TipoSolicitud extends AuditableEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank
    @Size(max = 100)
    @Column(name = "nombre", nullable = false, unique = true, length = 100)
    private String nombre;

    @Size(max = 500)
    @Column(name = "descripcion", length = 500)
    private String descripcion;

    // Prioridad sugerida por defecto para este tipo de solicitud
    @Column(name = "dias_atencion_estimados")
    private Integer diasAtencionEstimados;  // SLA estimado en días

    @Column(name = "activo", nullable = false)
    private boolean activo = true;

    // Relación inversa: no siempre necesitas este lado, inclúyelo solo si lo usas
    @OneToMany(mappedBy = "tipoSolicitud", fetch = FetchType.LAZY)
    private List<Solicitud> solicitudes = new ArrayList<>();

    protected TipoSolicitud() {}

    public TipoSolicitud(String nombre, String descripcion, Integer diasAtencionEstimados) {
        this.nombre = nombre;
        this.descripcion = descripcion;
        this.diasAtencionEstimados = diasAtencionEstimados;
    }

    // Getters y setters
    public Long getId() { return id; }
    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }
    public String getDescripcion() { return descripcion; }
    public void setDescripcion(String descripcion) { this.descripcion = descripcion; }
    public Integer getDiasAtencionEstimados() { return diasAtencionEstimados; }
    public void setDiasAtencionEstimados(Integer dias) { this.diasAtencionEstimados = dias; }
    public boolean isActivo() { return activo; }
    public void setActivo(boolean activo) { this.activo = activo; }
    public List<Solicitud> getSolicitudes() { return solicitudes; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof TipoSolicitud other)) return false;
        return id != null && id.equals(other.id);
    }

    @Override
    public int hashCode() { return getClass().hashCode(); }
}
```

### Entidad `Solicitud` (entidad central del sistema)

Esta es la entidad más importante del proyecto. Contiene todas las relaciones y el ciclo de vida completo.

```java
// src/main/java/co/edu/uniquindio/triage/domain/entity/Solicitud.java

package co.edu.uniquindio.triage.domain.entity;

import co.edu.uniquindio.triage.domain.enums.CanalOrigen;
import co.edu.uniquindio.triage.domain.enums.EstadoSolicitud;
import co.edu.uniquindio.triage.domain.enums.Prioridad;
import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;

import java.util.ArrayList;
import java.util.List;

@Entity
@Table(
    name = "solicitudes",
    indexes = {
        @Index(name = "idx_solicitud_estado", columnList = "estado"),
        @Index(name = "idx_solicitud_prioridad", columnList = "prioridad"),
        @Index(name = "idx_solicitud_usuario_solicitante",
               columnList = "usuario_solicitante_id"),
        @Index(name = "idx_solicitud_responsable", columnList = "responsable_id")
    }
)
public class Solicitud extends AuditableEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // ── Tipo de solicitud ────────────────────────────────────────────────

    @NotNull(message = "El tipo de solicitud es obligatorio")
    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "tipo_solicitud_id", nullable = false)
    private TipoSolicitud tipoSolicitud;

    // ── Contenido de la solicitud ─────────────────────────────────────────

    @NotBlank(message = "La descripción es obligatoria")
    @Size(min = 10, max = 3000)
    @Column(name = "descripcion", nullable = false, length = 3000)
    private String descripcion;

    @Size(max = 500)
    @Column(name = "justificacion_prioridad", length = 500)
    private String justificacionPrioridad;  // RF-03: justificación de la prioridad asignada

    // ── Estado y prioridad ────────────────────────────────────────────────

    @NotNull
    @Enumerated(EnumType.STRING)
    @Column(name = "estado", nullable = false, length = 50)
    private EstadoSolicitud estado = EstadoSolicitud.REGISTRADA;

    @Enumerated(EnumType.STRING)
    @Column(name = "prioridad", length = 20)
    private Prioridad prioridad;  // Se asigna al clasificar (RF-03)

    // ── Canal y usuarios ──────────────────────────────────────────────────

    @NotNull(message = "El canal de origen es obligatorio")
    @Enumerated(EnumType.STRING)
    @Column(name = "canal_origen", nullable = false, length = 50)
    private CanalOrigen canalOrigen;

    @NotNull(message = "El solicitante es obligatorio")
    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "usuario_solicitante_id", nullable = false)
    private Usuario usuarioSolicitante;

    // El responsable se asigna después (RF-05), puede ser null inicialmente
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "responsable_id")
    private Usuario responsable;

    // ── Cierre ────────────────────────────────────────────────────────────

    @Size(max = 1000)
    @Column(name = "observacion_cierre", length = 1000)
    private String observacionCierre;  // RF-08: obligatoria al cerrar

    // ── Historial ─────────────────────────────────────────────────────────

    @OneToMany(
        mappedBy = "solicitud",
        fetch = FetchType.LAZY,
        cascade = CascadeType.ALL,       // Al guardar solicitud, guarda el historial
        orphanRemoval = true             // Si se elimina un historial, lo borra de BD
    )
    @OrderBy("fechaCreacion ASC")        // Historial siempre ordenado por fecha
    private List<HistorialSolicitud> historial = new ArrayList<>();

    // ── Constructor vacío requerido por JPA ───────────────────────────────

    protected Solicitud() {}

    // ── Constructor para crear una nueva solicitud ─────────────────────────

    public Solicitud(TipoSolicitud tipoSolicitud,
                     String descripcion,
                     CanalOrigen canalOrigen,
                     Usuario usuarioSolicitante) {
        this.tipoSolicitud = tipoSolicitud;
        this.descripcion = descripcion;
        this.canalOrigen = canalOrigen;
        this.usuarioSolicitante = usuarioSolicitante;
        this.estado = EstadoSolicitud.REGISTRADA;
    }

    // ── Métodos de dominio (lógica de negocio dentro de la entidad) ─────────

    /**
     * Agrega una entrada al historial de la solicitud.
     * Usar siempre que cambie el estado o se realice alguna acción relevante.
     */
    public void agregarHistorial(String accion, String observacion, Usuario responsable) {
        var entrada = new HistorialSolicitud(this, accion, observacion, responsable);
        this.historial.add(entrada);
    }

    /**
     * Cambia el estado de la solicitud, validando que la transición sea válida.
     * Lanza excepción si la transición no está permitida.
     */
    public void cambiarEstado(EstadoSolicitud nuevoEstado,
                               String observacion,
                               Usuario usuarioResponsable) {
        if (!this.estado.puedeTransicionarA(nuevoEstado)) {
            throw new IllegalStateException(
                "No se puede pasar de %s a %s".formatted(this.estado, nuevoEstado)
            );
        }
        var accion = "Cambio de estado: %s → %s".formatted(this.estado, nuevoEstado);
        this.estado = nuevoEstado;
        agregarHistorial(accion, observacion, usuarioResponsable);
    }

    /**
     * Asigna un responsable a la solicitud (RF-05).
     */
    public void asignarResponsable(Usuario responsable, String observacion,
                                    Usuario usuarioAsignador) {
        this.responsable = responsable;
        agregarHistorial(
            "Asignación de responsable: " + responsable.getNombre(),
            observacion,
            usuarioAsignador
        );
    }

    /**
     * Clasifica la solicitud asignando prioridad (RF-03).
     */
    public void clasificar(Prioridad prioridad, String justificacion, Usuario clasificador) {
        this.prioridad = prioridad;
        this.justificacionPrioridad = justificacion;
        agregarHistorial(
            "Clasificación: prioridad " + prioridad.name(),
            justificacion,
            clasificador
        );
    }

    /**
     * Cierra la solicitud (RF-08). Solo si está en estado ATENDIDA.
     */
    public void cerrar(String observacionCierre, Usuario usuarioCierre) {
        if (this.estado != EstadoSolicitud.ATENDIDA) {
            throw new IllegalStateException(
                "Solo se pueden cerrar solicitudes en estado ATENDIDA"
            );
        }
        this.observacionCierre = observacionCierre;
        cambiarEstado(EstadoSolicitud.CERRADA, observacionCierre, usuarioCierre);
    }

    // ── Getters y setters ──────────────────────────────────────────────────

    public Long getId() { return id; }
    public TipoSolicitud getTipoSolicitud() { return tipoSolicitud; }
    public void setTipoSolicitud(TipoSolicitud tipo) { this.tipoSolicitud = tipo; }
    public String getDescripcion() { return descripcion; }
    public void setDescripcion(String descripcion) { this.descripcion = descripcion; }
    public String getJustificacionPrioridad() { return justificacionPrioridad; }
    public EstadoSolicitud getEstado() { return estado; }
    public Prioridad getPrioridad() { return prioridad; }
    public CanalOrigen getCanalOrigen() { return canalOrigen; }
    public Usuario getUsuarioSolicitante() { return usuarioSolicitante; }
    public Usuario getResponsable() { return responsable; }
    public String getObservacionCierre() { return observacionCierre; }
    public List<HistorialSolicitud> getHistorial() { return historial; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Solicitud other)) return false;
        return id != null && id.equals(other.id);
    }

    @Override
    public int hashCode() { return getClass().hashCode(); }
}
```

### Entidad `HistorialSolicitud`

```java
// src/main/java/co/edu/uniquindio/triage/domain/entity/HistorialSolicitud.java

package co.edu.uniquindio.triage.domain.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;

import java.time.LocalDateTime;

@Entity
@Table(
    name = "historial_solicitudes",
    indexes = {
        @Index(name = "idx_historial_solicitud_id", columnList = "solicitud_id")
    }
)
public class HistorialSolicitud {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotNull
    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "solicitud_id", nullable = false)
    private Solicitud solicitud;

    @NotBlank
    @Column(name = "accion", nullable = false, length = 200)
    private String accion;

    @Column(name = "observacion", length = 1000)
    private String observacion;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "usuario_responsable_id")
    private Usuario usuarioResponsable;

    // La fecha se registra automáticamente al crear el historial
    @Column(name = "fecha_accion", nullable = false)
    private LocalDateTime fechaAccion;

    protected HistorialSolicitud() {}

    public HistorialSolicitud(Solicitud solicitud,
                               String accion,
                               String observacion,
                               Usuario usuarioResponsable) {
        this.solicitud = solicitud;
        this.accion = accion;
        this.observacion = observacion;
        this.usuarioResponsable = usuarioResponsable;
        this.fechaAccion = LocalDateTime.now();
    }

    // Getters
    public Long getId() { return id; }
    public Solicitud getSolicitud() { return solicitud; }
    public String getAccion() { return accion; }
    public String getObservacion() { return observacion; }
    public Usuario getUsuarioResponsable() { return usuarioResponsable; }
    public LocalDateTime getFechaAccion() { return fechaAccion; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof HistorialSolicitud other)) return false;
        return id != null && id.equals(other.id);
    }

    @Override
    public int hashCode() { return getClass().hashCode(); }
}
```

### Repositories para todas las entidades

```java
// src/main/java/co/edu/uniquindio/triage/domain/repository/TipoSolicitudRepository.java
package co.edu.uniquindio.triage.domain.repository;

import co.edu.uniquindio.triage.domain.entity.TipoSolicitud;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public interface TipoSolicitudRepository extends JpaRepository<TipoSolicitud, Long> {
    Optional<TipoSolicitud> findByNombreIgnoreCase(String nombre);
    List<TipoSolicitud> findByActivoTrue();
    boolean existsByNombreIgnoreCase(String nombre);
}
```

```java
// src/main/java/co/edu/uniquindio/triage/domain/repository/UsuarioRepository.java
package co.edu.uniquindio.triage.domain.repository;

import co.edu.uniquindio.triage.domain.entity.Usuario;
import org.springframework.data.jpa.repository.EntityGraph;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;

import java.util.Optional;
import java.util.UUID;

@Repository
public interface UsuarioRepository extends JpaRepository<Usuario, UUID> {

    Optional<Usuario> findByEmail(String email);
    Optional<Usuario> findByNumeroIdentificacion(String numeroIdentificacion);
    boolean existsByEmail(String email);

    // Carga el usuario junto con sus roles (evita N+1 al verificar permisos)
    @EntityGraph(attributePaths = "roles")
    Optional<Usuario> findWithRolesByEmail(String email);

    // Para el UserDetailsService de Spring Security
    @Query("SELECT u FROM Usuario u JOIN FETCH u.roles WHERE u.email = :email AND u.activo = true")
    Optional<Usuario> findActiveUserWithRolesByEmail(String email);
}
```

```java
// src/main/java/co/edu/uniquindio/triage/domain/repository/HistorialSolicitudRepository.java
package co.edu.uniquindio.triage.domain.repository;

import co.edu.uniquindio.triage.domain.entity.HistorialSolicitud;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface HistorialSolicitudRepository extends JpaRepository<HistorialSolicitud, Long> {

    @Query("""
            SELECT h FROM HistorialSolicitud h
            JOIN FETCH h.usuarioResponsable
            WHERE h.solicitud.id = :solicitudId
            ORDER BY h.fechaAccion ASC
            """)
    List<HistorialSolicitud> findBySolicitudIdConUsuario(Long solicitudId);
}
```

```java
// src/main/java/co/edu/uniquindio/triage/domain/repository/RolRepository.java
package co.edu.uniquindio.triage.domain.repository;

import co.edu.uniquindio.triage.domain.entity.Rol;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface RolRepository extends JpaRepository<Rol, Long> {
    Optional<Rol> findByNombre(String nombre);
}
```

---

## 15. Migraciones con Flyway

### ¿Qué es Flyway y por qué usarlo?

Cuando `ddl-auto: update` hace cambios automáticos a tu esquema de BD, no tienes control ni trazabilidad de qué cambió, cuándo, ni quién. En producción esto es un desastre.

**Flyway** resuelve esto con scripts SQL versionados. Cada cambio al esquema es un script con un número de versión. Flyway ejecuta cada script exactamente una vez y lleva un registro de qué scripts ya fueron aplicados.

```
V1__crear_tablas_base.sql          → ejecutado en deploy inicial
V2__agregar_columna_dias_sla.sql   → ejecutado en deploy siguiente
V3__insertar_roles_iniciales.sql   → ejecutado en el siguiente deploy
```

Esto te da:
- **Trazabilidad:** sabes exactamente qué cambió en cada deploy.
- **Reproducibilidad:** puedes crear un ambiente nuevo desde cero con todos los scripts.
- **Colaboración:** todos en el equipo aplican los mismos cambios.

### Estructura de carpetas para Flyway

```
src/main/resources/
└── db/
    └── migration/
        ├── V1__crear_schema_inicial.sql
        ├── V2__insertar_datos_iniciales.sql
        └── V3__agregar_indices_adicionales.sql
```

La convención de nombres es `V{número}__{descripcion}.sql` (dos guiones bajos).

### V1 — Crear el esquema completo

```sql
-- src/main/resources/db/migration/V1__crear_schema_inicial.sql

-- =====================================================
-- V1: Creación del esquema inicial del sistema de Triage
-- Fecha: 2026-01-15
-- Autor: Equipo de desarrollo
-- =====================================================

-- Tabla de roles
CREATE TABLE roles (
    id          BIGSERIAL       PRIMARY KEY,
    nombre      VARCHAR(50)     NOT NULL UNIQUE,
    descripcion VARCHAR(200)
);

-- Tabla de usuarios
CREATE TABLE usuarios (
    id                      UUID            PRIMARY KEY DEFAULT gen_random_uuid(),
    nombre                  VARCHAR(150)    NOT NULL,
    email                   VARCHAR(100)    NOT NULL UNIQUE,
    numero_identificacion   VARCHAR(20)     NOT NULL UNIQUE,
    password_hash           VARCHAR(255)    NOT NULL,
    activo                  BOOLEAN         NOT NULL DEFAULT TRUE,
    fecha_creacion          TIMESTAMP       NOT NULL DEFAULT NOW(),
    fecha_modificacion      TIMESTAMP,
    creado_por              VARCHAR(100)    NOT NULL DEFAULT 'sistema',
    modificado_por          VARCHAR(100)
);

-- Tabla de relación usuarios-roles (N:M)
CREATE TABLE usuarios_roles (
    usuario_id  UUID    NOT NULL REFERENCES usuarios(id) ON DELETE CASCADE,
    rol_id      BIGINT  NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    PRIMARY KEY (usuario_id, rol_id)
);

-- Tabla de tipos de solicitud
CREATE TABLE tipos_solicitud (
    id                      BIGSERIAL       PRIMARY KEY,
    nombre                  VARCHAR(100)    NOT NULL UNIQUE,
    descripcion             VARCHAR(500),
    dias_atencion_estimados INTEGER,
    activo                  BOOLEAN         NOT NULL DEFAULT TRUE,
    fecha_creacion          TIMESTAMP       NOT NULL DEFAULT NOW(),
    fecha_modificacion      TIMESTAMP,
    creado_por              VARCHAR(100)    NOT NULL DEFAULT 'sistema',
    modificado_por          VARCHAR(100)
);

-- Tabla principal de solicitudes
CREATE TABLE solicitudes (
    id                      BIGSERIAL       PRIMARY KEY,
    tipo_solicitud_id       BIGINT          NOT NULL REFERENCES tipos_solicitud(id),
    descripcion             TEXT            NOT NULL,
    justificacion_prioridad VARCHAR(500),
    estado                  VARCHAR(50)     NOT NULL DEFAULT 'REGISTRADA',
    prioridad               VARCHAR(20),
    canal_origen            VARCHAR(50)     NOT NULL,
    usuario_solicitante_id  UUID            NOT NULL REFERENCES usuarios(id),
    responsable_id          UUID            REFERENCES usuarios(id),
    observacion_cierre      VARCHAR(1000),
    fecha_creacion          TIMESTAMP       NOT NULL DEFAULT NOW(),
    fecha_modificacion      TIMESTAMP,
    creado_por              VARCHAR(100)    NOT NULL DEFAULT 'sistema',
    modificado_por          VARCHAR(100)
);

-- Tabla de historial de solicitudes
CREATE TABLE historial_solicitudes (
    id                      BIGSERIAL       PRIMARY KEY,
    solicitud_id            BIGINT          NOT NULL REFERENCES solicitudes(id)
                                            ON DELETE CASCADE,
    accion                  VARCHAR(200)    NOT NULL,
    observacion             VARCHAR(1000),
    usuario_responsable_id  UUID            REFERENCES usuarios(id),
    fecha_accion            TIMESTAMP       NOT NULL DEFAULT NOW()
);

-- ── Índices para rendimiento ──────────────────────────────────────────────

CREATE INDEX idx_usuarios_email ON usuarios(email);
CREATE INDEX idx_usuarios_numero_identificacion ON usuarios(numero_identificacion);
CREATE INDEX idx_solicitudes_estado ON solicitudes(estado);
CREATE INDEX idx_solicitudes_prioridad ON solicitudes(prioridad);
CREATE INDEX idx_solicitudes_usuario_solicitante ON solicitudes(usuario_solicitante_id);
CREATE INDEX idx_solicitudes_responsable ON solicitudes(responsable_id);
CREATE INDEX idx_historial_solicitud_id ON historial_solicitudes(solicitud_id);
CREATE INDEX idx_tipos_solicitud_nombre ON tipos_solicitud(nombre);

-- ── Constraints de check para estados válidos ─────────────────────────────

ALTER TABLE solicitudes
    ADD CONSTRAINT chk_estado_valido
    CHECK (estado IN ('REGISTRADA', 'CLASIFICADA', 'EN_ATENCION', 'ATENDIDA', 'CERRADA'));

ALTER TABLE solicitudes
    ADD CONSTRAINT chk_prioridad_valida
    CHECK (prioridad IS NULL OR prioridad IN ('BAJA', 'MEDIA', 'ALTA', 'CRITICA'));

ALTER TABLE solicitudes
    ADD CONSTRAINT chk_canal_origen_valido
    CHECK (canal_origen IN ('PRESENCIAL', 'CORREO_ELECTRONICO', 'SISTEMA_ACADEMICO',
                            'TELEFONICO', 'PLATAFORMA_WEB'));
```

### V2 — Insertar datos iniciales

```sql
-- src/main/resources/db/migration/V2__insertar_datos_iniciales.sql

-- =====================================================
-- V2: Inserción de datos iniciales (roles y tipos de solicitud)
-- =====================================================

-- Roles del sistema
INSERT INTO roles (nombre, descripcion) VALUES
    ('ROLE_ADMIN',       'Administrador del sistema con acceso total'),
    ('ROLE_FUNCIONARIO', 'Funcionario académico que atiende solicitudes'),
    ('ROLE_ESTUDIANTE',  'Estudiante que registra y consulta sus solicitudes');

-- Tipos de solicitud iniciales
INSERT INTO tipos_solicitud (nombre, descripcion, dias_atencion_estimados) VALUES
    ('REGISTRO_ASIGNATURAS',    'Solicitud de registro o modificación de asignaturas', 5),
    ('HOMOLOGACION',            'Solicitud de homologación de materias de otras instituciones', 15),
    ('CANCELACION_ASIGNATURA',  'Cancelación de una asignatura matriculada', 3),
    ('SOLICITUD_CUPO',          'Solicitud de cupo en una asignatura con capacidad llena', 7),
    ('CONSULTA_ACADEMICA',      'Consulta general sobre proceso académico', 2),
    ('TRANSFERENCIA_INTERNA',   'Solicitud de transferencia entre jornadas o programas', 20),
    ('GRADO',                   'Trámites relacionados con el proceso de grado', 30),
    ('BECA_DESCUENTO',          'Solicitud de beca o descuento en matrícula', 10);

-- Usuario administrador por defecto
-- Contraseña: Admin2026! (hash BCrypt generado externamente)
-- NOTA: En producción, cambiar este usuario inmediatamente después del primer deploy
INSERT INTO usuarios (
    id,
    nombre,
    email,
    numero_identificacion,
    password_hash,
    activo,
    creado_por
) VALUES (
    gen_random_uuid(),
    'Administrador Sistema',
    'admin@uniquindio.edu.co',
    '0000000000',
    '$2a$12$someHashHere',  -- Reemplazar con hash BCrypt real en producción
    true,
    'sistema'
);

-- Asignar rol ADMIN al usuario administrador
INSERT INTO usuarios_roles (usuario_id, rol_id)
SELECT u.id, r.id
FROM usuarios u, roles r
WHERE u.email = 'admin@uniquindio.edu.co'
  AND r.nombre = 'ROLE_ADMIN';
```

### V3 — Ajuste posterior (ejemplo de evolución del esquema)

```sql
-- src/main/resources/db/migration/V3__agregar_numero_radicado.sql

-- =====================================================
-- V3: Agregar número de radicado único a cada solicitud
-- Este es un ejemplo de cómo evoluciona el esquema
-- =====================================================

-- Agregar columna de radicado
ALTER TABLE solicitudes
    ADD COLUMN numero_radicado VARCHAR(20);

-- Generar radicados para solicitudes existentes
UPDATE solicitudes
SET numero_radicado = 'RAD-' || LPAD(id::TEXT, 8, '0');

-- Hacer la columna NOT NULL y UNIQUE ahora que todos tienen valor
ALTER TABLE solicitudes
    ALTER COLUMN numero_radicado SET NOT NULL;

ALTER TABLE solicitudes
    ADD CONSTRAINT uq_solicitud_numero_radicado UNIQUE (numero_radicado);

CREATE INDEX idx_solicitudes_numero_radicado ON solicitudes(numero_radicado);
```

> **Regla de oro de Flyway:** Nunca modifiques un script que ya fue aplicado a la base de datos. Si cometiste un error en V1, crea un V2 que corrija el problema. Flyway verifica el checksum de los scripts y lanzará un error si detecta que cambiaste uno ya ejecutado.

---

## 16. Antes vs Ahora: EntityManager vs Spring Data JPA

### La forma antigua con EntityManager

Antes de Spring Data JPA, tenías que escribir toda la lógica de acceso a datos manualmente:

```java
// ❌ Forma antigua — EntityManager directo
@Repository
public class SolicitudRepositoryOld {

    @PersistenceContext
    private EntityManager em;

    public Solicitud findById(Long id) {
        return em.find(Solicitud.class, id);
    }

    public List<Solicitud> findByEstado(EstadoSolicitud estado) {
        TypedQuery<Solicitud> query = em.createQuery(
            "SELECT s FROM Solicitud s WHERE s.estado = :estado",
            Solicitud.class
        );
        query.setParameter("estado", estado);
        return query.getResultList();
    }

    public void save(Solicitud solicitud) {
        if (solicitud.getId() == null) {
            em.persist(solicitud);
        } else {
            em.merge(solicitud);
        }
    }

    public void delete(Long id) {
        Solicitud s = findById(id);
        if (s != null) em.remove(s);
    }

    public long count() {
        return em.createQuery("SELECT COUNT(s) FROM Solicitud s", Long.class)
                 .getSingleResult();
    }
}
```

### La forma moderna con Spring Data JPA

```java
// ✅ Forma moderna — Spring Data JPA
@Repository
public interface SolicitudRepository extends JpaRepository<Solicitud, Long> {
    List<Solicitud> findByEstado(EstadoSolicitud estado);
    // save, findById, delete, count → ya incluidos
}
```

### Tabla comparativa

| Aspecto | EntityManager directo | Spring Data JPA |
|---------|-----------------------|-----------------|
| Código para CRUD | ~50 líneas por entidad | 1 línea (extends) |
| Mantenimiento | Alto (código manual) | Bajo (generado) |
| Curva de aprendizaje | Alta | Baja |
| Flexibilidad | Total | Alta (+ @Query si necesitas) |
| Rendimiento | Igual | Igual |
| Consultas complejas | Naturaleza: nativo | @Query JPQL/SQL |
| ¿Cuándo usar? | Raramente, proyectos muy específicos | Siempre |

> **¿Debo olvidar EntityManager completamente?**
> No. Para operaciones avanzadas como batch processing, caché de segundo nivel, o consultas muy específicas, puede seguir siendo necesario. Pero para el 95% de los casos, Spring Data JPA es más que suficiente.

---

## 17. Errores comunes y troubleshooting

### Error 1: `LazyInitializationException`

```
org.hibernate.LazyInitializationException: 
  failed to lazily initialize a collection of role 
  'Solicitud.historial', could not initialize proxy - no Session
```

**Causa:** Intentaste acceder a una relación LAZY fuera del contexto de una transacción activa (por ejemplo, en el Controller, después de que el servicio cerró la transacción).

**Solución:**
```java
// En el servicio, cargar lo que necesitas DENTRO de la transacción
@Transactional(readOnly = true)
public SolicitudDetalleResponse obtenerConHistorial(Long id) {
    return solicitudRepository
        .findByIdWithHistorial(id)  // Usa JOIN FETCH o @EntityGraph
        .map(this::mapearAResponse)
        .orElseThrow(() -> new SolicitudNotFoundException(id));
}
```

### Error 2: `DetachedInstanceException`

```
org.hibernate.PersistenceException: 
  org.hibernate.exception.GenericJDBCException: 
  detached entity passed to persist
```

**Causa:** Intentaste persistir un objeto que fue cargado en una sesión anterior (está en estado "detached").

**Solución:** Usar `save()` (que hace `merge` si el objeto tiene ID) en lugar de operaciones directas. Asegúrate de que la operación esté dentro de una transacción.

### Error 3: `DataIntegrityViolationException` — violación de constraint único

```
ERROR: duplicate key value violates unique constraint "idx_usuario_email"
```

**Causa:** Intentaste insertar un valor duplicado en una columna con restricción `UNIQUE`.

**Solución:** Verificar antes de insertar:
```java
if (usuarioRepository.existsByEmail(email)) {
    throw new EmailYaRegistradoException("El email ya está registrado: " + email);
}
```

### Error 4: El `N+1` silencioso

No lanza excepción, pero hace tu aplicación muy lenta. Detéctalo habilitando el log de SQL en `dev`:

```yaml
spring:
  jpa:
    show-sql: true
    properties:
      hibernate:
        format_sql: true
logging:
  level:
    org.hibernate.SQL: DEBUG
    org.hibernate.orm.jdbc.bind: TRACE  # Muestra los parámetros
```

Si ves muchos `SELECT` iguales en el log, tienes un N+1.

### Error 5: `ConcurrentModificationException` al iterar y modificar

```java
// ❌ Incorrecto
for (HistorialSolicitud h : solicitud.getHistorial()) {
    solicitud.getHistorial().remove(h);  // Modificas la colección mientras iteras
}

// ✅ Correcto — usar iterator o removeIf
solicitud.getHistorial().removeIf(h -> condicion);
```

### Error 6: Flyway `Checksum mismatch`

```
ERROR: Migration V1__crear_schema_inicial.sql 
  checksum mismatch for migration version 1
```

**Causa:** Modificaste un script de Flyway que ya fue ejecutado.

**Solución:** Nunca modificar scripts ya aplicados. Si debes corregir algo, crea un nuevo script de versión mayor.

### Error 7: `@Transactional` no funciona

```java
// ❌ Incorrecto — llamar un método @Transactional desde dentro de la misma clase
@Service
public class SolicitudService {

    public void procesar(Long id) {
        this.guardarHistorial(id); // @Transactional NO se aplica (llamada interna)
    }

    @Transactional
    public void guardarHistorial(Long id) { ... }
}
```

**Causa:** Spring AOP intercepta las llamadas desde fuera de la clase, no las internas.

**Solución:** Extrae el método a otro servicio, o inyecta el propio servicio como dependencia (self-injection).

---

## 18. Ejercicios prácticos

### Reto 1 — Básico: Entidad y Repository

Crea la entidad `ConfiguracionTriage` con las siguientes características:
- ID autoincremental
- `clave` (String, único, max 100 chars)
- `valor` (String, max 500 chars)
- `descripcion` (String, max 300 chars)
- `activo` (boolean, default true)
- Hereda de `AuditableEntity`

Crea el Repository con:
- Método para buscar por clave
- Método para listar todas las configuraciones activas

Escribe el script Flyway `V4__crear_tabla_configuracion.sql` correspondiente.

---

### Reto 2 — Intermedio: Consultas complejas

Agrega al `SolicitudRepository` los siguientes métodos:

1. Encontrar las 5 solicitudes más antiguas en estado `REGISTRADA` o `CLASIFICADA` (sin responsable asignado).
2. Contar solicitudes agrupadas por `TipoSolicitud` y `EstadoSolicitud` (pista: usa `@Query` con JPQL y una proyección o `Object[]`).
3. Encontrar todas las solicitudes asignadas a un responsable específico que NO estén en estado `CERRADA`, paginadas.

---

### Reto 3 — Avanzado: Motor de reglas de priorización

Implementa un método `sugerirPrioridad(Solicitud solicitud)` en un servicio que use el patrón Strategy (ver Guía 02) para determinar la prioridad basado en:

- Si el tipo es `GRADO` o `HOMOLOGACION` → prioridad ALTA automáticamente.
- Si faltan menos de 7 días para el cierre del semestre → prioridad CRITICA.
- Todos los demás casos → prioridad MEDIA.

Agrega la `Prioridad` sugerida y su justificación a la solicitud antes de persistirla.

---

### Reto 4 — Desafío de arquitectura

El sistema requiere una nueva funcionalidad: **número de radicado automático** con el formato `RAD-2026-00001` (año-consecutivo con ceros a la izquierda).

Implementa esto usando:
1. Una secuencia PostgreSQL (`CREATE SEQUENCE seq_radicado START 1`).
2. Un `@PrePersist` en la entidad `Solicitud` que NO acceda a la BD (la secuencia se usa desde el script Flyway).
3. O, alternativamente, un método en el `SolicitudService` que genere el radicado antes de guardar.

Crea el script Flyway correspondiente y adapta la entidad.

---

## 19. Resumen y Cheat Sheet

### Anotaciones JPA esenciales

```java
// Clase
@Entity                     // Marca la clase como entidad
@Table(name = "tabla")      // Nombre de la tabla
@MappedSuperclass           // Clase base sin tabla propia

// ID
@Id                         // Llave primaria
@GeneratedValue(strategy = GenerationType.IDENTITY)  // Autoincremental
@GeneratedValue(strategy = GenerationType.UUID)      // UUID automático

// Columnas
@Column(name = "col", nullable = false, length = 100, unique = true)
@Enumerated(EnumType.STRING)  // Enums como texto
@Lob                          // Para datos grandes (TEXT en PostgreSQL)

// Relaciones
@ManyToOne(fetch = FetchType.LAZY) @JoinColumn(name = "fk_col")
@OneToMany(mappedBy = "campo", fetch = FetchType.LAZY, cascade = CascadeType.ALL)
@ManyToMany @JoinTable(name = "tabla_rel", joinColumns = ..., inverseJoinColumns = ...)
@OneToOne(fetch = FetchType.LAZY) @JoinColumn(name = "fk_col")

// Auditoría
@CreatedDate    @LastModifiedDate
@CreatedBy      @LastModifiedBy
```

### Spring Data JPA — métodos más usados

```java
repository.save(entity);                    // INSERT o UPDATE
repository.findById(id);                    // Optional<T>
repository.findAll();                       // List<T>
repository.findAll(pageable);               // Page<T>
repository.deleteById(id);                  // DELETE
repository.existsById(id);                  // boolean
repository.count();                         // long
repository.saveAll(list);                   // Batch

// Derivados por nombre — patrones
findBy{Campo}({valor})
findBy{Campo}And{Campo2}({v1}, {v2})
findBy{Campo}Not({valor})
findBy{Campo}In(List<T>)
findBy{Campo}IsNull()
findBy{Campo}Containing({texto})
countBy{Campo}({valor})
existsBy{Campo}({valor})
```

### Configuraciones importantes en application.yml

```yaml
spring:
  jpa:
    hibernate.ddl-auto: validate  # Con Flyway: validate
    open-in-view: false           # Siempre false
    show-sql: true                # Solo en dev
  flyway:
    enabled: true
    locations: classpath:db/migration
```

### Convención de nombres Flyway

```
V{versión}__{descripcion_con_guiones_bajos}.sql
V1__crear_tablas_base.sql
V2__insertar_datos_iniciales.sql
V3__agregar_columna_radicado.sql
```

---

## 20. Referencias y recursos adicionales

- [Spring Data JPA — Documentación oficial](https://docs.spring.io/spring-data/jpa/reference/index.html)
- [Hibernate User Guide 6.x](https://docs.jboss.org/hibernate/orm/6.6/userguide/html_single/Hibernate_User_Guide.html)
- [Flyway Documentation](https://documentation.red-gate.com/fd)
- [Jakarta Persistence 3.1 Spec](https://jakarta.ee/specifications/persistence/3.1/)
- [Baeldung — Spring Data JPA Tutorial](https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa)
- [Vlad Mihalcea Blog](https://vladmihalcea.com/) — El mejor recurso especializado en JPA/Hibernate avanzado, especialmente para el problema N+1 y optimización de consultas.

---

> **Autor:** José Alfredo Ramírez Espinosa  
> **Materia:** Programación Avanzada  
> **Programa:** Ingeniería de Sistemas y Computación  
> **Universidad del Quindío** | Armenia, Colombia  
> **Año:** 2026  
> *Material de uso académico. Todos los derechos reservados al autor.*
