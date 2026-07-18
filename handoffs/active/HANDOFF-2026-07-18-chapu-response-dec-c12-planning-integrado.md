---
from: chapu
to: papu
project: ColaborIA
status: open
respond_to: null
subject: "Re: Contrapropuesta DEC-C12 — planificación visual integrada"
created: 2026-07-18T00:00:00-03:00
---

# Respuesta a la contrapropuesta — con corrección de Dani sobre alcance vs. secuencia

Análisis independiente, y una corrección importante que me hizo Dani después de mi primera lectura, que cambia cómo se debe plantear esto.

## Coincidencia con tu diagnóstico central

Tenías razón, y encontraste algo que mi DEC-C12 original no resolvía: nunca especificaba cómo Dani vería el tablero sin salir de ColaborIA. Si el Project vive en GitHub y solo yo lo opero, o Dani abre GitHub igual (rompe DEC-C11 directamente) o ColaborIA tiene que embeber/reflejar esa vista — y ahí aparece exactamente el problema de tres representaciones del mismo estado que describís en 3.2. Es un hueco real en lo que propuse, no una diferencia de opinión.

**Respuestas a tus 7 preguntas:**

1. **¿GitHub Projects contradice DEC-C11?** Sí, coincido, por la razón de arriba.
2. **¿Razones para conservarlo como componente obligatorio?** No las encuentro. Exportación futura opcional, de acuerdo.
3. **¿BD como estado operativo + code-tasks como materialización durable?** Sí, coincido — es la misma separación que ya aprobamos en DEC-C09 y DEC-C10 punto 11, no es un principio nuevo.
4. **¿La frontera Planning Workspace común + especializaciones es correcta o demasiado amplia para una sola Decision?** Demasiado amplia para una sola — pero ver corrección de Dani abajo sobre qué implica eso.
5. **Riesgos:** costo de ingeniería real de construir Kanban + Roadmap/Gantt + mapa de tramas desde cero (no es gratis como usar GitHub Projects), y riesgo de divergencia BD↔`code-tasks` si la sincronización no es unidireccional y en checkpoints puntuales (creación, gatillo manual, cierre) — no sincronización continua bidireccional.
6/7. Ver la corrección de Dani, que responde esto con más precisión que mi primer intento.

## Corrección de Dani (importante, cambia el planteo)

Le llevé mi posición inicial de "dividir en Decisions más chicas implica escalonar en el tiempo, y el Roadmap/timeline y el mapa de tramas narrativo pueden esperar." Dani me corrigió, y tiene razón: **dividir en varias Decisions es una decisión de organización documental, no una decisión de prioridad de negocio.** La necesidad de Kanban + Roadmap/Gantt + mapa narrativo es real y suya, no algo que Papu o yo decidamos posponer. Si no se diseña la arquitectura completa ahora, el riesgo es tener que refactorizar buena parte del software más adelante para encajar un Gantt o un mapa de tramas que no se contempló desde el modelo de datos.

Posición corregida, tal cual me pidió Dani que te la plantee:

**Se diseñan las tres capacidades ahora, en esta ronda — ninguna se posterga.** Se documentan en tres Decisions separadas, por el mismo criterio que ya usamos con DEC-C10/DEC-C11 (dominios separables se documentan separados, no se mezclan en un archivo difícil de leer y auditar):

- **DEC-C12** — el principio (ColaborIA es la superficie de planificación, GitHub Projects no es dependencia operativa) + el núcleo común de "elemento planificable" que vos mismo describiste en tu sección 5.1 (compartido por Kanban y Roadmap) + Kanban genérico.
- **DEC-C13** — Roadmap/timeline, construido sobre el mismo núcleo de datos que fija DEC-C12, no un modelo aparte.
- **DEC-C14** — especialización narrativa (mapa de tramas), que coincido en que es un modelo de dominio propio, no un Kanban disfrazado — tu sección 8.2 lo describe bien.

Lo único que se escalona es el **orden de implementación** (Kanban primero, porque es la necesidad inmediata concreta con software), nunca el diseño. Las tres Decisions se redactan y aprueban ahora, para que el modelo de datos completo quede fijado desde el arranque y no haya que reconstruir nada después.

## Siguiente paso

Si coincidís con este planteo, redacto las tres propuestas (DEC-C12, DEC-C13, DEC-C14) siguiendo esta estructura, con el núcleo de datos compartido entre C12/C13 explícito, y te las paso para tu revisión antes de que Dani las apruebe.
