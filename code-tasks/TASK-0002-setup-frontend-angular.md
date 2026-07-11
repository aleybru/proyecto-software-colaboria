---
id: TASK-0002
status: done
repo_destino: aleybru/proyecto-software-colaboria-codigo
creado: 2026-07-11
aprobado_por: Dani
depende_de: []
decision_ref: DEC-C09
---

## Objetivo

Crear el esqueleto del frontend (Angular) de ColaborIA dentro de `/frontend/`, sin funcionalidad de negocio todavía, capaz de levantar localmente y comunicarse con el backend.

## Qué tiene que hacer Code (concreto)

1. Dentro de `/frontend/` (carpeta ya creada por TASK-0001, o crearla si TASK-0001 todavía no se ejecutó), inicializar un proyecto Angular estándar (última versión estable, TypeScript).

2. Configurar un servicio HTTP base apuntando a la URL del backend (configurable por entorno — no hardcodear localhost a fuego, usar el mecanismo de environments de Angular).

3. Agregar una pantalla mínima de verificación (ej. una vista simple que llame al endpoint `GET /health` del backend de TASK-0001 y muestre si respondió OK o no) — esto sirve como prueba de que front y back se comunican, no como funcionalidad real del producto.

4. Dejar el routing base configurado (aunque sea con una sola ruta), ya que va a ser un panel con múltiples vistas a futuro (proyectos, recursos, credenciales) y conviene que la estructura de routing esté desde el arranque en vez de agregarse después.

5. Actualizar `/README.md` en la raíz del repo (el que crea TASK-0001) con los pasos para levantar el frontend localmente.

## Criterio de éxito

- El proyecto Angular compila y levanta localmente (`ng serve` o equivalente) sin errores.
- La vista de verificación llama exitosamente al endpoint de healthcheck del backend cuando ambos están corriendo, y muestra el resultado en pantalla.
- La URL del backend es configurable por entorno, no hardcodeada.

## Contexto / restricciones

- No implementar todavía ninguna pantalla de negocio real (crear proyecto, ver recursos, gestionar credenciales) — esta tarea es exclusivamente el esqueleto + prueba de conectividad con el backend.
- No es necesario que TASK-0001 esté ejecutada y corriendo para que Code arranque esta tarea (el esqueleto de Angular no depende de que el backend exista para compilar), pero sí hace falta el backend corriendo para verificar el criterio de éxito de la vista de healthcheck — si Code ejecuta esta tarea antes de que el backend esté levantado, puede dejar la integración escrita y señalar en `## Resultado` que la verificación end-to-end queda pendiente hasta que el backend esté disponible.
- Misma nota que en TASK-0001 respecto a DEC-C08: decisiones técnicas menores no cubiertas acá pueden resolverse por Code y documentarse en `## Resultado`; decisiones de arquitectura no.

## Resultado

Ejecutado el 2026-07-11 en el repo `aleybru/proyecto-software-colaboria-codigo`, sobre `/frontend` (creado por TASK-0001, ya mergeada). TASK-0001 estaba disponible y se pudo levantar, así que la verificación end-to-end de esta tarea sí se hizo completa (no quedó pendiente).

### Qué se hizo

1. Proyecto Angular inicializado en `/frontend` con Angular CLI, nombre `colaboria-frontend` (routing habilitado, estilos SCSS, sin SSR).
2. `src/environments/environment.ts` / `environment.development.ts` generados con el schematic `ng generate environments`, con `apiBaseUrl` (`http://localhost:5269` por default) — configurable por entorno, no hardcodeado en componentes ni servicios.
3. `provideHttpClient()` agregado a `app.config.ts`.
4. Servicio `src/app/core/health.ts` (`Health`): `check()` hace `GET {apiBaseUrl}/health` con `responseType: 'text'` (el healthcheck de ASP.NET Core devuelve texto plano, no JSON).
5. Componente `src/app/features/health-check/health-check.ts` + template: llama a `Health.check()` al iniciar, muestra "Backend OK (…)", "Backend no disponible: …" o "Consultando…" según el estado (signal), con botón "Reintentar".
6. Routing base en `app.routes.ts`: una sola ruta (`''`) apuntando al componente de healthcheck, y `app.html` reemplazado por `<router-outlet />` (se sacó el template de bienvenida por defecto de Angular CLI).
7. Specs ajustados para que seguían pasando tras los cambios: `app.spec.ts` (se sacó el test que buscaba el texto de bienvenida, ya no aplica; se agregó `provideRouter([])` para que `<router-outlet>` resuelva en el test), `health.spec.ts` y `health-check.spec.ts` (se agregó `provideHttpClient()` + `provideHttpClientTesting()` porque el servicio ahora inyecta `HttpClient`).
8. `/README.md` raíz actualizado con sección "Frontend: cómo levantarlo localmente" (requisitos, instalar dependencias, configurar URL del backend, `ng serve`, build/tests) y una nota sobre CORS (ver decisión técnica abajo).
9. `.gitignore` raíz actualizado con `frontend/.angular/` (cache de build de Angular) además de `node_modules/`/`dist/` ya agregados en TASK-0001.

### Verificación real (no sólo compilación)

- `npx ng build`: compila sin errores, bundle inicial ~210 kB (57 kB transferido).
- `npx ng test`: 3/3 tests pasan (Vitest, el test runner por default de Angular 21).
- `npx ng serve` levantado en `http://localhost:4200`: verificado en browser real.
  - Con el backend apagado: la vista muestra "Backend no disponible: …" (confirmado con el backend efectivamente caído, no simulado).
  - Con el backend de TASK-0001 corriendo (`dotnet run` contra Postgres local real): la vista mostró **"Backend OK (Healthy)"**, confirmando la comunicación real front↔back de punta a punta.

### Decisiones técnicas menores tomadas (no contradicen DEC-C09)

- Versión de Angular: **21.2.19** en vez de la última absoluta (22.0.6). El Node.js instalado es 20.19.6; Angular CLI 22 requiere Node ≥22.22/24.15/26 y no es compatible. Angular 21 sí soporta Node ^20.19.0, así que es la "última versión estable" real disponible dado el runtime del entorno. Documentado acá porque la tarea pedía explícitamente "última versión estable".
- **CORS agregado en el backend** (`backend/ColaborIA.Api/Program.cs` y `appsettings.json`, sección `Cors:AllowedOrigins`, default `http://localhost:4200`): sin esto, la llamada real del browser (frontend en :4200) al backend (:5269) queda bloqueada por política de mismo origen aunque el servidor responda 200 — se confirmó en la práctica (el log del backend mostraba la query de salud ejecutándose, pero el browser reportaba `ERR_FAILED`/error opaco). Agregar CORS no es una pantalla de negocio ni cambia el schema de datos; es la costura mínima de infraestructura necesaria para que el criterio de éxito de esta tarea ("la vista llama exitosamente al endpoint... cuando ambos están corriendo") sea verificable de verdad, no sólo aspiracional. No contradice ninguna sección de DEC-C09.
- Estructura de carpetas dentro de `src/app/`: `core/` para servicios transversales (como `Health`) y `features/` para vistas (como `health-check`) — convención liviana pensada para que las próximas pantallas de negocio (proyectos, recursos, credenciales) tengan un lugar obvio donde caer, sin ser una arquitectura rígida.
- Test runner: Angular 21 usa Vitest por default (`@angular/build:unit-test`) en vez de Karma/Jasmine-runner clásico — no se tocó, es el default de la versión de CLI usada.

### Pendiente / no hecho en esta tarea (fuera de alcance)

- Ninguna pantalla de negocio real (crear proyecto, ver recursos, gestionar credenciales) — como indica la tarea, eso queda para tareas posteriores.
- No se hizo `git commit` ni push — queda en el working tree para revisión antes de confirmar el commit.
