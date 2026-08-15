---
id: TASK-0004
status: done
repo_destino: aleybru/proyecto-software-colaboria-codigo
creado: 2026-07-19
aprobado_por: Dani
depende_de: [TASK-0003]
decision_ref: DEC-C09, DEC-C16
---

## Objetivo

Integrar GitHub App como mecanismo de autenticación de la aplicación contra GitHub, conforme a la dirección ya fijada en DEC-C09 (no PAT manual) y precisada en DEC-C16 Bloque A (tokens efímeros generados bajo demanda, no guardados como credencial durable).

## Qué tiene que hacer Code (concreto)

1. Guardar, vía el servicio de credenciales de TASK-0003 (`provider: github_app`), la configuración durable de la App: App ID y clave privada (o referencia segura a ella, según lo que permita el flujo elegido) — **esto sí es durable y se cifra**.

2. Implementar la generación de **installation access tokens** bajo demanda:
   - el token efímero se genera en el momento en que se necesita (ej. al ejecutar una operación contra un repo), usando la App ID + clave privada guardadas;
   - **no se guarda el installation access token en la tabla `credentials` ni en ningún otro lugar persistente** — vive solo en memoria durante la operación que lo usa.

3. Endpoint de prueba de conexión: dado que la App está configurada, verificar que puede autenticarse y listar al menos las instalaciones/repos accesibles.

4. Estado visible de la integración: `no configurado` / `configurado, sin verificar` / `conectado` / `error` (con el detalle del error si lo hay, sin exponer la clave privada en el mensaje).

5. Documentar en `## Resultado` los pasos manuales que Dani tiene que haber hecho en GitHub antes de que esto funcione (registro de la App, instalación, permisos otorgados) — es un paso de configuración externa que esta tarea no automatiza, solo consume.

## Criterio de éxito

- Con la App ya registrada e instalada por Dani (paso manual previo, fuera del alcance de esta tarea), el endpoint de prueba de conexión confirma autenticación exitosa contra la API de GitHub.
- Se puede generar un installation access token válido bajo demanda y usarlo para una operación real de lectura contra un repo autorizado (ej. listar contenido de la raíz).
- El installation access token generado no aparece en ninguna tabla de la base de datos ni en logs.
- Si la App no está configurada o la clave es inválida, el estado de la integración lo refleja claramente, sin que la aplicación falle de forma opaca.

## Contexto / restricciones

- No reemplaza el token que Dani usa manualmente hoy para que Chapu opere el repo de gobernanza durante esta etapa conversacional — eso sigue siendo aparte, sin cambios, hasta que el flujo completo de MVP1.0 esté operativo.
- No implementar todavía operaciones de escritura reales contra repos (eso corresponde al Bloque E de DEC-C16, gateway de recursos tipado) — esta tarea es solo la autenticación y una prueba de lectura.
- Respetar DEC-C08: detalle de implementación del flujo de firma JWT para la App (librería específica, etc.) queda a criterio de Code.

## Resultado

Ejecutado el 2026-08-15 en el repo `aleybru/proyecto-software-colaboria-codigo`, sobre `backend/ColaborIA.Api` (TASK-0001/0002/0003 ya mergeadas).

**✅ Verificación end-to-end completa con una GitHub App real.** Inicialmente Dani no había respondido si ya tenía una App registrada, así que se implementó todo el código, se cubrió con tests, y se verificó cada pieza posible sin una App real (incluyendo una prueba contra la API real de GitHub con una clave RSA válida pero de una App inexistente). Más tarde, en la misma sesión, Dani confirmó que ya tenía la App registrada (App ID `4604548`) y pasó el App ID y la clave privada `.pem`. Con eso se completó la verificación real de punta a punta — ver sección "Verificación real" más abajo, que incluye el resultado con la App recién registrada (sin instalar todavía, `installations: []`) y después de que Dani la instaló (`installations: ["aleybru"]`, lectura real exitosa de dos repos).

### Qué se hizo

1. **Firma de JWT de la App** (`backend/ColaborIA.Api/Services/GitHubApp/GitHubAppJwtSigner.cs`): construye y firma (RS256, `RSA.SignData` + `RSASignaturePadding.Pkcs1`) el JWT que exige GitHub para autenticarse "como la App" (`iss`=App ID, `iat`/`exp` con el margen y el máximo de 10 minutos que pide la documentación de GitHub). Implementado a mano con `System.Security.Cryptography` (sin librería de JWT de terceros) — alcanza y evita una dependencia nueva sólo para esto.
2. **Config durable de la App** (`GitHubAppConfig.cs` + `POST /integrations/github-app/config` en `Endpoints/GitHubAppEndpoints.cs`): App ID + clave privada PEM se guardan cifrados vía el mismo mecanismo de TASK-0003 (`credentials`, `provider: github_app`, `kind: app_config`, `scope_type: global`). Valida que la clave sea un PEM RSA parseable antes de guardar (evita guardar una clave rota sin darse cuenta). Nunca prueba la conexión al guardar — separación explícita entre "guardar" y "verificar" (ver más abajo).
3. **Installation access tokens efímeros** (`Services/GitHubApp/GitHubAppService.cs`, método `TestReadAsync`): genera el token bajo demanda (`POST /app/installations/{id}/access_tokens` con el JWT), lo usa sólo dentro de esa misma llamada para leer la raíz de un repo, y lo descarta — vive únicamente en una variable local del método, nunca se persiste en `credentials` ni en ninguna otra tabla, nunca se loguea (confirmado, ver Verificación).
4. **Estado visible de la integración**:
   - `POST /integrations/github-app/config` devuelve `{"status": "configured_unverified"}` al guardar (no prueba nada todavía).
   - `GET /integrations/github-app/status` hace un chequeo **en vivo** contra la API real de GitHub (firma JWT, `GET /app`, `GET /app/installations`) y devuelve `not_configured` (sin config guardada) / `connected` (con el listado de logins de las instalaciones) / `error` (con `detail` legible, nunca con la clave privada ni ningún token).
   - `GET /integrations/github-app/test-read?owner=&repo=` hace la prueba de lectura real descripta en el punto 3; mismos tres estados.
5. **Pasos manuales que Dani tiene que hacer en GitHub** (documentados también en `/README.md`, sección "Configurar una GitHub App para ColaborIA"):
   1. GitHub → Settings → Developer settings → GitHub Apps → **New GitHub App**.
   2. Nombre/Homepage cualquiera (uso interno); sin webhook activo.
   3. Permisos mínimos para esta tarea: **Repository permissions → Contents: Read-only**.
   4. Generar y descargar la clave privada (`.pem`) — GitHub sólo la ofrece una vez.
   5. **Install App** en la cuenta/org, eligiendo los repos accesibles (ej. el propio repo de código/gobernanza de ColaborIA).
   6. Con el App ID (visible en la página de la App) y el contenido del `.pem`, llamar a `POST /integrations/github-app/config`.

### Verificación real (no sólo compilación)

- `dotnet build` / `dotnet test` (solución completa, config Release para no pisar un proceso de depuración que Dani tenía corriendo en Visual Studio durante la ejecución de esta tarea): compilan y pasan sin errores — 7/7 tests, incluyendo 4 nuevos específicos del firmador JWT (`GitHubAppJwtSignerTests`): estructura de 3 segmentos, claims `iss`/`iat`/`exp` correctos y duración ≤ 10 min, **firma verificada criptográficamente** contra la clave pública correspondiente (`RSA.VerifyData`), y que una clave PEM inválida hace fallar `CreateJwt` de forma explícita (no silenciosa).
- **Contra el backend real corriendo** (Postgres real, puerto separado para no chocar con el proceso de Dani):
  - `GET /integrations/github-app/status` sin config guardada → `{"status":"not_configured",...}`, sin error opaco.
  - `POST /integrations/github-app/config` con una clave PEM inválida (string cualquiera) → `400 Bad Request`, no guarda nada.
  - `POST /integrations/github-app/config` con un App ID inventado + una clave RSA-2048 real (generada localmente, no de una App real) → `200`, guarda cifrado.
  - `GET /integrations/github-app/status` con esa config → **llamó de verdad a `https://api.github.com/app`** y GitHub respondió `401` (esperable: el App ID no corresponde a ninguna App real) → el endpoint devolvió `{"status":"error","detail":"GitHub respondió 401 al verificar la identidad de la App."}`, sin excepción sin manejar ni 500. Esto confirma que la construcción del JWT, el `HttpClient` con sus headers (`Accept`, `User-Agent`, `X-GitHub-Api-Version`) y el circuito completo hacia la API real de GitHub funcionan correctamente — sólo falta una App real para llegar al camino "connected".
  - `GET /integrations/github-app/test-read` con la misma config falsa → mismo patrón, error claro (401), sin token generado.
  - **Logs revisados durante toda la prueba**: sólo aparecen método+URL+status code de cada request HTTP (logging por defecto de `HttpClientFactory`, sin headers ni body) — ninguna traza de la clave privada, el JWT ni ningún installation token.
  - Datos de prueba (App falsa) borrados de `credentials` al terminar esa parte.
- **Con la App real de Dani** (App ID `4604548`, clave `.pem` provista directamente por Dani en el chat, nunca commiteada ni pegada dos veces — se armó el request leyendo el `.pem` desde disco vía PowerShell):
  - `POST /integrations/github-app/config` → `200`, `{"status":"configured_unverified"}`.
  - `GET /integrations/github-app/status` (App recién registrada, todavía sin instalar) → `{"status":"connected","detail":null,"installations":[]}` — confirma que GitHub valida la identidad de la App (JWT correcto) aun antes de instalarla en ningún lado.
  - Dani instaló la App (Install App → cuenta `aleybru`). Reintento → `{"status":"connected","detail":null,"installations":["aleybru"]}`.
  - `GET /integrations/github-app/test-read` (sin `owner`/`repo`, autodetección) → `{"status":"connected","owner":"aleybru","repo":"proyecto-software-colaboria-codigo","entries":["file:.gitignore","file:README.md","dir:backend","dir:frontend"]}` — coincide exactamente con el contenido real de la raíz de ese repo.
  - `GET /integrations/github-app/test-read?owner=aleybru&repo=proyecto-software-colaboria` → `{"status":"connected",...,"entries":["file:CURRENT-STATE.md","file:INDICE.md","file:PROJECT.md","file:README.md","dir:code-tasks","dir:decisions","dir:handoffs","dir:open-questions","file:project.yml","dir:rules","dir:workspace"]}` — también coincide con el contenido real del repo de gobernanza.
  - Verificado en Postgres: sólo existe **una** fila en `credentials` con `provider='github_app'` (la config durable de la App) — ningún installation access token quedó persistido en ningún lado. Logs de esa corrida revisados igual que arriba: sin rastro del token.
  - **Esta config de la App real queda guardada en la base** (no se borró, a diferencia de las pruebas con datos falsos) — es la credencial durable que las próximas tareas (TASK-0005, TASK-0006, y el Bloque E de DEC-C16 más adelante) van a poder reutilizar.
  - El archivo temporal con la clave privada usado para armar el request se borró del disco al terminar.

### Decisiones técnicas menores tomadas (no contradicen DEC-C09/DEC-C10/DEC-C16)

- **App ID + clave privada guardados como un único registro JSON cifrado** (`{"appId":...,"privateKeyPem":...}`) en vez de dos filas separadas en `credentials` — son dos partes de un mismo secreto atómico (sin uno el otro no sirve), así que tiene más sentido rotarlos/reemplazarlos juntos.
- **`GET /status` hace el chequeo en vivo en cada llamada** en vez de persistir un campo de estado separado — evita bookkeeping adicional (última verificación, etc.) que la tarea no pedía explícitamente, y GitHub tolera perfectamente esta frecuencia de uso para V0.
- **Firma de JWT manual con `System.Security.Cryptography`** en vez de agregar `System.IdentityModel.Tokens.Jwt` u otra librería — el formato que exige GitHub es simple (RS256, 3 claims) y no vale la pena una dependencia nueva sólo para esto.
- **Instalación/repo objetivo de `test-read` autodetectados si no se pasan `owner`/`repo`** (primera instalación, primer repo accesible) — evita forzar a configurar algo adicional sólo para poder probar la lectura una vez que la App esté instalada.
- **Verificación de build/tests corrida en configuración `Release`, no `Debug`**: Dani tenía un proceso `ColaborIA.Api.exe` corriendo (aparentemente desde Visual Studio) que bloqueaba el binario de Debug; confirmé con él antes de tocarlo y me dijo que no lo terminara, así que usé `-c Release` (build a una carpeta de salida distinta) para no interferir con su sesión.

### Pendiente / no hecho en esta tarea

- Operaciones de escritura reales contra repos — Bloque E de DEC-C16, fuera de alcance explícito de esta tarea.
- No se hizo `git commit` ni push — queda en el working tree para revisión antes de confirmar el commit.
