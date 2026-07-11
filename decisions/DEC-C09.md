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

Ejemplos de recursos:

* `kind: governance_repo`, `provider: github`
* `kind: code_repo`, `provider: github`
* `kind: drive_root`, `provider: google_drive`

Esta decisión evita hardcodear recursos como columnas (`governance_repo`, `code_repo`, `drive_folder`) y permite agregar nuevos tipos de recursos o nuevos tipos de proyecto sin modificar el esquema principal de `projects`.

### 3. Tabla `credentials`

ColaborIA guardará las credenciales necesarias para operar integraciones.

Criterios obligatorios:

* cifrado reversible;
* clave de cifrado fuera de la DB;
* clave no versionada en repo;
* tokens nunca escritos en logs;
* scopes mínimos;
* separación conceptual por tipo de credencial.

Tipos iniciales previstos:

* GitHub;
* API keys LLM;
* futuras credenciales de integraciones.

**Alcance (por-proyecto vs. global): no resuelto en esta Decision — ver sección Pendiente.** No se asume ninguno de los dos por defecto.

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

## Pendiente

* Definir formato exacto de `config/agents/*.yml`.
* Definir schema concreto de `credentials`, **incluyendo si es por-proyecto o global** (abierto, sin default asumido).
* Definir estrategia técnica de cifrado.
* Ajustar convención de nombres de la sección 7 si el uso real lo justifica (queda como default, no como cierre definitivo).
* Definir primera code-task para implementar el esqueleto V0.
* **Definir stack técnico (lenguaje/framework de backend y frontend)** — no estaba cubierto por ninguna Decision anterior y es necesario antes de poder escribir una code-task ejecutable sin ambigüedad.
