# DEC-C17 — UI/UX Engine: generación de mockups funcionales validables

**Estado:** Decision vigente — aprobada por Dani el 2026-07-19. Decision **conceptual**: no tiene etapa, versión ni compromiso de roadmap asignado. Queda diseñada para no forzar refactor cuando se decida implementarla (mismo criterio que DEC-C12 a DEC-C15).
**Origen:** propuesta de Papu (`handoffs/active/HANDOFF-2026-07-19-papu-to-chapu-ideas-explorer-uiux-engines.md`), precisión de alcance extensa entre Dani y Chapu tras revisar capturas de Visily (`handoffs/active/HANDOFF-2026-07-19-chapu-precision-uiux-engine.md`), 4 precisiones finales de Papu incorporadas antes de aprobar.
**Relacionada con:** DEC-C11 (patrón de agente/adapter), DEC-C16 (fuera del alcance de MVP1.0), DEC-C18 (Explorer Engine — capacidad hermana, misma familia).

## Resumen

Capacidad para generar **mockups funcionales** (HTML/CSS/JS reales, no imágenes estáticas) de un caso de uso ya definido en conversación entre Dani, Papu y Chapu, con navegación e interacción real, para que Dani los valide y apruebe **antes** de que se programe el backend o el frontend definitivo. El objetivo es eliminar la incertidumbre de diseño antes de invertir en desarrollo, no generar la aplicación final automáticamente.

## Decisión

### 1. Qué NO es

No es un motor de renderizado propio construido desde cero, ni pretende competir con herramientas como Figma o Visily en generalidad. Es generación mediante LLM con rol especializado — la misma familia de capacidad que Chapu ya demostró en esta misma conversación (mockups del Kanban, Roadmap y sala de trabajo, con interacción real), aplicada de forma sistemática y con infraestructura propia.

### 2. Componentes

- **Sistema de diseño propio**: paleta, tipografía, componentes base — asegura consistencia visual entre pantallas generadas en momentos distintos.
- **Adapter especializado**: invocado por el orquestador (DEC-C11), configuración análoga a `config/agents/papu.yml`/`chapu.yml` — por ejemplo `config/agents/uiux_engine.yml` — con instrucciones de sistema de diseño y buenas prácticas de UX/UI modernas. No es literalmente Claude Code (la herramienta de terminal usada para code-tasks); es un rol de API con el mismo patrón de invocación que los demás agentes. Claude Code puede entrar más adelante como motor de ejecución si hace falta escribir archivos reales, pero es detalle de implementación, no de esta Decision.
- **Design Result**: objeto durable y versionado — el HTML/CSS/JS generado es un artefacto, **no la única fuente de verdad**. El Design Result estructura pantallas, componentes, tokens de diseño e interacciones de forma que puedan auditarse y reutilizarse, no solo "el código que quedó".
- **Canvas navegable en sandbox**: vista dentro de ColaborIA donde Dani recorre, prueba interacción y aprueba. **Corre en sandbox, sin acceso libre a backend real, credenciales ni integraciones** — un prototipo de validación no debe poder tocar nada sensible ni operar sobre datos reales.

### 3. Qué produce el engine

Mockups funcionales con navegación entre pantallas, botones que responden, transiciones y efectos ya definitivos (colores, fuentes, animaciones) — maquetación y UX/UI resueltas y aprobadas. **Sin conexión a datos reales ni lógica de negocio.**

### 4. Relación con la implementación real (Angular)

Etapa separada, no automática. El frontend Angular toma el Design Result aprobado como referencia exacta y lo reimplementa como componentes reutilizables con datos reales. Debe verificarse **conformidad visual y funcional** respecto del diseño aprobado — no alcanza con que "se vea parecido".

### 5. Visual Assets (boundary, no diseñado en esta Decision)

Logos, imágenes e ilustraciones quedan como **capacidad auxiliar separada**, conceptualmente distinta del motor UI/UX (que resuelve maquetación e interacción, no generación de arte). No se diseña en esta Decision.

### 6. Flujo de caso de uso

Conversación en la sala → caso de uso definido → engine genera mockup(s) según lo que Papu/Chapu indicaron → Dani navega y prueba en el Canvas (sandbox) → iteración hasta aprobar → Design Result queda como referencia definitiva → Angular lo traduce a componentes con datos reales, con verificación de conformidad → caso de uso cerrado.

## No objetivos de esta Decision

- Implementación — no forma parte de MVP1.0 (DEC-C16).
- Esquema físico de Postgres para almacenar Design Results.
- Diseño de la capacidad de Visual Assets (sección 5).
- Elección de tecnología concreta de sandbox para el Canvas.

## Pendiente

- Especificación arquitectónica, cuando se decida implementar.
- Definición de Visual Assets como capacidad propia (fuera de esta Decision).
