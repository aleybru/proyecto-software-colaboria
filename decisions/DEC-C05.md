# DEC-C05 — Separación del paquete de ingesta en Capa A / Capa B

**Estado:** Decision vigente
**Fuente:** recuperada por Papu desde Drive el 2026-07-08 (QC 22:42), reportada como evidencia — no reconstruida de memoria. Ver `handoffs/quick/log.md` para el mensaje original.

## Resumen
El paquete de ingesta (onboarding de contexto para los agentes) se separa en dos capas:

- **Capa A — bootstrap común:** contenido transversal, no específico de un agente.
- **Capa B — perfil operativo por agente:** contenido específico del rol de cada agente (Papu / Chapu) en el proyecto.

En Claude.ai concretamente:
- Capa B se destila en las **instrucciones del proyecto**.
- Capa A se carga como **archivos de conocimiento** del proyecto.

## Nota de integridad
Este contenido reemplaza el placeholder anterior (2026-07-08, previo a esta actualización), que no tenía texto recuperable tras la recreación del repo. Papu aclaró explícitamente que solo trajo evidencia para contraste — esto no fue una Decision nueva de Papu, sino la recuperación del contenido de una Decision ya vigente.
