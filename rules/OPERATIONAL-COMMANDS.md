# OPERATIONAL-COMMANDS.md — Comandos operativos de ColaborIA

Estos comandos son la aplicación concreta, en este proyecto, de los procedimientos de QC definidos en `PROTOCOLO-OPERATIVO-MULTIPROYECTO-v1.md` secciones 9.4 y 9.5. No son un protocolo distinto, son atajos de lenguaje natural que Dani usa para invocarlos.

## "Leé QC" / "Revisá QC" / "Mirá QC"
Chapu:
1. identifica el proyecto activo (ColaborIA);
2. abre el repo de gobernanza (`aleybru/proyecto-software-colaboria`);
3. lee `handoffs/quick/log.md` directamente vía bash/API — nunca espera que le peguen el contenido;
4. identifica el último mensaje, origen y destino;
5. identifica qué pide;
6. abre la fuente referenciada si existe;
7. responde a Dani con síntesis fiel;
8. no escribe en QC salvo pedido explícito.

## "Contestale por QC"
Chapu:
1. lee la versión más reciente de `handoffs/quick/log.md`;
2. obtiene el SHA actual del archivo;
3. redacta un mensaje breve;
4. lo agrega al final (append-only);
5. no modifica mensajes anteriores;
6. confirma éxito solo si la escritura ocurrió realmente (verificada, no asumida).

**Reconstruido:** sí, 2026-07-08 — contenido re-derivado directamente de las secciones 9.4/9.5 del protocolo común (fuente exacta disponible), no de memoria libre. Confiabilidad alta.
