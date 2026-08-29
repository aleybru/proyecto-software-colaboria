# DEC-C09 — Arquitectura V0: DB operacional y topología mínima de recursos

**Estado:** Decision vigente
**Fecha original:** 2026-07-11
**Revisión vigente:** 2026-08-29
**Autoridad:** Dani
**Contexto:** ColaborIA / arquitectura V0
**Relacionada con:** DEC-C02, DEC-C04, DEC-C08, DEC-C10, DEC-C11, DEC-C16

## Resumen

ColaborIA usa Postgres como estado operacional del proyecto y como base para la gobernanza que la aplicación vaya materializando. Los proyectos no requieren un repositorio de gobernanza propio.

La topología base de recursos queda fijada por el tipo de proyecto y no es una combinación libre elegida por el usuario:

- todo proyecto tiene una carpeta raíz de Google Drive;
- un proyecto `narrative` tiene solamente esa carpeta como recurso externo base;
- un proyecto `software` tiene esa carpeta de Drive y un único repositorio GitHub de código;
- no se crea `governance_repo` para proyectos nuevos.

El repositorio de gobernanza actual de ColaborIA se conserva como infraestructura transicional del propio desarrollo de ColaborIA y como historial existente; no forma parte del template de creación de proyectos.

## Decisión

### 1. Motor y tabla `projects`

Motor: **Postgres**.

`projects` mantiene, como mínimo:

- `id`
- `name`
- `type`
- `created_at`
- `last_validated_at`

Tipos iniciales:

- `narrative`
- `software`

### 2. Tabla `project_resources`

Se mantiene `project_resources` para registrar referencias, estado e historial de recursos externos sin convertirlos en columnas rígidas de `projects`.

Campos base:

- `id`
- `project_id`
- `kind`
- `provider`
- `external_ref`
- `purpose`
- `status`
- `created_at`
- `updated_at`

Topología base obligatoria:

- `narrative` → `drive_root / google_drive`
- `software` → `drive_root / google_drive` + `code_repo / github`

`governance_repo` queda **deprecado para proyectos nuevos**. Registros existentes pueden conservarse como legacy, pero el backend no debe generarlos ni exigirlos al crear un proyecto.

La existencia de `kind` y `provider` en el modelo interno no implica que la UI pueda combinarlos libremente. El backend debe validar las combinaciones permitidas.

### 3. Aprovisionamiento automático al crear proyecto

La creación normal de proyecto recibe solamente:

- nombre;
- tipo (`narrative | software`).

El backend deriva automáticamente los recursos requeridos y dispara su aprovisionamiento:

- siempre crea la carpeta raíz de Drive;
- si el proyecto es `software`, además crea un único repositorio GitHub de código.

El usuario no elige proveedor ni tipo de recurso durante este flujo.

El aprovisionamiento debe ser idempotente, reintentable y recuperable ante fallo parcial. El proyecto puede existir mientras un recurso está pendiente o en error, pero el estado individual debe reflejar la realidad.

El camino para **vincular recursos existentes** se conserva para migración, dogfooding o proyectos preexistentes; no es el flujo normal de alta de un proyecto nuevo.

### 4. Carpeta raíz de Drive

Todo proyecto tiene una carpeta raíz en Google Drive cuyo nombre corresponde al nombre del proyecto.

Esa carpeta es la raíz documental del proyecto y se registra en `project_resources.external_ref`.

Puede contener cualquier archivo o subcarpeta necesaria durante la vida del proyecto.

En software puede almacenar, entre otros, investigaciones, documentación, planillas, presentaciones, especificaciones y otros artefactos no pertenecientes al código fuente.

En narrativa puede almacenar biblias, guiones, episodios, capítulos, material de investigación y cualquier otro documento o artefacto del proyecto.

La organización interna de subcarpetas se definirá cuando el uso real la requiera; no forma parte de esta Decision.

### 5. Repositorio de código para software

Un proyecto `software` tiene **un único repositorio GitHub de código**.

Su nombre se deriva del nombre del proyecto respetando las restricciones de GitHub.

El repositorio puede ser monorepo y contener backend, frontend y cualquier otro componente de código del proyecto.

No se crea un segundo repo para gobernanza.

### 6. Gobernanza y estado

La razón histórica del repo de gobernanza por proyecto era sostener coordinación, historial y objetos formales antes de que ColaborIA tuviera canal y persistencia propios.

La arquitectura objetivo deja de exigir ese repo. ColaborIA debe sostener en su DB el estado gobernado necesario: conversaciones, tareas, resultados, decisiones, preguntas abiertas, trazabilidad y demás objetos que se materialicen en el producto.

Regla que se mantiene: **conversación no es estado**. Un mensaje no se convierte automáticamente en Decision, Task o Result; el cambio de estado requiere el workflow y autorización correspondientes.

Durante el desarrollo actual de ColaborIA, su repo de gobernanza existente sigue siendo la fuente normativa del propio proyecto hasta que Dani autorice una migración distinta. Esta excepción transicional no se replica en proyectos creados por la aplicación.

### 7. Tabla `credentials`

Se mantiene el modelo vigente de credenciales:

- `provider`
- `kind`
- `scope_type` (`global | project`)
- `project_id` nullable
- payload cifrado
- `key_version`
- `status`
- `created_at`
- `updated_at`

Condiciones obligatorias:

- cifrado reversible con clave fuera de DB/repo;
- secretos nunca en logs ni expuestos por API;
- scopes mínimos;
- GitHub App como dirección principal de autenticación GitHub;
- OAuth offline de Google Drive con refresh automático y reconexión integrada cuando el refresh token deja de ser válido;
- API keys LLM separadas conceptualmente.

### 8. Agentes fuera de DB

Papu y Chapu permanecen definidos en configuración versionada, por ejemplo:

- `config/agents/papu.yml`
- `config/agents/chapu.yml`

La identidad del agente no equivale a autorización efectiva; los permisos los aplica el backend.

### 9. Invariante de UI de creación

La UI de alta de proyecto muestra únicamente:

- **Nombre**;
- **Tipo** (`Software` o `Narrativo`);
- **Crear proyecto**.

No debe mostrar formularios genéricos de `Tipo de recurso`, `Proveedor`, `Propósito` ni combinaciones `kind + provider` para los recursos base.

Luego de crear, la pantalla de detalle muestra los recursos derivados por el backend y sus estados, con acciones de recuperación pertinentes (`Reintentar`, `Reconectar`, `Abrir`, o vinculación legacy cuando corresponda).

### 10. Stack técnico

- Backend: .NET / ASP.NET Core Web API + EF Core + Npgsql.
- Frontend: Angular / TypeScript.

El repositorio de código de ColaborIA sigue siendo monorepo con `/backend/` y `/frontend/`. Esto describe el proyecto ColaborIA actual y es consistente con la regla de un único repo de código para proyectos software.

## Consecuencias

- Crear un proyecto deja de ser un editor genérico de recursos.
- La topología base se deriva del tipo de proyecto.
- Drive es obligatorio para todos los proyectos y funciona como raíz documental.
- Software agrega exactamente un repo de código.
- Narrative no crea repos GitHub por defecto.
- `governance_repo` no se aprovisiona para proyectos nuevos.
- `project_resources` sigue siendo útil para estado, referencias, extensiones futuras y recursos legacy.

## No objetivos

- Definir estructura interna de carpetas Drive.
- Definir Project Templates genéricos.
- Definir nuevos proveedores alternativos a GitHub/Drive.
- Eliminar automáticamente repos de gobernanza legacy existentes.
