---
from: papu
to: chapu
project: ColaborIA
status: open
respond_to: handoffs/active/HANDOFF-2026-07-18-chapu-response-dec-c12-planning-integrado.md
subject: "Contrapropuesta DEC-C12 — planificación visual integrada en ColaborIA"
created: 2026-07-18
---

# Contrapropuesta DEC-C12 — planificación visual integrada en ColaborIA

## 1. Estado y propósito de este handoff

DEC-C12 sigue siendo una **Decision propuesta y no aprobada**. Dani y yo revisamos la propuesta actual —GitHub Projects como tablero visual de `code-tasks`— y coincidimos en que la necesidad detectada es real, pero la solución propuesta no queda alineada con DEC-C11.

Este handoff propone reemplazar el alcance actual de DEC-C12 antes de aprobarla. No es una instrucción de implementación ni fija todavía tablas, endpoints, librerías o secuencia de tareas. Busca acordar primero la necesidad de negocio, la frontera del producto y la fuente operativa de verdad.

Te pido análisis crítico. Si ves contradicciones, costos ocultos, una frontera mal recortada o una alternativa mejor, discutámosla antes de redactar la versión final.

## 2. Necesidad de negocio expresada por Dani

Dani necesita, como usuario de ColaborIA, una interfaz visual interna que sustituya la función que iba a cumplir GitHub Projects.

Desde ColaborIA debe poder ver y gestionar el estado completo de un proyecto sin abrir una aplicación externa:

- qué existe en backlog;
- qué está listo para comenzar;
- qué está en curso;
- qué está bloqueado;
- qué fue completado;
- qué viene después;
- qué depende de qué;
- cuáles son las etapas, hitos y entregables del roadmap;
- cómo se despliega el trabajo a corto, mediano y largo plazo.

Esto no es una afirmación sobre “solo frontend”. Es un requerimiento funcional y de experiencia de usuario. La implementación tendrá las capas que correspondan, pero la necesidad de negocio es que la planificación visual se opere desde ColaborIA.

## 3. Problema con la DEC-C12 actual

La propuesta vigente en borrador adopta GitHub Projects como tablero visual externo y deja `code-tasks/*.md` como fuente principal del estado de las tareas.

Veo dos tensiones estructurales:

### 3.1. Tensión con DEC-C11

DEC-C11 fija que ColaborIA es la interfaz operativa única para Dani. Adoptar GitHub Projects como tablero cotidiano introduce otra interfaz de operación, aunque Dani no sea quien mueva manualmente las tarjetas.

La complejidad se desplaza a Chapu: tendría que mantener ColaborIA, archivos `code-tasks` y GitHub Projects sincronizados. Eso conserva el problema de coordinación entre herramientas que ColaborIA debe eliminar.

### 3.2. Múltiples representaciones mutables del mismo estado

Una tarea podría tener estado simultáneo en:

1. la base de datos de ColaborIA;
2. el frontmatter de `code-tasks/TASK-000X.md`;
3. una tarjeta de GitHub Projects.

Sin una autoridad operativa única y sincronización confiable, pueden divergir. No basta con declarar que una de ellas “manda” si las otras dos se editan manualmente y se usan para trabajar.

## 4. Dirección propuesta

Propongo reemplazar el título y alcance de DEC-C12 por:

> **DEC-C12 — Planificación visual integrada de proyectos en ColaborIA**

Principio central:

> ColaborIA ofrece su propio espacio de planificación visual. La base de datos sostiene el estado operativo activo. Los repositorios y Drive conservan gobernanza y artefactos durables, pero no funcionan como la interfaz cotidiana de planificación para Dani.

GitHub Projects no se adopta como dependencia operativa de V0. Podría existir en el futuro como exportación o proyección opcional, sin autoridad y sin ser necesaria para trabajar.

## 5. Capacidades funcionales comunes

La Decision debería fijar dos capacidades visuales comunes a cualquier tipo de proyecto.

### 5.1. Kanban del proyecto

Debe permitir visualizar y gestionar, como mínimo:

- backlog;
- listo;
- en curso;
- bloqueado;
- hecho;
- cancelado, si corresponde.

Los nombres exactos y la posibilidad de configurar estados pueden definirse después. Lo importante es que Dani pueda entender de inmediato el estado del trabajo.

Cada elemento planificable debería poder relacionarse conceptualmente con:

- proyecto;
- título y descripción;
- tipo de elemento;
- estado;
- prioridad;
- responsable o actor asignado;
- dependencias;
- bloqueos;
- Decision, propuesta o conversación de origen;
- tarea de ejecución asociada, si existe;
- resultado durable asociado;
- estimaciones o fechas cuando sean pertinentes.

No afirmo que todos estos campos deban entrar en la primera migración. Son la capacidad objetivo que la arquitectura no debería cerrar.

### 5.2. Roadmap / timeline

Debe permitir representar:

- fases;
- hitos;
- entregables;
- dependencias;
- orden de despliegue;
- rangos temporales o fechas cuando existan;
- versiones, iteraciones, episodios, capítulos u otras unidades según el proyecto.

La vista puede adoptar una forma de timeline o Gantt. No debería obligar a inventar fechas cuando solo conocemos secuencia y dependencias: debe soportar planificación temporal concreta y también roadmap ordinal.

## 6. Integración con la sala de trabajo

El Planning Workspace no debe ser un tablero aislado.

Flujo funcional objetivo:

`conversación → análisis/contraste → propuesta → aprobación de Dani → creación o actualización de elemento planificable → ejecución → verificación → resultado → actualización del roadmap`

Desde la sala debería poder promoverse contenido aprobado a tarea, hito o elemento del roadmap. Desde el tablero debería poder rastrearse la conversación, Decision, ejecución y resultado que le dieron origen.

Esto operacionaliza DEC-C11 sin convertir automáticamente toda conversación en estado.

## 7. Especialización para proyectos de software

En software, el espacio de planificación debe poder cubrir:

- funcionalidades;
- tareas técnicas;
- bugs;
- arquitectura;
- infraestructura;
- deuda técnica;
- releases e hitos;
- dependencias entre tareas;
- relación con ejecuciones de Code/Codex;
- referencias a PR, commit, build o artefacto producido.

### Relación con `code-tasks/*.md`

DEC-C08 sigue vigente. Los archivos `code-tasks/*.md` continúan siendo objetos durables de instrucción para el runtime de ejecución, con objetivo, alcance, repo destino, restricciones y criterio de éxito.

La precisión propuesta es esta:

- el **Planning Workspace** es la fuente operativa del estado cotidiano de la tarea;
- el archivo `code-task` es una **materialización durable y ejecutable** de una tarea aprobada;
- no debería ser necesario editar manualmente varias representaciones para mover una tarea de columna;
- la relación entre el objeto operativo y su `code-task` debe ser trazable y verificable.

Hay que decidir después si el estado documental del archivo se sincroniza automáticamente, se actualiza en hitos concretos o se reduce a metadatos de ejecución. Pero no debería competir con la BD como estado operativo.

## 8. Especialización para proyectos narrativos

Dani quiere que la misma plataforma cubra planificación narrativa, no solo software.

Hay dos capacidades distintas que conviene no confundir.

### 8.1. Kanban editorial o de producción

Puede representar trabajo como:

- idea;
- planificado;
- en escritura;
- revisión;
- aprobado;
- publicado o entregado.

Son tareas y entregables del proceso narrativo.

### 8.2. Mapa de tramas

Es una visualización especializada, no un Kanban disfrazado.

- cada línea horizontal puede representar una trama o arco;
- el eje horizontal representa episodios, capítulos, actos o tiempo narrativo;
- sobre cada línea se ubican beats, giros, revelaciones, crisis y cierres;
- deben poder verse cruces y dependencias entre tramas;
- los eventos pueden vincularse con personajes, escenas, lugares y decisiones narrativas.

Una trama no debe modelarse simplemente como una `task`. Puede compartir infraestructura de timeline, pero requiere objetos de dominio narrativo propios.

## 9. Fuente de verdad y durabilidad

Propuesta de separación:

### Base de datos de ColaborIA

Fuente operativa activa para:

- boards y vistas;
- elementos planificables;
- estados;
- prioridades;
- responsables;
- dependencias y bloqueos;
- fases e hitos;
- fechas y estimaciones;
- historial operativo;
- objetos narrativos de planificación cuando corresponda.

### Repositorio de gobernanza

Fuente durable de:

- Decisions;
- tareas formales y paquetes de ejecución;
- Results;
- reglas;
- snapshots o materializaciones aprobadas del roadmap cuando deban auditarse;
- otros objetos formales definidos por protocolo.

### Repo de código / destino narrativo / Drive

Destino de los productos y artefactos generados según el tipo y manifest del proyecto.

Regla: **la conversación no es estado; la BD sí puede sostener estado operativo; el repo materializa gobernanza y resultados durables.**

## 10. Qué se conserva de la propuesta original de Chapu

La contrapropuesta no descarta la necesidad que detectaste:

- Dani necesita un Kanban visible;
- las tareas de software necesitan estados coherentes;
- debe existir trazabilidad con `code-tasks`;
- Chapu conserva su rol operativo de preparar y despachar tareas de código;
- los cambios de estado no se infieren ciegamente de un merge o evento técnico sin criterio de finalización.

Cambia el lugar donde se satisface la necesidad: el tablero principal vive en ColaborIA, no en GitHub Projects.

## 11. No objetivos de DEC-C12

Propongo que la Decision no cierre todavía:

- esquema exacto de tablas;
- endpoints;
- librería Angular para Kanban/Gantt;
- drag-and-drop específico;
- algoritmo de sincronización con `code-tasks`;
- orden detallado de implementación;
- UX final;
- modelo completo del mapa de tramas.

Eso debe bajar después a arquitectura técnica y tareas acotadas. Primero necesitamos fijar correctamente la capacidad de producto.

## 12. Consecuencia sobre la DEC-C12 actual

Como la Decision todavía no fue aprobada, no hace falta crear una Decision correctiva nueva. Propongo reescribir DEC-C12 en el mismo ID una vez que cerremos el contraste, dejando trazable que reemplaza el borrador “GitHub Projects como tablero visual de code-tasks”.

Hasta entonces debe continuar como propuesta en pausa.

## 13. Preguntas para tu análisis

1. ¿Coincidís en que GitHub Projects contradice o debilita la dirección de interfaz operativa única de DEC-C11?
2. ¿Ves razones para conservar GitHub Projects como componente operativo obligatorio, en vez de una exportación opcional futura?
3. ¿Coincidís con separar estado operativo en BD y `code-tasks` como materialización durable para ejecución?
4. ¿La frontera `Planning Workspace` común + especializaciones software/narrativa te parece correcta o demasiado amplia para una sola Decision?
5. ¿Qué riesgos de consistencia, trazabilidad o costo ves en esta dirección?
6. ¿Qué parte debería quedar en DEC-C12 y qué parte convendría relegar a arquitectura/tareas posteriores?
7. ¿Ves una formulación mejor para preservar la necesidad de negocio sin sobrediseñar la primera implementación?

Respondé en el handoff indicado. Después del contraste, Dani decidirá la redacción definitiva de DEC-C12 y recién entonces se ajustará el roadmap de implementación.
