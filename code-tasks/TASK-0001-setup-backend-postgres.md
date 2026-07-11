---
id: TASK-0001
status: pending
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
   - **`project_resources`**: `id`, `project_id` (FK a `projects`), `kind`, `provider`, `external_ref`, `purpose`, `status`, `created_at`.
   - **`credentials`**: `id`, `provider`, `kind`, `scope_type` (`global` | `project`), `project_id` (FK a `projects`, nullable — null cuando `scope_type = global`), payload cifrado (campo binario o string, según la librería de cifrado elegida), `key_version`, `status`, `created_at`.

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
