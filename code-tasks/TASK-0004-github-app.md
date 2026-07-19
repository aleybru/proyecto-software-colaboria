---
id: TASK-0004
status: pending
repo_destino: aleybru/proyecto-software-colaboria-codigo
creado: 2026-07-19
aprobado_por: Dani
depende_de: [TASK-0003]
decision_ref: DEC-C09, DEC-C16
---

## Objetivo

Integrar GitHub App como mecanismo de autenticación de la aplicación contra GitHub, conforme a la dirección ya fijada en DEC-C09 (no PAT manual) y precisada en DEC-C16 Bloque A (tokens efímeros generados bajo demanda, no guardados como credencial durable).

## Qué tiene que hacer Code (concreto)

1. Guardar, vía el servicio de credenciales de TASK-0003 (`provider: github_app`), la configuración durable de la App: App ID y clave privada (o referencia segura a ella, según lo que permita el flujo elegido) — **esto sí es durable y se cifra**.

2. Implementar la generación de **installation access tokens** bajo demanda:
   - el token efímero se genera en el momento en que se necesita (ej. al ejecutar una operación contra un repo), usando la App ID + clave privada guardadas;
   - **no se guarda el installation access token en la tabla `credentials` ni en ningún otro lugar persistente** — vive solo en memoria durante la operación que lo usa.

3. Endpoint de prueba de conexión: dado que la App está configurada, verificar que puede autenticarse y listar al menos las instalaciones/repos accesibles.

4. Estado visible de la integración: `no configurado` / `configurado, sin verificar` / `conectado` / `error` (con el detalle del error si lo hay, sin exponer la clave privada en el mensaje).

5. Documentar en `## Resultado` los pasos manuales que Dani tiene que haber hecho en GitHub antes de que esto funcione (registro de la App, instalación, permisos otorgados) — es un paso de configuración externa que esta tarea no automatiza, solo consume.

## Criterio de éxito

- Con la App ya registrada e instalada por Dani (paso manual previo, fuera del alcance de esta tarea), el endpoint de prueba de conexión confirma autenticación exitosa contra la API de GitHub.
- Se puede generar un installation access token válido bajo demanda y usarlo para una operación real de lectura contra un repo autorizado (ej. listar contenido de la raíz).
- El installation access token generado no aparece en ninguna tabla de la base de datos ni en logs.
- Si la App no está configurada o la clave es inválida, el estado de la integración lo refleja claramente, sin que la aplicación falle de forma opaca.

## Contexto / restricciones

- No reemplaza el token que Dani usa manualmente hoy para que Chapu opere el repo de gobernanza durante esta etapa conversacional — eso sigue siendo aparte, sin cambios, hasta que el flujo completo de MVP1.0 esté operativo.
- No implementar todavía operaciones de escritura reales contra repos (eso corresponde al Bloque E de DEC-C16, gateway de recursos tipado) — esta tarea es solo la autenticación y una prueba de lectura.
- Respetar DEC-C08: detalle de implementación del flujo de firma JWT para la App (librería específica, etc.) queda a criterio de Code.
