# DEC-C15 — Cronología interna del mundo narrativo

**Estado:** Decision propuesta — pendiente de aprobación explícita de Dani
**Fecha:** 2026-07-18
**Origen:** división de la DEC-C14 original, propuesta por Papu tras revisión con Dani de un prototipo funcional traído a la conversación.
**Relacionada con:** DEC-C14 (estructura de presentación — distinto dominio, vinculado por referencia), DEC-C13 (tiempo de producción — distinto eje).

## Resumen

Cubre cuándo ocurrieron realmente los hechos dentro del mundo narrativo, independientemente de cuándo se cuentan al público (eso es DEC-C14) o de cuándo se produce el proyecto (eso es DEC-C13). Un hecho ocurrido quince años antes puede presentarse como flashback en un episodio tardío — "cuándo ocurrió" y "cuándo se cuenta" no son el mismo atributo ni el mismo eje, y no deben modelarse como si lo fueran.

Fija arquitectura conceptual, no esquema físico de Postgres — mismo criterio que DEC-C12/C13/C14.

## Decisión

### 1. Objeto conceptual: acontecimiento narrativo

Hecho canónico dentro del mundo de la historia. Incluye: qué ocurrió, cuándo ocurrió en la cronología interna, duración interna aproximada, personajes/lugares/tramas involucrados, y causas y consecuencias.

### 2. Atributos de la cronología interna

Año o fecha (del mundo narrativo, no del calendario real), edad de personajes, generaciones, períodos históricos, sucesión causal real, eventos anteriores al protagonista, infancia/juventud/presente de los personajes, y períodos omitidos o resumidos en la narración.

### 3. Independencia respecto a la presentación (DEC-C14)

Un acontecimiento existe y tiene su lugar en la cronología interna independientemente de si, cuándo, o cuántas veces se presenta al público. La relación con sus presentaciones (DEC-C14) es de referencia — un acontecimiento puede tener cero, una, o varias presentaciones asociadas.

### 4. Causalidad histórica

La cronología interna debe soportar relaciones de causa/consecuencia entre acontecimientos, no solo orden temporal — para poder responder no solo "qué pasó cuándo" sino "qué generó qué".

### 5. Vista sincronizada

Es una de las proyecciones sincronizadas descritas en DEC-C14 sección 6 — comparte el principio de que un cambio en un acontecimiento se refleja en todas las vistas relacionadas (mapa de tramas, estructura de presentación, cronología) sin crear fuentes de verdad competidoras.

## No objetivos de esta Decision

- Esquema físico de Postgres.
- Estructura de presentación narrativa (DEC-C14).
- Tiempo de producción (DEC-C13).
- Implementación — no hay proyecto narrativo activo hoy que la requiera de forma inmediata.

## Pendiente

- Especificación arquitectónica, cuando exista un proyecto narrativo activo que la necesite.
