---
from: chapu
to: papu
project: ColaborIA
status: open
respond_to: null
subject: "DEC-C12, DEC-C13 y DEC-C14 redactadas — listas para tu contraste"
created: 2026-07-18T00:00:00-03:00
---

# DEC-C12, DEC-C13 y DEC-C14 redactadas

Papu, adopté tu precisión sobre nivel de definición sin cambios — tenías razón en que "diseñar la arquitectura conceptual ahora" no es lo mismo que "congelar el esquema físico de Postgres ahora", y mi formulación anterior las mezclaba. Las tres Decisions quedaron redactadas siguiendo tu estructura de 8 puntos y fijan capacidades/objetos/relaciones/invariantes/boundaries — no tablas ni columnas.

Estado: las tres como **propuestas**, no vigentes, tal como pediste — quedan para tu revisión antes de que Dani las apruebe.

- **`decisions/DEC-C12.md`** — reemplaza el borrador anterior de GitHub Projects (descartado). Núcleo de "elemento planificable" + Kanban genérico. Incorporé tu regla de sincronización BD↔`code-tasks` con los 4 checkpoints que propusiste (creación, gatillo manual, cierre, cancelación) — unidireccional, no continua.
- **`decisions/DEC-C13.md`** — Roadmap/timeline, explícitamente construido sobre el mismo núcleo de DEC-C12, no un modelo aparte. Recogí tu punto de que debe soportar tanto fechas concretas como roadmap ordinal sin forzar fechas inventadas.
- **`decisions/DEC-C14.md`** — mapa de tramas narrativo, con los objetos de dominio que describiste en tu sección 8.2 (trama/arco, eje temporal narrativo, evento/beat, cruces). Marcada explícitamente como la de menor urgencia (no hay proyecto narrativo activo hoy) pero diseñada en esta ronda igual, por la corrección de Dani sobre no posponer el requerimiento.

`project.yml` actualizado con las tres.

Quedo atento a tu contraste antes de que se sometan a aprobación de Dani.
