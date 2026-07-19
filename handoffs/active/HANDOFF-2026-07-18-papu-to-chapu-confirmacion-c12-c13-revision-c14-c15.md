---
from: papu
to: chapu
project: ColaborIA
status: open
respond_to: handoffs/active/HANDOFF-2026-07-18-chapu-response-revision-c14-c15.md
subject: "Confirmación DEC-C12 y DEC-C13; revisión sustantiva DEC-C14 y posible DEC-C15"
created: 2026-07-18
---

# Confirmación DEC-C12 y DEC-C13; revisión sustantiva DEC-C14

Dani revisó conmigo las tres propuestas y fija estos estados:

1. **DEC-C12 queda aprobada en su redacción actual.** Podés marcarla como vigente y actualizar `project.yml`.
2. **DEC-C13 queda aprobada en dirección y contenido**, con una precisión conceptual que debe incorporarse antes de marcarla vigente. Con ese ajuste integrado, no requiere reabrir la decisión de negocio.
3. **DEC-C14 todavía no queda aprobada.** La referencia visual y funcional revisada con Dani muestra que el dominio es más amplio que la redacción actual. Debe reformularse y, si el boundary resulta más limpio, dividirse en DEC-C14 y DEC-C15.

## 1. Precisión obligatoria para DEC-C13

La maqueta de roadmap/timeline representa correctamente la necesidad de negocio: planificación del trabajo en el tiempo, con fechas reales cuando existan y roadmap ordinal cuando no.

La Decision debe distinguir jerarquías de planificación. No toda barra del roadmap es una tarea. El núcleo conceptual debe admitir, como mínimo, una estructura equivalente a:

`fase → iniciativa/frente de trabajo → hito → tarea o elemento accionable`

Los nombres definitivos pueden quedar abiertos, pero deben preservarse estas diferencias:

- **fase**: agrupación estratégica o etapa mayor del proyecto;
- **iniciativa/capability/frente**: resultado amplio que requiere varios hitos y tareas;
- **hito**: condición o entregable verificable que marca progreso;
- **tarea**: unidad de trabajo accionable y asignable.

Un elemento superior puede agregar progreso desde sus elementos hijos, pero no debe convertirse artificialmente en una “tarea gigante”. El Kanban opera principalmente sobre trabajo accionable; el roadmap puede mostrar fases, iniciativas, hitos y tareas con distintos niveles de detalle.

Con esta precisión incorporada, **DEC-C13 queda aprobada por Dani** y puede marcarse vigente.

## 2. Referencias visuales revisadas

Revisé con Dani tres referencias funcionales:

- el Kanban que preparaste, como ejemplo válido de DEC-C12;
- el roadmap/Gantt que preparaste, como ejemplo válido de DEC-C13;
- un prototipo funcional previo de gestor de tramas sincronizadas que Dani trajo explícitamente a esta conversación como referencia para precisar DEC-C14.

No hace falta copiar literalmente esas interfaces. Sirven para fijar qué problema debe resolver cada dominio.

## 3. Distinción que DEC-C14 debe corregir

La redacción actual mezcla parcialmente “episodios, capítulos y actos” con “tiempo de la historia”. Son ejes diferentes.

Deben distinguirse al menos tres dimensiones:

### 3.1. Tiempo de producción — pertenece a DEC-C13

Es el calendario real de trabajo para producir el proyecto:

- cuándo diseñamos;
- cuándo escribimos;
- cuánto tiempo estimamos;
- qué depende de qué;
- qué hitos deben alcanzarse en semanas o meses.

Esto es roadmap/Gantt de producción. No pertenece al mapa de tramas.

### 3.2. Orden y estructura de presentación del relato

Representa cómo se cuenta la historia al público:

- episodio o capítulo;
- acto;
- apertura;
- detonante;
- puntos de giro;
- confrontación;
- midpoint;
- crisis;
- clímax;
- resolución;
- orden de aparición de hechos y revelaciones;
- flashbacks, anticipaciones y repeticiones.

Acá también importa el **espacio narrativo o tiempo de pantalla estimado**. En el prototipo, las tarjetas pueden redimensionarse y ocupan porcentajes del episodio o de una unidad dramática. Si un episodio dura 50 o 60 minutos, ese porcentaje permite estimar cuántos minutos ocupa un beat, una escena futura o un conjunto de escenas.

La duración sigue siendo estimada hasta que la escena se escriba, pero debe poder modelarse y ajustarse.

### 3.3. Cronología interna de la historia

Representa cuándo ocurrieron realmente los hechos dentro del mundo narrativo, independientemente de cuándo se cuentan:

- año o fecha;
- edad de personajes;
- generaciones;
- períodos históricos;
- sucesión causal real;
- eventos anteriores al protagonista;
- infancia, juventud y presente;
- períodos omitidos o resumidos.

Un hecho ocurrido quince años antes puede presentarse como flashback en un episodio tardío. Por eso “cuándo ocurrió” y “cuándo se cuenta” no pueden ser el mismo atributo ni el mismo eje.

## 4. Objetos conceptuales que la revisión debe contemplar

No propongo congelar tablas ni nombres físicos, pero la arquitectura conceptual debería distinguir algo equivalente a:

### Acontecimiento narrativo

Hecho canónico dentro del mundo de la historia:

- qué ocurrió;
- cuándo ocurrió en la cronología interna;
- duración interna aproximada;
- personajes, lugares y tramas involucrados;
- causas y consecuencias.

### Presentación o beat dramático

Forma y posición mediante la cual ese acontecimiento se presenta al público:

- episodio/capítulo;
- acto y función dramática;
- orden de presentación;
- porcentaje o rango dentro de la unidad narrativa;
- tiempo de pantalla o extensión estimada;
- escena o conjunto de escenas asociado;
- trama/subtrama desde la que se presenta.

Un mismo acontecimiento puede:

- mostrarse una vez;
- revelarse parcialmente en varios momentos;
- aparecer como flashback;
- ser contado por un personaje;
- reinterpretarse más adelante.

Por eso el acontecimiento y su presentación no deberían duplicarse como si fueran el mismo objeto.

## 5. Vistas sincronizadas

El mapa de tramas debe concebirse como varias proyecciones sincronizadas sobre objetos narrativos compartidos, no como tableros independientes mantenidos manualmente.

Como mínimo, la dirección conceptual debe preservar:

1. **mapa por tramas y subtramas**;
2. **estructura dramática / orden del relato**;
3. **cronología interna de la historia**.

Un cambio en un objeto debe reflejarse en las vistas correspondientes sin crear fuentes de verdad competidoras.

## 6. Tramas, beats y relaciones semánticas

La referencia funcional muestra que no alcanza con ubicar tarjetas en una grilla.

La Decision debe admitir:

- tramas principales;
- subtramas;
- beats/eventos;
- actos y unidades estructurales;
- tarjetas redimensionables;
- porcentajes o rangos dentro de episodios/capítulos;
- estimación de minutos o extensión;
- cruces entre tramas;
- relaciones semánticas tipadas.

Tipos de relación candidatos —no vocabulario cerrado—:

- causa;
- consecuencia;
- convergencia;
- paralelo;
- eco;
- presagio;
- revelación;
- contradicción;
- resolución de una promesa narrativa.

Los huecos también son información: una trama que desaparece durante un tramo puede ser una pausa deliberada, una discontinuidad o una parte todavía no resuelta. El sistema debe permitir detectarlos, no rellenarlos automáticamente.

## 7. Posible división DEC-C14 / DEC-C15

Te pido que evalúes el boundary y elijas la separación más auditable.

Una opción razonable es:

- **DEC-C14 — Mapa de tramas y estructura de presentación narrativa**: tramas/subtramas, beats, actos, función dramática, orden del relato, porcentajes, tiempo de pantalla estimado, relaciones y vistas sincronizadas de presentación.
- **DEC-C15 — Cronología interna del mundo narrativo**: acontecimientos canónicos, fechas/edades/generaciones, duración interna, causalidad histórica y vínculo con una o varias presentaciones en el relato.

Otra opción es mantener una única DEC-C14 si podés demostrar que el boundary sigue claro y el documento no mezcla dos dominios difíciles de auditar.

La división documental no implica que una capacidad se postergue. Ambas deben diseñarse ahora; solo se escalona la implementación.

## 8. Próximo paso

1. Marcá DEC-C12 vigente.
2. Incorporá la jerarquía conceptual indicada en DEC-C13 y marcala vigente.
3. Reformulá DEC-C14 o proponé la división C14/C15.
4. Mantené C14 —y C15 si se crea— como propuestas pendientes.
5. Pasámelas para contraste antes de que Dani las apruebe.

No generes todavía tareas de implementación para C14/C15. Primero cerremos correctamente el dominio narrativo.
