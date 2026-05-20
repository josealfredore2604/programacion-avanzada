# 📘 Sistema de Triage y Gestión de Solicitudes Académicas

## Contexto Institucional

El Programa de Ingeniería de Sistemas y Computación cuenta con una comunidad
de más de 1.400 estudiantes, docentes y administrativos. De manera permanente
se realizan diversas solicitudes académicas y administrativas (registro de
asignaturas, homologaciones, cupos, etc.) a través de múltiples canales como
atención presencial, correo electrónico y sistemas académicos.

### El Problema

Actualmente la gestión de estas solicitudes es ineficiente debido a la falta
de una estructura unificada, la ausencia de mecanismos formales de
clasificación y la carencia de trazabilidad clara. Esto genera sobrecarga
operativa y respuestas inoportunas para los estudiantes.

---

## Propósito y Objetivos

Aplicar conocimientos de arquitectura empresarial, patrones de diseño,
Spring Boot y Angular para resolver este problema real.

- Registrar solicitudes de manera estructurada
- Clasificar y priorizar mediante reglas de negocio
- Asignar responsables de forma controlada
- Gestionar el ciclo de vida completo con historial auditable
- Proveer información clara sobre el estado de cada trámite

---

## Organización

| Aspecto | Detalle |
|---------|---------|
| **Equipo** | Máximo 3 integrantes |
| **Metodología** | Desarrollo incremental — 3 hitos de 5 semanas |
| **Backend** | Java + Spring Boot |
| **Frontend** | TypeScript + Angular |
| **Persistencia** | ORM + API REST |

---

## Requisitos Funcionales

### RF-01 — Registro de Solicitudes
Registrar una solicitud almacenando: tipo, descripción, canal de origen
(CSU, correo, SAC, telefónico, etc.), fecha/hora de registro e
identificación del solicitante.

### RF-02 — Clasificación de Solicitudes
Clasificar según su tipo:
- Registro de asignaturas
- Homologación
- Cancelación de asignaturas
- Solicitud de cupos
- Consulta académica

### RF-03 — Priorización de Solicitudes
Asignar prioridad basada en: tipo de solicitud, impacto académico y fecha
límite. La prioridad debe quedar registrada con su justificación.

### RF-04 — Gestión del Ciclo de Vida
Gestionar los estados con transiciones coherentes: