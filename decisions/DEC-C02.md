# DEC-C02 — Dos repositorios: infraestructura transicional del desarrollo de ColaborIA (no topología de producto)

**Estado:** Decision vigente
**Reconstruido:** sí — ver nota original en DEC-C01.
**Revisión vigente:** 2026-08-30 — alcance acotado explícitamente tras la revisión de DEC-C09/C10/C11/C16 (2026-08-29), que fijó la topología real de recursos que ColaborIA crea para proyectos.

## Resumen

**Para el desarrollo actual de ColaborIA como proyecto en sí mismo** (no como producto), se usan dos repositorios de GitHub separados:
- `aleybru/proyecto-software-colaboria` — repositorio de gobernanza (decisiones, arquitectura, handoffs, code-tasks).
- `aleybru/proyecto-software-colaboria-codigo` — repositorio de código (monorepo backend/frontend).

**Esto es una excepción transicional/histórica**, propia de cómo se viene desarrollando ColaborIA mientras no existía todavía la aplicación misma. **No es, y nunca fue pensada como, la topología de recursos que ColaborIA crea para proyectos nuevos** — eso quedó fijado explícitamente por DEC-C09/DEC-C11/DEC-C16 (revisión 2026-08-29):

- proyecto `narrative` → carpeta raíz de Google Drive, sin repo de código;
- proyecto `software` → carpeta raíz de Google Drive + **un único** repo de código;
- **nunca** se crea `governance_repo` para un proyecto nuevo — la gobernanza/estado durable de los proyectos que gestiona ColaborIA vive en la base de datos de ColaborIA (DEC-C09/DEC-C10), no en un repo de GitHub aparte.

Esta Decision (DEC-C02) queda limitada, desde esta revisión, exclusivamente a describir la infraestructura del propio desarrollo de ColaborIA — no se usa como referencia de arquitectura de producto. Para eso, la fuente es DEC-C09/DEC-C11/DEC-C16 vigentes.

## Pendiente

- Definir cuándo se migra o retira esta infraestructura transicional (repo de gobernanza) una vez que ColaborIA pueda gestionar su propia gobernanza en su propia base de datos — mismo pendiente ya señalado en DEC-C16.
