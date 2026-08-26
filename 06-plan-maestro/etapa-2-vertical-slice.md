# Etapa 2 — El vertical slice (plan de etapa)

**Estado: en progreso** (arrancada 2026-08-25, con E1 cerrada). El loop completo, a través de todos los repos:

> **login → conectar → volar → matar un Vex → recoger su carga → volver a base → refinado automático → almacén → vender al NPC**

**Gate:** el slice jugable end-to-end en dev, con reconexión limpia y sin ninguna pieza legada en el camino.

## Lo que E1 y el trabajo previo ya dejaron listo

- Contrato con codegen propio probado (roundtrip byte-exacto C#↔GDScript) — falta extenderlo del spike (3 mensajes) al catálogo completo.
- BD `mexorbit_dev` viva en MySQL 8.4.11 local (3307) con el esquema y las semillas del slice.
- Diseño de auth validado (ticket Ed25519, sesión única, reconexión).
- Arte del slice completo (Phoenix, Vex, estación, fondo, efectos, props) en `mex-orbit-art`.
- Dirección de UI congelada (sistema de diseño N + skill `mexorbit-ui`).

## Iteraciones

Cada una termina en algo observable; el orden respeta las dependencias.

1. ✅ **I1 — El contrato completo** (2026-08-25): `schema/messages.yaml` con **37 mensajes + 2 structs** (sesión, mundo, combate, loot/economía, chat); el generador aprende `repeated`, submensajes, `bool` y `sint` zigzag, y valida el esquema (ids/tags únicos, nombres reservados — `Error` colisionaba con el tipo nativo de Godot y se renombró `ErrorReply`). Roundtrip byte-exacto C#↔GDScript verificado incluyendo `CollectResult` con drops repetidos y `StorageDelta` con delta negativo.
2. ✅ **I2 — La api mínima** (2026-08-25): ASP.NET minimal api sobre `mexorbit_dev`; login con Argon2id, session token opaco (solo hash), game ticket JWT Ed25519 de 60 s, auditoría y rate limit; `seed` idempotente (odrack + testbot con Phoenix equipada en 1-1); registro tras `registration_open`. Probado por curl: ticket VALIDO verificado por firma, 401 + auditoría con password malo, 403 con registro cerrado.
3. ✅ **I3 — El game server mínimo** (2026-08-25): ws en 5200 (TLS = infra en prod), handshake Hello con validación Ed25519 del ticket (jti un solo uso), **sesión única demostrada** (doble login → `REPLACED` en BD y `SessionReplaced` al viejo), heartbeat ping/pong, tick fijo 80 ms blindado (la lección del TickManager), mapa y 15 Vex cargados de BD, `MoveIntent`→seq→clamp→**eco autoritativo** y write-behind de `player_ship_state` verificado. Observable cumplido: el cliente de consola (`tools/console-client`) vuela por el 1-1 — login api → Welcome → EnterMap → 16 spawns → 3 intents con ecos avanzando.
4. ✅ **I4 — El cliente nuevo** (2026-08-25): proyecto Godot 4 limpio con los tokens N (`n_theme.gd`, fuentes Michroma/Exo 2/JetBrains incluidas), login con la secuencia real, mundo con el fondo del pipeline escalado a los límites de `EnterMap`, entidades orientadas al rumbo con nombre y vida, **vuelo con predicción optimista + reconciliación** contra el eco, y HUD mínimo N (panel NAVE + estado). Observable cumplido por autotest con captura: la Phoenix vectorizada volando por el 1-1, 16 entidades, y la posición **persistida de la sesión anterior** como punto de partida. Pendientes que pasan a I5–I7: menú/radar/chat como ventanas N, estación en el protocolo (`EnterMap` no transmite su posición — se agrega en I6), parallax del fondo.
5. ✅ **I5 — Combate y loot** (2026-08-25): láser con autoridad total (rango 600, golpe cada 500 ms, escudo primero, eventos POST-daño), muerte del Vex con recompensa relativa + ledger y respawn desde BD, caja con la mezcla de la **zona** (60/30/10 verificado exacto) y cantidad del NPC, `CollectBox` validado por distancia **en el server** con recogida parcial por capacidad, bodega volante persistida transaccionalmente antes de responder. Migración `.2`: Vex a 800/400 (el seed daba TTK >3 min — injugable). Observable cumplido por autotest del loop completo: matar → caja → recoger → Bodega 40/100, 10.400 C y ledger íntegro (`NPC_KILL` + `CARGO_PICKUP`).
6. ✅ **I6 — La base y la economía del slice** (2026-08-26): la estación viaja en `EnterMap` (posición y radio) y se dibuja con su anillo de zona segura; entrar en rango abre el panel de base (ventana N). **Descarga transaccional** bodega→almacén con **refinado automático desde BD** (30 Asterium + 20 Nebulium + 10 Coronium → 1 Aurorium, gratis y sin colas) y **venta al NPC** con precios de BD, `SELECT FOR UPDATE` y escrituras relativas. `economy_ledger` registra las seis razones del loop. Observable cumplido por autotest del **slice completo**: matar Vex → caja → base → descargar → refinar → vender, con la receta verificada exacta en BD.
7. ✅ **I7 — Reconexión y cierre del gate** (2026-08-25): una caída de socket ya **no** saca la nave del mundo — `LeaveCmd("DROPPED")` abre una **ventana de gracia de 60 s** (el heartbeat también, en vez de dropear) y el tick barre solo a los que la agotan (`TIMEOUT`). El primer frame de una conexión puede ser `Hello` **o** `Resume`: el token del `Welcome` se resuelve por hash contra la sesión viva, `OnResume` **intercambia el puerto** del mismo slot y re-sincroniza el mundo con el mismo método que el join — misma nave, misma carga, misma posición. **Chat tipado por el enlace del juego** (el legado tenía socket aparte y una gramática de texto sin escapar): ventana COMMS del sistema N con pestañas GLOBAL/FACCIÓN, Enter para enfocar, reparto por facción en el server y eco al emisor. Observable cumplido por el autotest e2e con TestBot: Vex → caja → base → refinado → venta → chat ida y vuelta → **corte de red** → regreso con la nave intacta, todo en una corrida.

**Gate E2 cerrado.** El loop vertical completo corre de punta a punta y se verifica solo.

### Después del gate
- **IA de los NPCs** (2026-08-25): portada la máquina de tres estados del server legado (buscar / aproximarse / esperar), con su vagabundeo por **todo el mapa** — que es lo que hace que el sector se sienta vivo — y sus vicios corregidos (destino sorteado fuera de los límites reales, aggro fijo en código, el bucle que se quedaba con el último jugador de la lista). **Pasivo deja de ser inofensivo**: recibir un golpe convierte a cualquier bicho en agresor, como el `ReceiveAttack` del legado; solo el **Ferox** caza por iniciativa. Con ello entra el combate NPC→jugador, la **muerte del jugador** (la bodega volante se queda flotando en una caja: transferencia, no destrucción) y la **reaparición** en la base. Destapó que `npc_catalog.damage` nunca se había calibrado contra nadie: el autotest moría en 15 s contra un Ferox, y la migración `.7` fija el daño por *cuánto te cuesta matar a cada bicho*.

- **Mobiliario del mapa** (2026-08-25): el **portal** del 1-1 entra como dato — migración `.3` con la fila en `map_portal` y el mapa vecino `1-2` que exige su FK (*"el destino existe por construcción"*); viaja **completo en `EnterMap`** (struct `MapPortal`), como manda la spec para portales, estaciones y POIs. En el cliente es `PortalNode`: aro quieto, vórtice que gira y late, y la etiqueta del sector destino debajo; clic = rumbo al portal. **El salto en sí es de E3.** La **caja de carga** deja de estar hardcodeada y pasa a `data/props/cargo-box.json`, como el resto de los assets. El minimapa gana los rombos de estación y portal.

## Reglas de la etapa

- Código en inglés, comentarios en español; UI bajo el skill `mexorbit-ui`; todo número de juego se lee de BD, jamás se hardcodea.
- El prototipo legado (Azure) es referencia de comportamiento, no fuente de código.
- La cuenta del usuario y TestBot separadas desde la semilla (una sesión por cuenta).
- Cada iteración termina con commit+push en los repos tocados y actualización de este plan.
