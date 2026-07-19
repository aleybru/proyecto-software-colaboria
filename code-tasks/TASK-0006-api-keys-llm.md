---
id: TASK-0006
status: pending
repo_destino: aleybru/proyecto-software-colaboria-codigo
creado: 2026-07-19
aprobado_por: Dani
depende_de: [TASK-0003]
decision_ref: DEC-C11, DEC-C16
---

## Objetivo

Gestionar las API keys de Anthropic (Chapu) y OpenAI (Papu) que la sala de chat (DEC-C10, Bloque D de DEC-C16) va a necesitar para invocar a los agentes reales.

## Qué tiene que hacer Code (concreto)

1. Endpoint para dar de alta/reemplazar la API key de Anthropic y la de OpenAI, vía el servicio de credenciales de TASK-0003 (`provider: anthropic` / `provider: openai`, `scope_type: global` — no son credenciales por proyecto).

2. Endpoint de revocación (cambia `status`, no borra el registro — mismo criterio que TASK-0003).

3. Validación de conectividad real contra cada proveedor: una llamada mínima de prueba (ej. un request simple y barato a la API correspondiente) para confirmar que la key es válida, sin necesidad de esperar a que falle en la sala de chat.

4. Estado visible por key: `no configurada` / `configurada, sin verificar` / `válida` / `inválida o revocada`.

## Criterio de éxito

- Se puede dar de alta una key de Anthropic y una de OpenAI, y el endpoint de validación confirma conectividad real contra cada proveedor.
- Una key inválida (ej. revocada del lado del proveedor) se refleja como tal al validar, sin que la aplicación falle de forma opaca.
- Las keys no aparecen en logs ni se devuelven en texto plano por ningún endpoint.

## Contexto / restricciones

- No implementar todavía la invocación real de agentes en la sala de chat (eso es parte del Bloque D de DEC-C16, tarea posterior) — esta tarea es solo gestión y validación de las credenciales.
- `scope_type: global`, no `project` — estas keys no varían por proyecto (a diferencia de, por ejemplo, credenciales de GitHub que sí podrían ser por proyecto a futuro).
- Respetar DEC-C08: elección de qué endpoint específico de cada proveedor usar como "llamada de prueba mínima" queda a criterio de Code, documentado en `## Resultado`.
