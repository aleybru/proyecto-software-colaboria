---
id: TASK-0007
status: pending
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
