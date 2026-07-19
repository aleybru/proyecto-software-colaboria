# DEC-C18 — Explorer Engine: exploración estructurada de ideas antes de comprometer desarrollo

**Estado:** Decision vigente — aprobada por Dani el 2026-07-19. Decision **conceptual**: no tiene etapa, versión ni compromiso de roadmap asignado.
**Origen:** propuesta de Papu (`handoffs/active/HANDOFF-2026-07-19-papu-to-chapu-ideas-explorer-uiux-engines.md`), sin objeciones de Chapu ni ajustes de alcance — a diferencia de DEC-C17, esta se aprueba conforme a la propuesta original.
**Relacionada con:** DEC-C11 (patrón de agente/adapter), DEC-C16 (fuera del alcance de MVP1.0), DEC-C17 (UI/UX Engine — capacidad hermana, misma familia, boundaries propios).

## Resumen

Formaliza y potencia como capacidad nativa de ColaborIA algo que hoy se hace manualmente: cuando Dani trae una idea todavía inmadura, explorarla de forma estructurada, trazable y crítica — precisar el problema, investigar referencias/competidores, evaluar viabilidad, identificar riesgos — antes de convertirla en proyecto o comprometer desarrollo.

## Decisión

### 1. Propósito

Recibir una idea inmadura y producir una exploración estructurada antes de que se convierta en proyecto o se comprometa desarrollo.

### 2. Capacidades

- Precisar la idea, el problema y sus variantes.
- Identificar usuarios y necesidad potencial.
- Investigar productos, competidores y alternativas existentes.
- Explorar mercado y contexto público disponible.
- Evaluar viabilidad técnica, operativa y económica.
- Identificar supuestos, dependencias, costos y riesgos.
- Distinguir evidencia, inferencia e incertidumbre.
- Producir pros, contras y escenarios alternativos.
- Recomendar explorar más, reformular, avanzar o descartar — **sin quitarle a Dani la autoridad de decisión**. El engine evalúa y produce recomendaciones; el go/no-go final siempre es de Dani.

### 3. Resultado esperado: Exploration Result

Objeto durable y estructurado, con fuentes, nivel de confianza, contradicciones, preguntas abiertas y recomendación argumentada. **No es automáticamente una Decision ni crea un proyecto** — es insumo para que Dani decida.

### 4. Relación con los agentes

El engine puede orquestar análisis complementarios de Papu y Chapu, búsquedas web, y evaluadores especializados. La diversidad de perspectiva importa — no debe reducirse a un único prompt que entrega una conclusión aparentemente definitiva y cerrada.

### 5. Infraestructura compartida con DEC-C17

Ambos engines (este y UI/UX Engine) comparten infraestructura potencial: Project Orchestrator, Context Builder, Source Registry, workflows multiagente, presupuestos y límites, objetos Result/Proposal/Decision, auditoría y versionado, adapters de herramientas y modelos. Conservan boundaries propios — no se fusionan en un "motor universal" que mezcle investigación, diseño, assets y ejecución en una caja negra.

## No objetivos de esta Decision

- Implementación — no forma parte de MVP1.0 (DEC-C16).
- Esquema físico de Postgres para el Exploration Result.
- Definición de presupuesto/límites concretos de búsqueda o inferencia por exploración.

## Pendiente

- Especificación arquitectónica, cuando se decida implementar.
- Definir criterio de calidad/suficiencia de evidencia antes de producir una recomendación (para evitar conclusiones con apariencia de certeza sobre evidencia débil).
