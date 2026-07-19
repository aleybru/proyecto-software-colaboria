---
id: TASK-0005
status: pending
repo_destino: aleybru/proyecto-software-colaboria-codigo
creado: 2026-07-19
aprobado_por: Dani
depende_de: [TASK-0003]
decision_ref: DEC-C16
---

## Objetivo

Integrar Google Drive vía OAuth con acceso offline, conforme al Bloque A de DEC-C16 (agregado explícitamente tras la corrección de Papu — estaba ausente de la redacción original de esa Decision).

## Qué tiene que hacer Code (concreto)

1. Implementar el flujo de OAuth de Google con `access_type=offline` (para obtener refresh token), con los scopes mínimos necesarios para leer/escribir en la carpeta de Drive del proyecto (no acceso completo a todo el Drive de la cuenta).

2. Flujo de consentimiento inicial: Dani autoriza una vez, vía navegador, siguiendo el flujo estándar de OAuth de Google.

3. Guardar el refresh token, vía el servicio de credenciales de TASK-0003 (`provider: google_drive`), cifrado.

4. Implementar la renovación automática del access token usando el refresh token guardado, sin requerir que Dani vuelva a autorizar en cada uso.

5. Endpoint de prueba de conexión: verificar que puede autenticarse y listar el contenido de la carpeta Drive configurada.

6. Manejo de revocación/invalidez: si el refresh token deja de ser válido (Dani revocó el acceso desde su cuenta de Google, u otro motivo), el estado de la integración lo refleja claramente y ofrece el camino para volver a autorizar — no falla en silencio.

## Criterio de éxito

- Dani completa el consentimiento una sola vez y la integración queda operativa sin volver a pedir autorización en usos posteriores.
- El endpoint de prueba de conexión lista correctamente el contenido de la carpeta Drive configurada.
- El refresh token no aparece en logs ni se expone por ningún endpoint.
- Si se revoca el acceso desde Google, la aplicación lo detecta y lo muestra como error de estado, no como una falla no explicada.

## Contexto / restricciones

- No implementar todavía operaciones de escritura reales de archivos/documentos (Decisions, código, etc.) — eso es Bloque E de DEC-C16, tarea posterior. Esta tarea es solo autenticación y una prueba de lectura.
- Scopes mínimos: no pedir permisos de Drive más amplios de los estrictamente necesarios para operar sobre la carpeta del proyecto.
- Respetar DEC-C08: elección de librería cliente de Google OAuth para .NET queda a criterio de Code.
