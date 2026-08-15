---
id: TASK-0005
status: done
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

## Resultado

Ejecutado el 2026-08-15 en el repo `aleybru/proyecto-software-colaboria-codigo`, sobre `backend/ColaborIA.Api` (rama basada en `main` antes de que se mergeara TASK-0004 — TASK-0005 sólo depende de TASK-0003, ya mergeada, así que no hacía falta esperar).

**✅ Verificación end-to-end completa con un client OAuth real y consentimiento real de Dani**, hecha en la misma sesión (igual que terminó pasando con TASK-0004). Dani registró un OAuth client en Google Cloud Console, agregó su cuenta como test user, y completó el consentimiento vía navegador. En el camino apareció y se corrigió un bug real (ver más abajo) que sin esta verificación real habría quedado sin detectar.

### Qué se hizo

1. **Modelo de credentials para Drive** (`Services/GoogleDrive/GoogleDriveConfig.cs`): dos credenciales separadas bajo `provider: google_drive` (a diferencia de GitHub App, acá sí tiene sentido separarlas — tienen ciclos de vida distintos):
   - `kind: oauth_client_config` (`client_id`, `client_secret`, `redirect_uri`, `driveFolderId`) — config durable del proyecto OAuth, la registra Dani una vez en Google Cloud Console.
   - `kind: refresh_token` — el refresh token en sí, **sí se persiste cifrado** (a diferencia del installation token de GitHub, que es efímero): así lo pide DEC-C16 Bloque A ("renovación automática vía refresh token") y el punto 3 de esta tarea explícitamente.
2. **Flujo OAuth** (`Services/GoogleDrive/GoogleDriveService.cs` + `Endpoints/GoogleDriveEndpoints.cs`):
   - `POST /integrations/google-drive/config` — guarda la config del cliente OAuth cifrada. No inicia el consentimiento.
   - `GET /integrations/google-drive/authorize` — arma la URL de consentimiento de Google (`access_type=offline`, `prompt=consent` para garantizar que Google devuelva refresh_token incluso en re-consentimientos, y un `state` de un solo uso contra CSRF — ver `OAuthStateStore.cs`, en memoria del proceso).
   - `GET /integrations/google-drive/callback` — destino del redirect de Google; valida el `state`, intercambia el `code` por tokens, guarda el `refresh_token` cifrado. Si Google no devuelve refresh_token (podría pasar si el consentimiento previo nunca se revocó), lo señala explícitamente con el link a revocar accesos previos en `myaccount.google.com/permissions` y reintentar.
3. **Renovación automática** (`GetFreshAccessTokenAsync` en `GoogleDriveService`): cada operación (`status`, `test-read`) intercambia el refresh token guardado por un access token nuevo en el momento — el access token **nunca se persiste**, vive sólo en una variable local durante esa operación (mismo patrón que el installation token de TASK-0004).
4. **Detección de revocación**: si Google responde `invalid_grant` al intentar renovar (típico cuando Dani revocó el acceso desde su cuenta), el estado devuelto es `error` con el mensaje explícito "El refresh token ya no es válido... Volvé a autorizar vía GET /integrations/google-drive/authorize" — no una falla opaca.
5. **Endpoints de estado y lectura**: `GET /integrations/google-drive/status` (chequeo en vivo: renueva token + `GET /drive/v3/about` para confirmar la cuenta) y `GET /integrations/google-drive/test-read?folderId=` (lista `GET /drive/v3/files` filtrando por `'<folderId>' in parents`, usando el folder de la config si no se pasa parámetro).
6. **Scope pedido: sólo lectura** (`https://www.googleapis.com/auth/drive.readonly`), no el scope completo de Drive — es el mínimo necesario para lo que esta tarea efectivamente prueba (lectura). El scope de escritura se agrega recién cuando el Bloque E de DEC-C16 implemente operaciones de escritura reales, para no pedir un permiso más amplio del que se usa hoy (documentado también como restricción explícita de esta tarea).
7. **Pasos manuales que Dani tiene que hacer en Google Cloud Console** (documentados en `/README.md`, sección "Configurar OAuth de Google Drive para ColaborIA"): habilitar la Drive API, configurar la pantalla de consentimiento OAuth, crear un OAuth client ID de tipo Web application con el *redirect URI* de `/integrations/google-drive/callback`, y llamar a `POST /integrations/google-drive/config` con esos datos + el ID de la carpeta de Drive del proyecto.

### Verificación real (no sólo compilación)

- `dotnet build` / `dotnet test` (Release, mismo motivo que TASK-0004: Dani tenía un proceso de depuración corriendo): compilan y pasan sin errores — 16/16 tests al cierre, 9 nuevos de esta tarea:
  - `OAuthStateStoreTests` (5): un `state` se puede consumir una vez y no dos, un `state` desconocido o vacío no valida, y cada `state` generado es distinto.
  - `GoogleDriveAuthorizationUrlTests` (5, uno de los cuales solapa con el anterior en cobertura): la URL apunta al endpoint de autorización de Google, pide `access_type=offline` + `prompt=consent`, pide únicamente el scope de sólo-lectura, hace round-trip correcto de `client_id`/`redirect_uri`/`state`, y **nunca incluye el `client_secret`** en la URL (que es pública, va en la barra de direcciones del navegador de Dani).
  - `GoogleTokenResponseTests` (2, agregados después como regresión del bug descripto abajo): deserializan una respuesta de token real de Google (éxito y error) y confirman que los campos snake_case se mapean bien.
- **Contra el backend real corriendo** (Postgres real, puerto separado de la sesión de depuración de Dani):
  - `GET /integrations/google-drive/status`, `/authorize` y `/test-read` sin config guardada → los tres devuelven `not_configured` limpio, sin error opaco.
  - `POST /integrations/google-drive/config` con `clientId` vacío → `400`, no guarda nada.
  - `POST /integrations/google-drive/config` con datos bien formados pero falsos (client OAuth inexistente) → `200`, guarda cifrado.
  - `GET /integrations/google-drive/authorize` con esa config → devolvió una URL real y bien formada apuntando a `accounts.google.com`.
  - `GET /integrations/google-drive/callback` con un `state` inventado (no emitido por `/authorize`) → `error`, "state inválido o expirado" — protección CSRF funcionando.
  - `GET /integrations/google-drive/callback` con un `state` válido (recién emitido) pero un `code` falso → **llamó de verdad a `https://oauth2.googleapis.com/token`**, Google respondió `401`/`invalid_client` (esperable: el client OAuth no existe), y el endpoint devolvió `{"status":"error","detail":"Google respondió invalid_client: "}` sin excepción sin manejar. Confirma que el intercambio de código, el `HttpClient`, y el circuito completo hacia la API real de Google funcionan.
  - `GET /integrations/google-drive/status` después de eso → `not_configured` (correcto: como el callback falló, nunca se guardó un refresh token).
  - **Logs revisados durante toda la prueba**: sólo método+URL+status code por request (logging por defecto de `HttpClientFactory`, que además enmascara el query string por default) — ninguna traza del `client_secret`, el `code`, ni ningún token. **Corregí en el camino** un bug que yo mismo había introducido al escribir `GoogleDriveService`: originalmente pasaba el access token como query string (`?access_token=...`) en las llamadas a la API de Drive, lo cual sí habría podido quedar expuesto en logs más verbosos — lo cambié a header `Authorization: Bearer` antes de probar nada, mismo patrón que ya había usado en TASK-0004 para GitHub.
  - Datos de prueba (client OAuth falso) borrados de `credentials` al terminar esa parte.
- **Con el client OAuth real de Dani y consentimiento real vía navegador**:
  - `POST /integrations/google-drive/config` con Client ID/Secret reales (provistos por Dani directamente en el chat, nunca commiteados ni reescritos dos veces — se armaron vía PowerShell a un archivo temporal, se usaron, y se borraron) + el `driveFolderId` real → `200`.
  - Primer intento de consentimiento → Google bloqueó con `403 access_denied` porque la cuenta de Dani no estaba en la lista de test users de la pantalla de consentimiento OAuth (app en modo Testing). Dani la agregó desde Google Cloud Console (**Público/Audience → Test users**) — sin necesidad de publicar la app, lo cual habría disparado el proceso de verificación de Google para los scopes de Drive, innecesario para una herramienta de un solo usuario.
  - Consentimiento completado tres veces más, pero **cada vez** `GET /integrations/google-drive/callback` devolvía `{"status":"error","detail":"Google no devolvió un refresh_token..."}` pese a que Dani sí completaba el consentimiento correctamente (incluso probamos revocar el acceso previo desde `myaccount.google.com/permissions` y reintentar en una ventana de incógnito con sesión nueva — mismo resultado).
  - **Causa raíz encontrada: bug real en `GoogleApiDtos.cs`.** `GoogleTokenResponse` no tenía `[JsonPropertyName]` para los campos del endpoint de token de Google, que vienen en snake_case (`access_token`, `refresh_token`, `expires_in`, `error_description`). `PropertyNameCaseInsensitive` sólo ignora mayúsculas/minúsculas, **no** la diferencia entre snake_case y PascalCase — así que `AccessToken`/`RefreshToken` quedaban silenciosamente `null` en cada deserialización exitosa, aunque Google sí mandaba los valores. El síntoma ("no devolvió un refresh_token") era técnicamente cierto para el objeto deserializado, pero engañoso sobre la causa real: no era un problema de Google, era que nunca estábamos leyendo su respuesta correctamente.
  - **Corregido** agregando `[JsonPropertyName(...)]` explícito a los cuatro campos snake_case del DTO. Se agregó `GoogleTokenResponseTests.cs` (2 tests) como regresión: deserializa una respuesta de éxito y una de error con los nombres reales que manda Google, verificando que los campos se lean bien — para que este tipo de bug no pueda reaparecer en silencio.
  - **Con el fix aplicado, cuarto intento de consentimiento real**: `GET /integrations/google-drive/callback` → `{"status":"connected","detail":"Autorización completa. El refresh token quedó guardado cifrado."}`.
  - `GET /integrations/google-drive/status` → `{"status":"connected","detail":null,"accountEmail":"aleybru@gmail.com"}` — cuenta real confirmada.
  - `GET /integrations/google-drive/test-read` (sin `folderId`, autodetección desde la config) → `{"status":"connected","detail":null,"folderId":"1fXYHbsNkdJ5MqXEhDqJ1P3486HVZ6qDl","entries":[]}` — lista vacía confirmada por Dani como correcta (la carpeta está vacía de verdad), es decir la llamada real a `GET /drive/v3/files` funcionó y devolvió el resultado esperado, no un error disfrazado de lista vacía.
  - Verificado en Postgres: exactamente 2 filas para `provider='google_drive'` (`oauth_client_config` y `refresh_token`), ambas cifradas — ningún access token quedó persistido en ningún lado.
  - **Esta config y el refresh token reales quedan guardados en la base** (no se borraron) — listos para que TASK-0006 y el Bloque E de DEC-C16 los reutilicen.

### Decisiones técnicas menores tomadas (no contradicen DEC-C09/DEC-C16)

- **Refresh token y config del cliente OAuth como dos filas separadas** en `credentials` (a diferencia de GitHub App en TASK-0004, donde App ID + clave privada quedaron en una sola fila) — acá tienen ciclos de vida genuinamente distintos: la config del cliente OAuth la fija Dani una vez en Google Cloud Console y casi no cambia; el refresh token se reemplaza cada vez que Dani reconecta (revocación, reconsentimiento, etc.).
- **Protección CSRF vía `state` de un solo uso en memoria** (`OAuthStateStore`, `ConcurrentDictionary` con TTL de 10 min) en vez de persistirlo en DB o usar cookies de sesión — la app no tiene sesiones de usuario todavía (herramienta personal de un solo operador), así que un store en memoria del proceso alcanza; se pierde si la app se reinicia a mitad del flujo, aceptable para V0.
- **Scope `drive.readonly`, no `drive` completo ni `drive.file`**: `drive.file` sólo daría acceso a archivos creados o abiertos explícitamente por la app (exigiría un flujo de Picker que no existe todavía), y pedir el scope de escritura completo hoy violaría "scopes mínimos" de DEC-C16 ya que esta tarea no implementa ninguna escritura real. Se documenta acá porque implica una vuelta de consentimiento adicional cuando el Bloque E agregue escritura (el usuario va a tener que volver a autorizar con el scope ampliado).
- **`access_token` pasado por header `Authorization: Bearer`, nunca por query string** — ver el bug que corregí en la sección de verificación arriba.
- **Integración implementada a mano con `HttpClient`** (OAuth2 authorization code + refresh, Drive API v3 REST) en vez del cliente oficial `Google.Apis.Drive.v3` — consistente con el enfoque de TASK-0004 para GitHub, evita una dependencia grande (el paquete oficial trae toda la superficie generada de la API) para un flujo que en HTTP crudo es perfectamente manejable. **Contracara real de esta decisión**: implementarlo a mano fue justamente la causa del bug de deserialización descripto arriba — un cliente oficial probablemente lo habría evitado. Se mantiene la decisión (la dependencia grande sigue sin justificarse para un solo flujo), pero queda registrado como el costo concreto de la alternativa elegida, no como algo gratis.
- **Verificación de build/tests corrida en `Release`**, no `Debug`, mismo motivo documentado en TASK-0004 (proceso de depuración de Dani corriendo).

### Pendiente / no hecho en esta tarea

- Scope de escritura y operaciones de escritura reales — Bloque E de DEC-C16, fuera de alcance explícito de esta tarea.
- No se hizo `git commit` ni push — queda en el working tree para revisión antes de confirmar el commit.
