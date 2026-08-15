---
id: TASK-0007
status: done
repo_destino: aleybru/proyecto-software-colaboria-codigo
creado: 2026-08-15
aprobado_por: Dani
depende_de: []
decision_ref: DEC-C09, DEC-C16
---

## Objetivo

Implementar el modelo y los endpoints mínimos de `projects` y `project_resources` (DEC-C09), incluyendo el camino para **vincular recursos ya existentes**, no solo crear nuevos. Sin aprovisionamiento externo todavía (eso es TASK-0008) — esta tarea es solo el registro.

## Qué tiene que hacer Code (concreto)

1. Confirmar/completar la entidad `projects` conforme a DEC-C09: `id`, `name`, `type` (`narrative` | `software`), `created_at`, `last_validated_at`.

2. Confirmar/completar la entidad `project_resources` conforme a DEC-C09: `id`, `project_id`, `kind`, `provider`, `external_ref`, `purpose`, `status`, `created_at`, `updated_at`.

3. Endpoints CRUD mínimos:
   - crear proyecto (nombre + tipo);
   - listar proyectos;
   - obtener detalle de un proyecto con sus recursos;
   - agregar un recurso a un proyecto por dos caminos distintos: **vincular uno ya existente** (recibe `external_ref` real, ej. URL de un repo o carpeta ya creados) o **registrar la intención de crear uno nuevo** (queda en un estado que TASK-0008 va a completar con el aprovisionamiento real).

4. Validaciones mínimas: no permitir duplicar el mismo `external_ref` dentro del mismo proyecto; `type` restringido a los dos valores válidos.

## Criterio de éxito

- **Caso real, no solo de prueba:** registrar **ColaborIA misma** como proyecto (`type: software`) y vincular sus tres recursos ya existentes: repo de gobernanza (`aleybru/proyecto-software-colaboria`), repo de código (`aleybru/proyecto-software-colaboria-codigo`), y la carpeta de Drive `Proyecto COLABORIA`. Esto es dogfooding real, no un dato inventado — usar las referencias reales.
- El endpoint de detalle de ese proyecto devuelve los tres recursos vinculados correctamente.
- Un segundo proyecto de prueba (puede ser sintético) confirma que el flujo de creación (sin vincular) también funciona y queda en el estado esperado para que TASK-0008 lo complete.
- Intentar vincular el mismo `external_ref` dos veces al mismo proyecto es rechazado.

## Contexto / restricciones

- No implementar todavía la creación real de repos/carpetas (eso es TASK-0008) — el camino de "crear nuevo" en esta tarea solo registra la intención en la base de datos.
- No implementar todavía la UI (eso es TASK-0009) — esta tarea es solo backend/API.
- Respetar DEC-C08: detalles menores de validación o forma exacta de los endpoints quedan a criterio de Code.

## Resultado

Ejecutado el 2026-08-15 en el repo `aleybru/proyecto-software-colaboria-codigo`, sobre `backend/ColaborIA.Api` (rama basada en `main` con TASK-0001 a TASK-0006 ya mergeadas — TASK-0007 no depende de ninguna, pero el schema de `projects`/`project_resources` que necesitaba ya existía desde TASK-0001).

**✅ Dogfooding real completo**: ColaborIA quedó registrada como proyecto con sus tres recursos reales vinculados.

### Qué se hizo

1. **Entidades `projects`/`project_resources`**: ya estaban completas desde TASK-0001 y coinciden exactamente con lo que pedía esta tarea — confirmado con `dotnet ef migrations has-pending-model-changes` (sin cambios pendientes). No se generó ninguna migración nueva.
2. **Endpoints de proyectos** (`Endpoints/ProjectsEndpoints.cs`):
   - `POST /projects` — crea (`name` + `type`, validando que `type` sea exactamente `narrative` o `software`).
   - `GET /projects` — lista.
   - `GET /projects/{id}` — detalle con sus recursos (`Include(p => p.Resources)`).
3. **Dos caminos distintos para agregar un recurso**, como pedía explícitamente el punto 3 de la tarea:
   - `POST /projects/{id}/resources/link` — vincula un recurso **ya existente**: exige `externalRef` no vacío, rechaza (`409 Conflict`) si el mismo `externalRef` ya está vinculado a ese proyecto, y lo guarda con `status: "active"`.
   - `POST /projects/{id}/resources/provision` — registra la **intención** de crear un recurso nuevo: no recibe (ni acepta) `externalRef`, lo guarda con `status: "pending_provisioning"` y `external_ref: null`, dejado explícitamente para que TASK-0008 lo complete con el aprovisionamiento real y actualice `status`/`external_ref`.
4. `/README.md` actualizado con los 5 endpoints nuevos.

### Verificación real (no sólo compilación)

- `dotnet build` / `dotnet test` (Release, mismo motivo de las tareas anteriores): compilan y pasan sin errores — 32/32 tests, 2 nuevos (`ProjectsEndpointsTests`: parseo de `type`, valores válidos e inválidos).
- **Contra el backend real corriendo** (Postgres real):
  - **Caso real de dogfooding**: creado el proyecto `ColaborIA` (`type: software`) y vinculados sus tres recursos reales:
    - `governance_repo` → `https://github.com/aleybru/proyecto-software-colaboria`
    - `code_repo` → `https://github.com/aleybru/proyecto-software-colaboria-codigo`
    - `drive_root` → `https://drive.google.com/drive/u/0/folders/1YJR0LuKloo9VgGPpXHFUVJj692sJPSxL` (carpeta real "Proyecto COLABORIA", el link lo pasó Dani — el primer ID que yo había asumido, reutilizando el que se usó para probar Drive en TASK-0005, resultó ser una carpeta distinta; Dani corrigió antes de que quedara guardado mal).
    - `GET /projects/{id}` sobre ese proyecto devolvió los tres recursos correctamente.
  - **Proyecto sintético de prueba** (`Proyecto Prueba TASK-0007`, `type: narrative`): se le agregó un recurso vía `/resources/provision` (sin `externalRef`) → quedó con `status: "pending_provisioning"` y `externalRef: null`, exactamente el estado que necesita TASK-0008 para completarlo. Confirmado en el detalle del proyecto. Este proyecto sintético se borró de la base al terminar (cascada borró también su recurso); el proyecto `ColaborIA` real **se dejó guardado**.
  - **Duplicado de `externalRef`**: intentar vincular de nuevo `https://github.com/aleybru/proyecto-software-colaboria` al mismo proyecto → `409 Conflict`, rechazado como pedía el criterio de éxito.
  - Validaciones adicionales probadas: `type` inválido → `400`; `name` vacío → `400`; proyecto inexistente → `404`.
  - Logs revisados durante toda la prueba: nada anómalo (sólo SQL parametrizado, sin valores sensibles involucrados en esta tarea).

### Decisiones técnicas menores tomadas (no contradicen DEC-C09/DEC-C16)

- **Dos endpoints separados** (`/resources/link` y `/resources/provision`) en vez de un único endpoint con un campo que indique la intención — la tarea pedía explícitamente "dos caminos distintos"; separarlos en rutas distintas es más autoexplicativo que inferir la intención de la presencia/ausencia de `externalRef` en un único body.
- **`status: "pending_provisioning"`** como valor concreto para el camino de "crear nuevo" — no estaba fijado por DEC-C09 (que sólo define la columna `status` en general), así que se eligió un valor legible que TASK-0008 puede buscar explícitamente para saber qué recursos le faltan aprovisionar.
- **Chequeo de `externalRef` duplicado sólo aplica al camino `/link`** (el camino `/provision` nunca tiene `externalRef`, así que no puede colisionar) — consistente con que Postgres no cuenta múltiples `NULL` como duplicados en una comparación de igualdad tampoco.
- **`GET /projects/{id}` no filtra por proyecto inexistente antes del `Include`** — un solo query con `FirstOrDefaultAsync` que ya trae los recursos, en vez de dos queries separadas (uno para chequear existencia y otro para traer detalle) — más simple y una sola ida a la base.

### Pendiente / no hecho en esta tarea

- Aprovisionamiento real de recursos en estado `pending_provisioning` — TASK-0008.
- UI para proyectos/recursos — TASK-0009.
- No se hizo `git commit` ni push — queda en el working tree para revisión antes de confirmar el commit.
