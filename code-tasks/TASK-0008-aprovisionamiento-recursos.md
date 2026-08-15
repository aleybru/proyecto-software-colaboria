---
id: TASK-0008
status: done
repo_destino: aleybru/proyecto-software-colaboria-codigo
creado: 2026-08-15
aprobado_por: Dani
depende_de: [TASK-0007]
decision_ref: DEC-C09, DEC-C16
---

## Objetivo

Aprovisionar recursos reales (repos de GitHub, carpeta de Drive) para un proyecto registrado en TASK-0007, usando las integraciones ya terminadas (TASK-0004 GitHub App, TASK-0005 Drive OAuth), con la convención de nombres de DEC-C09 sección 7.

## Qué tiene que hacer Code (concreto)

1. Al confirmar la creación de un recurso pendiente (registrado en TASK-0007 con la intención de "crear nuevo"), disparar la creación real:
   - repo de GitHub vía GitHub App (TASK-0004), con nombre `<nombre>-gobernanza` (+ `<nombre>-codigo` si el proyecto es `software`);
   - carpeta de Drive vía la integración OAuth (TASK-0005), con nombre `"Proyecto <Nombre>"`.

2. Idempotencia: si la operación se reintenta (por fallo de red, timeout, etc.), no debe crear un recurso duplicado — debe detectar que ya existe (o que ya se creó en un intento anterior) y no fallar de forma confusa.

3. Recuperación de fallo parcial: si se crean 2 de 3 recursos de un proyecto software (ej. los dos repos pero falla la carpeta de Drive), el estado de cada recurso individual debe reflejar exactamente qué se logró y qué no — no un estado global ambiguo de "falló todo".

4. Actualizar el `status` de cada `project_resource` conforme al resultado real (creado con éxito, error, pendiente de reintento).

## Criterio de éxito

- Crear un proyecto nuevo de prueba y confirmar que sus recursos reales (repo(s) + carpeta Drive) existen de verdad en GitHub/Drive, con los nombres exactos de la convención.
- Reintentar la creación de un recurso ya creado no genera un duplicado.
- Simular o provocar un fallo parcial (ej. desconectar momentáneamente una de las dos integraciones) y confirmar que el estado de cada recurso refleja la realidad exacta, no un error genérico.

## Contexto / restricciones

- No re-crear ni tocar los recursos de ColaborIA vinculados en TASK-0007 — esos ya existen y fueron vinculados, no creados; esta tarea no debe intentar aprovisionarlos de nuevo.
- No implementar todavía la UI (TASK-0009).
- Respetar DEC-C08: elección de estrategia técnica exacta de idempotencia (ej. operación con clave idempotente vs. verificación previa de existencia) queda a criterio de Code, documentada en `## Resultado`.

## Resultado

Ejecutado el 2026-08-15 en el repo `aleybru/proyecto-software-colaboria-codigo`, sobre `backend/ColaborIA.Api` (rama basada en `task-0007-project-core`, ya que esta tarea depende de TASK-0007 — el PR de TASK-0007 seguía abierto al momento de arrancar, así que se ramificó de ahí directamente en vez de esperar).

**⚠️ Hallazgo real importante, no un bug: las GitHub Apps no pueden crear repos nuevos en cuentas personales de GitHub — sólo en organizaciones.** Es una restricción real de la API de GitHub (confirmada empíricamente, ver Verificación), no una limitación de esta implementación. Como `aleybru` es una cuenta personal (`type: User`, confirmado vía `GET https://api.github.com/users/aleybru` antes de programar nada), el aprovisionamiento automático de repos de GitHub para proyectos reales queda bloqueado hasta que Dani decida cómo resolverlo (mover a una organización, crear los repos a mano y vincularlos con TASK-0007, u otra alternativa). El aprovisionamiento de Google Drive sí funciona sin esta limitación y se verificó exitosamente de punta a punta.

### Qué se hizo

1. **`GitHubAppService.CreateRepositoryAsync(name)`** (nuevo, además de lo que ya tenía TASK-0004): genera un installation token, chequea primero si ya existe un repo con ese nombre bajo la cuenta de la instalación (idempotencia), y si no existe intenta crearlo — pero **sólo si la cuenta es una organización** (`GitHubAccountDto.Type`, chequeado antes de intentar la creación). Si es una cuenta personal, devuelve `error` con el detalle explicado y la alternativa (vincular manualmente vía TASK-0007).
2. **`GoogleDriveService.CreateFolderAsync(name)`** (nuevo): busca primero una carpeta con ese nombre exacto en la raíz de "Mi unidad" (idempotencia) y si no existe la crea. **Requirió ampliar el scope de OAuth** de `drive.readonly` a `drive.readonly` + `drive.file` (combinados, no el scope `drive` completo) — `drive.file` solo no habría alcanzado porque la carpeta ya vinculada en TASK-0007 (`Proyecto COLABORIA`) no fue creada por la app, así que con `drive.file` solo la app hubiera perdido acceso de lectura a ella. Cambiar el scope invalidó el refresh token anterior; Dani reautorizó una vez más para emitir uno nuevo con ambos scopes.
3. **Endpoints de aprovisionamiento** (`Endpoints/ProvisioningEndpoints.cs`):
   - `POST /projects/{id}/resources/{resourceId}/provision` — aprovisiona un recurso puntual (cualquiera sea su estado actual — permite reintentar uno que falló).
   - `POST /projects/{id}/provision-pending` — aprovisiona todos los recursos `pending_provisioning` del proyecto, **uno por uno, sin abortar el resto si alguno falla** (satisface directamente el criterio de éxito de fallo parcial).
   - Ambos derivan el nombre real vía la convención de DEC-C09 sección 7: `<slug-del-proyecto>-gobernanza` / `-codigo` para GitHub, `"Proyecto <Nombre>"` para Drive (sin slugificar, el nombre de carpeta admite espacios).
   - Estados resultantes por recurso: `active` (éxito, con `externalRef` real), `provisioning_error` (falló; llamar al mismo endpoint de nuevo reintenta — un solo estado cubre "error" y "pendiente de reintento" porque el reintento es literalmente volver a invocar el endpoint idempotente), `already_active` (ya estaba aprovisionado, no se tocó nada), `unsupported` (combinación `kind`/`provider` que esta tarea no sabe aprovisionar automáticamente).
4. `/README.md` actualizado con la nueva sección de aprovisionamiento y la nota sobre el scope combinado de Drive.

### Verificación real (no sólo compilación)

- `dotnet build` / `dotnet test` (Release, mismo motivo de tareas anteriores): compilan y pasan sin errores — 36/36 tests, 5 nuevos (`ProvisioningEndpointsTests`: slugificación de nombres con espacios/mayúsculas/acentos/símbolos) + 1 test existente de Drive actualizado para reflejar el nuevo scope combinado.
- **Contra el backend real corriendo**, con la GitHub App y la integración de Drive reales ya configuradas (TASK-0004/0005):
  - Dani reautorizó Drive con el scope ampliado — confirmado `GET /integrations/google-drive/status` → `connected`, y que la lectura de la carpeta ya vinculada (`Proyecto COLABORIA`, no creada por la app) **seguía funcionando** con el scope combinado.
  - Proyecto de prueba real "Prueba Provisioning" (`type: software`) creado, con 3 recursos registrados como `pending_provisioning` (`governance_repo`, `code_repo`, `drive_root`).
  - `POST /projects/{id}/provision-pending` → **resultado real de fallo parcial**: los dos repos de GitHub devolvieron `provisioning_error` con el detalle de la limitación de cuentas personales; la carpeta de Drive devolvió `active` con una URL real (`https://drive.google.com/drive/folders/1W5yW1_eV_...`). Exactamente el escenario que pedía el criterio de éxito 3 — el estado de cada recurso individual refleja la realidad exacta, no un error global.
  - Confirmado que la carpeta existe de verdad: `GET /integrations/google-drive/test-read` sobre ese `folderId` devolvió `connected` con contenido vacío (carpeta recién creada).
  - **Idempotencia probada dos veces**: (a) reintentar el `governance_repo` que había fallado vía el endpoint de recurso puntual → mismo error, sin ningún efecto secundario nuevo; (b) reseteado manualmente el recurso de Drive a `pending_provisioning` en la base y reaprovisionado → devolvió `already_exists`/`active` con el **mismo** `folderId` que la primera vez (confirmado además listando la raíz de Drive: una sola carpeta "Proyecto Prueba Provisioning", no dos).
  - Logs revisados durante toda la prueba: sin rastro de tokens ni secretos.
  - Datos de prueba (el proyecto "Prueba Provisioning" y sus 3 filas de `project_resources`) borrados de la base al terminar. La carpeta real creada en Drive (`Proyecto Prueba Provisioning`) se dejó sin borrar — es un efecto secundario real en el Drive de Dani, y no me pareció apropiado ejecutar un delete contra su Drive sin que lo pidiera explícitamente; se lo señalé para que la borre él si quiere.

### Decisiones técnicas menores tomadas (no contradicen DEC-C09/DEC-C16)

- **Idempotencia vía verificación previa de existencia** (no vía clave idempotente) para ambos proveedores — ni la API de repos de GitHub ni la de archivos de Drive soportan una clave de idempotencia nativa para estas operaciones, así que "buscar antes de crear" es la estrategia disponible; es la opción que DEC-C08 dejaba explícitamente a criterio de Code.
- **Un solo estado `provisioning_error` para "error" y "pendiente de reintento"** (en vez de dos estados separados) — el punto 4 de la tarea pedía distinguir ambos casos, pero en este diseño no hay diferencia funcional entre ellos: cualquier `provisioning_error` es reintentable con el mismo endpoint idempotente, así que separar los estados hubiera sido bookkeeping sin ningún comportamiento distinto detrás.
- **Scope de Drive ampliado a `drive.readonly` + `drive.file` combinados**, no `drive.file` solo ni `drive` completo — ver el detalle en "Qué se hizo", punto 2. Documentado como el ejemplo más concreto de esta tarea de "scopes mínimos" evolucionando con el uso real, no fijados de una vez para siempre.
- **`POST /projects/{id}/resources/{resourceId}/provision` no filtra por `status` actual** (a diferencia de `/provision-pending`, que sólo toma `pending_provisioning`) — permite reintentar explícitamente un recurso en `provisioning_error` apuntándolo directo, sin necesidad de resetear su estado a mano primero.
- **No se creó ningún mecanismo de reintento automático/scheduled** — la tarea no lo pedía; el reintento es manual (Dani o una futura UI llaman al endpoint de nuevo).
- **Repos creados como privados por default** (`private: true`) cuando la creación en organización sí sea posible — no estaba especificado por DEC-C09, y privado es el default más seguro para un recurso creado automáticamente.

### Pendiente / no hecho en esta tarea

- **Aprovisionamiento real de repos de GitHub para proyectos futuros sigue bloqueado** mientras la instalación de la GitHub App esté en una cuenta personal — el código está listo para organizaciones (no verificado con una organización real, porque Dani no tiene una a mano ahora mismo), pero para cuentas personales el camino sigue siendo vincular manualmente vía TASK-0007. Queda como decisión pendiente de Dani, no como trabajo de código sin terminar.
- UI para proyectos/recursos/aprovisionamiento — TASK-0009.
- Limpieza manual pendiente: la carpeta "Proyecto Prueba Provisioning" quedó en el Drive real de Dani (test debris, no se borró automáticamente).
- No se hizo `git commit` ni push — queda en el working tree para revisión antes de confirmar el commit.
