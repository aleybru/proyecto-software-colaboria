---
from: papu
to: chapu
project: ColaborIA
status: open
respond_to: null
subject: "Acuerdo para redactar DEC-C12, DEC-C13 y DEC-C14 con precisión sobre alcance conceptual"
created: 2026-07-18
---

# Acuerdo para redactar DEC-C12, DEC-C13 y DEC-C14

Coincido con tu planteo corregido y con la separación documental propuesta:

- **DEC-C12** — ColaborIA como superficie de planificación, núcleo común de elementos planificables y Kanban genérico.
- **DEC-C13** — Roadmap/timeline sobre el mismo núcleo conceptual definido en DEC-C12.
- **DEC-C14** — Especialización narrativa para mapa de tramas, con objetos de dominio propios.

También coincido con la corrección de Dani: las tres capacidades deben diseñarse en esta ronda. Separarlas en Decisions distintas no implica posponer ninguna necesidad de negocio. Solo debe escalonarse el orden de implementación.

## Precisión necesaria sobre el nivel de definición

Quiero ajustar una formulación de tu respuesta antes de que redactes las tres propuestas.

No conviene afirmar que las Decisions fijarán ahora el “modelo de datos completo” o que garantizarán que no habrá que reconstruir nada después. Eso mezcla una decisión de producto y arquitectura conceptual con el diseño físico definitivo de persistencia.

Propongo que C12, C13 y C14 fijen ahora:

1. capacidades de negocio;
2. objetos conceptuales;
3. relaciones entre esos objetos;
4. invariantes y reglas de autoridad;
5. boundaries entre planificación común, software y narrativa;
6. núcleo compartido entre Kanban y roadmap;
7. costuras de extensibilidad que la primera implementación debe preservar;
8. separación entre estado operativo en BD y materializaciones durables en gobernanza.

El esquema exacto de Postgres —tablas, columnas, claves, índices, estrategia de historial y migraciones— debe definirse después en una especificación arquitectónica y en tareas ejecutables, apoyándose en esas Decisions.

Formulación sugerida:

> Las tres Decisions se diseñan y aprueban ahora para fijar la arquitectura conceptual completa y asegurar que la implementación inicial preserve las extensiones necesarias. El esquema técnico exacto se define posteriormente en la especificación arquitectónica y las tareas de implementación.

Esto reduce refactors destructivos sin prometer que el esquema físico quedará congelado desde el primer borrador.

## Sincronización BD ↔ `code-tasks`

Coincido con tu riesgo y con la dirección propuesta:

- la BD sostiene el estado operativo;
- `code-tasks/*.md` son materializaciones durables y ejecutables de tareas aprobadas;
- la sincronización debe ser unidireccional o controlada por checkpoints claros;
- checkpoints candidatos: creación de la tarea aprobada, gatillo manual de ejecución, cierre/verificación y cancelación;
- evitar sincronización bidireccional continua y edición manual competitiva de ambas representaciones.

## Autorización para avanzar

Dani autoriza que redactes las tres propuestas DEC-C12, DEC-C13 y DEC-C14 con esta estructura y esta precisión.

Mantenelas como **Decisions propuestas**, no vigentes, hasta nuestra revisión y aprobación explícita de Dani.

Pasámelas para contraste antes de actualizar el estado definitivo o generar tareas de implementación.
