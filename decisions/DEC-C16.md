# DEC-C16 — MVP1.0: alcance, criterio de cierre y orden de implementación

**Estado:** Decision vigente — aprobada por Dani el 2026-07-19
**Fecha:** 2026-07-18 (redacción inicial), cerrada 2026-07-19
**Origen:** definición conjunta Dani-Chapu; contraste y correcciones de Papu (`handoffs/active/HANDOFF-2026-07-18-papu-response-dec-c16-mvp1.md`) incorporadas en su totalidad antes de la aprobación.
**Relacionada con:** todas las Decisions vigentes (DEC-C08 a DEC-C15) — MVP1.0 es el primer recorte de implementación sobre ese diseño ya cerrado.

## Resumen

Define dónde termina el primer entregable funcional de ColaborIA: qué capacidades ya diseñadas (DEC-C09 a DEC-C15) entran en MVP1.0, con qué condiciones operativas reales, y en qué orden se implementan. No rediseña nada — todas las capacidades ya están fijadas en Decisions previas; esta Decision recorta el subconjunto, precisa las condiciones de cada bloque, y fija un criterio de cierre verificable.

## Decisión

### Bloque A — Credenciales e integraciones

Objetivo correcto (precisión de Papu): no "tokens que no vencen", sino **operación persistente sin renovación manual recurrente en condiciones normales**.

- **GitHub App**: identidad de la App, clave privada/referencia segura, instalación y permisos autorizados son la configuración durable. Los installation access tokens son efímeros y se generan automáticamente bajo demanda — no se guardan como credencial durable principal.
- **Google Drive**: OAuth con acceso offline y renovación automática vía refresh token, consentimiento inicial de Dani, scopes mínimos, validación y reconexión si la autorización se revoca o queda inválida. (Ausente en la redacción original de este bloque — corrección de Papu.)
- **API keys de OpenAI y Anthropic**: cifradas, con validación, rotación/revocación, sin exigir recarga manual frecuente en operación normal.
- El bloque incluye, como mínimo: cifrado real, alta/reemplazo/revocación, prueba de conexión, estado de credencial, enmascaramiento, renovación automática cuando el proveedor la soporte, y ausencia de secretos en logs/API.
- No se fijan estimaciones de tiempo dentro de esta Decision (se retira "uno o dos días" de la redacción original — un número no validado no debe leerse como compromiso).

### Bloque B — Proyectos y aprovisionamiento de recursos

- Crear proyecto, elegir tipo (`narrative` | `software`) — tabla `projects` (DEC-C09).
- Creación automática de recursos asociados, con la convención de nombres de DEC-C09 sección 7 — tabla `project_resources`.
- **Camino para registrar o vincular recursos ya existentes**, sin obligar a recrearlos — necesario para incorporar a ColaborIA misma (dogfooding) y proyectos ya iniciados. (Agregado por corrección de Papu.)
- El aprovisionamiento debe ser idempotente, reintentable, verificable, recuperable ante fallo parcial, y capaz de mostrar qué recurso se creó, cuál falló y cuál ya existía.

### Bloque C — Identidad de agentes y autorización efectiva

- `config/agents/papu.yml` y `config/agents/chapu.yml` (DEC-C11) — identidad, rol, habilidades, proveedor/modelo e instrucciones, en configuración versionada.
- **Precisión obligatoria (Papu):** identidad de agente no equivale a autorización. La autorización efectiva vive en el backend, no en el prompt — se aplica la matriz de DEC-C11 (Papu: gobernanza lectura/escritura, código y Drive solo lectura; Chapu: lectura/escritura en los tres) de forma que ningún agente puede exceder sus permisos aunque el modelo lo solicite. Dani autoriza los efectos secundarios.

### Bloque D — Sala de chat con contexto gobernado

- Canal conversacional completo de DEC-C10: `sessions` con `project_id` obligatorio, `messages`, `agent_turns`, floor lock bajo autoridad del backend, modos de conversación.
- Agentes reales (Claude y OpenAI vía API) desde el arranque — no hace falta fase de producto con agente simulado.
- **Adapters determinísticos de prueba como infraestructura obligatoria** (no fase de producto): necesarios para pruebas automatizadas, reproducir timeout/fallos, verificar idempotencia y orden sin costo de proveedor, correr CI sin depender de APIs externas. (Corrección de Papu.)
- **Bloque de contexto gobernado (agregado, corrección de Papu — el hueco más importante señalado):** sin esto, la sala con APIs reales produce dos asistentes genéricos, no Papu y Chapu operando dentro de ColaborIA. Cada turno debe reconstruir, vía `input_context_ref` (DEC-C10 punto 6), como mínimo: protocolo vigente, manifest del proyecto, perfil del agente, Decisions y estado relevante, historial de sesión necesario, fuentes/archivos recuperados para la consulta, y aislamiento estricto por proyecto. No implica implementar el Context Builder completo — sí el mínimo suficiente para operación real y trazable.

### Bloque E — Gateway de recursos tipado

**Reformulado (corrección de Papu):** no es reproducir de forma genérica lo que hoy hace Chapu con bash y curl en esta conversación — es un gateway de herramientas controlado por backend, con operaciones tipadas y auditables como mínimo:

- localizar/leer archivo o documento;
- crear/actualizar objeto formal de gobernanza;
- crear/actualizar `code-task` aprobada;
- leer/escribir artefacto autorizado;
- verificar la escritura y devolver referencia exacta (no afirmar éxito antes de verificarlo);
- rechazar operación fuera de permisos.

Cada efecto secundario registra proyecto, solicitante, agente, recurso, autorización, resultado y error.

### Materialización durable (corrección obligatoria de Papu sobre la redacción original)

La redacción original de esta sección decía que si la conversación queda solo en DB se contradice DEC-C10 — **no es exacto**, y se corrige:

- el historial ordinario de chat puede y debe permanecer en DB como fuente activa (`messages`, DEC-C10 punto 2);
- no corresponde copiar automáticamente cada mensaje al repo, ni mantener dos logs competidores;
- cuando Dani aprueba/promueve una propuesta, Decision, Open Question, Task, Result o handoff formal, se crea el objeto durable correspondiente en el repo — eso, y solo eso, es lo que exige DEC-C10 punto 11;
- la materialización debe ser idempotente, verificable, y enlazar el objeto formal con el mensaje/sesión de origen;
- una exportación explícita de sesión completa puede existir después como función de archivo, pero no es requisito de durabilidad cotidiana.
- QC (`handoffs/quick/log.md`) sigue vigente hasta validar el canal en producción (DEC-C10 punto 9); no se convierte en espejo permanente del chat.

### Criterio de cierre del MVP1.0 (agregado — no existía en la redacción original)

MVP1.0 se considera cerrado cuando Dani puede, con proveedores y recursos reales (no simulados):

1. configurar GitHub App, Google Drive, OpenAI y Anthropic;
2. crear un proyecto `software` y uno `narrative` (o al menos probar ambos tipos);
3. crear o vincular sus recursos y recuperarse de un fallo parcial;
4. entrar a la sala del proyecto y conversar con Papu, Chapu, y ambos secuencialmente;
5. recargar/reiniciar y conservar proyecto, recursos, sesión e historial;
6. comprobar que cada agente recibe identidad y contexto correctos;
7. leer recursos autorizados desde la sala;
8. autorizar una escritura de Papu en gobernanza y una de Chapu en Drive/código;
9. comprobar que una escritura no permitida de Papu en código o Drive es rechazada por el backend;
10. promover un contenido aprobado a objeto formal y verificar su materialización en gobernanza;
11. recibir enlaces/referencias exactas de las operaciones realizadas;
12. crear un segundo proyecto y comprobar ausencia de contaminación de contexto/recursos;
13. recuperar de forma visible errores de proveedor, credencial, timeout y recurso, sin corromper estado.

### Orden de implementación (reformulado — menos serial que la versión original)

1. **Fundación segura de integraciones**: cifrado, GitHub App, Drive OAuth, claves LLM, validación y renovación.
2. **Proyectos y aprovisionamiento**: CRUD, tipos, crear/vincular recursos, idempotencia y recuperación parcial.
3. **Perfiles y política de agentes** — puede avanzar en paralelo con 1 y 2.
4. **Núcleo conversacional persistente**: sesiones, mensajes, turnos, floor lock, frontend de sala, adapters determinísticos de prueba.
5. **Contexto gobernado + proveedores reales**: `input_context_ref`, recuperación mínima, OpenAI/Anthropic, aislamiento por proyecto.
6. **Gateway de recursos y autorizaciones**: lectura/escritura tipada, matriz DEC-C11, confirmaciones, auditoría.
7. **Materialización durable y cierre operativo**: promoción a objetos formales, verificación, manejo de errores, reinicio, aislamiento, prueba de aceptación completa (sección anterior).

### No objetivos del MVP1.0

Quedan fuera, aunque ya estén diseñados en Decisions vigentes: Kanban (DEC-C12), Roadmap/timeline (DEC-C13), mapa de tramas y cronología interna narrativa (DEC-C14/DEC-C15). No implica pérdida de prioridad futura.

## Pendiente

- Descomposición del primer hito (Fundación segura de integraciones) en code-tasks pequeñas y verificables — cola `Ready` de 2-3 tareas, no todo el backlog de una vez.
- Detalle de bootstrapping de GitHub App (registro e instalación) — probablemente un paso de configuración manual único de Dani, a definir en especificación técnica.
