# DEC-C09 — Arquitectura V0: DB operacional mínima, recursos vinculados y gobernanza en repo

**Estado:** Decision vigente
**Fecha:** 2026-07-11
**Autoridad:** Dani
**Contexto:** ColaborIA / arquitectura inicial V0
**Relacionada con:** DEC-C02, DEC-C04, DEC-C08
**Origen:** propuesta redactada por Papu, revisada por Chapu (3 ajustes de precisión incorporados: motor de DB, alcance de `credentials`, convención de nombres) y aprobada por Dani el 2026-07-11.

## Resumen

ColaborIA arrancará con una arquitectura V0 acotada: una aplicación web de uso personal de Dani que permite crear proyectos, registrar/crear sus recursos asociados y guardar credenciales necesarias para operar integraciones.

La base de datos propia existirá desde V0, pero no será la fuente de verdad de la gobernanza del proyecto. La gobernanza durable —decisiones, QC, handoffs, estado formal, preguntas abiertas y objetos equivalentes— seguirá viviendo en el repositorio de gobernanza versionado.

La DB se usará como registro operacional mínimo para proyectos, recursos externos asociados y credenciales cifradas.

## Decisión

Para V0 se adopta el siguiente modelo base:

### 0. Motor de base de datos

**Postgres.** Ya está instalado y corriendo en el entorno local de Dani — sin costo de setup adicional. Se descarta SQLite explícitamente (no aporta ventaja dado que Postgres ya está disponible, y SQLite habría exigido montar algo nuevo).

### 1. Tabla `projects`

Guarda la identidad mínima del proyecto dentro de ColaborIA.

Campos iniciales:

* `id`
* `name`
* `type`
* `created_at`
* `last_validated_at`

El campo `type` tendrá inicialmente valores acotados:

* `narrative`
* `software`

Estos valores no implican que el diseño quede cerrado a esos dos tipos para siempre.

### 2. Tabla `project_resources`

Los recursos externos del proyecto no se guardarán como columnas fijas en `projects`.

Se crea una tabla separada `project_resources` para registrar recursos vinculados al proyecto.

Campos iniciales:

* `id`
* `project_id`
* `kind`
* `provider`
* `external_ref`
* `purpose`
* `status`
* `created_at`
* `updated_at`

Ejemplos de recursos:

* `kind: governance_repo`, `provider: github`
* `kind: code_repo`, `provider: github`
* `kind: drive_root`, `provider: google_drive`

Esta decisión evita hardcodear recursos como columnas (`governance_repo`, `code_repo`, `drive_folder`) y permite agregar nuevos tipos de recursos o nuevos tipos de proyecto sin modificar el esquema principal de `projects`.

### 3. Tabla `credentials`

ColaborIA guardará las credenciales necesarias para operar integraciones.

**Actualizado el 2026-07-11** (resuelve el pendiente de alcance por-proyecto/global que había quedado abierto en la versión original de esta Decision). Definido por Dani y Papu; incorporado a la Decision por Chapu.

**Dirección de mecanismo de auth con GitHub:** no se basa en PATs manuales por repositorio como mecanismo principal de la aplicación en producción. Se modela vía **GitHub App / installation access tokens**: instalación con permisos acotados, tokens temporales por repositorio/permiso. Para V0 se puede implementar el schema y las costuras de configuración/cifrado primero, y fasear la interacción real con GitHub App si conviene — pero la dirección queda decidida desde el arranque, no como algo a resolver después.

*(Nota de alcance: esto es sobre cómo la aplicación ColaborIA se autentica contra GitHub en producción — no afecta el mecanismo actual de acceso de Chapu al repo de gobernanza durante esta etapa de trabajo conversacional, que sigue siendo token por sesión provisto por Dani.)*

Campos iniciales del modelo (`scope_type` resuelve el pendiente de alcance):

* `id`
* `provider` (ej. `github_app`, `llm_api`, futuros)
* `kind`
* `scope_type` (`global` | `project`)
* `project_id` (nullable — null cuando `scope_type = global`)
* payload cifrado
* `key_version`
* `status`
* `created_at`
* `updated_at`

Criterios obligatorios (sin cambios respecto a la versión original):

* cifrado reversible;
* clave de cifrado fuera de la DB;
* clave no versionada en repo;
* tokens/secretos nunca escritos en logs ni en texto plano;
* no exponer secretos por API ni por Swagger/documentación autogenerada;
* scopes mínimos;
* separación conceptual por tipo de credencial (GitHub App/installation tokens, API keys LLM, futuras credenciales — no se mezclan conceptualmente aunque compartan tabla).

Tipos iniciales previstos: GitHub (vía GitHub App), API keys LLM (Claude/GPT), futuras credenciales de integraciones.

### 4. Agentes fuera de DB

Las identidades de agentes no se guardarán en DB durante V0.

Papu y Chapu quedarán en configuración estática o archivos de configuración, por ejemplo:

* `config/agents/papu.yml`
* `config/agents/chapu.yml`

El formato exacto queda pendiente de definición.

La razón es separar:

* identidad/rol del agente, que es relativamente estático;
* estado dinámico del proyecto, que sí pertenece al runtime operacional.

### 5. Alcance funcional V0

V0 debe cubrir:

* crear proyecto;
* elegir tipo de proyecto;
* crear o registrar recursos asociados;
* guardar credenciales necesarias;
* ejecutar bootstrap básico del proyecto;
* dejar registrado el vínculo entre proyecto y recursos externos.

Para proyecto narrativo:

* repo de gobernanza;
* carpeta Drive.

Para proyecto software:

* repo de gobernanza;
* repo de código;
* carpeta Drive.

### 6. Context Engine y Governance Engine

Context Engine y Governance Engine se reconocen como boundaries conceptuales importantes para ColaborIA, pero no se implementarán completos en V0.

En V0:

* las decisiones siguen en repo de gobernanza;
* QC sigue en repo de gobernanza;
* handoffs siguen en repo de gobernanza;
* estado formal sigue en repo de gobernanza;
* la DB no duplica esos objetos como fuente de verdad.

La arquitectura debe preservar costuras para incorporar capacidades de Context/Governance más adelante, sin convertir el backend en una colección rígida de condicionales `narrative/software`.

### 7. Convención de nombres para recursos creados automáticamente

Convención adoptada como default de arranque (no cerrada de forma permanente, puede ajustarse si el uso real lo justifica):

* Repos: `<nombre>-gobernanza` (+ `<nombre>-codigo` si el proyecto es de tipo `software`).
* Carpeta Drive raíz: `"Proyecto <Nombre>"`.

Es el mismo patrón que ya usa ColaborIA como proyecto propio.

### 8. Stack técnico

Resuelto el 2026-07-11 (pendiente de la versión original de esta Decision).

* **Backend:** .NET / ASP.NET Core Web API, con Entity Framework Core y Npgsql como driver de Postgres.
* **Frontend:** Angular (TypeScript).

Motivo: Dani ya conoce ambos stacks — para una herramienta interna de un solo desarrollador, sin roadmap de escalado, la familiaridad del mantenedor pesa más que una elección "óptima en abstracto". Adicionalmente, ambos frameworks son opinados (convenciones claras de routing, DI, forms, migraciones) en vez de minimalistas — eso reduce la carga de decisiones de bajo nivel que un solo desarrollador tendría que sostener a lo largo del tiempo. EF Core además da migraciones de base de datos versionadas de forma nativa, alineado con la necesidad de evolucionar `projects` / `project_resources` / `credentials` sin fricción.

## No objetivos de V0

No forman parte de esta decisión para V0:

* multiusuario;
* producto comercial;
* marketplace de agentes;
* workflows genéricos completos;
* DB completa de tasks/runs/results/decisions/approvals/budgets/auditoría;
* implementación completa de Project Templates;
* implementación completa de Context Engine;
* implementación completa de Governance Engine.

## Rationale

ColaborIA es inicialmente una herramienta interna y personal de Dani, no un producto comercial ni un MVP clásico orientado a mercado.

Por eso se evita sobreconstruir desde el primer paso.

Al mismo tiempo, se evita subdiseñar: el modelo `project_resources` permite que el sistema crezca sin rehacer el esquema básico cuando aparezcan nuevos recursos, proveedores o tipos de proyecto.

La DB propia es necesaria para operar la aplicación y manejar credenciales/sesiones, pero el repo de gobernanza sigue siendo la fuente durable y versionada de verdad para las decisiones y coordinación formal del proyecto.

## Consecuencias

* La DB arranca chica y pragmática.
* El repo de gobernanza conserva autoridad sobre el estado formal del proyecto.
* Se reduce el riesgo de dos fuentes de verdad contradictorias.
* Se deja un camino claro para extender recursos y tipos de proyecto.
* El primer code-task puede enfocarse en esqueleto de app, conexión a Postgres, modelo base y configuración inicial.

### 9. Estructura de repositorio de código (monorepo)

Definido el 2026-07-11 por Dani y Chapu.

Un solo repo de código (ya fijado por DEC-C02 — no se agrega un repo nuevo), con backend y frontend como carpetas separadas dentro de él:

```
/backend/     → solución .NET (Web API, EF Core, migraciones)
/frontend/    → proyecto Angular
/README.md    → cómo levantar ambos localmente
```

Motivo: proyecto de un solo desarrollador, sin necesidad de aislar releases o permisos entre BE/FE; cambios que tocan ambos lados a la vez quedan en un solo commit. Cada carpeta mantiene sus propias dependencias (`.csproj` / `package.json`) — comparten repo, no build.

**Nota (2026-07-11):** los campos `created_at`/`updated_at` de `project_resources` (sección 2) y `credentials` (sección 3) se precisaron explícitamente a partir de una ambigüedad señalada por Papu en revisión de TASK-0001 (DEC-C09 decía "timestamps" en genérico). Ambas tablas llevan ambos campos, dado que sus campos `status` (y `key_version` en `credentials`) son mutables en el tiempo.

## Pendiente

* Definir formato exacto de `config/agents/*.yml`.
* ~~Definir schema concreto de `credentials`, incluyendo si es por-proyecto o global.~~ **Resuelto el 2026-07-11 — ver sección 3.**
* Definir estrategia técnica de cifrado específica (algoritmo, manejo de `key_version`).
* Ajustar convención de nombres de la sección 7 si el uso real lo justifica (queda como default, no como cierre definitivo).
* Definir primera code-task para implementar el esqueleto V0.
* ~~Definir stack técnico (lenguaje/framework de backend y frontend).~~ **Resuelto el 2026-07-11 — ver sección 8.**
* Definir fase de implementación real de GitHub App (sección 3) — V0 puede arrancar con schema + costuras solamente, la interacción real con GitHub App puede fasearse.
