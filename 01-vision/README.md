# Visión y leyes del proyecto

## Qué es MexOrbit (nombre temporal)

Un MMO espacial top-down en tiempo real que rescata lo que hizo grande al género en su era dorada — el estatus visible, los clanes, el respeto al tiempo del jugador, la progresión que se siente — y le quita todo lo que lo mató: el pay-to-win, el azar opaco, el tedio monetizado y el treadmill que dejaba atrás a todos. Diseñado para **converger**: un server donde 200 jugadores se sienten un mundo vivo y donde la promesa "algún día llego" es matemáticamente verdadera.

## La ley suprema: el invariante de convergencia

> **Todo jugador casual alcanza el full de la temporada en ~4.5–5 de sus 6 meses; el tryhard en ~2–2.5.** Nadie termina una temporada atrasado por diseño. El contenido nuevo de cada ciclo jamás supera lo que el jugador gratuito dedicado puede ganar en ese ciclo.

DarkOrbit murió por violar esto todos los años (la demostración con números: [la cuenta final del retail](../02-investigacion/economia-darkorbit-2010-retail.md), §17). Cualquier propuesta que rompa el invariante se rechaza sin discusión adicional.

## Las leyes derivadas (los anti-patrones prohibidos)

Cada una nace de un anti-patrón documentado del retail (los 11 de la [autopsia](../02-investigacion/economia-darkorbit-2010-retail.md), §18):

1. **El dinero jamás compra poder** — compra tiempo ajeno (Starbond → Credits → mercado de jugadores), conveniencia y cosmética. Ningún verbo, ningún stat, ningún techo se vende.
2. **Cero azar opaco** — toda probabilidad es publicada, todo objetivo tiene pity visible ("me faltan 3"). Sin loot boxes: además de tóxicas, son ilegales o inviables en mercados clave (UE).
3. **El tedio se elimina, no se monetiza** — si un jugador pagaría por no hacer algo, ese algo se rediseña o se automatiza gratis.
4. **Cada tier es una parada real o una decisión** — cero tiers señuelo, cero precios ancla para empujar al premium.
5. **La inversión del jugador nunca se invalida** — lo nuevo se construye sobre lo ganado ("tu Quasar se mejora, no se tira").
6. **El tiempo se respeta por contrato** — sesiones de 20 minutos valen la pena, sin decay, sin FOMO, sin castigos a los desconectados.
7. **Sumideros voluntarios antes que punitivos** — el jugador paga con gusto lo que compra poder ahora (munición, recubrimientos); los castigos son mínimos y comprensibles (taller, carga).
8. **Cero mecánicas idle** — todo se gana volando, peleando o comerciando. Sin timers de producción, sin fábricas.

## Convenciones del proyecto

- **Documentación en español.** Los nombres propios del juego (Asterium, Quasar, Orion, Vex…) no se traducen en ningún idioma.
- **Código en inglés, comentarios en español** — en todos los repos.
- **"MexOrbit" y `mex-orbit` son el nombre de trabajo** en repos, namespaces y paquetes; el nombre comercial se decidirá después (nada de logos definitivos ni branding profundo todavía).
- **Papel antes que código**: documento de diseño → revisión → implementación.
- Las cinco familias de nombres del juego (materiales = elementos hipotéticos · legendarios = fenómenos extremos · equipo = códigos funcionales · naves = constelaciones · aliens = taxonomía en latín oscuro) están definidas en los [Guidelines §11](../03-guidelines/guidelines-generales.md).
