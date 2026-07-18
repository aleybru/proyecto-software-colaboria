# DEC-C10 — Canal conversacional V0: modelo de datos, floor lock y política de durabilidad

**Estado:** Decision vigente — aprobada por Dani el 2026-07-18
**Fecha:** 2026-07-18
**Origen:** propuesta inicial de Papu (`handoffs/active/HANDOFF-2026-07-18-papu-to-chapu-canal-conversacional-v0.md`), análisis y 6 ajustes de Chapu (`handoffs/active/HANDOFF-2026-07-18-chapu-response-canal-conversacional-v0.md`), acuerdo de Papu incorporando ajustes propios (`handoffs/active/HANDOFF-2026-07-18-papu-response-canal-e-interfaz-unica.md`), confirmado por Dani en conversación directa con Chapu.
**Relacionada con:** DEC-C08, DEC-C09, DEC-C11 (interfaz operativa única — aplica el canal definido acá).

## Resumen

Se define el modelo de datos y las reglas operativas del canal conversacional que permite a Dani, Papu y Chapu coordinar trabajo dentro de ColaborIA, con control de turnos ("floor lock") gestionado exclusivamente por el backend — nunca por los agentes.

## Decisión

### 1. `sessions.project_id` obligatorio

Toda sesión de conversación pertenece a un proyecto. Sin esto, el backend no puede resolver qué protocolo (L0), manifest (L1), agentes, fuentes ni estado de proyecto (L4) corresponde cargar en el contexto de cada agente invocado. No se admite `sessions` sin proyecto asociado.

### 2. Fuente de verdad de contenido: `messages` vs. bitácora de ejecución: `agent_turns`

- **`messages`** es la fuente de verdad de **qué se dijo** — lo que se muestra en el chat, sin excepción.
- **`agent_turns`** es la bitácora de **cómo se generó** cada respuesta (modelo invocado, tiempos, estado de ejecución, contexto usado) — trazabilidad y debug, no contenido a mostrar directamente.

Al completarse un turno de agente exitosamente, se crea un `message` nuevo. `agent_turns.output_text` (si existe como campo) es un buffer de trabajo, no una segunda fuente de verdad del contenido.

### 3. Idempotencia del mensaje de entrada

Todo mensaje entrante desde el frontend debe llevar una clave de idempotencia (`client_message_id` o equivalente) para que un doble envío (doble click, reintento de red) no genere `message`/ronda duplicados.

### 4. Timeouts, retry y recuperación de turnos

Todo `agent_turn` tiene un timeout explícito. Un turno que excede el timeout no queda indefinidamente en `running` — pasa a `failed` y queda disponible para reintento manual. El mecanismo exacto de retry (automático acotado vs. siempre manual) queda para la especificación técnica de TASK-0003, pero el principio de que ningún turno puede quedar huérfano sin límite de tiempo es de esta Decision.

### 5. Política de fallo del primer turno en ronda con segundo agente pendiente

Si el primer turno de una ronda falla y hay un segundo turno programado (modo `both_sequential`), el segundo turno **no se ejecuta automáticamente sin contexto del primero** — la ronda completa pasa a estado que requiere decisión explícita (reintentar el primero, cancelar la ronda, o continuar sin el primero solo si así se indica explícitamente). No se asume ningún comportamiento por default.

### 6. `input_context_ref` como referencia trazable, no campo decorativo

Cada `agent_turn` debe poder reconstruir exactamente qué contexto se le dio al agente invocado: protocolo L0, manifest L1 del proyecto, estado L4 relevante, y el historial de `messages` de la sesión hasta ese punto. `input_context_ref` apunta a ese paquete de forma verificable — no es un campo de texto libre sin estructura definida.

### 7. Modos de conversación: `both_sequential` para V0, `both_blind` a futuro

Para V0, el modo `both_sequential` (el segundo agente ve la respuesta del primero) cubre coordinación operativa y conversación general. Se reconoce explícitamente la tensión con la revisión ciega que pide el protocolo L0 (sección 11) para evitar anclaje cuando hay juicio o contraste de posiciones real de por medio. Una variante `both_blind` (el segundo agente no ve la respuesta del primero hasta responder el suyo) queda señalada como necesidad futura, no incluida en V0.

### 8. Floor lock y orden de turnos bajo autoridad exclusiva del backend

El control de quién habla y cuándo nunca queda a criterio de un agente LLM invocado — es una decisión de infraestructura del backend, determinística y auditable.

### 9. Deprecación formal de `handoffs/quick/log.md` como mecanismo cotidiano

Una vez que el canal conversacional de ColaborIA esté validado y operativo, `handoffs/quick/log.md` deja de ser el mecanismo de coordinación del día a día entre Dani, Papu y Chapu. Esta deprecación es efectiva **desde la validación del canal en producción**, no desde la fecha de esta Decision — hasta entonces, QC-archivo sigue siendo el mecanismo vigente.

### 10. `handoffs/active/` se conserva

Los handoffs formales (análisis extensos, evidencia, desacuerdos sustantivos, resultados que deben quedar como referencia durable) siguen usando `handoffs/active/` tal como está definido en el protocolo común — el canal conversacional no reemplaza este mecanismo, salvo que en el futuro se defina una migración equivalente dentro de ColaborIA.

### 11. Política de durabilidad

La base de datos operacional (DEC-C09) puede ser la fuente activa del canal conversacional en tiempo real, pero **todo objeto que cambie el estado del proyecto o deba ser auditable tiene que materializarse en el repositorio de gobernanza** — mediante exportación, sincronización, o creación explícita del objeto formal correspondiente (Decision, Open Question, Result). Esto preserva el principio de DEC-C09: la DB no reemplaza la gobernanza durable versionada.

## No objetivos de esta Decision

- No define el mecanismo técnico exacto de exportación DB → repo (queda para TASK-0003 o una tarea posterior).
- No implementa `both_blind` — solo lo reconoce como necesidad futura.
- No resuelve permisos de acceso por agente a los distintos repos/Drive — eso se resuelve en DEC-C11.

## Pendiente

- Especificación técnica completa para TASK-0003 (modelo de datos exacto, mecanismo de retry, formato de `input_context_ref`).
- Mecanismo concreto de exportación/sincronización DB → repo de gobernanza (punto 11).
- Fecha o criterio de validación que dispare la deprecación efectiva de QC-archivo (punto 9).
