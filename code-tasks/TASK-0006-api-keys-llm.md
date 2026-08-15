---
id: TASK-0006
status: done
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

## Resultado

Ejecutado el 2026-08-15 en el repo `aleybru/proyecto-software-colaboria-codigo`, sobre `backend/ColaborIA.Api` (rama basada en `main` en el mismo punto que TASK-0004/0005 — TASK-0006 sólo depende de TASK-0003, ya mergeada).

**✅ Verificación end-to-end completa**, incluyendo ambos proveedores con keys reales provistas por Dani en el chat durante la misma sesión (nunca commiteadas, usadas una vez contra el backend local y descartadas de la conversación).

### Qué se hizo

Esta tarea resultó bastante más chica que TASK-0004/0005 porque los puntos 1 y 2 del pedido (alta/reemplazo y revocación de las keys) **ya los cubre el mecanismo genérico de `credentials` de TASK-0003 sin ningún cambio**: `POST /credentials` con `provider: "anthropic"` o `"openai"`, `kind: "api_key"`, `scopeType: "global"` da de alta o reemplaza; `POST /credentials/{id}/revoke` revoca. No se agregaron endpoints nuevos para eso — sería duplicar lógica ya escrita y probada.

Lo único nuevo:

1. **`Services/LlmKeys/LlmKeyValidationService.cs`**: busca la credencial activa (`provider`+`kind: api_key`+`scope: global`) para el proveedor pedido, la descifra, y hace una llamada mínima real de sólo lectura contra la API del proveedor:
   - **Anthropic**: `GET https://api.anthropic.com/v1/models` con headers `x-api-key` y `anthropic-version`.
   - **OpenAI**: `GET https://api.openai.com/v1/models` con `Authorization: Bearer`.
   
   Ambas son la llamada de prueba mínima estándar recomendada por cada proveedor para validar una key sin costo (no invocan al modelo, sólo listan metadata). Elegidas porque son gratis, rápidas, y no requieren elegir un modelo específico de antemano.
2. **`GET /integrations/llm/{provider}/status`** (`Endpoints/LlmKeysEndpoints.cs`): mapea el status code de la respuesta del proveedor a los cuatro estados que pedía la tarea:
   - `200` → `valid`.
   - `401`/`403` → `invalid_or_revoked`.
   - Cualquier otro código, o error de red → `error` (con detalle legible, sin exponer la key).
   - Sin credencial activa para ese proveedor → `not_configured`.
   
   El cuarto estado que pedía la tarea, "configurada, sin verificar", es el estado implícito entre el `POST /credentials` que da de alta la key y la primera vez que se llama a este endpoint — no es algo que este endpoint devuelva (hace el chequeo en vivo siempre que se lo llama), mismo criterio que "configured_unverified" en TASK-0004/0005 lo devolvía el paso de guardar, no el de verificar.
3. **Distinción entre "revocada por nosotros" y "revocada/inválida según el proveedor"**: si Dani llama a `POST /credentials/{id}/revoke`, la fila pasa a `status: revoked` en la DB y el filtro de `LlmKeyValidationService` (que sólo busca `status: active`) deja de encontrarla — el endpoint de status devuelve `not_configured`, no `invalid_or_revoked`. Ese segundo estado es específicamente para cuando la credencial sigue activa en nuestra DB pero el *proveedor* la rechaza (revocada desde su lado, expirada, etc.) — son dos cosas conceptualmente distintas y el código las trata distinto a propósito.
4. `/README.md` actualizado con el endpoint nuevo y un ejemplo de alta reusando `/credentials`.

### Verificación real (no sólo compilación)

- `dotnet build` / `dotnet test` (Release, mismo motivo de las tareas anteriores — proceso de depuración de Dani corriendo): compilan y pasan sin errores — 11/11 tests en esta rama (4 de TASK-0003 + 7 nuevos de `LlmKeyValidationServiceTests`: mapeo `200→valid`, `401`/`403→invalid_or_revoked`, otros códigos→`error`, y proveedor no soportado→`error` sin tocar DB/red).
- **Contra el backend real corriendo** (Postgres real, puerto separado de la sesión de Dani):
  - `GET /integrations/llm/anthropic/status` y `/openai/status` sin ninguna key configurada → `not_configured` en ambos, limpio.
  - `GET /integrations/llm/mistral/status` (proveedor no soportado) → `error` explícito, no 500.
  - Key falsa de Anthropic (`sk-ant-fake-invalid-key-...`) dada de alta vía `POST /credentials` → `GET /integrations/llm/anthropic/status` **llamó de verdad a `api.anthropic.com`**, que respondió `401` → `invalid_or_revoked`.
  - Mismo patrón con una key falsa de OpenAI contra `api.openai.com` → `401` → `invalid_or_revoked`.
  - `POST /credentials/{id}/revoke` sobre la key falsa de Anthropic → `GET .../status` pasó a `not_configured` (no `invalid_or_revoked`), confirmando la distinción del punto 3 de arriba.
  - **Con keys reales de Dani** (una de Anthropic, una de OpenAI, provistas directamente en el chat y usadas una sola vez contra el backend local, nunca commiteadas ni reescritas en ningún archivo): `POST /credentials` para cada una, y `GET /integrations/llm/{provider}/status` devolvió **`{"status":"valid","detail":null}` en ambos casos** — confirmación real y completa del criterio de éxito, cerrando también el camino "válida" que con keys falsas no se podía probar.
  - **Logs revisados en cada paso**: ninguna key (falsa o real) apareció en ningún momento — `HttpClientFactory` sólo loguea método+URL+status code, nunca headers ni body.
  - Verificado en Postgres al final: 2 filas activas (`anthropic`, `openai`), ambas cifradas — **se dejaron guardadas** (no se borraron, son las keys reales que Dani va a usar), listas para cuando el Bloque D de DEC-C16 implemente la sala de chat con agentes reales.

### Decisiones técnicas menores tomadas (no contradicen DEC-C09/DEC-C11/DEC-C16)

- **No se agregaron endpoints nuevos de alta/revocación**: se reutilizan `POST /credentials` y `POST /credentials/{id}/revoke` de TASK-0003 tal cual, sin ningún endpoint específico de LLM para esto — evita duplicar lógica ya escrita y probada, y la tarea no pedía explícitamente que fueran endpoints nuevos, sólo que existiera el mecanismo ("vía el servicio de credenciales de TASK-0003").
- **`GET /v1/models` como llamada de prueba mínima** para ambos proveedores — gratis, no consume tokens, no exige elegir un modelo, y es el patrón recomendado por ambos proveedores para este propósito.
- **Un único endpoint `GET /integrations/llm/{provider}/status`** parametrizado por proveedor, en vez de rutas separadas `/anthropic/status` y `/openai/status` — la lógica de validación es idéntica salvo el header de auth y la URL, así que un solo endpoint con un `switch` interno es más simple que duplicar rutas.
- **"configurada, sin verificar" como estado implícito, no devuelto explícitamente** — mismo razonamiento aplicado en TASK-0004/0005 para "configured_unverified".

### Pendiente / no hecho en esta tarea

- Invocación real de agentes en la sala de chat — Bloque D de DEC-C16, fuera de alcance explícito de esta tarea.
- No se hizo `git commit` ni push — queda en el working tree para revisión antes de confirmar el commit.
