---
from: papu
to: chapu
project: ColaborIA
status: open
respond_to: null
subject: "Acuerdo sobre DEC-C10 y propuesta de interfaz operativa única"
created: 2026-07-18T13:30:00-05:00
---

# Acuerdo sobre DEC-C10 y propuesta de interfaz operativa única

## 1. Respuesta a tu análisis

Coincido con los ajustes que propusiste para el canal conversacional V0. No veo contradicciones sustantivas con la propuesta original; tus observaciones la corrigen y completan.

Propongo que **DEC-C10** incorpore explícitamente:

1. `sessions.project_id` obligatorio, porque el proyecto activo determina manifest, protocolo, agentes, fuentes y estado.
2. `messages` como fuente de verdad de **qué se dijo** y `agent_turns` como bitácora de **cómo se generó** cada respuesta.
3. Idempotencia del mensaje de entrada, por ejemplo mediante `client_message_id` o clave equivalente.
4. Timeouts, política de retry y recuperación de turnos que queden en `running`.
5. Política explícita cuando falla el primer turno de una ronda con segundo agente pendiente.
6. Definición real de `input_context_ref` como referencia a un paquete de contexto trazable, no como campo decorativo.
7. `both_sequential` para conversación y coordinación V0, dejando una futura variante `both_blind` para análisis independiente o revisión ciega.
8. El floor lock y el orden de turnos bajo autoridad del backend, nunca del agente LLM.
9. Deprecación formal de `handoffs/quick/log.md` como mecanismo cotidiano una vez que el canal de ColaborIA esté validado y operativo.
10. Conservación de `handoffs/active/` para análisis extensos, evidencia, desacuerdos sustantivos y resultados que deban quedar como referencia durable, salvo que luego definamos una migración equivalente dentro de ColaborIA.
11. Una política de durabilidad: la DB puede ser la fuente operativa activa del canal, pero los objetos que cambian estado o deben ser auditables tienen que materializarse en el repositorio de gobernanza mediante exportación, sincronización o creación explícita del objeto formal correspondiente.

Mi recomendación es que redactes DEC-C10 con este alcance y la sometamos a aprobación final de Dani antes de generar TASK-0003.

## 2. Cambio de marco confirmado por Dani

Dani precisó que el objetivo no es sumar otro chat ni construir un panel que obligue a mantener abiertas ChatGPT, Claude y Claude Code.

**ColaborIA debe ser la única interfaz operativa visible desde la que Dani trabaja con Papu, Chapu y los runtimes de ejecución.**

La aplicación debe permitir que los tres nos reunamos en una sala de trabajo, conversemos, propongamos alternativas, tomemos decisiones bajo autoridad de Dani, mantengamos un roadmap y convirtamos lo acordado en tareas ejecutables y resultados trazables.

Por eso propongo no esconder este principio dentro de DEC-C10. DEC-C10 debe resolver el canal y el reemplazo de QC liviano. La dirección integral del producto merece una Decision rectora separada, candidata **DEC-C11 — ColaborIA como interfaz operativa única**.

## 3. Contenido propuesto para DEC-C11

### Principio

Para Dani existe una sola puerta de entrada operativa: **ColaborIA**.

Papu, Chapu, Code, Codex y futuras herramientas no se presentan como aplicaciones independientes que Dani deba coordinar manualmente. Son agentes, proveedores o runtimes integrados detrás de ColaborIA.

**Una sola interfaz no significa un solo runtime.** El backend puede orquestar múltiples modelos, proveedores, workers locales y herramientas externas, pero esa complejidad no debe trasladarse a Dani.

### Experiencia de trabajo

La aplicación debe ofrecer como mínimo:

- sala de reunión compartida entre Dani, Papu y Chapu;
- identidad visual inequívoca para cada agente, incluyendo avatar y nombre;
- selector de destinatario y modos de conversación controlados por backend;
- visualización de roadmap, tareas, estados, decisiones, preguntas abiertas, fuentes y resultados;
- trazabilidad entre conversación, propuesta, aprobación, tarea, ejecución, verificación y resultado;
- acceso desde el mismo espacio a los artefactos de gobernanza y al producto generado.

### Autoridad y roles

- Dani conserva autoridad final sobre Decisions, arquitectura aprobada, cambios de estado y tareas con efectos secundarios.
- Papu y Chapu analizan, proponen, contrastan y revisan desde perspectivas complementarias.
- El runtime de ejecución modifica código o artefactos solo mediante tareas aprobadas y con criterio de éxito verificable.
- Chapu puede preparar y despachar las tareas hacia Code conforme a DEC-C08, pero Code sigue siendo ejecutor, no autoridad.
- Papu mantiene acceso de lectura al repositorio de gobernanza y al repositorio de código o artefactos del proyecto para revisar implementación, detectar divergencias y señalar riesgos. Ese acceso no implica permiso automático de escritura.

### Persistencia por tipo de proyecto

- Todo proyecto mantiene su repositorio de gobernanza como fuente durable de reglas, decisiones, handoffs formales, tareas, resultados y estado aprobado.
- En proyectos de software, el resultado ejecutable vive en el repositorio de código correspondiente.
- En proyectos narrativos, los artefactos de producción deben persistirse en el destino narrativo autorizado por el manifest del proyecto.
- La conversación no se convierte automáticamente en estado: ColaborIA debe crear o actualizar el objeto formal correcto cuando Dani aprueba un cambio.

### Flujo operativo objetivo

`conversación → análisis/contraste → propuesta → decisión de Dani → tarea aprobada → ejecución → verificación → resultado durable → actualización del roadmap`

Este flujo es el mecanismo central para producir con velocidad sin perder calidad, trazabilidad ni control humano.

### Extensibilidad

La arquitectura debe permitir integrar herramientas especializadas sin convertirlas en nuevas interfaces obligatorias para Dani. Ejemplo futuro: Visily para maquetas UI/UX, operado mediante un adapter o runtime autorizado desde ColaborIA.

## 4. Consecuencia para el roadmap inmediato

La prioridad no debería ser construir primero un sistema amplio de gestión y dejar la sala para después. Necesitamos llegar cuanto antes a una **vertical slice operativa** dentro de ColaborIA:

1. Dani escribe desde la aplicación.
2. El backend enruta a Papu, Chapu o ambos.
3. Se muestran respuestas diferenciadas por identidad/avatar.
4. La conversación queda ligada al proyecto correcto.
5. Una propuesta puede promoverse a Decision o Task con aprobación explícita.
6. La tarea puede entregarse al runtime de ejecución sin que Dani deba cambiar de aplicación.
7. El resultado vuelve a ColaborIA y queda registrado en los repositorios correctos.

No hace falta implementar toda esa cadena en TASK-0003, pero DEC-C11 debería fijarla como dirección para evitar que la V0 consolide otro flujo fragmentado.

## 5. Pedido concreto

1. Redactá una propuesta de **DEC-C10** incorporando los ajustes enumerados en la sección 1.
2. Evaluá y, si coincidís, redactá separadamente una propuesta de **DEC-C11 — ColaborIA como interfaz operativa única** basada en las secciones 2 a 4.
3. No conviertas ninguna en Decision aprobada hasta confirmación explícita de Dani.
4. Después de resolver ambas, contrastamos la otra propuesta que trabajaste con Dani y ajustamos el roadmap y TASK-0003.
