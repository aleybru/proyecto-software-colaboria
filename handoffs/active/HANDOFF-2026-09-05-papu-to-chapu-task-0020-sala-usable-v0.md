# HANDOFF — Propuesta TASK-0020: sala usable V0

**Fecha:** 2026-09-05
**De:** Papu
**Para:** Chapu
**Autoridad:** Dani
**Estado:** propuesta para contraste; **no redactar todavía `code-tasks/TASK-0020.md`**
**Refs:** DEC-C10, DEC-C16, TASK-0018, TASK-0019

## Motivo

Dani quiere cortar el ciclo de backend-only y empezar a **usar ColaborIA ya**, aunque la primera sala funcione todavía con adapters determinísticos. La prioridad de este corte no es conectar OpenAI/Anthropic todavía: es volver visible, operable y dogfoodeable el canal que ya existe para poder probar UX, persistencia, routing, floor lock, idempotencia y aislamiento desde la aplicación real.

Revisé `main` del repo de código antes de proponer:

- Angular hoy tiene rutas `projects`, `projects/new`, `projects/:id` y `health`; no existe `features/room`.
- `project-detail` ya es el punto natural de entrada desde un proyecto.
- El backend ya expone crear sesión, postear mensaje de usuario y leer transcript.
- `ConversationOrchestrationService` de TASK-0019 sigue interno, sin endpoint HTTP.
- `DeterministicFakeAgentAdapter` sigue siendo el adapter registrado para esta capa y produce respuestas fake determinísticas.
- TASK-0019 cerró routing `papu | chapu | both_sequential`, floor lock de ronda + turno, orden V0 `papu → chapu`, idempotencia de ronda y no-continuación tras fallo.

## Propuesta de TASK-0020

### Objetivo

Implementar el primer **vertical slice usable de la sala**:

`Angular → API → message → conversation_round → agent_turn(s) → transcript persistido`

Desde el detalle de un proyecto, Dani debe poder abrir la sala del proyecto, crear o reabrir una sesión, ver el transcript, elegir `Papu | Chapu | Ambos`, enviar un mensaje y ver el resultado persistido.

Mientras siga usando adapters determinísticos, la UI debe mostrar sin ambigüedad:

> **Modo de prueba — respuestas determinísticas; proveedores reales todavía no conectados a la sala.**

No debe parecer que esas respuestas ya provienen de Papu/Chapu reales vía proveedor.

## Backend mínimo

### 1. Listado de sesiones del proyecto

Agregar una lectura equivalente a:

`GET /projects/{projectId}/sessions`

Objetivo: permitir reabrir conversaciones persistidas después de reload/restart. En V0 no hacen falta título, búsqueda, rename ni delete; fecha/ID o metadata mínima alcanza.

Debe preservar aislamiento por `projectId`: un proyecto nunca lista ni revela sesiones de otro.

### 2. Operación de dominio única para “Enviar”

No quiero que Angular coordine:

1. `POST message`
2. después `StartRound`

como dos operaciones independientes. Esa separación deja al frontend a cargo de una invariante de dominio que pertenece al backend.

Propongo una operación conceptual:

`POST /projects/{projectId}/sessions/{sessionId}/rounds`

Body mínimo:

```json
{
  "clientMessageId": "...",
  "content": "...",
  "mode": "papu | chapu | both_sequential"
}
```

La operación debe encapsular persistir/reusar el mensaje de usuario y disparar/reusar la ronda correspondiente.

Respuesta tipada mínima a validar/refinar:

- `finished`
- `already_exists`
- `mode_conflict`
- `floor_busy`
- `agent_registry_unavailable`
- `agent_unavailable`
- `session_not_found`

más `roundId`, `roundStatus` y `detail` cuando corresponda.

### 3. Invariante de atomicidad del gesto “Enviar”

Este es uno de los puntos principales para tu contraste.

Si el floor está ocupado, una segunda acción de “Enviar” **no debería dejar un `message` nuevo persistido sin una ronda asociada**.

Mi propuesta conceptual:

- mensaje + claim inicial de ronda forman una única unidad de dominio;
- si la request no puede ganar el floor, no queda un mensaje nuevo huérfano;
- retry del mismo `clientMessageId` + mismo `mode` reutiliza/devuelve el envío/ronda existentes sin duplicar turns;
- mismo `clientMessageId` + otro `mode` produce conflicto tipado;
- **no** mantener una transacción DB abierta mientras corre el adapter;
- sólo el claim inicial debe quedar consistente/atómico; después la ejecución usa la orquestación y atomicidad ya probadas por TASK-0018/0019.

Revisá cuál es el refactor mínimo de `ConversationService` / `ConversationOrchestrationService` para conseguir esto sin duplicar routing, floor ni persistencia de turns.

### 4. Lifecycle al exponer orquestación a HTTP

Mientras el orquestador era interno, una cancelación HTTP no existía como problema real de producto. Al conectarlo al browser sí.

Una ronda que ganó el floor **no puede quedar `running` indefinidamente** porque Dani recargó/cerró la pestaña, se cortó la conexión o se canceló el request.

TASK-0020 debe introducir una política determinística y verificable para que una excepción/cancelación durante ejecución termine o recupere la ronda sin dejar el floor pegado. No prescribo scheduler/background worker si no hace falta; sí exijo el invariante.

Atacá especialmente si la ejecución HTTP síncrona alcanza con `try/finally` + transición terminal, o si el boundary correcto necesita separar claim/ejecución de otra manera para no acoplar lifetime del trabajo al request.

## Frontend Angular V0

Agregar `features/room` y una ruta de sala accesible desde `project-detail` mediante una acción clara tipo **“Abrir sala”**.

La sala V0 debe incluir:

- lista simple de sesiones del proyecto;
- “Nueva conversación”;
- selección/reapertura de sesión;
- transcript central con autor visible: `Dani`, `Papu`, `Chapu`;
- selector de destino `Papu | Chapu | Ambos (Papu → Chapu)`;
- textarea + `Enviar`;
- `clientMessageId` generado por Angular y conservado durante retries de la misma acción;
- bloqueo del envío actual mientras está in-flight;
- al terminar la request, **releer transcript desde DB/API** en vez de fabricar respuestas localmente;
- consultar `/agents` para disponibilidad del registro y explicar/deshabilitar cuando corresponda;
- banner visible de **modo de prueba determinístico**.

El orden fijo de `both_sequential` debe verse en la UI (`Ambos (Papu → Chapu)`), no quedar escondido.

### Estados UX mínimos y explícitos

- cargando sala/sesiones;
- procesando ronda;
- `floor_busy`;
- ronda `stopped`;
- AgentRegistry no disponible;
- agente no disponible/deprecated;
- error de red con retry conservando `clientMessageId`;
- sesión/proyecto no encontrados.

Nada de “algo salió mal” como única información cuando el backend ya tiene un resultado tipado.

Si Papu falla en `both_sequential`, la UI no debe fabricar ni esperar una respuesta inexistente de Chapu.

## Aceptación real / dogfooding

La tarea no se considera cerrada sólo por unit tests. Debe hacerse una verificación real con backend + Angular + Postgres:

1. abrir el proyecto ColaborIA desde la UI;
2. entrar a la sala;
3. crear una sesión;
4. enviar al menos un mensaje a Papu, uno a Chapu y uno a Ambos;
5. confirmar respuestas determinísticas y autoría correcta;
6. recargar browser y comprobar historial;
7. reiniciar backend y comprobar persistencia;
8. abrir segunda pestaña sobre la misma sesión y forzar solapamiento/floor conflict sin mensajes ni rondas duplicadas/huérfanas;
9. abrir otro proyecto y comprobar que no puede ver la sesión/historial de ColaborIA.

Tests automáticos esperados:

- backend: endpoint de send, idempotencia, mode conflict, floor busy sin mensaje huérfano, aislamiento y cancelación/excepción;
- frontend: ruta, listado/cambio de sesión, modos, retry con mismo `clientMessageId`, estados tipados y reload de transcript;
- sin regresiones sobre los **141/141** tests backend existentes y la suite frontend vigente.

## Fuera de TASK-0020

- OpenAI/Anthropic reales;
- Context Builder real / `input_context_ref` reconstructible;
- streaming / SSE / WebSocket;
- tools / attachments;
- approvals / gateway;
- objetos formales (Decision/Task/Result/etc.);
- títulos, búsqueda, rename o delete de sesiones;
- `both_blind`;
- orden configurable de agentes;
- implementación del UI/UX Engine de DEC-C17;
- diseño visual final.

Mi posición es **no absorber providers reales + Context Builder en esta misma task**. Primero hacemos visible y testeable todo el canal con fuente determinística; la siguiente tarea reemplaza el adapter y agrega contexto sobre una sala ya operativa. Eso permite separar fallas de canal/UI de fallas de provider/contexto.

## Contraste solicitado a Chapu

Revisá esta propuesta contra DEC-C10/DEC-C16 y el código real, y atacá especialmente:

1. **Atomicidad/shape** de `Enviar → message + claim de ronda`: ¿la invariante propuesta es correcta y cuál es el boundary mínimo?
2. **Cancelación/lifecycle HTTP:** ¿qué mecanismo mínimo evita rondas `running`/floor pegado sin meter infraestructura prematura?
3. **Listar/reabrir sesiones:** ¿es mínimo necesario para una sala realmente usable o hay scope que conviene sacar?
4. **Providers reales:** ¿existe una razón técnica fuerte para incluirlos en TASK-0020, o conviene mantenerlos fuera como propongo?
5. Cualquier contradicción real con lo ya aprobado o con la implementación actual que deba resolverse antes de redactar la tarjeta.

Si no aparece una Decision nueva real, respondé con contraste y recomendación. **No redactes todavía `TASK-0020.md`**: después del contraste Dani cierra el corte y te da el go explícito para crearla.