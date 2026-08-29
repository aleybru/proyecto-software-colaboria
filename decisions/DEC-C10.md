# DEC-C10 — Canal conversacional V0: modelo de datos, floor lock y durabilidad

**Estado:** Decision vigente
**Fecha original:** 2026-07-18
**Revisión vigente:** 2026-08-29
**Autoridad:** Dani
**Relacionada con:** DEC-C08, DEC-C09, DEC-C11, DEC-C16

## Resumen

Se define el canal conversacional de ColaborIA y su trazabilidad. La revisión de 2026-08-29 elimina la dependencia arquitectónica de un repositorio de gobernanza por proyecto: la DB de ColaborIA pasa a sostener conversación, ejecución y objetos formales del producto. El repo de gobernanza actual de ColaborIA queda como infraestructura transicional del desarrollo del propio sistema.

## Decisión

### 1. `sessions.project_id` obligatorio

Toda sesión pertenece a un proyecto. El backend usa ese vínculo para resolver identidad de agentes, contexto, fuentes, permisos y aislamiento.

### 2. `messages` vs. `agent_turns`

- `messages` es la fuente de verdad de qué se dijo.
- `agent_turns` es la bitácora de cómo se generó una respuesta: agente, proveedor/modelo, tiempos, estado, contexto y error.

Un turno exitoso produce un `message`. Un buffer interno de `agent_turns` no es una segunda fuente de contenido.

### 3. Idempotencia

Todo mensaje entrante debe tener una clave de idempotencia (`client_message_id` o equivalente) para evitar duplicados por doble click, retry o pérdida de red.

### 4. Timeouts y recuperación

Todo `agent_turn` tiene timeout explícito. Un turno no puede quedar indefinidamente en `running`. Los fallos deben quedar trazables y disponibles para recuperación o reintento según política del backend.

### 5. Rondas con ambos agentes

En `both_sequential`, si falla el primer agente, el segundo no continúa automáticamente como si hubiera recibido una respuesta válida. La ronda requiere una acción explícita o una política determinística definida por backend.

### 6. `input_context_ref`

Cada turno debe poder reconstruir exactamente el contexto entregado al agente: protocolo aplicable, manifest/configuración del proyecto, perfil del agente, estado/Decisions relevantes en DB, historial necesario, fuentes recuperadas y referencias externas utilizadas.

`input_context_ref` es trazable y verificable, no texto decorativo.

### 7. Modos de conversación

V0 incluye `both_sequential`. `both_blind` queda como capacidad futura para revisión independiente cuando el caso lo requiera.

### 8. Floor lock

El backend controla exclusivamente quién habla y en qué orden. Ningún LLM controla el floor lock ni puede saltarse el routing determinado por ColaborIA.

### 9. QC actual y transición

`handoffs/quick/log.md` sigue vigente **para el desarrollo actual de ColaborIA** hasta que el canal de la aplicación esté validado para reemplazarlo.

No se crea ni se exige un QC basado en archivo para proyectos nuevos creados por ColaborIA. Su coordinación cotidiana vive en la DB y en la sala del proyecto.

### 10. Handoffs formales

El concepto de handoff formal se conserva porque tiene valor de dominio: análisis extenso, evidencia, desacuerdo sustantivo o transferencia de trabajo.

En la arquitectura objetivo, un handoff de un proyecto creado por ColaborIA es un objeto durable administrado por la aplicación/DB. El uso actual de `handoffs/active/` en el repo de gobernanza de ColaborIA es una implementación transicional del proceso de desarrollo, no un recurso que deba aprovisionarse para cada proyecto.

### 11. Durabilidad y gobernanza

La DB de ColaborIA es la fuente durable del canal y de los objetos formales que la propia aplicación gestione: Decisions, Open Questions, Tasks, Results, handoffs, aprobaciones, trazabilidad e historial operativo.

Reglas:

- la conversación ordinaria permanece en `messages`;
- conversación no es estado;
- promover contenido a Decision, Task, Result u otro objeto requiere el workflow y autorización correspondientes;
- un objeto formal debe conservar referencia a su conversación/sesión de origen;
- las transiciones relevantes deben ser auditables;
- no se requiere materializar esos objetos en un repo de gobernanza por proyecto.

Los artefactos documentales del proyecto se almacenan en su carpeta de Drive. El código de un proyecto software vive en su único repo de código.

El repo de gobernanza actual de ColaborIA permanece como excepción transicional y normativa del desarrollo de ColaborIA hasta decisión explícita de migración.

## No objetivos

- Implementar `both_blind` en V0.
- Definir todavía el esquema físico completo de todos los objetos de gobernanza.
- Migrar automáticamente el historial existente del repo de gobernanza de ColaborIA a la DB.

## Pendiente

- Especificación física de `sessions`, `messages`, `agent_turns` y objetos formales.
- Criterio de validación que permita retirar QC-archivo del desarrollo de ColaborIA.
- Estrategia futura de migración del corpus histórico del repo de gobernanza, si Dani decide realizarla.
