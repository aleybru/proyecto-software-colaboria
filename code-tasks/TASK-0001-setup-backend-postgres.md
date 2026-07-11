---
id: TASK-0001
status: done
repo_destino: aleybru/proyecto-software-colaboria-codigo
creado: 2026-07-11
aprobado_por: Dani
depende_de: []
decision_ref: DEC-C09
---

## Objetivo

Crear el esqueleto del monorepo de código de ColaborIA y el backend inicial (.NET / ASP.NET Core Web API) con conexión a Postgres y la primera migración de Entity Framework Core, cubriendo las tres tablas base definidas en DEC-C09.

## Qué tiene que hacer Code (concreto)

1. En el repo `aleybru/proyecto-software-colaboria-codigo` (repo vacío — inicializarlo desde cero), crear la estructura de monorepo definida en DEC-C09 sección 9:
   ```
   /backend/
   /frontend/
   /README.md
   ```
   `/frontend/` puede quedar vacío en esta tarea (se completa en TASK-0002).

2. Dentro de `/backend/`, crear una solución .NET con un proyecto ASP.NET Core Web API (nombre sugerido: `ColaborIA.Api`, ajustable si Code tiene una convención mejor — no es una restricción dura).

3. Agregar Entity Framework Core con el paquete Npgsql (driver de Postgres) como proveedor de base de datos.

4. Definir los `DbSet`/entidades correspondientes a las tres tablas de DEC-C09:

   - **`projects`**: `id`, `name`, `type` (`narrative` | `software`), `created_at`, `last_validated_at`.
   - **`project_resources`**: `id`, `project_id` (FK a `projects`), `kind`, `provider`, `external_ref`, `purpose`, `status`, `created_at`, `updated_at`.
   - **`credentials`**: `id`, `provider`, `kind`, `scope_type` (`global` | `project`), `project_id` (FK a `projects`, nullable — null cuando `scope_type = global`), payload cifrado (campo binario o string, según la librería de cifrado elegida), `key_version`, `status`, `created_at`, `updated_at`.

   Para `type` y `scope_type`, usar el mecanismo de EF Core que corresponda para restringir valores (enum de .NET mapeado, o check constraint — a criterio de Code).

5. Generar la primera migración de EF Core con estas tres tablas.

6. Configurar la cadena de conexión a Postgres vía configuración externa (`appsettings.json` + variables de entorno / user-secrets para el valor real) — **no hardcodear credenciales de conexión en el código ni commitear una cadena de conexión con password real**.

7. Dejar un endpoint mínimo de healthcheck (ej. `GET /health`) que confirme que la API levanta y puede conectar a la base.

8. Agregar `/README.md` en la raíz del repo con los pasos para levantar el backend localmente (requisitos, cómo aplicar la migración, cómo correr la API).

## Criterio de éxito

- El proyecto compila sin errores (`dotnet build`).
- La migración de EF Core se genera correctamente y puede aplicarse contra una instancia local de Postgres (`dotnet ef database update` o equivalente), creando las tres tablas con los campos especificados.
- La API levanta localmente y el endpoint de healthcheck responde OK cuando la conexión a la base está configurada correctamente.
- No hay ninguna credencial ni cadena de conexión con password en texto plano commiteada al repo.

## Contexto / restricciones

- No implementar todavía lógica de negocio, autenticación, ni endpoints CRUD reales sobre estas tablas — esta tarea es exclusivamente el esqueleto + conexión + schema base. Eso queda para tareas posteriores.
- No implementar todavía la integración real con GitHub App (DEC-C09 sección 3) — el campo de payload cifrado en `credentials` puede quedar como columna simple por ahora, sin la lógica de cifrado/descifrado implementada (eso es un pendiente explícito no bloqueante de DEC-C09).
- Respetar DEC-C08: esta tarea es ejecución de algo ya definido y aprobado — si durante la implementación aparece una decisión de diseño no cubierta acá (por ejemplo, nombre exacto de un campo, tipo de dato específico), Code puede tomar la decisión técnica menor razonable y dejarla documentada en la sección `## Resultado`, pero no debe tomar decisiones de arquitectura que contradigan DEC-C09.

## Resultado

Ejecutado el 2026-07-11 en el repo `aleybru/proyecto-software-colaboria-codigo`, working copy local (sin push todavía).

### Qué se hizo

1. Estructura de monorepo creada en la raíz: `/backend/`, `/frontend/` (vacío, para TASK-0002), `/README.md` actualizado, `.gitignore` agregado (bin/obj/.NET, node_modules/dist Angular, editores).
2. Solución .NET en `/backend/`: `ColaborIA.slnx` con el proyecto `ColaborIA.Api` (ASP.NET Core Web API, `net10.0`, plantilla mínima sin controllers).
3. Paquetes agregados: `Npgsql.EntityFrameworkCore.PostgreSQL` (10.0.3), `Microsoft.EntityFrameworkCore.Design` (10.0.9), `Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore` (10.0.9).
4. Entidades en `backend/ColaborIA.Api/Models/`: `Project`, `ProjectResource`, `Credential`, más los enums `ProjectType` (`Narrative`/`Software`) y `CredentialScopeType` (`Global`/`Project`). Campos según DEC-C09 secciones 1-3, incluyendo la precisión de `created_at`/`updated_at` en `project_resources` y `credentials` de la nota del 2026-07-11.
5. `DbContext` en `backend/ColaborIA.Api/Data/ColaborIaDbContext.cs`: mapea las tres tablas (`projects`, `project_resources`, `credentials`) con nombres de columna snake_case. Los enums se restringen vía `ValueConverter` a los literales exactos de DEC-C09 en minúscula (`narrative`/`software`, `global`/`project`), no el nombre por defecto de .NET — así el valor en la columna DB queda legible y alineado 1:1 con la decisión. Además se agregó un `CHECK` constraint (`ck_credentials_scope_project_id`) que obliga `project_id IS NULL` cuando `scope_type = 'global'` y `project_id IS NOT NULL` cuando `scope_type = 'project'`, cubriendo la regla de negocio de la sección 3 que un enum por sí solo no expresa.
6. `payload cifrado` de `credentials` implementado como columna simple `bytea` (`encrypted_payload`), sin lógica de cifrado — conforme al pendiente explícito no bloqueante de DEC-C09.
7. Migración inicial generada: `backend/ColaborIA.Api/Data/Migrations/20260711215309_InitialCreate.cs`.
8. Connection string: `appsettings.json` sólo tiene un placeholder (`Username=CHANGE_ME;Password=CHANGE_ME`). El valor real para desarrollo local se guardó vía `dotnet user-secrets` (fuera del repo, en el secret store de usuario) — no hay ninguna credencial real commiteada.
9. Endpoint `GET /health` agregado (`AddHealthChecks().AddDbContextCheck<ColaborIaDbContext>()` + `MapHealthChecks("/health")`), reemplazando el endpoint `weatherforecast` de la plantilla (removido junto con el record `WeatherForecast`, sin uso en el dominio).
10. `README.md` raíz actualizado con estructura del monorepo y pasos para levantar el backend local (requisitos, configurar connection string vía user-secrets, aplicar migración, correr la API, verificar `/health`).

### Verificación real (no sólo compilación)

- `dotnet build`: compila sin errores (sólo advertencia `NU1903` preexistente de la plantilla de .NET 10 sobre `Microsoft.OpenApi` transitivo — no hay versión parcheada disponible todavía en NuGet a la fecha).
- `dotnet ef database update` corrido contra una instancia Postgres 18 local real (`ColaboriaDB`, rol `ColaboriaAdmin` con `CREATEDB`): creó la base y las tres tablas. Verificado con `psql \dt` y `\d credentials` que el esquema, el FK y el `CHECK` constraint quedaron como se esperaba.
- `dotnet run` levantado localmente y `GET http://localhost:5269/health` respondió `200 Healthy`, confirmando que la API levanta y valida la conexión real a la base.
- Revisado `git status`/`git add -n` antes de terminar: ningún archivo con credenciales reales quedaría staged: `appsettings.json` sólo tiene el placeholder, `bin/`/`obj/` quedan excluidos por `.gitignore`, y el user-secrets con la password real vive fuera del repo.

### Decisiones técnicas menores tomadas (no contradicen DEC-C09)

- Nombre del proyecto: `ColaborIA.Api` (el sugerido en la tarea).
- Mecanismo elegido para restringir `type`/`scope_type`: enum de .NET + `ValueConverter` a string lowercase (en vez de enum de Postgres nativo), por simplicidad de migraciones futuras si se agregan valores.
- Se agregó el `CHECK` constraint de consistencia `scope_type`/`project_id` en `credentials` porque esa regla (sección 3 de DEC-C09) no la cubre un enum por sí solo.
- Herramienta `dotnet-ef` global actualizada de 9.0.10 a 10.0.9 para que coincida con el runtime del proyecto (evita warnings de versión, no afecta el repo).

### Pendiente / no hecho en esta tarea (fuera de alcance, según DEC-C09)

- Cifrado real del payload de `credentials` (algoritmo, manejo de `key_version`) — pendiente explícito de DEC-C09.
- Integración real con GitHub App — fuera de alcance de V0/esta tarea.
- No se hizo `git commit` ni push — quedan los archivos en el working tree para que Dani revise antes de confirmar el commit.
