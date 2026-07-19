---
id: TASK-0003
status: pending
repo_destino: aleybru/proyecto-software-colaboria-codigo
creado: 2026-07-19
aprobado_por: Dani
depende_de: []
decision_ref: DEC-C09, DEC-C10, DEC-C16
---

## Objetivo

Implementar la infraestructura de cifrado y la tabla `credentials` que el resto del Bloque A de DEC-C16 (GitHub App, Drive OAuth, API keys de LLM) va a necesitar. Sin esta tarea, ninguna de las siguientes (TASK-0004, 0005, 0006) puede ejecutarse.

## Qué tiene que hacer Code (concreto)

1. En `backend/ColaborIA.Api`, agregar un servicio de cifrado reversible (simétrico, ej. AES) para el payload de credenciales:
   - la clave de cifrado se lee de una variable de entorno (o user-secrets en desarrollo), **nunca hardcodeada ni versionada en el repo**;
   - el servicio expone como mínimo `Encrypt(string plaintext) -> byte[]` y `Decrypt(byte[] ciphertext) -> string`.

2. Agregar la entidad y `DbSet` de `credentials` con el schema ya definido en DEC-C09/DEC-C10 (con los ajustes de timestamps de DEC-C09):
   - `id`
   - `provider` (ej. `github_app`, `google_drive`, `anthropic`, `openai` — usar un tipo que permita agregar proveedores futuros sin migración, ej. string con validación, no enum rígido de EF Core)
   - `kind`
   - `scope_type` (`global` | `project`)
   - `project_id` (FK nullable a `projects` — null cuando `scope_type = global`)
   - payload cifrado (tipo binario, ej. `bytea` en Postgres)
   - `key_version` (para poder rotar la clave de cifrado en el futuro sin invalidar credenciales existentes)
   - `status` (ej. `active`, `invalid`, `revoked`)
   - `created_at`, `updated_at`
   - `CHECK` constraint: `project_id` obligatorio si `scope_type = 'project'`, prohibido si `scope_type = 'global'` (mismo patrón que Code ya usó en TASK-0001 para esta misma tabla — reutilizar el criterio).

3. Generar la migración de EF Core correspondiente.

4. Endpoints mínimos (sin UI todavía, solo API):
   - crear/reemplazar una credencial (recibe el valor en texto plano, lo cifra antes de guardar, nunca lo devuelve en la respuesta);
   - listar credenciales por proyecto o globales, **sin exponer el payload** — devolver `id`, `provider`, `kind`, `scope_type`, `status`, timestamps, y un indicador enmascarado (ej. últimos 4 caracteres si aplica, o simplemente "configurada"/"no configurada");
   - revocar una credencial (cambia `status`, no borra el registro).

5. Verificar explícitamente que ningún log de la aplicación (ni de EF Core en modo desarrollo) imprime el valor en texto plano de una credencial — mismo criterio que Code ya aplicó en TASK-0001.

## Criterio de éxito

- La migración se aplica correctamente contra Postgres.
- Un valor guardado vía el endpoint de creación puede recuperarse descifrado internamente (verificable con una prueba o script, no expuesto por API) y coincide exactamente con el valor original.
- El endpoint de listado nunca devuelve el payload en texto plano ni cifrado crudo.
- Los logs de la aplicación, revisados durante una prueba de creación/lectura, no contienen el valor en texto plano de ninguna credencial.
- El `CHECK` constraint rechaza una fila con `scope_type = 'project'` sin `project_id`, y una con `scope_type = 'global'` con `project_id` no nulo.

## Contexto / restricciones

- No implementar todavía la integración real con GitHub App, Drive ni las API keys de LLM — eso es TASK-0004, 0005 y 0006, que dependen de esta.
- No es necesario resolver todavía renovación automática de tokens (eso es específico de cada integración y se resuelve en su propia tarea).
- Respetar DEC-C08: decisiones técnicas menores (ej. algoritmo exacto de cifrado dentro de lo que ofrezca el ecosistema .NET estándar) quedan a criterio de Code y se documentan en `## Resultado`; no se cambia el schema de `credentials` ya fijado en DEC-C09/DEC-C10 sin señalarlo explícitamente.
