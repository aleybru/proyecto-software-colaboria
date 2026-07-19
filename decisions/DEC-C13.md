# DEC-C13 — Roadmap y timeline sobre el núcleo de planificación

**Estado:** Decision vigente — aprobada por Dani el 2026-07-18, con la jerarquía de la sección 2.1 incorporada como precisión obligatoria de Papu antes de la aprobación.
**Fecha:** 2026-07-18
**Origen:** contrapropuesta de Papu, corrección de alcance de Dani, acuerdo de Papu sobre nivel de definición — mismo origen que DEC-C12.
**Relacionada con:** DEC-C12 (núcleo de "elemento planificable" del que esta Decision depende — no define un modelo aparte), DEC-C11.

## Resumen

Se define la capacidad de Roadmap/timeline de ColaborIA, construida **sobre el mismo núcleo conceptual de elemento planificable fijado en DEC-C12** — no es un modelo de datos independiente. Fija arquitectura conceptual, no esquema físico de Postgres (ver DEC-C12 para el criterio de nivel de definición, que aplica igual acá).

## Decisión

### 1. Capacidad de negocio

Dani necesita, sin salir de ColaborIA, un mapa de cómo avanza el desarrollo — de software o de una historia — a corto, mediano y largo plazo: fases, hitos, entregables, dependencias, orden de despliegue.

### 2. Objetos conceptuales

Sobre el elemento planificable de DEC-C12, se agregan estas relaciones/atributos cuando el elemento participa del roadmap: fase o hito al que pertenece, entregable asociado, dependencias de orden con otros elementos, rango temporal o fecha (cuando exista), y unidad de agrupación según el proyecto (versión, iteración, episodio, capítulo, u otra que el manifest del proyecto defina).

### 2.1. Jerarquía de planificación (precisión obligatoria de Papu, incorporada antes de aprobación)

No toda barra del roadmap es una tarea. El núcleo conceptual admite, como mínimo:

`fase → iniciativa/frente de trabajo → hito → tarea o elemento accionable`

- **fase**: agrupación estratégica o etapa mayor del proyecto.
- **iniciativa/capability/frente**: resultado amplio que requiere varios hitos y tareas.
- **hito**: condición o entregable verificable que marca progreso.
- **tarea**: unidad de trabajo accionable y asignable (se vincula con el elemento planificable de DEC-C12, y en software con `code-tasks/*.md` según la sección 4 de esa Decision).

Los nombres definitivos de estos niveles pueden ajustarse en la especificación técnica, pero la distinción entre ellos debe preservarse. Un elemento superior (fase, iniciativa, hito) puede agregar progreso desde sus elementos hijos, pero no debe convertirse artificialmente en una tarea gigante que oculte el detalle real de trabajo. El Kanban (DEC-C12) opera principalmente sobre el nivel de tarea/elemento accionable; el roadmap puede mostrar los cuatro niveles con distinto grado de detalle según la vista.

### 3. Invariante: roadmap ordinal y temporal conviven

La vista no debe obligar a inventar fechas cuando solo se conoce secuencia y dependencias. Debe soportar tanto planificación temporal concreta (fechas reales, tipo Gantt) como roadmap puramente ordinal (orden y dependencia, sin fecha). Un elemento sin fecha sigue siendo válido en el roadmap.

### 4. Relación con el Kanban (DEC-C12)

Un elemento planificable puede existir en el Kanban, en el roadmap, o en ambos simultáneamente — son dos vistas del mismo objeto subyacente, no dos objetos distintos que haya que mantener sincronizados. Mover un elemento de estado en el Kanban no requiere una operación aparte para reflejarlo en el roadmap.

### 5. Especialización software

Cubre releases, hitos de infraestructura/arquitectura, y dependencias entre tareas técnicas — reutilizando la relación con `code-tasks/*.md` ya fijada en DEC-C12 sección 4 (mismos checkpoints de sincronización, no una regla nueva).

### 6. Boundary con mapa de tramas narrativo (DEC-C14)

El roadmap narrativo (fases de producción: idea → escritura → publicación, con fechas/orden) usa esta Decision. El **mapa de tramas** (arcos narrativos, beats, cruces entre tramas a lo largo de episodios/capítulos) es un modelo de dominio distinto — comparte infraestructura de timeline si conviene técnicamente, pero no se modela como una lista de elementos planificables genéricos. Se define en DEC-C14.

### 7. Fuente de verdad y durabilidad

Mismas reglas que DEC-C12 sección 7: BD como estado operativo activo, repo de gobernanza como fuente durable de snapshots aprobados del roadmap cuando deban auditarse.

## No objetivos de esta Decision

- Esquema físico de Postgres.
- Elección de librería de visualización (Gantt/timeline) para Angular.
- Modelo del mapa de tramas narrativo (DEC-C14).
- Orden de implementación respecto a DEC-C12 y DEC-C14 (Kanban tiene prioridad de implementación por ser la necesidad inmediata con software, pero las tres se diseñan en esta misma ronda).

## Pendiente

- Especificación arquitectónica que baje esta Decision a esquema físico.
- Definición de la primera tarea ejecutable de esta capacidad.
