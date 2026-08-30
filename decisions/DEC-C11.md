# DEC-C11 — ColaborIA como interfaz operativa única

**Estado:** Decision vigente
**Fecha original:** 2026-07-18
**Revisión vigente:** 2026-08-29
**Autoridad:** Dani
**Relacionada con:** DEC-C08, DEC-C09, DEC-C10, DEC-C16

## Resumen

ColaborIA es la única puerta de entrada operativa para Dani. Papu, Chapu y los runtimes de ejecución operan detrás de la aplicación.

La revisión de 2026-08-29 elimina el repositorio de gobernanza como recurso obligatorio de cada proyecto. La gobernanza operativa y durable de los proyectos creados por ColaborIA vive en la DB de la aplicación; Drive es la raíz documental de todos los proyectos y los proyectos software agregan un único repo de código.

## Decisión

### 1. Principio

Para Dani existe una sola interfaz operativa: **ColaborIA**.

La complejidad de proveedores, herramientas y runtimes queda detrás del backend.

### 2. Experiencia mínima

ColaborIA debe permitir, desde el mismo espacio:

- entrar a un proyecto;
- conversar con Papu, Chapu o ambos;
- ver identidad inequívoca de cada agente;
- consultar estado, decisiones, preguntas abiertas, tareas, resultados y fuentes;
- acceder a los archivos de Drive del proyecto;
- acceder al repo de código cuando el proyecto sea software;
- rastrear conversación → propuesta → aprobación → tarea → ejecución → resultado.

### 3. Autoridad y roles

- Dani conserva autoridad final sobre Decisions, arquitectura aprobada, cambios de estado y efectos secundarios.
- Papu y Chapu analizan, proponen, contrastan y revisan.
- El runtime de ejecución modifica código o artefactos solamente mediante acciones autorizadas y verificables.
- Identidad de agente no equivale a autorización efectiva.

### 4. Matriz objetivo de accesos

Para proyectos operados dentro de ColaborIA:

| Recurso | Papu | Chapu |
|---|---|---|
| Estado/gobernanza del proyecto en ColaborIA (DB, vía backend) | lectura + escritura autorizada | lectura + escritura autorizada |
| Repo de código (solo software) | solo lectura | lectura + escritura autorizada |
| Carpeta raíz de Drive | solo lectura | lectura + escritura autorizada |

**Aclaración de lectura de la matriz (2026-08-30, sin cambio de la matriz en sí):** en esta matriz, "lectura + escritura autorizada" significa que la escritura está dentro del techo de autoridad del agente, pero **cada operación con efecto real requiere opt-in explícito de Dani** conforme a DEC-C04. Nunca equivale a escritura libre ni permite superar una celda definida como `denied` en una política de autorización más granular (ver DEC-C16 Bloque C — techo de autoridad por operación).

Toda escritura con efecto real sigue sometida a la autorización de Dani y a las reglas del backend. La capacidad técnica nunca equivale por sí sola a permiso.

**Excepción transicional:** el repo de gobernanza actual de ColaborIA continúa accesible a Papu y Chapu según el workflow vigente de desarrollo. No forma parte de la topología de recursos de proyectos nuevos.

### 5. Persistencia por tipo de proyecto

Para cualquier proyecto:

- la DB de ColaborIA contiene estado, conversación, decisiones, tareas, resultados, preguntas abiertas, handoffs y trazabilidad;
- la carpeta raíz de Google Drive contiene los documentos y artefactos de archivo del proyecto.

Para proyectos `software` además:

- existe un único repositorio GitHub de código;
- ese repo puede funcionar como monorepo para backend, frontend y demás componentes de código.

Para proyectos `narrative`:

- no se crea repo GitHub por defecto;
- sus artefactos documentales viven en Drive y su estado gobernado vive en ColaborIA.

La conversación no se convierte automáticamente en estado: la promoción a objeto formal requiere workflow y autorización.

### 6. Flujo operativo objetivo

```text
conversación → análisis/contraste → propuesta → decisión de Dani → tarea aprobada
→ ejecución → verificación → resultado durable → actualización de estado/roadmap
```

El resultado durable se persiste según su naturaleza:

- estado/gobernanza → DB;
- documento/archivo → Drive;
- código → repo de código del proyecto software.

### 7. Consecuencia para la creación de proyectos

La interfaz no pide al usuario construir manualmente la topología de recursos.

Crear proyecto requiere:

- nombre;
- tipo (`narrative | software`).

El backend deriva y aprovisiona automáticamente:

- siempre: carpeta raíz de Drive;
- software: además un único repo GitHub de código.

No hay selector genérico de proveedor/tipo para estos recursos base.

### 8. Extensibilidad

Se pueden agregar herramientas o recursos futuros, pero no deben convertir la creación básica de proyecto en un formulario genérico de infraestructura.

## Consecuencia para el roadmap inmediato

La vertical slice sigue siendo prioritaria:

1. Dani entra/crea un proyecto.
2. ColaborIA conoce automáticamente sus recursos base.
3. Dani conversa con Papu/Chapu desde ese proyecto.
4. El backend aplica identidad, contexto y permisos correctos.
5. Contenido aprobado puede promoverse a estado formal en DB.
6. Tareas autorizadas pueden enviarse al runtime de ejecución.
7. Los resultados vuelven a ColaborIA y se persisten en DB, Drive o repo de código según corresponda.

## No objetivos

- Definir todavía toda la UI de sala.
- Definir el esquema físico completo de la gobernanza en DB.
- Eliminar el repo histórico de gobernanza de ColaborIA.
