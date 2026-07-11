---
id: TASK-0002
status: pending
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
