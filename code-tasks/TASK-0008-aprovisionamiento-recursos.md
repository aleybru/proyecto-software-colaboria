---
id: TASK-0008
status: pending
repo_destino: aleybru/proyecto-software-colaboria-codigo
creado: 2026-08-15
aprobado_por: Dani
depende_de: [TASK-0007]
decision_ref: DEC-C09, DEC-C16
---

## Objetivo

Aprovisionar recursos reales (repos de GitHub, carpeta de Drive) para un proyecto registrado en TASK-0007, usando las integraciones ya terminadas (TASK-0004 GitHub App, TASK-0005 Drive OAuth), con la convención de nombres de DEC-C09 sección 7.

## Qué tiene que hacer Code (concreto)

1. Al confirmar la creación de un recurso pendiente (registrado en TASK-0007 con la intención de "crear nuevo"), disparar la creación real:
   - repo de GitHub vía GitHub App (TASK-0004), con nombre `<nombre>-gobernanza` (+ `<nombre>-codigo` si el proyecto es `software`);
   - carpeta de Drive vía la integración OAuth (TASK-0005), con nombre `"Proyecto <Nombre>"`.

2. Idempotencia: si la operación se reintenta (por fallo de red, timeout, etc.), no debe crear un recurso duplicado — debe detectar que ya existe (o que ya se creó en un intento anterior) y no fallar de forma confusa.

3. Recuperación de fallo parcial: si se crean 2 de 3 recursos de un proyecto software (ej. los dos repos pero falla la carpeta de Drive), el estado de cada recurso individual debe reflejar exactamente qué se logró y qué no — no un estado global ambiguo de "falló todo".

4. Actualizar el `status` de cada `project_resource` conforme al resultado real (creado con éxito, error, pendiente de reintento).

## Criterio de éxito

- Crear un proyecto nuevo de prueba y confirmar que sus recursos reales (repo(s) + carpeta Drive) existen de verdad en GitHub/Drive, con los nombres exactos de la convención.
- Reintentar la creación de un recurso ya creado no genera un duplicado.
- Simular o provocar un fallo parcial (ej. desconectar momentáneamente una de las dos integraciones) y confirmar que el estado de cada recurso refleja la realidad exacta, no un error genérico.

## Contexto / restricciones

- No re-crear ni tocar los recursos de ColaborIA vinculados en TASK-0007 — esos ya existen y fueron vinculados, no creados; esta tarea no debe intentar aprovisionarlos de nuevo.
- No implementar todavía la UI (TASK-0009).
- Respetar DEC-C08: elección de estrategia técnica exacta de idempotencia (ej. operación con clave idempotente vs. verificación previa de existencia) queda a criterio de Code, documentada en `## Resultado`.
