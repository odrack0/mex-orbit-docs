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
4. **I4 — El cliente nuevo** (`mex-orbit-client`, Godot): proyecto limpio con el theme de la dirección N; login (secuencia real), mundo con los assets del slice, vuelo con predicción+reconciliación, HUD mínimo (menú, nave, radar, chat como ventanas N). *Observable: volar en 1-1 viendo la Phoenix sobre el fondo generado.*
5. **I5 — Combate y loot**: selección, láser, muerte del Vex, `BoxSpawn`, `CollectBox` validado por distancia, bodega volante (`player_cargo_hold`), respawn del Vex. *Observable: matar un Vex y ver la carga en la bodega.*
6. **I6 — La base y la economía del slice**: descarga en estación, refinado automático 30/20/10, almacén (`player_resource_balance` con la regla de escritura), venta al NPC, `economy_ledger` registrando todo. *Observable: el loop completo del slice de una sentada.*
7. **I7 — Reconexión y cierre del gate**: resume con gracia de 60 s, matar el proceso del cliente a media pelea y volver; chat tipado; pasada e2e completa con cuenta TestBot. *Observable: el gate demostrado en video/log.*

## Reglas de la etapa

- Código en inglés, comentarios en español; UI bajo el skill `mexorbit-ui`; todo número de juego se lee de BD, jamás se hardcodea.
- El prototipo legado (Azure) es referencia de comportamiento, no fuente de código.
- La cuenta del usuario y TestBot separadas desde la semilla (una sesión por cuenta).
- Cada iteración termina con commit+push en los repos tocados y actualización de este plan.
