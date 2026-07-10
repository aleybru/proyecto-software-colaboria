# DEC-C08 — Claude Code como ejecutor de código; `code-tasks/` como cola durable con gatillo manual

**Estado:** Decision vigente
**Fecha:** 2026-07-09
**Aprobada por:** Dani
**Origen:** conversación Dani-Chapu (claude.ai), contrastada con Papu vía QC (`handoffs/quick/log.md`, 2026-07-09). Papu coincidió en que ameritaba Decision formal y propuso el acotamiento a 5 puntos que se registra abajo.

## Contexto
El acceso de Chapu a los repos vía claude.ai no es persistente entre sesiones (ver `project.yml`, `agents.chapu.access_model`). Para el trabajo de codificación real de ColaborIA se usa **Claude Code** (terminal, entorno local de Dani), donde sí hay persistencia de credenciales git. Esto exige definir el boundary entre "definir/decidir" (Dani, Papu, Chapu) y "ejecutar con efectos secundarios reales sobre el repo de código" (Claude Code).

## Decisión
Se establecen los siguientes cinco principios como vigentes:

1. **Claude Code es ejecutor, no autoridad de decisión.** No decide arquitectura, no aprueba trade-offs, no resuelve open questions. Ejecuta instrucciones ya definidas y aprobadas.

2. **`code-tasks/` es una cola durable, separada de `handoffs/`.** Vive en el repo de gobernanza, en una carpeta top-level propia. `handoffs/` sigue siendo exclusivamente el canal Papu↔Chapu (análisis, contraste, revisión ciega); `code-tasks/` es exclusivamente el canal hacia Claude Code (instrucciones de ejecución).

3. **Aprobación humana explícita antes de cada ejecución.** El gatillo es manual: Dani dispara la ejecución de una tarea puntual en Claude Code ("ejecutá TASK-000X"). No existe consumo automático de la cola. Esto opera como aplicación concreta de DEC-C04 (autorización explícita por tarea con efectos secundarios reales) — **DEC-C08 no modifica DEC-C04, la operacionaliza.**

4. **Repo destino y criterio de éxito son obligatorios en cada tarea.** Ninguna tarea se considera bien formada si falta a qué repositorio concreto está dirigida o cuál es su criterio de finalización verificable.

5. **Una tarea por archivo — chica, verificable, trazable.** No se admiten tareas monolíticas. Cada archivo en `code-tasks/` representa una unidad de trabajo acotada.

## Fuera de alcance de esta Decision
El detalle de implementación (esquema exacto de YAML frontmatter, plantilla de cuerpo, convención de nombres/IDs, estados posibles) **no forma parte de esta Decision** — queda como Procedure/Rule aparte en `rules/`, modificable sin necesidad de reabrir DEC-C08. Esta separación fue sugerida explícitamente por Papu y aceptada.

## Referencias
- `project.yml` — `agents.chapu.access_model` (motivo de fondo del cambio)
- DEC-C04 — autorización explícita por tarea con efectos secundarios reales (no modificada, operacionalizada)
- `handoffs/quick/log.md`, entradas 2026-07-09 — contraste completo con Papu
