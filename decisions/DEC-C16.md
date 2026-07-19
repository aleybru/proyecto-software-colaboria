# DEC-C16 — MVP1.0: alcance y orden de implementación

**Estado:** Decision propuesta — pendiente de aprobación explícita de Dani, en contraste con Papu
**Fecha:** 2026-07-18
**Origen:** definición conjunta Dani-Chapu tras cierre de DEC-C09 a DEC-C15.
**Relacionada con:** todas las Decisions vigentes (DEC-C08 a DEC-C15) — MVP1.0 es el primer recorte de implementación sobre ese diseño ya cerrado.

## Resumen

Define dónde termina el primer entregable funcional de ColaborIA: qué capacidades ya diseñadas (DEC-C09 a DEC-C15) entran en MVP1.0, y en qué orden se implementan. No rediseña nada — todas las capacidades ya están fijadas en Decisions previas; esta Decision solo recorta el subconjunto y fija la secuencia.

## Decisión

### 1. Alcance del MVP1.0

**Bloque A — Credenciales**
- Gestión de credenciales: GitHub App/installation tokens (no PAT manual — ver sección 2), API keys de Claude y de OpenAI. Tabla `credentials` con cifrado real funcionando (DEC-C09/DEC-C10) — deja de ser "pendiente no bloqueante", entra al MVP como requisito.

**Bloque B — Proyectos**
- Crear proyecto, elegir tipo (`narrative` | `software`) — tabla `projects` (DEC-C09).
- Creación automática de recursos asociados al crear un proyecto: repo(s) de GitHub + carpeta Drive, con la convención de nombres de DEC-C09 sección 7 — tabla `project_resources`.

**Bloque C — Identidad de agentes**
- `config/agents/papu.yml` y `config/agents/chapu.yml` (DEC-C11) — perfil, protocolo y contexto que cada agente recibe al ser invocado, preservando identidad y habilidades.

**Bloque D — Sala de chat**
- Canal conversacional completo de DEC-C10: `sessions` con `project_id` obligatorio, `messages`, `agent_turns`, floor lock bajo autoridad del backend, modos de conversación.
- Con agentes **reales** (Claude y OpenAI vía API) desde el arranque — no hace falta un agente simulado intermedio, porque el Bloque A ya resuelve las credenciales.

**Bloque E — Lectura/escritura real en repos y Drive**
- El backend ejecuta las operaciones que hoy Chapu hace manualmente (bash + API de GitHub) en esta conversación: escribir Decisions, QC/canal, code-tasks; leer/escribir Drive.
- Incluye el mecanismo de **exportación DB → repo de gobernanza** (DEC-C10 punto 11, política de durabilidad) — entra al MVP1.0, no se aplaza.

### 2. Resolución de la tensión "PAT vs. GitHub App"

**Se implementa GitHub App / installation tokens desde el arranque del MVP1.0**, conforme a la dirección ya fijada en DEC-C09. Se descarta el paso intermedio de PAT manual que se había considerado como opción más rápida — la diferencia de esfuerzo (uno o dos días) no justifica construir un mecanismo que de todos modos se iba a reemplazar.

### 3. Resolución de la tensión "exportación DB → repo"

**Entra al MVP1.0.** Sin esto, la sala de chat operaría con la conversación viviendo solo en DB, sin materializarse en el repo de gobernanza — lo que contradice la política de durabilidad ya aprobada en DEC-C10 punto 11. Se acepta el costo adicional (estimado en días, no en una fase completa aparte).

### 4. Orden de implementación propuesto

Dependencias entre bloques, no solo preferencia:

1. **Bloque A (Credenciales)** — primero. Todo lo demás depende de tener dónde guardar tokens de GitHub App y API keys.
2. **Bloque B (Proyectos)** — depende de A (crear recursos reales requiere credenciales funcionando).
3. **Bloque C (Identidad de agentes)** — puede avanzar en paralelo a A/B, dependencia baja.
4. **Bloque D (Sala de chat)** — depende de A (API keys de agentes) y de B (`sessions.project_id` requiere que exista al menos un proyecto).
5. **Bloque E (Lectura/escritura real + exportación)** — depende de A y D (necesita la sala funcionando para tener contenido que exportar).

### 5. No objetivos del MVP1.0

Quedan fuera, aunque ya estén diseñados en Decisions vigentes:
- Kanban (DEC-C12).
- Roadmap/timeline (DEC-C13).
- Mapa de tramas y cronología interna narrativa (DEC-C14/DEC-C15).

Este alcance no implica que estas capacidades pierdan prioridad futura — solo que MVP1.0 se cierra sin ellas.

## Pendiente

- Especificación técnica de cada bloque, bajada a code-tasks concretas por Chapu, siguiendo el orden de la sección 4.
- Detalle de bootstrapping de GitHub App (registro de la app en GitHub, instalación) — probablemente un paso de configuración manual único de Dani antes de que el Bloque A pueda operar, a definir en la especificación técnica.
