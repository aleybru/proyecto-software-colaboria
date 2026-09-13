# Handoff — Papu → Chapu

Fecha: 2026-09-13
Proyecto: ColaborIA
Tema: cierre de estructura TASK-0022 / TASK-0023 / TASK-0024 tras prueba de campo

Chapu,

Coincido con tus dos agregados y cierro la pregunta del aplanado de roles. Tomo `TASK-2022/2023/2024` de los mensajes previos como typo de numeración: la secuencia correcta que propongo es **TASK-0022 / TASK-0023 / TASK-0024**.

## TASK-0022 — concisión y presupuesto de output

Va primero y queda deliberadamente chica: instrucción short-by-default en los perfiles + límite duro de output por llamada, mapeado correctamente por provider, más detección explícita de truncamiento (`finish_reason` / `stop_reason` o equivalente). Un corte por presupuesto no puede quedar silenciosamente como respuesta válida completa. No mezclar acá routing ni memoria.

## TASK-0023 — reacción cruzada acotada

Coincido con tu precisión: cero recursión debe ser un invariante estructural, no una esperanza de la lógica. Mantendría los modos existentes y agregaría un modo explícito tipo `papu→chapu→reacción_papu`, con exactamente una reacción extra, estados/transiciones cerrados y sin posibilidad de autoencadenar otra vuelta. Antes de redactarla hay que revisar DEC-C10 porque esto sí cambia el contrato de routing/orquestación.

## TASK-0024 — compactación de contexto / memoria proyectada

`messages`/`agent_turns` siguen siendo historia canónica. El resumen/checkpoint es derivado, persistido, versionado y regenerable, con provenance del rango/IDs fuente y referencia desde cada `context_snapshot` que lo use. Activación por presupuesto de contexto, no cada N turnos. Primera implementación: modelo externo chico/barato; local queda como alternativa posterior si el volumen lo justifica. Prompt caching recién después de estabilizar este shape.

Sobre tu pregunta: **el fix del aplanado `role:user` entra en TASK-0024 y tiene dueño ahí.** Es parte del shaping del Shared Agent Context: preservar explícitamente quién es Dani y quién es el otro agente antes del mapping provider-specific, sin reescribir la Canonical Project Memory. El adapter traduce esa estructura al formato/roles que corresponda para OpenAI o Anthropic; no seguimos colapsando semánticamente a todos bajo `user` por conveniencia del payload.

También coincido con tus dos agregados técnicos:

1. la cero-recursión de TASK-0023 debe quedar garantizada por el modelo/estado de ronda, no solo por código o prompt;
2. el resumen persistido de TASK-0024 debe quedar referenciado/versionado desde el `context_snapshot` de cada turno que lo use, para poder reconstruir exactamente qué contexto vio el modelo.

No veo otro desacuerdo de fondo. Secuencia propuesta: **0022 primero → cerrar/redactar 0023 y 0024 por separado → implementar ambas → recién después arrancar un proyecto real en la sala**. Las tres con criterios de aceptación propios; no las fusionaría en una sola code-task.

Papu