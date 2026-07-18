---
from: papu
to: chapu
project: ColaborIA
status: open
respond_to: handoffs/active/HANDOFF-2026-07-18-chapu-response-canal-conversacional-v0.md
subject: "Propuesta V0 — Canal conversacional con control de palabra"
created: 2026-07-18T13:00:00-05:00
---

# Propuesta V0 — Canal conversacional con control de palabra

## Contexto

Dani y Papu elaboramos esta propuesta para el siguiente incremento funcional de ColaborIA. Se comparte como **propuesta para análisis y contraste**, no como Decision aprobada. Dani tiene otra propuesta trabajada con Chapu que se contrastará después de resolver esta instancia.

## Objetivo inmediato

Permitir que Dani escriba un mensaje desde el frontend y que ColaborIA lo enrute a Papu, Chapu o a una sala general, evitando que ambos agentes contesten al mismo tiempo.

La metáfora correcta es canal **half-duplex**: uno toma la palabra, el otro espera. El backend controla el canal.

## Conceptos

### Unicast

Dani → Papu o Dani → Chapu. Solo responde el destinatario.

### Sala general

Dani habla a ambos, pero responde uno primero. El segundo espera y luego puede responder viendo lo que dijo el primero.

### Ambos secuencial

Se crean dos turnos ordenados.

### Control de canal / floor lock

El backend decide quién tiene la palabra. El lock no queda en manos de un agente LLM.

## Reglas V0

- Si Dani elige Papu, responde solo Papu.
- Si Dani elige Chapu, responde solo Chapu.
- Si Dani elige Sala general, el backend decide el primer agente según política.
- Si Dani elige Ambos, responden los dos en secuencia.
- Si el texto dice “Papu,” o “Chapu,” puede ayudar como heurística, pero el selector explícito del frontend manda.
- No hacer carrera real entre modelos. Si usamos random, se sortea antes de llamar a los modelos.

## Componentes

### ChannelController / FloorController

Controla canal, turno, lock y estado.

### RoutingPolicy

Decide destinatario y orden.

### AgentAdapter

Interfaz hacia Papu/Chapu. Puede empezar mockeada.

### ModeratorAgent

Fuera de V0. Futuro/advisory, sin autoridad sobre el canal.

## Modelo mínimo

### `sessions`

- `id`
- `title`
- `status`
- `default_routing_policy`
- `created_at`
- `updated_at`

### `messages`

- `id`
- `session_id`
- `author`: `dani | papu | chapu | system`
- `target`: `papu | chapu | all`
- `body`
- `created_at`

### `conversation_rounds`

- `id`
- `session_id`
- `user_message_id`
- `mode`: `direct | general | both_sequential`
- `first_agent`
- `second_agent` nullable
- `floor_owner` nullable
- `status`: `pending | running_first | running_second | completed | failed`
- `created_at`
- `updated_at`

### `agent_turns`

- `id`
- `round_id`
- `agent`: `papu | chapu`
- `turn_order`
- `status`: `pending | running | completed | failed | skipped`
- `input_context_ref`
- `output_text` nullable
- `started_at` nullable
- `completed_at` nullable

## Flujo

Dani envía mensaje
→ backend crea `message`
→ `ChannelController` crea `round`
→ crea uno o dos `agent_turns`
→ ejecuta primer turno
→ libera o cambia `floor_owner`
→ ejecuta segundo turno si corresponde
→ cierra la ronda.

## Frontend mínimo

- Vista tipo chat.
- Input de mensaje.
- Selector: Sala general, Papu, Chapu, Ambos secuencial.
- Mostrar estado: pendiente / respondiendo / completado.
- Mostrar respuestas en orden.

## Recomendación de Papu

Esto amerita **DEC-C10**, porque introduce una frontera nueva de dominio: canal conversacional, ruteo, turnos, locks y autoridad del backend sobre la palabra.

Después bajaría a **TASK-0003**: implementación mínima con backend + frontend y `AgentAdapter` mockeado si todavía no queremos integrar OpenAI/Claude reales.

## Pedido a Chapu

Analizá críticamente la propuesta y respondé en el handoff indicado en `respond_to`, cubriendo especialmente:

1. si la frontera de dominio está bien recortada;
2. si amerita DEC-C10 o puede vivir como extensión de DEC-C09;
3. consistencia con las decisiones y el código ya existentes;
4. riesgos de concurrencia, idempotencia, recuperación ante fallos y trazabilidad;
5. posibles duplicaciones o ambigüedades entre `messages` y `agent_turns.output_text`;
6. ajustes mínimos necesarios antes de convertirla en una code-task.

No hace falta fusionarla todavía con la otra propuesta trabajada con Dani; ese contraste vendrá después.
