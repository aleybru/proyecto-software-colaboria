---
id: TASK-0003
status: done
repo_destino: aleybru/proyecto-software-colaboria-codigo
creado: 2026-07-19
aprobado_por: Dani
depende_de: []
decision_ref: DEC-C09, DEC-C10, DEC-C16
---

## Objetivo

Implementar la infraestructura de cifrado y la tabla `credentials` que el resto del Bloque A de DEC-C16 (GitHub App, Drive OAuth, API keys de LLM) va a necesitar. Sin esta tarea, ninguna de las siguientes (TASK-0004, 0005, 0006) puede ejecutarse.

## Qué tiene que hacer Code (concreto)

1. En `backend/ColaborIA.Api`, agregar un servicio de cifrado reversible (simétrico, ej. AES) para el payload de credenciales:
   - la clave de cifrado se lee de una variable de entorno (o user-secrets en desarrollo), **nunca hardcodeada ni versionada en el repo**;
   - el servicio expone como mínimo `Encrypt(string plaintext) -> byte[]` y `Decrypt(byte[] ciphertext) -> string`.

2. Agregar la entidad y `DbSet` de `credentials` con el schema ya definido en DEC-C09/DEC-C10 (con los ajustes de timestamps de DEC-C09):
   - `id`
   - `provider` (ej. `github_app`, `google_drive`, `anthropic`, `openai` — usar un tipo que permita agregar proveedores futuros sin migración, ej. string con validación, no enum rígido de EF Core)
   - `kind`
   - `scope_type` (`global` | `project`)
   - `project_id` (FK nullable a `projects` — null cuando `scope_type = global`)
   - payload cifrado (tipo binario, ej. `bytea` en Postgres)
   - `key_version` (para poder rotar la clave de cifrado en el futuro sin invalidar credenciales existentes)
   - `status` (ej. `active`, `invalid`, `revoked`)
   - `created_at`, `updated_at`
   - `CHECK` constraint: `project_id` obligatorio si `scope_type = 'project'`, prohibido si `scope_type = 'global'` (mismo patrón que Code ya usó en TASK-0001 para esta misma tabla — reutilizar el criterio).

3. Generar la migración de EF Core correspondiente.

4. Endpoints mínimos (sin UI todavía, solo API):
   - crear/reemplazar una credencial (recibe el valor en texto plano, lo cifra antes de guardar, nunca lo devuelve en la respuesta);
   - listar credenciales por proyecto o globales, **sin exponer el payload** — devolver `id`, `provider`, `kind`, `scope_type`, `status`, timestamps, y un indicador enmascarado (ej. últimos 4 caracteres si aplica, o simplemente "configurada"/"no configurada");
   - revocar una credencial (cambia `status`, no borra el registro).

5. Verificar explícitamente que ningún log de la aplicación (ni de EF Core en modo desarrollo) imprime el valor en texto plano de una credencial — mismo criterio que Code ya aplicó en TASK-0001.

## Criterio de éxito

- La migración se aplica correctamente contra Postgres.
- Un valor guardado vía el endpoint de creación puede recuperarse descifrado internamente (verificable con una prueba o script, no expuesto por API) y coincide exactamente con el valor original.
- El endpoint de listado nunca devuelve el payload en texto plano ni cifrado crudo.
- Los logs de la aplicación, revisados durante una prueba de creación/lectura, no contienen el valor en texto plano de ninguna credencial.
- El `CHECK` constraint rechaza una fila con `scope_type = 'project'` sin `project_id`, y una con `scope_type = 'global'` con `project_id` no nulo.

## Contexto / restricciones

- No implementar todavía la integración real con GitHub App, Drive ni las API keys de LLM — eso es TASK-0004, 0005 y 0006, que dependen de esta.
- No es necesario resolver todavía renovación automática de tokens (eso es específico de cada integración y se resuelve en su propia tarea).
- Respetar DEC-C08: decisiones técnicas menores (ej. algoritmo exacto de cifrado dentro de lo que ofrezca el ecosistema .NET estándar) quedan a criterio de Code y se documentan en `## Resultado`; no se cambia el schema de `credentials` ya fijado en DEC-C09/DEC-C10 sin señalarlo explícitamente.

## Resultado

Ejecutado el 2026-07-19 en el repo `aleybru/proyecto-software-colaboria-codigo`, sobre `backend/ColaborIA.Api` (TASK-0001/0002 ya mergeadas).

### Qué se hizo

1. **Servicio de cifrado** (`backend/ColaborIA.Api/Services/ICredentialEncryptionService.cs` + `AesGcmCredentialEncryptionService.cs`): AES-256-GCM (autenticado). `Encrypt(string) -> byte[]` devuelve `[nonce(12)][tag(16)][ciphertext]` concatenado; `Decrypt(byte[]) -> string` lo revierte. La clave se lee de `Credentials:EncryptionKey` (config/env/user-secrets) y **nunca** se hardcodea ni se commitea — `appsettings.json` sólo tiene el placeholder `CHANGE_ME_BASE64_AES256_KEY`. Si la clave falta o tiene el tamaño incorrecto, el servicio lanza una excepción explícita al primer uso (no al arrancar la app, así `/health` y el resto de la API siguen funcionando aunque la clave de cifrado no esté configurada todavía).
2. **Schema de `credentials`**: ya existía completo desde TASK-0001 (`id`, `provider` string libre, `kind`, `scope_type` enum mapeado a `global`/`project`, `project_id` FK nullable, `encrypted_payload` bytea, `key_version`, `status`, `created_at`/`updated_at`, y el `CHECK ck_credentials_scope_project_id` con exactamente el patrón que pedía esta tarea). Confirmado con `dotnet ef migrations has-pending-model-changes` → "No changes have been made to the model since the last migration." **No se generó una migración nueva** porque no hacía falta ningún cambio de schema — se documenta acá porque la tarea pedía explícitamente el paso 3 ("generar la migración correspondiente") y in TASK-0001 ya se había adelantado.
3. **Endpoints** (`backend/ColaborIA.Api/Endpoints/CredentialsEndpoints.cs`):
   - `POST /credentials` — crea o reemplaza (upsert por `provider`+`kind`+`scopeType`+`projectId`); cifra `value` antes de tocar la DB; nunca devuelve el valor ni el payload cifrado en la respuesta.
   - `GET /credentials/global` y `GET /credentials/projects/{projectId}` — listan sin exponer el payload; devuelven `id`, `provider`, `kind`, `scopeType`, `projectId`, `status`, timestamps y `valueIndicator` (`"configurada"`/`"no configurada"`, la opción más simple que ofrecía la tarea, para no derivar ningún indicador del valor real).
   - `POST /credentials/{id}/revoke` — cambia `status` a `revoked`, no borra la fila.
   - Validación de scope/projectId consistente con el `CHECK` de DB (devuelve 400 antes de llegar a la DB si `scopeType='project'` sin `projectId` o `scopeType='global'` con `projectId`).
4. **Tests**: proyecto nuevo `backend/ColaborIA.Api.Tests` (xUnit) con `CredentialEncryptionServiceTests` — round-trip exacto, nonces distintos por llamada (mismo texto plano → ciphertexts distintos), y que el constructor falla explícitamente si falta la clave o tiene el tamaño incorrecto.
5. `/README.md` raíz actualizado: cómo configurar `Credentials:EncryptionKey` vía user-secrets, documentación de los 3 endpoints, y cómo correr `dotnet test`.

### Verificación real (no sólo compilación)

- `dotnet build` (solución completa, API + Tests): compila sin errores.
- `dotnet test`: 4/4 tests pasan.
- **Extremo a extremo contra Postgres real** (backend corriendo, DB `ColaboriaDB`):
  - `POST /credentials` con un valor de prueba (`sk-ant-test-secret-value-XYZ123`, scope `global`, provider `anthropic`) → `201`, respuesta sin el valor ni el payload.
  - `GET /credentials/global` → lista la credencial con `valueIndicator: "configurada"`, sin ningún rastro del payload.
  - **Descifrado verificado fuera de la API** (script aparte en un directorio temporal, no en el repo): se leyó `encrypted_payload` directamente de Postgres vía `psql`, se descifró con la misma clave de user-secrets replicando la lógica de `AesGcmCredentialEncryptionService`, y el resultado coincidió exactamente (mismo string, misma longitud) con el valor original enviado al endpoint.
  - **Logs revisados durante la prueba**: EF Core no logueó el valor en texto plano ni el payload cifrado — los parámetros aparecen como `?` en el log (sensitive data logging no está habilitado), y el código de los endpoints tampoco loguea `request.Value` en ningún punto.
  - `POST /credentials/{id}/revoke` → cambia `status` a `revoked`, la fila sigue existiendo y aparece en el listado con `valueIndicator: "no configurada"`.
  - **`CHECK` constraint probado directamente en Postgres** (insert manual vía `psql`, no vía la API): una fila con `scope_type='project'` y `project_id NULL` fue rechazada; una fila con `scope_type='global'` y `project_id` no nulo (apuntando a un proyecto de prueba creado ad hoc) también fue rechazada. Ambos casos y los datos de prueba se limpiaron de la base al terminar.

### Decisiones técnicas menores tomadas (no contradicen DEC-C09/DEC-C10)

- **Algoritmo de cifrado**: AES-256-GCM (`System.Security.Cryptography.AesGcm`, estándar de .NET) en vez de AES-CBC+HMAC — GCM es autenticado nativamente (integridad + confidencialidad en un solo primitivo), evita tener que combinar dos primitivos a mano.
- **Formato del payload almacenado**: `[nonce(12)][tag(16)][ciphertext]` concatenado en un solo `bytea` — evita columnas adicionales sólo para nonce/tag, y es suficiente para descifrar sin ambigüedad.
- **`provider` sigue siendo string libre** (ya lo era desde TASK-0001, no un enum de EF Core) — cumple el pedido explícito de esta tarea de poder agregar proveedores futuros sin migración.
- **Rutas de listado separadas** (`/credentials/global` vs `/credentials/projects/{id}`) en vez de un único endpoint con query param — la tarea no fijaba la forma exacta ("listar por proyecto o globales"), y las rutas explícitas evitan ambigüedad sobre qué se está pidiendo.
- **`valueIndicator`**: se usó la opción más simple mencionada por la tarea ("configurada"/"no configurada") en vez de derivar algo tipo "últimos 4 caracteres", para no correr ningún riesgo de filtrar información derivada del secreto real.
- **Validación de la clave de cifrado es perezosa** (al primer uso, no al arrancar la app): así el resto de la API (incluido `/health`, ya crítico desde TASK-0001) no se rompe si todavía no se configuró `Credentials:EncryptionKey` en un entorno nuevo.

### Pendiente / no hecho en esta tarea (fuera de alcance, según la tarea y DEC-C16)

- Integración real con GitHub App, Drive OAuth y API keys de LLM — eso es TASK-0004, 0005 y 0006, que dependen de esta.
- Rotación de clave de cifrado (`key_version` > 1) — el campo existe y está pensado para esto, pero la lógica de rotación no se implementó (no la pedía esta tarea).
- No se hizo `git commit` ni push — queda en el working tree para revisión antes de confirmar el commit.
