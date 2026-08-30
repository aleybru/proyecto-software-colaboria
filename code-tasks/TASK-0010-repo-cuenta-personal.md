---
id: TASK-0010
status: cancelled
repo_destino: aleybru/proyecto-software-colaboria-codigo
creado: 2026-08-15
aprobado_por: Dani
depende_de: [TASK-0008]
decision_ref: DEC-C09, DEC-C16
motivo_cancelacion: superseded
---

**Cancelada el 2026-08-30 — motivo: `superseded`.**

El objetivo de esta tarea (resolver la creación de repos de GitHub en cuenta personal) quedó finalmente resuelto por una implementación posterior, ejecutada fuera de este mecanismo formal y reconstruida retrospectivamente como **`TASK-0013`** (ver `code-tasks/TASK-0013.md`), mediante GitHub App **user authorization** (`POST /user/repos` con user access token, no con installation token) — un camino distinto y más simple que los tres que esta tarea proponía probar en orden.

Se conserva este archivo sin modificar su contenido original (abajo) por trazabilidad histórica — es la tarea tal como se aprobó y quedó pendiente, nunca llegó a dispararse en Claude Code.

---


## Objetivo

Resolver el bloqueo encontrado en TASK-0008: `CreateRepositoryAsync` hoy solo prueba `POST /orgs/{owner}/repos` con installation token, y falla de entrada si la cuenta de destino es personal (`User`), como es el caso de `aleybru`. Esta tarea prueba, en orden, tres caminos alternativos antes de resignarse a que la única opción sea crear el repo a mano y vincularlo (TASK-0007).

## Contexto de la investigación previa (no repetir, partir de acá)

- El código actual (rama ya mergeada de TASK-0008) detecta `installation.Account.Type != "Organization"` y devuelve error explícito sin intentar nada más.
- La documentación oficial de GitHub y los reportes de la comunidad son contradictorios sobre si `POST /user/repos` acepta installation token directamente. No hay respuesta confiable sin probarlo contra la cuenta real.
- Contraste completo entre Papu y Chapu disponible en `handoffs/quick/log.md`, entradas del 2026-08-15 sobre este tema, y en el `## Resultado` de `code-tasks/TASK-0008-aprovisionamiento-recursos.md`.

## Qué tiene que hacer Code (concreto)

Probar, **en este orden**, deteniéndose en el primero que funcione:

1. **`POST /user/repos` con installation token** (mismo cliente/token que ya usa el código para `/orgs/{org}/repos`, cambio mínimo). Si la cuenta de la instalación es personal, intentar este endpoint antes de devolver error.

2. **Si falla (1):** probar `POST /repos/{template_owner}/{template_repo}/generate` con installation token, usando un repo plantilla mínimo (crear uno vacío si hace falta para la prueba, documentar cuál se usó).

3. **Si falla (2):** implementar generación de **user access token** (user-to-server) siguiendo el flujo que documenta GitHub para GitHub Apps, y usar ese token específicamente para la creación del repo personal. Las operaciones posteriores sobre ese repo siguen usando installation token normalmente — el user access token es solo para este paso puntual.

4. **Si las tres fallan**, no inventar una cuarta alternativa por cuenta propia — dejar el error documentado con el detalle exacto de cada intento (código de estado, mensaje de GitHub) en `## Resultado`, y el camino de creación manual + vínculo (ya soportado por TASK-0007) queda como el mecanismo válido para cuentas personales, sin bloquear nada más del sistema.

En cualquiera de los casos donde algo funcione, actualizar `CreateRepositoryAsync` para que la cuenta personal use el camino que resultó exitoso, dejando el camino de organización intacto tal como está.

## Criterio de éxito

- Se registra explícitamente, con evidencia real (no simulada), cuál de los tres caminos funcionó — o que ninguno funcionó, con el detalle exacto de cada fallo.
- Si alguno funcionó: crear un repo de prueba real en la cuenta `aleybru` usando el nuevo camino, confirmar que existe, y limpiar el repo de prueba al finalizar (o dejarlo documentado si no corresponde borrarlo sin permiso — mismo criterio que Code ya aplicó con la carpeta de Drive de TASK-0008).
- La ruta de organización (`/orgs/{owner}/repos`) sigue funcionando sin cambios — no se toca lo que ya andaba.
- El `## Resultado` deja trazabilidad completa: qué se probó, en qué orden, con qué resultado exacto de la API de GitHub en cada intento.

## Contexto / restricciones

- No crear una organización de GitHub como solución — eso fue descartado explícitamente por Papu y Dani salvo que los tres caminos de esta tarea fallen y se abra una nueva conversación al respecto.
- No volver a PAT manual — descartado por la misma razón.
- No modificar DEC-C09, DEC-C10 ni DEC-C16 — esta tarea es iteración técnica dentro de lo ya aprobado, no cambio de arquitectura.
- Si el paso 3 (user access token) requiere algún dato de configuración adicional que la App no tiene todavía (ej. habilitar un flujo de autorización de usuario en la configuración de la GitHub App), documentarlo como paso manual pendiente de Dani en `## Resultado`, igual que se hizo con el bootstrapping de la App en TASK-0004.
