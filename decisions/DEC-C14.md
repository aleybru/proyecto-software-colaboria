# DEC-C14 — Mapa de tramas y estructura de presentación narrativa

**Estado:** Decision vigente — aprobada por Dani el 2026-07-18
**Fecha:** 2026-07-18
**Reemplaza:** la versión anterior de DEC-C14 ("Especialización narrativa: mapa de tramas"), que mezclaba presentación y cronología interna en un solo dominio. Dividida en esta Decision y DEC-C15 tras revisión de un prototipo funcional que Dani trajo a la conversación.
**Origen:** revisión de Papu con Dani (`handoffs/active/HANDOFF-2026-07-18-papu-to-chapu-confirmacion-c12-c13-revision-c14-c15.md`).
**Relacionada con:** DEC-C12, DEC-C13 (roadmap de producción — distinto del eje de esta Decision), DEC-C15 (cronología interna — distinto dominio, vinculado por referencia).

## Resumen

Cubre cómo se presenta una historia al público: estructura dramática, orden del relato, tramas y subtramas, y su despliegue a lo largo de episodios/capítulos — incluyendo tiempo de pantalla estimado. **No** cubre cuándo ocurrieron los hechos dentro del mundo narrativo (eso es DEC-C15) ni el calendario real de producción (eso es DEC-C13).

Fija arquitectura conceptual, no esquema físico de Postgres — mismo criterio que DEC-C12/C13.

## Decisión

### 1. Tres ejes que no deben confundirse

- **Tiempo de producción** (DEC-C13): calendario real de trabajo — cuándo se escribe, cuánto se estima, qué hitos hay que alcanzar. No pertenece a esta Decision.
- **Orden y estructura de presentación del relato** (esta Decision): cómo se cuenta la historia al público — episodio/capítulo, acto, apertura, detonante, puntos de giro, confrontación, midpoint, crisis, clímax, resolución, orden de aparición de hechos y revelaciones, flashbacks, anticipaciones, repeticiones.
- **Cronología interna de la historia** (DEC-C15): cuándo ocurrieron realmente los hechos dentro del mundo narrativo, independiente de cuándo se cuentan.

### 2. Objetos conceptuales

- **Presentación o beat dramático**: forma y posición mediante la cual un acontecimiento (objeto definido en DEC-C15) se presenta al público. Incluye episodio/capítulo, acto y función dramática, orden de presentación, porcentaje o rango dentro de la unidad narrativa, tiempo de pantalla o extensión estimada, escena o conjunto de escenas asociado, y trama/subtrama desde la que se presenta.
- **Trama/subtrama**: unidad narrativa que se despliega a lo largo de la estructura de presentación.
- **Acto/unidad estructural**: agrupación dentro de la estructura dramática.

Un mismo acontecimiento (DEC-C15) puede tener múltiples presentaciones: mostrarse una vez, revelarse parcialmente en varios momentos, aparecer como flashback, ser contado por un personaje, o reinterpretarse más adelante. **El acontecimiento y su presentación no se duplican como si fueran el mismo objeto** — la presentación referencia al acontecimiento, no lo repite.

### 3. Tarjetas dimensionables y tiempo de pantalla

El prototipo revisado con Dani usa tarjetas redimensionables que ocupan un porcentaje del episodio o unidad dramática — si un episodio dura 50 o 60 minutos, ese porcentaje permite estimar cuántos minutos ocupa un beat, una escena futura, o un conjunto de escenas. La duración es estimada hasta que la escena se escribe, pero debe poder modelarse y ajustarse desde el arranque.

### 4. Relaciones semánticas tipadas

No alcanza con ubicar tarjetas en una grilla. Se admiten relaciones entre beats/tramas con tipos como (vocabulario no cerrado): causa, consecuencia, convergencia, paralelo, eco, presagio, revelación, contradicción, resolución de una promesa narrativa.

### 5. Huecos como información

Una trama que desaparece durante un tramo puede ser una pausa deliberada, una discontinuidad, o una parte todavía no resuelta. El sistema debe permitir **detectar** huecos, no rellenarlos automáticamente ni asumir resolución.

### 6. Vistas sincronizadas, no tableros independientes

El mapa de tramas se concibe como proyecciones sincronizadas sobre objetos narrativos compartidos. Como mínimo: mapa por tramas y subtramas, y estructura dramática/orden del relato (la tercera vista, cronología interna, vive en DEC-C15 pero debe poder cruzarse con estas). Un cambio en un objeto se refleja en las vistas correspondientes sin crear fuentes de verdad competidoras.

### 7. Relación con DEC-C15

Un acontecimiento narrativo (objeto de DEC-C15) puede tener una o varias presentaciones (objeto de esta Decision). La relación es de referencia, no de duplicación — "cuándo ocurrió" vive en DEC-C15, "cuándo y cómo se cuenta" vive acá.

## No objetivos de esta Decision

- Esquema físico de Postgres.
- Modelo completo de personajes/lugares (pertenece a la capa L4 de cada proyecto narrativo).
- Cronología interna del mundo narrativo (DEC-C15).
- Implementación — no hay proyecto narrativo activo hoy que la requiera de forma inmediata.

## Pendiente

- Especificación arquitectónica, cuando exista un proyecto narrativo activo que la necesite.
- Validar contra el prototipo funcional que Dani trajo como referencia, si se decide reutilizar patrones de esa interfaz.
