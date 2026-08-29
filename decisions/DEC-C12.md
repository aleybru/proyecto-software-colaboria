# DEC-C12 — ColaborIA como superficie de planificación: núcleo común y Kanban

**Estado:** Decision vigente
**Fecha original:** 2026-07-18
**Revisión vigente:** 2026-08-29
**Autoridad:** Dani
**Relacionada con:** DEC-C08, DEC-C09, DEC-C10, DEC-C11, DEC-C13, DEC-C14

## Resumen

ColaborIA construye su propio espacio de planificación. GitHub Projects no es una dependencia operativa. El Kanban y el Roadmap comparten un núcleo conceptual de elemento planificable.

La revisión de 2026-08-29 elimina la dependencia de un repo de gobernanza por proyecto: la DB es fuente operativa y durable del estado de planificación que ColaborIA gestiona.

## Decisión

### 1. Capacidad de negocio

Dani debe poder ver y gestionar dentro de ColaborIA qué está en backlog, listo, en curso, bloqueado, hecho o cancelado, además de prioridades, responsables y dependencias.

### 2. Elemento planificable

Un elemento planificable puede relacionarse con:

- proyecto;
- título y descripción;
- tipo;
- estado;
- prioridad;
- responsable/actor;
- dependencias y bloqueos;
- Decision/propuesta/conversación de origen;
- ejecución asociada;
- resultado asociado;
- estimaciones o fechas cuando correspondan.

La Decision fija el dominio, no el esquema físico definitivo.

### 3. Kanban genérico

Estados mínimos conceptuales:

- backlog;
- listo;
- en curso;
- bloqueado;
- hecho;
- cancelado.

Los nombres finales y configurabilidad se definen en implementación.

### 4. Tareas de ejecución de software

La DB de ColaborIA es la fuente de verdad del estado operativo y durable de la tarea.

Para el desarrollo actual de ColaborIA, `code-tasks/*.md` continúa existiendo por DEC-C08 como mecanismo transicional que Claude Code consume. Esta excepción histórica **no implica que cada proyecto software necesite un repo de gobernanza ni que el producto objetivo deba materializar sus tareas en uno**.

En la arquitectura objetivo, el backend debe poder entregar al runtime de ejecución una tarea aprobada y trazable desde ColaborIA sin requerir un repositorio de gobernanza del proyecto. El formato técnico del payload de ejecución se definirá cuando se implemente ese gateway.

Mientras siga vigente el workflow actual de desarrollo de ColaborIA, la sincronización con `code-tasks/*.md` se hace únicamente en los checkpoints definidos por DEC-C08, evitando dos estados editables en paralelo.

### 5. Boundary narrativo

El Kanban genérico puede organizar trabajo narrativo, pero el mapa de tramas y la cronología son dominios propios definidos en DEC-C14 y DEC-C15.

### 6. Integración con sala

Flujo objetivo:

```text
conversación → análisis/contraste → aprobación → elemento planificable
→ ejecución → verificación → resultado → actualización de roadmap
```

Todo elemento debe poder rastrear su conversación, Decision, ejecución y resultado de origen.

### 7. Fuente de verdad y durabilidad

- **DB de ColaborIA**: fuente operativa y durable de boards, elementos, estados, prioridades, responsables, dependencias, historial y objetos formales relacionados.
- **Drive**: documentos y artefactos de archivo del proyecto.
- **Repo de código**: código del proyecto software.

No se requiere un repo de gobernanza por proyecto ni snapshots obligatorios del Kanban en GitHub.

La regla `conversación no es estado` se mantiene: el contenido sólo se convierte en planificación formal mediante el workflow correspondiente.

## No objetivos

- Esquema físico exacto de Postgres.
- Librería Angular o mecánica concreta de drag-and-drop.
- Roadmap/timeline (DEC-C13) y dominio narrativo (DEC-C14/C15).
- Migrar ahora el workflow transicional `code-tasks` del desarrollo de ColaborIA.

## Pendiente

- Especificación física de planificación en Postgres.
- Contrato de despacho de tareas al runtime de ejecución sin depender de repo de gobernanza por proyecto.
