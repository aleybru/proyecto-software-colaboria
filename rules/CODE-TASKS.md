# CODE-TASKS.md — Procedimiento y esquema de tarjetas para `code-tasks/`

**Tipo:** Rule / Procedure (no es Decision — implementa DEC-C08)
**Fecha:** 2026-07-09
**Vigencia:** aplica a toda tarea creada en `code-tasks/` del repo de gobernanza.

Este documento define el *cómo*. El *qué se decidió* (boundary, autorización, principios) vive en `decisions/DEC-C08.md` y no se repite acá. Si hay conflicto entre este Procedure y DEC-C08, DEC-C08 tiene autoridad.

---

## 1. Qué es una code-task

Una code-task es un archivo Markdown en `code-tasks/`, con YAML frontmatter + cuerpo estructurado, que instruye a Claude Code a ejecutar una unidad de trabajo pequeña y verificable sobre el repo de código (`aleybru/proyecto-software-colaboria-codigo`).

No es un Handoff (eso es Papu↔Chapu, análisis y contraste). Una code-task se escribe **después** de que la discusión y la aprobación de Dani ya ocurrieron — es el resultado empaquetado para ejecución, no el lugar donde se decide.

## 2. Convención de nombre y ID

- Archivo: `code-tasks/TASK-NNNN-slug-corto.md`
- `NNNN`: correlativo de 4 dígitos, con ceros a la izquierda, secuencial (no se reutilizan números aunque una tarea se cancele).
- `slug-corto`: 2-4 palabras en minúscula separadas por guiones, descriptivas (ej. `TASK-0001-setup-repo-base.md`).

## 3. Esquema del YAML frontmatter (obligatorio)

```yaml
---
id: TASK-0001
status: pending          # pending | in_progress | done | blocked | cancelled
repo_destino: aleybru/proyecto-software-colaboria-codigo
creado: 2026-07-09
aprobado_por: Dani
depende_de: []            # lista de ids de otras code-tasks, si hay orden estricto
decision_ref: DEC-C08     # Decision(es) que justifican o enmarcan esta tarea, si aplica
---
```

Campos obligatorios sin excepción (por DEC-C08, punto 4): `id`, `status`, `repo_destino`, criterio de éxito (ver sección 4, va en el cuerpo).

## 4. Esquema del cuerpo (obligatorio)

```md
## Objetivo
Qué problema resuelve esta tarea, en 1-3 líneas.

## Qué tiene que hacer Code (concreto)
Instrucción operativa, sin ambigüedad. Si involucra archivos o módulos
específicos, nombrarlos. Si involucra una API o contrato, especificarlo.

## Criterio de éxito
Cómo se verifica que la tarea está terminada. Preferentemente verificable
por máquina (test que pasa, build que compila, comando que corre sin error) —
en línea con DEC-C03 (modo automático acotado solo para criterio verificable).
Si el criterio es subjetivo, decirlo explícitamente y aclarar que requiere
revisión humana antes de marcar `done`.

## Contexto / restricciones
Cualquier limitación, decisión previa vinculante, o cosas que NO debe hacer
Code (ej. "no modificar el schema de la tabla X sin nueva code-task").
```

## 5. Ciclo de vida de una tarea

1. Papu y Chapu discuten y definen el contenido (con revisión ciega si el tema lo amerita — ver protocolo común sección 11).
2. Dani aprueba el contenido.
3. Chapu escribe el archivo en `code-tasks/` con `status: pending`, hace commit y push, y avisa a Dani que está lista.
4. Dani, cuando quiere, va a Claude Code y dispara la ejecución explícitamente ("ejecutá TASK-000X"). **Este paso es siempre manual** (DEC-C08, punto 3) — ninguna automatización dispara este paso por Dani.
5. Claude Code ejecuta, y al terminar actualiza el propio archivo:
   - cambia `status` a `done`, `blocked` o deja `in_progress` si quedó a mitad;
   - agrega una sección `## Resultado` al final del cuerpo con qué se hizo, qué archivos tocó, y el resultado de la verificación del criterio de éxito.
6. El archivo actualizado queda como el `Result` trazable de esa tarea (protocolo común, sección 8) — no se borra, no se mueve a otra carpeta salvo que en el futuro se decida archivar tareas `done` en una subcarpeta (no definido todavía).

## 6. Qué NO cubre este Procedure

- No define el stack técnico ni convenciones de código — eso es contenido de cada tarea o de un documento de arquitectura aparte, todavía no escrito.
- No define si Code puede encadenar varias tareas en una sola invocación — por ahora, una invocación de Dani = una tarea (implícito en el gatillo manual de DEC-C08, punto 3), pero no está escrito como regla explícita. Señalado como ambigüedad menor a resolver si se vuelve relevante en la práctica.
