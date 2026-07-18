# DEC-C12 — GitHub Projects como tablero visual de code-tasks

**Estado:** Decision propuesta — pendiente de aprobación explícita de Dani
**Fecha:** 2026-07-18
**Origen:** propuesta inicial de Chapu contrastada con Dani (`Chapu Agent/ColaborIA - Propuesta GitHub Projects.docx`, Drive), acotada por Dani en conversación directa.
**Relacionada con:** DEC-C08, DEC-C09 (code-tasks/), DEC-C11 (matriz de accesos — esta Decision depende de que Chapu sea quien tiene escritura en repo de código).

## Resumen

Se adopta GitHub Projects (v2) como tablero visual del trabajo de codificación, **acotado exclusivamente a proyectos de tipo software y a tareas de código** — no como herramienta general de gestión de todo el trabajo del proyecto (se descarta la Opción B más amplia que Chapu había propuesto originalmente en el documento de Drive).

El Project **complementa** los archivos `code-tasks/*.md` — no los reemplaza. Cada ítem del Project corresponde 1:1 a un archivo `code-tasks/TASK-000X.md` existente. La fuente de verdad de qué tiene que hacer Code sigue siendo el archivo Markdown (DEC-C08, `rules/CODE-TASKS.md`); el Project es una vista de estado, no una instrucción de ejecución.

## Decisión

### 1. Alcance

Exclusivo para proyectos de tipo `software` con tareas de código gestionadas vía `code-tasks/` (DEC-C08). No se usa como backlog general para otro tipo de trabajo (decisiones pendientes, preguntas abiertas, tareas narrativas) — eso sigue viviendo donde ya vivía.

### 2. Propiedad y control del tablero

Conforme a la matriz de accesos de DEC-C11 (Chapu es el único agente con escritura en el repo de código), **Chapu es el único que crea y mueve tarjetas en el Project**. Papu y Dani no operan el tablero directamente — participan en la definición del contenido de las tareas por conversación.

### 3. Flujo operativo

1. Dani, Papu y Chapu conversan (por el canal de DEC-C10, o por el mecanismo vigente hasta que ese canal esté validado) sobre qué tareas de código hacen falta.
2. Esa conversación produce una **lista depurada y final** de tareas — no es el Project el lugar donde se decide qué hacer, es donde se refleja lo ya decidido.
3. Chapu recibe la lista depurada y:
   - crea el archivo `code-tasks/TASK-000X.md` correspondiente para cada tarea (esquema ya definido en `rules/CODE-TASKS.md`);
   - crea el ítem correspondiente en el Project, vinculado conceptualmente al archivo (mismo `id`/número de tarea en el título del ítem, como mínimo).
4. A medida que el trabajo avanza y se acuerda en la conversación que una tarea está terminada, Chapu:
   - actualiza el `status` en el YAML frontmatter del archivo `code-tasks/TASK-000X.md` (ya definido en `rules/CODE-TASKS.md`);
   - mueve la tarjeta correspondiente en el Project al estado equivalente.

El estado de una tarea se decide en la conversación, no se infiere automáticamente de ningún evento técnico (por ejemplo, un merge de PR no mueve la tarjeta por sí solo — el criterio sigue siendo la conversación y el gatillo manual, coherente con DEC-C08 punto 3).

### 4. Correspondencia de estados

Las columnas del Project deben corresponder exactamente a los valores ya definidos en `rules/CODE-TASKS.md` para `status` en el frontmatter — no se crea un vocabulario de estados nuevo y distinto:

| Columna del Project | `status` en YAML |
|---|---|
| Backlog | `pending` |
| En curso | `in_progress` |
| Bloqueado | `blocked` |
| Hecho | `done` |
| Cancelado | `cancelled` |

### 5. Vínculo al repo

El Project se vincula al repositorio de gobernanza (`aleybru/proyecto-software-colaboria`), no al repo de código — mismo razonamiento ya validado en la prueba técnica: el repo de gobernanza es el único que todo proyecto tiene garantizado, y aunque esta Decision acota el *uso* del Project a tareas de código, no hay razón para que el *anclaje* dependa del tipo de proyecto.

## Estado de la prueba técnica previa

Ya se verificó viabilidad técnica antes de esta Decision: se creó un Project de prueba ("ColaborIA - TEST", vinculado al repo de gobernanza) con 4 tareas de ejemplo, confirmado por lectura directa de la API. Detalle completo en el documento de Drive referenciado arriba.

Hallazgo técnico relevante para la implementación futura: los tokens fine-grained de GitHub no soportan todavía el permiso "Projects" para proyectos de cuenta personal (gap documentado por GitHub). La automatización de este flujo (si se lleva a código en vez de operarse manualmente vía Chapu con token clásico) debe tenerlo en cuenta.

## No objetivos de esta Decision

- No convierte al Project en fuente de verdad de ningún contenido — eso sigue siendo `code-tasks/*.md`, Decisions, y el resto de la gobernanza en repo.
- No automatiza el movimiento de tarjetas por eventos de git (commits, PRs, merges) — el movimiento es decidido en conversación y ejecutado por Chapu.
- No extiende el uso del Project a proyectos narrativos ni a gestión general de trabajo no relacionado con código.

## Pendiente

- Decidir si se reutiliza el Project de prueba ("ColaborIA - TEST") como base real (renombrándolo y limpiando las 4 tareas de ejemplo) o se descarta y se crea uno nuevo.
- Revocar o regenerar el token clásico usado en la prueba técnica (quedó expuesto en texto plano durante la sesión de trabajo).
- Si en el futuro se decide automatizar este flujo desde el backend de ColaborIA (en vez de que Chapu lo opere conversacionalmente), definir cómo se resuelve el gap de permisos de tokens fine-grained señalado arriba.
