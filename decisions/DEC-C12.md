# DEC-C12 — ColaborIA como superficie de planificación: núcleo común y Kanban

**Estado:** Decision vigente — aprobada por Dani el 2026-07-18
**Fecha:** 2026-07-18
**Reemplaza:** el borrador anterior de DEC-C12 ("GitHub Projects como tablero visual de code-tasks"), descartado tras contraste entre Papu, Chapu y Dani.
**Origen:** contrapropuesta de Papu (`handoffs/active/HANDOFF-2026-07-18-papu-to-chapu-contrapropuesta-dec-c12-planning-integrado.md`), corrección de alcance de Dani (diseñar las tres capacidades ahora, escalonar solo la implementación), acuerdo final de Papu sobre nivel de definición (`handoffs/active/HANDOFF-2026-07-18-papu-response-dec-c12-c13-c14.md`).
**Relacionada con:** DEC-C08, DEC-C09, DEC-C10, DEC-C11, DEC-C13 (Roadmap, construida sobre el núcleo definido acá), DEC-C14 (especialización narrativa).

## Resumen

GitHub Projects se descarta como dependencia operativa de V0 — contradice DEC-C11 (ColaborIA como interfaz operativa única) al introducir una segunda superficie que hay que mantener sincronizada, aunque Dani no la toque directamente. **ColaborIA construye su propio espacio de planificación**, con un núcleo conceptual de "elemento planificable" compartido entre el Kanban (esta Decision) y el Roadmap (DEC-C13).

Esta Decision fija **arquitectura conceptual**: capacidades de negocio, objetos, relaciones, invariantes y boundaries. No fija el esquema físico de Postgres (tablas, columnas, índices, migraciones) — eso se define después en especificación arquitectónica y tareas ejecutables, apoyándose en lo que se fija acá.

## Decisión

### 1. Capacidad de negocio

Dani, sin salir de ColaborIA, debe poder ver y gestionar: qué hay en backlog, qué está listo, en curso, bloqueado, hecho o cancelado; qué depende de qué. GitHub Projects no se adopta como componente obligatorio — puede existir a futuro como exportación o proyección opcional, sin autoridad.

### 2. Objeto conceptual central: elemento planificable

Un elemento planificable puede relacionarse conceptualmente con: proyecto, título y descripción, tipo de elemento, estado, prioridad, responsable/actor asignado, dependencias, bloqueos, Decision/propuesta/conversación de origen, tarea de ejecución asociada (si existe), resultado durable asociado, y estimaciones o fechas cuando corresponda. No todos estos campos necesitan existir en la primera implementación — es la capacidad objetivo que la arquitectura no debe cerrar de antemano.

### 3. Kanban genérico

Estados mínimos: backlog, listo, en curso, bloqueado, hecho, cancelado (si corresponde). Los nombres exactos y la posibilidad de configurar estados por proyecto quedan abiertos a diseño posterior — el invariante es que Dani entienda de inmediato el estado del trabajo.

### 4. Especialización software: relación con `code-tasks/*.md`

DEC-C08 sigue vigente sin modificaciones. Se fija esta relación:

- el elemento planificable (BD) es la fuente del **estado operativo cotidiano** de una tarea;
- el archivo `code-tasks/TASK-000X.md` es la **materialización durable y ejecutable** de una tarea ya aprobada — lo que Claude Code lee para ejecutar (sin cambios respecto a `rules/CODE-TASKS.md`);
- la relación entre ambos debe ser trazable y verificable, pero **no es sincronización bidireccional continua** — evita el riesgo de que la misma tarea tenga estados divergentes en dos lugares editados en paralelo.

**Checkpoints de sincronización** (unidireccional, BD → archivo, en momentos puntuales, no continuo):
- creación de la tarea aprobada (se genera el archivo `code-tasks`);
- gatillo manual de ejecución (DEC-C08 punto 3);
- cierre/verificación (se actualiza `status` en el frontmatter);
- cancelación.

Fuera de esos checkpoints, no se edita el archivo en paralelo a la BD.

### 5. Boundary con la planificación narrativa

El Kanban genérico (esta Decision) cubre trabajo tipo backlog/producción para cualquier tipo de proyecto, incluido narrativo (ej. idea → planificado → en escritura → revisión → aprobado → publicado). La especialización narrativa de **mapa de tramas** (arcos, beats, cruces entre tramas) es un modelo de dominio distinto, no una vista del Kanban — se define en DEC-C14, no acá.

### 6. Integración con la sala de trabajo (DEC-C10, DEC-C11)

Flujo objetivo: conversación → análisis/contraste → propuesta → aprobación de Dani → creación/actualización de elemento planificable → ejecución → verificación → resultado → actualización del roadmap. Desde la sala debe poder promoverse contenido aprobado a elemento planificable; desde el tablero debe poder rastrearse la conversación/Decision/ejecución/resultado de origen. Esto opera DEC-C11 sin que toda conversación se vuelva estado automáticamente (protocolo común, sección 8).

### 7. Fuente de verdad y durabilidad

- **Base de datos**: fuente operativa activa — boards, elementos planificables, estados, prioridades, responsables, dependencias, historial operativo.
- **Repositorio de gobernanza**: fuente durable de Decisions, tareas formales (`code-tasks`), Results, reglas, y snapshots aprobados del roadmap cuando deban auditarse.
- Regla heredada de DEC-C09/DEC-C10: la conversación no es estado; la BD puede sostener estado operativo; el repo materializa gobernanza y resultados durables.

## No objetivos de esta Decision

- Esquema exacto de tablas, columnas, claves, índices, estrategia de historial y migraciones de Postgres.
- Endpoints, librería Angular específica para el Kanban, mecánica de drag-and-drop.
- Algoritmo técnico exacto de sincronización BD ↔ `code-tasks` (solo se fija la regla de checkpoints unidireccionales, no la implementación).
- Roadmap/timeline (DEC-C13) y mapa de tramas narrativo (DEC-C14) — se referencian pero se definen en sus propias Decisions.
- Orden de implementación (queda para roadmap de tareas posterior a la aprobación de las tres Decisions).

## Pendiente

- Especificación arquitectónica que baje esta Decision a esquema físico de Postgres.
- Definición de la primera tarea ejecutable de esta capacidad (fuera del alcance de esta Decision).
