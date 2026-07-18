# DEC-C11 — ColaborIA como interfaz operativa única

**Estado:** Decision propuesta — pendiente de aprobación explícita de Dani
**Fecha:** 2026-07-18
**Origen:** propuesta de Papu (`handoffs/active/HANDOFF-2026-07-18-papu-response-canal-e-interfaz-unica.md`, secciones 2-4), matriz de accesos definida y corregida por Dani en conversación directa con Chapu (incluye una ronda de corrección sobre lectura de un diagrama dibujado a mano).
**Relacionada con:** DEC-C08, DEC-C09, DEC-C10 (el canal conversacional que esta Decision usa como mecanismo).

## Resumen

Dani deja de coordinar manualmente entre aplicaciones separadas (ChatGPT para Papu, Claude para Chapu, Claude Code para ejecución). **ColaborIA se convierte en la única puerta de entrada operativa** desde la que Dani trabaja con Papu, Chapu y los runtimes de ejecución. Papu, Chapu, Code y futuras herramientas son agentes/runtimes integrados detrás de ColaborIA, no aplicaciones independientes.

Una sola interfaz no implica un solo runtime: el backend puede orquestar múltiples modelos, proveedores y herramientas externas, pero esa complejidad no se traslada a Dani.

## Decisión

### 1. Principio

Para Dani existe una sola puerta de entrada operativa: **ColaborIA**.

### 2. Experiencia de trabajo mínima

- sala de reunión compartida entre Dani, Papu y Chapu (usa el canal de DEC-C10);
- identidad visual inequívoca para cada agente (avatar y nombre);
- selector de destinatario y modos de conversación, controlados por el backend;
- visualización de roadmap, tareas, estados, decisiones, preguntas abiertas, fuentes y resultados;
- trazabilidad entre conversación → propuesta → aprobación → tarea → ejecución → verificación → resultado;
- acceso desde el mismo espacio a los artefactos de gobernanza y al producto generado.

### 3. Autoridad y roles

- Dani conserva autoridad final sobre Decisions, arquitectura aprobada, cambios de estado y tareas con efectos secundarios (sin cambios respecto al protocolo común y DEC-C04).
- Papu y Chapu analizan, proponen, contrastan y revisan desde perspectivas complementarias (sin cambios respecto al protocolo común).
- El runtime de ejecución (Claude Code u otro) modifica código o artefactos solo mediante tareas aprobadas con criterio de éxito verificable (sin cambios respecto a DEC-C08 — sigue siendo ejecutor, no autoridad).
- Chapu puede preparar y despachar tareas hacia el runtime de ejecución conforme a DEC-C08.

### 4. Matriz de accesos por agente y recurso

Definida explícitamente por Dani, con precisión mayor a la del borrador original de Papu:

| Recurso | Papu | Chapu |
|---|---|---|
| Repo de gobernanza | lectura + escritura | lectura + escritura |
| Repo de código | solo lectura | lectura + escritura |
| Carpeta Drive | solo lectura | lectura + escritura |

Razón: no mezclar los accesos de cada agente más de lo necesario. Papu prioriza arquitectura y trade-offs (rol ya definido en el protocolo común, sección 7.2) — le alcanza con lectura de código y Drive para detectar divergencias y señalar riesgos, sin necesidad de escritura ahí. Chapu prioriza consistencia entre decisiones documentadas y código real (mismo protocolo) — ese rol requiere escritura en los tres.

El acceso de lectura de Papu a repo de código y Drive **no implica permiso automático de escritura** — la matriz es explícita y no se infiere de la capacidad técnica disponible.

### 5. Persistencia por tipo de proyecto

- Todo proyecto mantiene su repositorio de gobernanza como fuente durable de reglas, decisiones, handoffs formales, tareas y estado aprobado (sin cambios respecto al protocolo común).
- En proyectos de software, el resultado ejecutable vive en el repositorio de código correspondiente.
- En proyectos narrativos, los artefactos de producción se persisten en el destino narrativo autorizado por el manifest del proyecto.
- La conversación no se convierte automáticamente en estado — ColaborIA crea o actualiza el objeto formal correcto solo cuando Dani aprueba un cambio (aplica el principio "conversación no es estado" del protocolo común, sección 8).

### 6. Flujo operativo objetivo

```
conversación → análisis/contraste → propuesta → decisión de Dani → tarea aprobada
→ ejecución → verificación → resultado durable → actualización del roadmap
```

### 7. Extensibilidad

La arquitectura permite integrar herramientas especializadas sin que se conviertan en nuevas interfaces obligatorias para Dani (ejemplo futuro: una herramienta de maquetas UI/UX, operada mediante un adapter autorizado desde ColaborIA, no como aplicación aparte que Dani deba abrir).

## Consecuencia para el roadmap inmediato

La prioridad no es construir primero un sistema amplio de gestión y dejar la sala de trabajo para después. El objetivo de corto plazo es una **vertical slice operativa** dentro de ColaborIA:

1. Dani escribe desde la aplicación.
2. El backend enruta a Papu, Chapu o ambos.
3. Se muestran respuestas diferenciadas por identidad/avatar.
4. La conversación queda ligada al proyecto correcto (DEC-C10, punto 1).
5. Una propuesta puede promoverse a Decision o Task con aprobación explícita de Dani.
6. La tarea puede entregarse al runtime de ejecución sin que Dani cambie de aplicación.
7. El resultado vuelve a ColaborIA y queda registrado en los repositorios correctos, según la matriz de accesos de la sección 4.

No es necesario implementar toda esta cadena en una sola tarea — pero el diseño de las tareas siguientes debe apuntar hacia esta dirección para no consolidar un flujo fragmentado en V0.

## No objetivos de esta Decision

- No define el detalle técnico de implementación del backend orquestador (queda para tareas posteriores).
- No resuelve el mecanismo exacto de "promoción" de propuesta a Decision/Task dentro de la UI (queda para diseño de producto posterior).

## Pendiente

- Diseño de UI/UX concreto para la sala de trabajo compartida.
- Definición de la vertical slice como tarea o tareas ejecutables concretas.
- Cómo se refleja la matriz de accesos de la sección 4 a nivel técnico (permisos por agente en el backend, no solo declarativo en esta Decision).
