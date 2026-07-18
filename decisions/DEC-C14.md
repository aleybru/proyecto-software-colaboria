# DEC-C14 — Especialización narrativa: mapa de tramas

**Estado:** Decision propuesta — pendiente de aprobación explícita de Dani
**Fecha:** 2026-07-18
**Origen:** contrapropuesta de Papu, corrección de alcance de Dani, acuerdo de Papu sobre nivel de definición — mismo origen que DEC-C12 y DEC-C13.
**Relacionada con:** DEC-C12 (Kanban editorial/producción, distinto de esta Decision), DEC-C13 (roadmap narrativo de fases, distinto de esta Decision), DEC-C11.

## Resumen

Se define la capacidad de **mapa de tramas** para proyectos narrativos — un modelo de dominio propio, no una vista adicional del Kanban ni del roadmap genérico. Fija arquitectura conceptual, no esquema físico de Postgres (mismo criterio de nivel de definición que DEC-C12 y DEC-C13).

Esta Decision es la de menor urgencia de las tres — ColaborIA hoy no tiene un proyecto narrativo activo (el manifest de ColaborIA es de tipo software) — pero se diseña en esta misma ronda para no forzar un refactor cuando la necesidad aparezca, según el criterio que fijó Dani para las tres Decisions en conjunto.

## Decisión

### 1. Capacidad de negocio

Para un proyecto narrativo, Dani necesita ver, sin salir de ColaborIA, cómo se despliegan las tramas de una historia a lo largo del tiempo narrativo: qué pasa en qué momento, qué tramas se cruzan, y cómo evolucionan los arcos.

### 2. Distinción de dos capacidades narrativas (no confundir)

- **Kanban editorial/de producción** (cubierto por DEC-C12, sin necesidad de una Decision aparte): estados de trabajo como idea, planificado, en escritura, revisión, aprobado, publicado/entregado — son tareas y entregables del proceso, mismo modelo de elemento planificable que cualquier otro proyecto.
- **Mapa de tramas** (esta Decision): visualización especializada de la estructura narrativa en sí, no del proceso de producción.

### 3. Objetos conceptuales del mapa de tramas

- **Trama/arco**: unidad narrativa que se despliega a lo largo del tiempo de la historia (no confundir con una `task`).
- **Eje temporal narrativo**: episodios, capítulos, actos, o la unidad que el proyecto use — es tiempo de la historia, no tiempo real de producción (eso es DEC-C13).
- **Evento/beat**: giro, revelación, crisis, cierre — ubicado sobre una trama en un punto del eje temporal narrativo.
- **Vínculo**: un evento puede relacionarse con personajes, escenas, lugares y decisiones narrativas ya existentes en el proyecto (fuentes del repo de gobernanza narrativo, según el protocolo común L4).
- **Cruce entre tramas**: relación explícita cuando un evento de una trama afecta o depende de otra.

### 4. Invariante de modelado

Una trama no se representa como una lista de elementos planificables genéricos con estado backlog/hecho — el mapa de tramas requiere sus propios objetos de dominio (sección 3), aunque pueda reutilizar infraestructura técnica de visualización de línea de tiempo si conviene en la implementación.

### 5. Relación con DEC-C12 y DEC-C13

- Comparte con DEC-C13 el concepto de eje temporal y visualización tipo timeline, pero el eje es narrativo (tiempo de la historia), no de producción (tiempo real).
- No comparte el modelo de "elemento planificable" de DEC-C12 — una trama y un beat no son tareas.

### 6. Fuente de verdad y durabilidad

Mismas reglas que DEC-C12/DEC-C13: BD como estado operativo activo para la visualización y edición del mapa; el repo de gobernanza del proyecto narrativo conserva las fuentes narrativas formales (personajes, decisiones canónicas, etc.) según el protocolo común — el mapa de tramas en BD referencia esas fuentes, no las reemplaza ni las duplica como autoridad.

## No objetivos de esta Decision

- Esquema físico de Postgres.
- Modelo completo y definitivo de todos los atributos posibles de personajes/escenas/lugares (eso pertenece a la capa L4 de cada proyecto narrativo, no a esta Decision de arquitectura).
- Elección de librería de visualización.
- Implementación — no hay proyecto narrativo activo hoy que la requiera de forma inmediata; queda diseñada para no bloquear una implementación futura sin refactor.

## Pendiente

- Especificación arquitectónica que baje esta Decision a esquema físico, cuando exista un proyecto narrativo activo que la necesite.
- Validar este modelo contra un caso real (por ejemplo, si en el futuro se integra un proyecto narrativo existente a ColaborIA) antes de implementar.
