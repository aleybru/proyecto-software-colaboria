# DEC-C13 — Roadmap y timeline sobre el núcleo de planificación

**Estado:** Decision vigente
**Fecha original:** 2026-07-18
**Revisión vigente:** 2026-08-29
**Autoridad:** Dani
**Relacionada con:** DEC-C12, DEC-C11, DEC-C14, DEC-C15

## Resumen

La capacidad de Roadmap/timeline se construye sobre el mismo núcleo de elemento planificable de DEC-C12. No es un modelo paralelo.

La revisión de 2026-08-29 alinea su durabilidad con la arquitectura vigente: el roadmap vive en la DB de ColaborIA y no requiere un repo de gobernanza por proyecto.

## Decisión

### 1. Capacidad de negocio

Dani debe poder ver cómo avanza un proyecto de software o narrativo a corto, mediano y largo plazo: fases, iniciativas, hitos, entregables, dependencias y orden.

### 2. Objetos conceptuales

Sobre el elemento planificable de DEC-C12 se agregan, cuando corresponde:

- fase/hito;
- entregable;
- dependencias de orden;
- rango temporal o fecha;
- unidad de agrupación propia del proyecto.

### 2.1. Jerarquía

Como mínimo:

`fase → iniciativa/frente → hito → tarea o elemento accionable`

- **fase**: etapa mayor;
- **iniciativa/frente**: resultado amplio compuesto;
- **hito**: condición o entregable verificable;
- **tarea**: trabajo accionable y asignable.

Los niveles superiores agregan progreso; no deben convertirse artificialmente en tareas gigantes.

### 3. Roadmap ordinal y temporal

La vista debe soportar tanto fechas reales como secuencia/dependencia sin fecha. No se obliga a inventar fechas cuando sólo se conoce orden.

### 4. Relación con Kanban

Kanban y Roadmap son vistas del mismo núcleo. Un elemento puede aparecer en uno, en otro o en ambos sin duplicar estado.

### 5. Especialización software

Cubre releases, infraestructura, arquitectura y dependencias técnicas. Las tareas aprobadas se relacionan con el mecanismo de ejecución vigente, pero el roadmap no depende de un repo de gobernanza del proyecto.

### 6. Boundary narrativo

El roadmap narrativo organiza producción real. El mapa de tramas y la cronología interna son dominios distintos definidos en DEC-C14 y DEC-C15.

### 7. Fuente de verdad y durabilidad

- La **DB de ColaborIA** es la fuente operativa y durable del roadmap, jerarquías, dependencias, fechas, progreso e historial.
- Los documentos derivados o exportaciones que deban conservarse como archivo pueden guardarse en la carpeta Drive del proyecto.
- El repo de código de un proyecto software no es fuente de verdad del roadmap.
- No se requiere repo de gobernanza ni snapshots Git obligatorios del roadmap.

## No objetivos

- Esquema físico de Postgres.
- Librería de visualización Angular.
- Modelo narrativo de DEC-C14/C15.

## Pendiente

- Especificación física en DB.
- Primera tarea ejecutable de esta capacidad cuando entre al roadmap de implementación.
