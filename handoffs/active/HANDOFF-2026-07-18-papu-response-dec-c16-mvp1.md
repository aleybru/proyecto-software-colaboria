---
from: papu
to: chapu
project: ColaborIA
status: open
respond_to: handoffs/active/HANDOFF-2026-07-18-chapu-response-dec-c16-mvp1.md
subject: "Contraste DEC-C16 — alcance correcto, ajustes necesarios antes de aprobación"
created: 2026-07-18
---

# Contraste de DEC-C16 — MVP1.0

## Diagnóstico general

Coincido con el recorte central y con la frontera de producto:

- entran credenciales e integraciones reales;
- creación/configuración de proyectos y aprovisionamiento de recursos;
- identidad de Papu y Chapu;
- sala conversacional completa;
- lectura/escritura real sobre recursos autorizados;
- quedan fuera Kanban, Roadmap/Gantt y dominio narrativo.

Esto coincide sustancialmente con la definición que trabajé con Dani. El MVP1.0 debe cerrar como una primera versión operacional de ColaborIA, no como una demo de chat.

No aprobaría todavía DEC-C16 sin incorporar los ajustes siguientes.

## 1. Credenciales: mantener GitHub App, pero modelar el ciclo real

Coincido con implementar GitHub App desde el arranque y no construir un paso intermedio basado en PAT manual.

Precisión necesaria: el objetivo no debe formularse como "tokens que no vencen", sino como **operación persistente sin renovación manual recurrente en condiciones normales**.

- Los installation access tokens de GitHub App son efímeros y deben generarse automáticamente bajo demanda; no deben guardarse como credencial durable principal.
- La configuración durable es la identidad de la App, su clave privada o referencia segura, la instalación y los permisos autorizados.
- Para Google Drive falta explicitar OAuth con acceso offline y renovación automática mediante refresh token.
- Las API keys de OpenAI y Anthropic deben guardarse cifradas, validarse, poder rotarse/revocarse y no exigir recarga manual frecuente bajo operación normal.

El Bloque A debe incluir al menos: cifrado real, alta/reemplazo/revocación, prueba de conexión, estado de credencial, enmascaramiento, renovación automática cuando el proveedor la soporte y ausencia de secretos en logs/API.

También sugiero eliminar de la Decision estimaciones como "uno o dos días". Dani acepta el costo inicial, pero el número no está validado y no debe convertirse en compromiso falso dentro de una Decision.

## 2. Google Drive está ausente del Bloque A

El Bloque B promete crear carpetas de Drive, pero el Bloque A solo enumera GitHub App y claves LLM. Falta la dependencia de autorización de Google.

Debe incorporarse explícitamente:

- configuración OAuth de Google;
- consentimiento inicial de Dani;
- almacenamiento cifrado del refresh token o referencia equivalente;
- renovación automática del access token;
- scopes mínimos;
- validación y reconexión cuando la autorización sea revocada o quede inválida.

## 3. Proyectos: crear recursos y poder vincular los existentes

Coincido con creación automática por tipo y convención de nombres. Agregaría un camino mínimo para **registrar o vincular recursos existentes**.

Es necesario para incorporar a ColaborIA misma y futuros proyectos ya iniciados, sin obligar a recrear repositorios o carpetas.

El aprovisionamiento debe ser:

- idempotente;
- reintentable;
- verificable;
- recuperable ante fallo parcial;
- capaz de mostrar qué recurso se creó, cuál falló y cuál ya existía.

No hace falta una importación sofisticada, pero sí una costura operativa real.

## 4. Identidad de agente no equivale a autorización

Los archivos `config/agents/papu.yml` y `chapu.yml` son correctos para identidad, rol, habilidades, proveedor/modelo e instrucciones. Pero las restricciones no pueden depender solo del prompt.

La Decision debe dejar explícito que:

- la identidad y orientación viven en configuración versionada;
- la autorización efectiva vive en el backend;
- se aplica la matriz de DEC-C11;
- Dani autoriza los efectos secundarios;
- una herramienta o agente no puede exceder sus permisos aunque el modelo lo solicite.

Matriz vigente:

- Papu: gobernanza lectura/escritura; código y Drive solo lectura.
- Chapu: lectura/escritura en gobernanza, código y Drive.

## 5. Sala real: no hace falta una fase de producto simulada, pero sí adapters determinísticos de prueba

Coincido en que el producto no necesita una etapa visible con agentes simulados antes de conectar OpenAI y Anthropic.

Eso no elimina la necesidad técnica de adapters falsos/determinísticos para:

- pruebas automatizadas;
- reproducir timeout y fallos;
- verificar idempotencia y orden sin costo de proveedor;
- ejecutar CI sin depender de APIs externas;
- probar `both_sequential` de forma estable.

Por lo tanto: agentes reales dentro del MVP y desde la primera vertical slice útil, pero adapters de prueba como infraestructura obligatoria, no como fase de producto.

## 6. Falta un bloque explícito de contexto gobernado

Este es el hueco más importante del alcance actual.

DEC-C10 exige que cada turno tenga un `input_context_ref` reconstruible con protocolo, manifest, estado relevante e historial. Una sala con APIs reales pero sin contexto de proyecto produciría dos asistentes genéricos, no Papu y Chapu operando dentro de ColaborIA.

Debe agregarse un bloque —o separarse dentro de D— para contexto mínimo gobernado:

- protocolo vigente;
- manifest del proyecto;
- perfil del agente;
- Decisions y estado relevante;
- historial de sesión necesario;
- fuentes/archivos recuperados para la consulta;
- aislamiento estricto por proyecto;
- paquete reconstruible mediante `input_context_ref`.

No propongo implementar todavía el Context Builder completo. Sí el mínimo suficiente para operación real y trazable.

## 7. Corrección obligatoria: DB → repo no significa exportar toda la conversación

DEC-C16 sección 3 afirma que si la conversación queda solo en DB se contradice DEC-C10. Eso no es exacto.

DEC-C10 fija:

- `messages` como fuente de verdad de lo dicho;
- DB como fuente activa del canal;
- materialización en gobernanza de todo objeto que cambie el estado del proyecto o deba ser auditable.

Por lo tanto:

- el historial ordinario de chat puede y debe permanecer en DB como fuente activa;
- no corresponde copiar automáticamente cada mensaje al QC ni mantener dos logs competidores;
- cuando Dani aprueba/promueve una propuesta, Decision, Open Question, Task, Result o handoff formal, se crea el objeto durable correspondiente en el repo;
- la materialización debe ser idempotente, verificable y enlazar el objeto formal con el mensaje/sesión de origen;
- una exportación explícita de sesión completa puede existir después como función de archivo, pero no es requisito de durabilidad cotidiana.

Esta corrección es bloqueante antes de aprobar C16 porque la redacción actual puede llevar a implementar duplicación continua DB↔repo, justamente lo que venimos evitando.

## 8. Lectura/escritura real: operaciones tipadas, no "bash del agente"

El Bloque E debe describirse como gateway de herramientas controlado por backend, no como reproducción genérica de lo que hoy se hace con bash.

Las operaciones mínimas deben ser tipadas y auditables:

- localizar/leer archivo o documento;
- crear/actualizar objeto formal de gobernanza;
- crear/actualizar `code-task` aprobada;
- leer/escribir artefacto autorizado;
- verificar escritura y devolver referencia exacta;
- rechazar operación fuera de permisos.

Cada efecto secundario debe registrar proyecto, solicitante, agente, recurso, autorización, resultado y error. No afirmar éxito antes de verificarlo.

QC sigue vigente hasta validar el canal en producción y después se depreca para coordinación cotidiana; no debe convertirse en espejo permanente del chat.

## 9. Falta definir el criterio de cierre del MVP1.0

Los cinco bloques dicen qué construir, pero todavía no fijan de forma verificable dónde termina el MVP.

Propongo agregar una prueba integral de aceptación. MVP1.0 termina cuando Dani puede:

1. configurar GitHub App, Google Drive, OpenAI y Anthropic;
2. crear un proyecto `software` y uno `narrative`, o al menos probar ambos tipos;
3. crear o vincular sus recursos y recuperarse de un fallo parcial;
4. entrar a la sala del proyecto y conversar con Papu, Chapu y ambos secuencialmente;
5. recargar/reiniciar y conservar proyecto, recursos, sesión e historial;
6. comprobar que cada agente recibe identidad y contexto correctos;
7. leer recursos autorizados desde la sala;
8. autorizar una escritura de Papu en gobernanza y una de Chapu en Drive/código;
9. comprobar que una escritura no permitida de Papu en código o Drive es rechazada por backend;
10. promover un contenido aprobado a objeto formal y verificar su materialización en gobernanza;
11. recibir enlaces/referencias exactas de las operaciones realizadas;
12. crear un segundo proyecto y comprobar ausencia de contaminación de contexto/recursos;
13. recuperar de forma visible errores de proveedor, credencial, timeout y recurso sin corromper estado.

Cuando ese recorrido funciona con proveedores y recursos reales, el MVP1.0 está cerrado.

## 10. Orden de implementación recomendado

Mantendría la lógica general, pero evitaría cinco bloques completamente seriales. Propongo estos hitos:

1. **Fundación segura de integraciones**: cifrado, GitHub App, Drive OAuth, claves LLM, validación y renovación.
2. **Proyectos y aprovisionamiento**: CRUD, tipos, crear/vincular recursos, idempotencia y recuperación parcial.
3. **Perfiles y política de agentes**: puede avanzar en paralelo con 1 y 2.
4. **Núcleo conversacional persistente**: sesiones, mensajes, turnos, floor lock, frontend de sala y adapters determinísticos de prueba.
5. **Contexto gobernado + proveedores reales**: `input_context_ref`, recuperación mínima, OpenAI/Anthropic y aislamiento por proyecto.
6. **Gateway de recursos y autorizaciones**: lectura/escritura tipada, matriz DEC-C11, confirmaciones y auditoría.
7. **Materialización durable y cierre operativo**: promoción a objetos formales, verificación, errores, reinicio, aislamiento y prueba integral.

Esto permite tareas pequeñas y verificables, con una cola Ready corta, sin redactar todo el backlog ejecutable de una vez.

## Conclusión

La frontera de MVP1.0 propuesta es correcta y está alineada con lo que Dani y yo definimos. Los cambios anteriores no amplían el producto hacia MVP2.0; hacen explícitas dependencias y condiciones necesarias para que MVP1.0 sea realmente operativo.

Solicito que ajustes DEC-C16 y me pases la nueva versión para una última revisión antes de que Dani la marque vigente. No generes todavía la primera code-task: primero cerremos C16 y el roadmap de hitos; después descomponemos el primer hito en tareas pequeñas y preparamos una cola `Ready` de dos o tres tareas.
