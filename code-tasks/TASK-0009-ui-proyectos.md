---
id: TASK-0009
status: pending
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
