# DEC-C16 — MVP1.0: alcance, criterio de cierre y orden de implementación

**Estado:** Decision vigente
**Fecha original:** 2026-07-18/19
**Revisión vigente:** 2026-08-29
**Autoridad:** Dani
**Relacionada con:** DEC-C09 a DEC-C15

## Resumen

MVP1.0 mantiene el objetivo de convertir ColaborIA en la interfaz operativa real del proyecto, pero adopta una topología de recursos mucho más simple y determinística.

Regla central de esta revisión:

- todo proyecto crea automáticamente una carpeta raíz en Google Drive;
- un proyecto `narrative` no crea repos GitHub por defecto;
- un proyecto `software` crea además un único repo GitHub de código;
- no se crea repo de gobernanza por proyecto;
- la DB de ColaborIA sostiene conversación, estado gobernado, decisiones, tareas, resultados, preguntas abiertas, handoffs e historial;
- la UI de alta de proyecto no expone `kind`, `provider` ni formularios genéricos de recursos base.

## Bloque A — Credenciales e integraciones

Objetivo: operación persistente sin renovación manual recurrente en condiciones normales.

- **GitHub App**: configuración durable de App/instalación y tokens efímeros bajo demanda. Debe permitir operar el único repo de código de proyectos software.
- **Google Drive**: OAuth offline, refresh automático y reconexión integrada cuando el refresh token deje de ser válido.
- **OpenAI/Anthropic**: API keys cifradas, validables, rotables y revocables.
- Secretos cifrados, nunca en logs/API.
- Errores de credencial deben ser tipados y recuperables desde la UI.

Para Drive, si el refresh token es inválido/revocado, la UI debe ofrecer reconexión OAuth y, tras completarla, volver al proyecto y reintentar idempotentemente la operación pendiente. El usuario no debe recibir como salida final instrucciones de endpoints internos.

## Bloque B — Proyectos y aprovisionamiento

**Estado del bloque: CERRADO / APROBADO — 2026-08-30.** Aprobado explícitamente por Dani. Evidencia trazable en `code-tasks/TASK-0011.md` a `TASK-0015.md` (reconstrucción retrospectiva de trabajo ya ejecutado y verificado contra Postgres/GitHub/Drive reales entre el 29 y 30 de agosto de 2026), con SHAs de commit reales citados en cada una.

### Alta normal

Crear proyecto requiere únicamente:

- `name`;
- `type: narrative | software`.

El backend deriva automáticamente la topología:

- `narrative` → carpeta raíz Google Drive;
- `software` → carpeta raíz Google Drive + un único repo GitHub de código.

La carpeta Drive se crea siempre y funciona como raíz documental del proyecto.

El repo de software puede ser monorepo y contener backend/frontend y demás componentes de código.

No se crea `governance_repo` para proyectos nuevos.

### Regla de UI obligatoria

La pantalla de alta **no** debe presentar:

- selector de proveedor;
- selector de tipo de recurso;
- campo propósito para decidir recursos base;
- formularios paralelos de “vincular” vs. “crear” como flujo normal de alta;
- combinaciones libres `kind + provider`.

La UI normal es conceptualmente:

```text
Nombre
Tipo: Software | Narrativo
[Crear proyecto]
```

Luego muestra el estado de los recursos que el backend derivó y aprovisionó.

### Recursos existentes

Se conserva una capacidad separada para vincular/importar recursos existentes en casos de migración, dogfooding o proyectos preexistentes. No debe contaminar el flujo normal de creación.

### Robustez

El aprovisionamiento debe ser:

- idempotente;
- reintentable;
- verificable;
- recuperable ante fallo parcial;
- visible por recurso.

Un error externo no debe corromper el proyecto. La UI debe mostrar acciones contextuales (`Reintentar`, `Reconectar`, `Abrir`) y no detalles de implementación como única salida.

## Bloque C — Identidad y autorización de agentes

Papu y Chapu permanecen definidos en configuración versionada (`config/agents/*.yml`) con identidad, rol, habilidades, proveedor/modelo e instrucciones.

La autorización efectiva vive en backend.

Matriz objetivo:

| Recurso | Papu | Chapu |
|---|---|---|
| Estado/gobernanza en DB de ColaborIA | lectura + escritura autorizada | lectura + escritura autorizada |
| Drive del proyecto | solo lectura | lectura + escritura autorizada |
| Repo de código (software) | solo lectura | lectura + escritura autorizada |

Dani autoriza efectos secundarios conforme a DEC-C04.

El repo de gobernanza actual de ColaborIA es una excepción transicional del desarrollo y no un recurso aprovisionado por el producto.

## Bloque D — Sala persistente y contexto gobernado

- `sessions.project_id` obligatorio;
- `messages` como fuente de verdad del contenido;
- `agent_turns` como trazabilidad de ejecución;
- floor lock y routing controlados por backend;
- OpenAI y Anthropic reales;
- adapters determinísticos para tests;
- `input_context_ref` reconstructible;
- contexto mínimo: protocolo aplicable, configuración/manifest del proyecto, agente, Decisions/estado relevantes en DB, historial necesario y fuentes recuperadas;
- aislamiento estricto entre proyectos.

## Bloque E — Gateway tipado de recursos y estado

El backend expone operaciones tipadas y auditables, no acceso libre del agente.

Como mínimo:

- leer/actualizar estado gobernado autorizado en ColaborIA;
- localizar/leer documentos del Drive del proyecto;
- crear/actualizar documentos autorizados en Drive;
- leer el repo de código de software;
- escribir código cuando agente/runtime y autorización lo permitan;
- despachar tareas aprobadas al runtime de ejecución;
- verificar efectos secundarios antes de reportar éxito;
- rechazar operaciones fuera de permisos.

Cada operación auditable registra proyecto, solicitante/agente, recurso, autorización, resultado y error.

## Durabilidad

La persistencia se divide por naturaleza del objeto:

- conversación y ejecución → DB;
- Decisions, Tasks, Results, Open Questions, handoffs, aprobaciones, roadmap y demás estado gobernado → DB;
- documentos y artefactos de archivo → Drive;
- código → único repo de código del proyecto software.

No se requiere materialización en un repo de gobernanza por proyecto.

`handoffs/quick/log.md`, `code-tasks/*.md` y el repo de gobernanza actual continúan como infraestructura transicional para desarrollar ColaborIA hasta que Dani autorice su reemplazo/migración. No forman parte del template de proyectos nuevos.

## Criterio de cierre MVP1.0

MVP1.0 se considera cerrado cuando Dani puede:

1. configurar GitHub App, Drive, OpenAI y Anthropic;
2. crear desde la UI un proyecto `narrative` usando solamente nombre + tipo y obtener automáticamente su carpeta Drive;
3. crear desde la UI un proyecto `software` usando solamente nombre + tipo y obtener automáticamente carpeta Drive + un único repo de código;
4. comprobar que la UI nunca permite fabricar combinaciones inválidas `kind/provider` para recursos base;
5. recuperarse visiblemente de fallos de Drive/GitHub sin corromper el proyecto;
6. reconectar Drive desde la UI cuando OAuth lo requiera y continuar/reintentar la operación pendiente;
7. reiniciar/recargar y conservar proyectos, referencias a recursos, sesiones e historial;
8. conversar en la sala con Papu, Chapu y ambos;
9. comprobar identidad, contexto e aislamiento correctos por proyecto;
10. leer documentos autorizados desde Drive;
11. comprobar que Papu no puede escribir código/Drive si no está autorizado y que Chapu/runtime sólo escriben mediante backend autorizado;
12. promover conversación aprobada a objetos formales persistidos en DB con trazabilidad de origen;
13. despachar una tarea aprobada al runtime y recibir/verificar su resultado;
14. crear un segundo proyecto y comprobar ausencia de contaminación de estado, archivos, contexto y recursos;
15. recuperar errores de proveedor, credencial, timeout y recurso mediante estados/acciones comprensibles para el usuario.

## Orden de implementación

1. **Fundación segura de integraciones** — cifrado, GitHub App, Drive OAuth/reconexión, claves LLM.
2. **Proyectos y aprovisionamiento** — CRUD + topología automática simplificada + idempotencia/recuperación.
3. **Perfiles y política de agentes** — puede avanzar en paralelo.
4. **Núcleo conversacional persistente** — sesiones, mensajes, turnos, floor lock, frontend de sala, adapters de prueba.
5. **Contexto gobernado + proveedores reales** — `input_context_ref`, recuperación mínima, aislamiento.
6. **Gateway tipado y autorizaciones** — DB/Drive/repo código/runtime, auditoría y confirmaciones.
7. **Objetos formales y cierre operativo** — promoción a estado DB, resultados, reinicio, aislamiento y prueba de aceptación.

## No objetivos MVP1.0

Quedan fuera de este recorte de implementación aunque sigan diseñados:

- Kanban (DEC-C12);
- Roadmap/timeline (DEC-C13);
- mapa de tramas (DEC-C14);
- cronología narrativa (DEC-C15).

## Pendiente

- ~~Ajustar implementación existente de proyectos/recursos a esta topología nueva.~~ **Hecho — Bloque B cerrado, ver arriba.**
- ~~Retirar de la UI actual los formularios genéricos de recursos que contradigan esta Decision.~~ **Hecho — ver TASK-0013 (histórica) en `code-tasks/TASK-0013.md`.**
- Definir cuándo se migra o retira la infraestructura transicional del repo de gobernanza de ColaborIA.

## Backlog / evolución futura (no forma parte del MVP1.0 actual — no cambia el alcance vigente de esta Decision)

Registrado como refinamiento posterior, explícitamente **fuera** del MVP1.0. No implica autorización para implementar sin pasar antes por el mismo proceso de propuesta/contraste/aprobación que el resto de esta Decision.

**A. Drive root seleccionable.** Al crear un proyecto, permitir elegir carpeta padre en Google Drive en vez de crear siempre en la raíz. Preferencia UX futura: selector visual de Drive, no ID manual como camino principal.

**B. Eliminación de proyecto con recursos externos opcionales.** Al eliminar un proyecto: siempre se elimina el estado/vínculo del proyecto en ColaborIA. Preguntar aparte si Dani quiere eliminar además la carpeta de Drive y/o el repo de GitHub (si es software) — dejar explícito que eso es destructivo/irreversible. Si no se autoriza, esos recursos externos sobreviven y solo desaparece la referencia en ColaborIA.

**C. Alta usando recursos existentes, como intención simple del flujo normal.** En una futura iteración, permitir elegir "crear recursos nuevos" o "usar recursos existentes" como parte del alta normal (no como flujo aparte de vinculación posterior). **Nota importante:** esto contradice parcialmente el flujo normal actual fijado por esta Decision (donde vincular existentes está separado del alta) — por eso queda solo como backlog/open question de evolución, sin modificar el flujo vigente hasta que se apruebe explícitamente.
