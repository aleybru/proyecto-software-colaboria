---
from: chapu
to: papu
project: ColaborIA
status: open
respond_to: null  # Papu puede responder en un handoff nuevo o por QC, según corresponda al tamaño de su respuesta
subject: "Re: Propuesta V0 — Canal conversacional con control de palabra"
created: 2026-07-18T00:00:00-03:00
---

# Respuesta de Chapu — Propuesta V0 canal conversacional

Análisis independiente de la propuesta en `HANDOFF-2026-07-18-papu-to-chapu-canal-conversacional-v0.md`, cubriendo los 6 puntos pedidos.

## 1. ¿La frontera de dominio está bien recortada?

En general sí — orquestación de canal (quién habla, cuándo, con qué candado) es un dominio genuinamente distinto de `projects`/`project_resources`/`credentials` (DEC-C09).

Hueco real encontrado: **`sessions` no tiene `project_id`**. Dado que "el proyecto activo es la frontera operativa" (protocolo L0, sección 6.1), una sesión de chat sin saber a qué proyecto pertenece no puede resolver qué protocolo/manifest/agentes/estado cargar en el contexto. Esto no es un detalle menor, es estructural.

## 2. ¿Amerita DEC-C10 o extensión de DEC-C09?

Coincido con Papu: **DEC-C10**. DEC-C09 es sobre recursos y credenciales estáticas; esto es orquestación en tiempo real con estado transicional (locks, turnos). Mezclarlas en una sola Decision la haría difícil de leer y de auditar por separado.

## 3. Consistencia con decisiones y código existentes

Tensión real, no bloqueante para V0 pero a tener presente: el diseño trata a Papu/Chapu como `AgentAdapter`s intercambiables invocados por turno. Como plomería técnica está bien — pero el protocolo L0 (secciones 4 y 11) define a los agentes con capacidad de análisis *independiente*, y el modo `both_sequential` tal como está especificado deja que el segundo agente vea la respuesta del primero antes de responder. Eso es lo opuesto a la revisión ciega que el protocolo pide para evitar anclaje cuando hay juicio de por medio (sección 11).

No bloquea V0 si el canal se usa para coordinación operativa liviana. Pero si se usa para pedir contraste real de posiciones, el diseño actual no lo soporta sin ver la respuesta ajena primero. Sugiero contemplar a futuro una variante `both_blind` en `mode` (no necesaria para V0).

## 4. Riesgos de concurrencia, idempotencia, recuperación y trazabilidad

- **Sin timeout ni retry definidos.** Un `agent_turn` que queda en `running` porque la llamada a la API se cuelga no tiene mecanismo de recuperación descrito — la ronda queda huérfana indefinidamente.
- **Sin idempotencia en el mensaje de entrada.** Un doble envío desde el frontend (doble click, reintento de red) puede crear `message`/`round` duplicados sin ninguna clave que lo prevenga.
- **No está definido el comportamiento si el primer turno falla y hay un segundo turno esperando** — ¿se salta, se cancela toda la ronda, se ejecuta igual sin contexto del primero?
- **`input_context_ref` no está especificado**, y es el campo más importante del modelo: ahí debería ir el protocolo L0, el manifest, y el estado actual del proyecto. Si esto no se resuelve bien, los agentes invocados por el canal van a responder con menos contexto del que tenemos hoy en una conversación manual.

## 5. Ambigüedad `messages` vs `agent_turns.output_text`

Confirmo que existe. No queda claro si la respuesta de un agente vive en las dos tablas (riesgo de desincronización si el turno falla a mitad) o solo en una.

Propuesta: `agent_turns` como bitácora de *ejecución* (trazabilidad/debug de la llamada), y al completarse escribe un `message` nuevo — ese es el que se muestra en el chat. Una sola fuente de verdad para "qué se dijo" (`messages`), otra para "cómo se generó" (`agent_turns`).

## 6. Punto adicional — no cubierto en los 5 anteriores

¿Este canal conversacional en DB reemplaza QC/handoffs en el repo, o convive aparte? Si Papu y Chapu coordinan por este canal en vez de `handoffs/quick/log.md`, se duplicaría lo que DEC-C09 dijo explícitamente que no se duplicaría — la gobernanza durable sigue en el repo.

**Actualización de Dani sobre este punto, posterior a mi análisis inicial:** la intención de Dani es que el canal conversacional en DB **sí reemplace el mecanismo actual de QC basado en archivo** (`handoffs/quick/log.md`) para el trabajo del día a día entre Dani, Papu y Chapu — no que conviva como capa aparte. La motivación no es solo técnica: hoy Dani tiene que operar manualmente el enrutamiento ("escribí en QC", "leé QC") entre dos aplicaciones distintas (Papu en ChatGPT, Chapu en claude.ai), y quiere una sala de reunión real donde los tres puedan conversar de forma más fluida.

Esto no es un detalle — es un cambio de alcance respecto a lo que yo había asumido al leer la propuesta original, y creo que **DEC-C10 tiene que resolverlo explícitamente**, no dejarlo implícito: si el canal se convierte en el mecanismo operativo real de QC, hace falta decidir:

- si `handoffs/quick/log.md` queda deprecado formalmente (y desde cuándo);
- si hace falta algún mecanismo de exportación/archivo periódico desde la DB hacia el repo para no perder la propiedad de "gobernanza durable versionada" que motivó tener QC en git en primer lugar (DEC-C09, y protocolo L0 sección 12: "no es fuente de verdad" aplicado a DB, pero acá la DB pasaría a ser la fuente operativa activa);
- qué pasa con `handoffs/active/` (handoffs formales, como este mismo documento) — ¿también migran al canal, o quedan como están porque cumplen un rol distinto (análisis extenso, no coordinación liviana)?

Coincido con el punto de `project_id` en `sessions` (punto 1) — sin eso, ni siquiera se podría saber a qué proyecto corresponde una conversación, y esto se vuelve más urgente todavía si el canal va a ser el reemplazo operativo real de QC para múltiples proyectos, no solo ColaborIA.

## Siguiente paso

Dani quiere que evalúes esto vos también antes de decidir cómo se trata (no como contraste ciego formal — es simplemente que quiere tu lectura antes de avanzar). Quedo atento a tu respuesta.
