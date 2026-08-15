---
id: TASK-0009
status: done
repo_destino: aleybru/proyecto-software-colaboria-codigo
creado: 2026-08-15
aprobado_por: Dani
depende_de: [TASK-0007, TASK-0008]
decision_ref: DEC-C16
---

## Objetivo

Primera pantalla de Angular funcional para gestionar proyectos, de forma que Dani pueda hacer el flujo completo de TASK-0007/0008 desde la aplicación, sin depender de Swagger ni de llamadas manuales a la API.

## Qué tiene que hacer Code (concreto)

1. Vista de listado de proyectos: nombre, tipo, cantidad de recursos, estado general.

2. Vista de creación de proyecto: nombre + selección de tipo (`software` | `narrative`).

3. Vista de detalle de un proyecto: recursos vinculados/creados, con su estado individual (conforme a TASK-0008), y la opción explícita de:
   - **vincular un recurso existente** (campo para pegar la referencia real);
   - **crear un recurso nuevo** (dispara el flujo de TASK-0008).

   Esta distinción debe ser clara en la UI — no un único botón ambiguo "agregar recurso".

4. Manejo visible de estados de error (ej. si TASK-0008 reporta un fallo parcial, se ve en esta pantalla, no queda oculto).

## Criterio de éxito

- Dani puede, sin usar Swagger ni curl, entrar a la aplicación y ver el proyecto ColaborIA (registrado en TASK-0007) con sus tres recursos vinculados correctamente mostrados.
- Dani puede crear un proyecto nuevo desde la UI, elegir tipo, y elegir explícitamente "vincular" o "crear" para al menos un recurso, viendo el resultado real (éxito o error) reflejado en pantalla.
- La distinción entre "vincular" y "crear" es clara para alguien que use la pantalla por primera vez, sin tener que preguntar qué hace cada botón.

## Contexto / restricciones

- No es una pantalla de administración amplia — cubre exactamente el flujo de TASK-0007/0008, nada más.
- No implementar todavía la sala de chat (DEC-C10) ni ninguna otra capacidad de DEC-C16 — esta tarea es específicamente la UI de proyectos.
- Respetar DEC-C08: detalles de estilo visual menores quedan a criterio de Code; el sistema de diseño formal (si DEC-C17 se implementa en el futuro) no es prerrequisito de esta tarea.

## Resultado

Ejecutado el 2026-08-15 en el repo `aleybru/proyecto-software-colaboria-codigo`, sobre `frontend/colaboria-frontend` + un ajuste aditivo menor en `backend/ColaborIA.Api` (rama basada en `main`, que ya tenía TASK-0007/0008 mergeadas).

**✅ Verificación end-to-end completa en un browser real**, incluyendo el caso de dogfooding explícito del criterio de éxito.

### Qué se hizo

1. **Ajuste aditivo en el backend** (`Endpoints/ProjectsEndpoints.cs`): `GET /projects` ahora incluye `resourceCount` y `overallStatus` (`sin_recursos`|`completo`|`pendiente`|`con_errores`, calculado a partir del `status` de los recursos) en `ProjectSummaryResponse`. No estaba en el alcance original de TASK-0007 y el endpoint no lo devolvía — hacía falta para que el listado (punto 1 de esta tarea) pudiera mostrar "cantidad de recursos, estado general" sin N+1 requests. Cambio puramente aditivo (dos campos nuevos), no rompe nada existente. Cubierto con 4 tests nuevos de `ComputeOverallStatus`.
2. **`core/projects.ts`**: servicio Angular con modelos TS que matchean los DTOs del backend, y métodos para los 5 endpoints relevantes (`list`, `get`, `create`, `linkResource`, `registerProvisioningIntent`, `provisionResource`).
3. **`features/projects/project-list`**: tabla con nombre, tipo, cantidad de recursos y badge de estado general; link a detalle y a "+ Nuevo proyecto".
4. **`features/projects/project-create`**: form nombre + tipo (select), navega al detalle del proyecto recién creado.
5. **`features/projects/project-detail`** (la vista más grande): lista de recursos con badge de estado individual; y **dos formularios visualmente separados** para agregar un recurso, cada uno con su propio título y explicación:
   - *"Vincular uno ya existente"* — pide `externalRef` real (obligatorio), sólo registra.
   - *"Crear uno nuevo"* — sin campo de `externalRef`; al enviar, registra la intención **y en el mismo paso dispara el aprovisionamiento real** (llama a `registerProvisioningIntent` y encadena `provisionResource` para el recurso recién creado), mostrando el resultado real (éxito o el detalle exacto del error) en pantalla.
   
   Recursos en `pending_provisioning` o `provisioning_error` muestran un botón ("Aprovisionar ahora" / "Reintentar") que llama a `provisionResource` de nuevo.
6. **Routing**: `/` redirige a `/projects` (la pantalla de proyectos pasa a ser la de arranque); `/projects`, `/projects/new`, `/projects/:id`, `/health` (la vista de TASK-0002 se mantiene, ahora accesible desde un nav). Nav mínimo agregado en `app.html`/`app.ts`.
7. Estilos globales mínimos (`styles.scss`) — badges de estado, botones, forms — sin sistema de diseño formal (DEC-C17 no implementado, no era prerrequisito).

### Verificación real (no sólo compilación)

- Backend: `dotnet build`/`dotnet test` — 40/40 tests.
- Frontend: `npx ng build` / `npx ng test` — 7/7 tests (incluye los specs nuevos de `Projects`, `ProjectList`, `ProjectCreate`, `ProjectDetail`).
- **En un browser real** (backend con el código de esta sesión en un puerto separado del proceso de Dani, frontend en `ng serve`, apuntado temporalmente al backend de prueba y revertido al terminar):
  - **Caso de dogfooding del criterio de éxito**: `/projects` mostró `ColaborIA | Software | 3 | Completo`; entrando al detalle se vieron los tres recursos reales de TASK-0007 (los dos repos de GitHub y la carpeta de Drive), cada uno con su URL real y estado "Activo" — exactamente lo que pedía el criterio, sin usar Swagger ni curl.
  - **Creado un proyecto nuevo desde la UI** ("Proyecto Prueba UI", software) → navegó automáticamente a su detalle, vacío.
  - **"Vincular"**: cargado un `externalRef` de prueba → el recurso apareció de inmediato como "Activo" con esa URL.
  - **"Crear" con Drive**: enviado el form sin `externalRef` → **se creó una carpeta real en Drive** (aprovisionamiento real de TASK-0008 disparado desde la UI), mostró "Aprovisionado correctamente." y el recurso pasó a "Activo" con la URL real de la carpeta creada.
  - **"Crear" con GitHub**: mismo flujo → la UI mostró el badge rojo **"Error al aprovisionar"** y, debajo del formulario, el detalle completo y real de la limitación de cuentas personales de GitHub (documentada en TASK-0008) — el fallo parcial quedó visible, no oculto, tal como pedía el criterio de éxito.
  - **"Reintentar"** sobre ese recurso en error → mismo resultado, consistente, sin efectos secundarios extraños.
  - Consola del navegador revisada: sin errores.
  - Logs del backend revisados durante toda la prueba: sin rastro de tokens ni secretos.
  - Datos de prueba (`Proyecto Prueba UI` y sus 3 recursos) borrados de la base al terminar; el proyecto `ColaborIA` real, intacto. La carpeta real creada en Drive durante esta prueba quedó en el Drive real de Dani (mismo criterio que TASK-0008: no me pareció mi lugar borrar algo de su Drive sin que lo pida).

### Decisiones técnicas menores tomadas (no contradicen DEC-C09/DEC-C16)

- **Extender `GET /projects` en vez de agregar un endpoint nuevo** para cantidad de recursos/estado general — es el mismo recurso, sólo con más campos; agregar un endpoint paralelo hubiera sido duplicar la fuente de la lista sin necesidad.
- **"Crear recurso nuevo" encadena registrar + aprovisionar en una sola acción de UI** (dos llamadas HTTP seguidas desde el componente, no un endpoint de backend nuevo que las combine) — la tarea pedía que la acción "dispare el flujo de TASK-0008", y encadenar los dos endpoints ya existentes desde el frontend evita agregar un endpoint compuesto sólo para esto.
- **`kind`/`provider` como `<select>` con las 3+2 combinaciones que TASK-0008 sabe aprovisionar** (no campos de texto libre) — reduce el riesgo de que Dani escriba algo que el backend no reconozca y caiga en la rama "unsupported" sin necesidad.
- **`/` redirige a `/projects`, no a `/health`** — la pantalla de proyectos es ahora "la" pantalla de la aplicación (objetivo explícito de esta tarea); `/health` se conserva accesible vía nav para no perder la vista de TASK-0002, pero deja de ser la de arranque.
- **Sin sistema de diseño**: estilos mínimos funcionales en `styles.scss`, coherente con que DEC-C17 (sistema de diseño formal) explícitamente no es prerrequisito de esta tarea.
- **Verificación con un backend de prueba propio en otro puerto**, apuntando el frontend ahí temporalmente (revertido antes de terminar) — Dani tenía su propio backend corriendo en el puerto de siempre (5269) con el código de una sesión anterior; usar un puerto separado con el build de hoy evitó pisarlo y permitió probar los campos nuevos (`resourceCount`/`overallStatus`) que su proceso todavía no tenía compilados.

### Pendiente / no hecho en esta tarea

- Sala de chat (DEC-C10) y cualquier otra capacidad de DEC-C16 — fuera de alcance explícito de esta tarea.
- Sistema de diseño formal (DEC-C17) — no implementado, no era prerrequisito.
- Limpieza manual pendiente: quedó una carpeta real "Proyecto Proyecto Prueba UI" en el Drive real de Dani (test debris, no se borró automáticamente — mismo criterio que TASK-0008).
- No se hizo `git commit` ni push — queda en el working tree para revisión antes de confirmar el commit.
