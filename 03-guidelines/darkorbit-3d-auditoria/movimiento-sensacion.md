# Auditoría: sensación de movimiento de las naves (§5–§6 guidelines DO 3D)

**Fecha**: 2026-08-29 · **Auditor**: sesión de auditoría técnica (solo lectura)
**Referencia**: `mex-orbit-v1\mex-orbit-docs\03-guidelines\darkorbit-3d-guidelines.md` §5, §6, §14
**Código**: `MexOrbit.GodotClient\game\scenes\entities\entity.gd`, `game\ships\ship_sprite_2d.gd`, `game\ships\drone_ring_2d.gd`, `game\ships\projectile_2d.gd`, `game\scenes\world\world.gd`, `net\protocol\incoming.gd`

**Contexto clave**: el cliente Godot es 2D top-down con sheets de 32 frames horneados y declara replicar el **cliente Flash 2D** original, no el 3D. Varias divergencias contra estos guidelines son fieles al 2D de referencia; se marcan igualmente porque la "ley de oro #2" (modelo tosco, vista sedosa) es lo que se audita.

---

## Ítem 1 — Interpolación de posición lineal destino→destino · ✅ (con ⚠️ en recálculo por speed)

**DO**: TweenLite LINEAL destino→destino, `duración = distancia/speed`, tween reutilizado, sin snapping ni extrapolación; si cambia `speed` en vuelo se relanza al mismo destino recalculando duración.

**MexOrbit**:
- `entity.gd:833-903` — `move_with_time(dest, ms)` guarda `_origin = position` (arranca del punto interpolado actual, sin saltos) y `_process` hace `position = _origin.lerp(_target, clampf(_elapsed_ms/_travel_ms, 0, 1))`. Lineal puro, `t` clampeado → **cero extrapolación, cero snapping**. Sin tween de Godot: lerp manual por frame, cero asignaciones — equivalente (mejor) al "tween reutilizado".
- Duración: héroe optimista `world.gd:1178-1180` `time_ms = dist/speed*1000`; follow de grupo igual (`world.gd:1366-1367`). Otras entidades: el server manda los ms restantes (`incoming.gd:195` `time_ms`) → mismo modelo que `Movement.ActualPosition` del GameServer. ✅
- ⚠️ **Gap**: `world.gd:1369-1370` — al llegar `SET_SPEED` solo hace `hero_data.speed = data.speed`. El viaje EN CURSO del héroe **no se relanza** con la nueva duración. DO relanza el tween al mismo destino recalculando `dist/speed`. Consecuencia: tras un buff/debuff de velocidad en vuelo, la nave local llega antes/después que en el server hasta el siguiente click (desincronía visible en frenadas de combate y travel mode).

**Corrección**: en el case `"speed"`, si `hero.is_moving()` relanzar `hero.move_with_time(hero.target_position(), hero.position.distance_to(hero.target_position()) / maxf(speed,1.0) * 1000.0)`.

---

## Ítem 2 — Giro visual suavizado, heading del modelo instantáneo · ⚠️

**DO**: heading del modelo = asignación instantánea (`atan2+π`); la vista lo alcanza con **QuadEaseOut d=0.2 s** (exponencial, ~16 %/frame a 60 fps, constante efectiva ≈0.1 s), camino angular corto.

**MexOrbit** (`ship_sprite_2d.gd`):
- Heading del modelo instantáneo ✅: `entity.gd:822-829` `_face_current()` fija el rumbo en seco al recibir destino/objetivo, con prioridad ataque > destino (idéntica a DO §5.1), re-evaluado cuando se mueve cualquiera de los dos (`entity.gd:882-889`).
- Semántica `atan2 + 180°` (popa) ✅: `ship_sprite_2d.gd:135`.
- Zona muerta 2 px ✅: `DEAD_ZONE := 2.0` (`ship_sprite_2d.gd:19,133`) = el "destino a ≤2 u no re-apunta" de DO.
- Camino corto ✅: `ship_sprite_2d.gd:144` `fposmod(target - _visual_angle + 180, 360) - 180`.
- ⚠️ **La curva y la constante difieren**: `TURN_TIME := 0.1` con `create_tween().tween_method(...)` **LINEAL** de duración FIJA (`ship_sprite_2d.gd:17,140-150`). DO usa ease exponencial con d=0.2: un giro de 170° tarda visiblemente más que uno de 10° y desacelera al llegar. En MexOrbit todo giro dura 0.1 s a velocidad angular constante → los giros grandes se sienten "de resorte" (1700°/s) y paran en seco. Es fiel al Flash 2D, pero es exactamente la mitad de la "sensación" que la ley de oro #2 atribuye al ease.

**Corrección (traducción a sprite 2D)**: sustituir el tween por avance por frame en `_process`: `_visual_angle += diff_corta * min(1.0, (delta/0.2)*(2.0 - delta/0.2))` (fórmula QuadEaseOut incremental de §15) o `1 - exp(-delta/0.1)`. Los 32 frames cuantizan a 11.25°, así que el efecto se percibe sobre todo en giros >45° (más frames intermedios visibles, llegada amortiguada).

---

## Ítem 3 — Banking derivado del error angular + modo combate invertido · ❌

**DO**: `targetRoll = +deltaY·1` clamp ±20° d=0.2 en crucero; **invertido** `−deltaY·2` clamp ±10° d=0.08 moviéndose con objetivo de ataque; vuelve solo a 0; flags `tilting=false` por nave.

**MexOrbit**: **no existe ningún equivalente**. Búsqueda exhaustiva de `skew|banking|roll|tilt` en `game\` → 0 resultados. Los sheets son 32 frames de yaw puro (sin variantes de roll), `ship_sprite_2d.gd` no aplica `skew`, ni `scale` asimétrica, ni rotación del nodo al girar. El giro es solo el cambio de frame. Tampoco hay modo combate (la única diferencia atacando es la prioridad de rumbo del ítem 2).

**Impacto**: es el gap #1 de sensación. En DO el alabeo es la señal visual de "estoy girando" y el modo invertido es la firma del combate. Sin él, el giro 2D se lee como un disco que rota.

**Corrección (equivalentes 2D, de menor a mayor costo)**:
1. **Squash de eje**: durante el giro, `_body.scale.x = cos(deg_to_rad(roll_visual))` con `roll_visual = clamp(deltaY_pendiente, -20, +20)` suavizado con el mismo ease d=0.2 (d=0.08, ×−2, ±10° con `attack_target`); el sprite se "aplasta" perpendicular al rumbo simulando el alabeo. Barato y sorprendentemente efectivo con top-down.
2. **Skew del sprite**: `_sprite.transform` con shear proporcional al roll (misma envolvente).
3. **Frames horneados por roll**: el pipeline `mexorbit-asset-3d` ya hornea PNG desde el modelo 3D — hornear 3 bandas (roll −20/0/+20) × 32 yaw y elegir banda por signo/magnitud del error angular. Es la traducción fiel de "sprites horneados".
En cualquier caso el estado ya está disponible: `deltaY` es la distancia entre el target de `set_heading` y `_visual_angle`, y `attack_target != null && _moving` da el modo combate.

---

## Ítem 4 — Hover idle + NPCs girando en idle · ⚠️

**DO**: solo parado; amplitud pos (5,5,5) u y rot (5,5,5)°; Lissajous `x=sin·cos, y=sin², z=cos·sin`, fase 0.5 rad/s (~6.3 s), **fase inicial aleatoria por nave**; al arrancar tween a 0 en **0.5 s**. NPCs idle: giro `±180°·rand` cada **2000+5000·rand ms**.

**MexOrbit** (`ship_sprite_2d.gd:368-381, 195-199`):
- Hover solo parado ✅ (`_apply_idle_state` corta al moverse; gate de calidad "ship" 0 además lo apaga).
- Amplitud/forma ⚠️: bobbing 1D `position.y` 0→+3 px→0, `TRANS_SINE` yoyo, `BOB_TIME 1.2` (periodo 2.4 s). Réplica del Flash 2D. Contra DO: sin componente X, sin rotación oscilante, periodo 2.4 s vs 6.3 s (más nervioso), y en Godot `y+` es ABAJO — la nave "cuelga" en vez de flotar hacia arriba (DO: `sin²` = solo hacia arriba).
- ❌ **Sin fade-out al arrancar**: `_apply_idle_state` hace `_bob_tween.kill(); _body.position.y = 0.0` — la nave puede saltar hasta 3 px en seco al iniciar un vuelo. DO tweenéa a 0 en 0.5 s.
- ❌ **Sin fase aleatoria por nave**: todos los tweens de bob arrancan en fase 0 → una flota parada respira al unísono (justo lo que la ley visual #3 de DO llama "aspecto de videojuego barato").
- NPCs idle ✅ **exacto**: `_idle_timer = 2.0 + randf() * 5.0` (2–7 s) y `set_heading(_visual_angle + (randf()-0.5)*360.0)` (±180°) — `ship_sprite_2d.gd:52-53, 195-199`; se cancela al moverse (`set_moving` reinicia el timer) y usa el suavizado de giro normal, igual que DO §5.4.

**Corrección**: (a) al pasar a moving, tweenar `_body.position` a cero en 0.5 s en vez de resetear; (b) arrancar el bob con fase aleatoria (p. ej. `_bob_tween` precedido por un tween parcial, o pasar a un oscilador en `_process` con `_bob_phase = randf()*TAU`); (c) opcional: subir el periodo hacia ~4–6 s y sumar un `rotation_degrees` de ±1–2° al `_body` para acercarse al carácter Lissajous.

---

## Ítem 5 — Thrusters por estado + estela ribbon · ⚠️ / ❌

**DO**: escala objetivo moviendo 1 / idle jugador **0.7** / idle NPC 0, ×3 con speed-buff, lerp 20 %/frame, siguen el banking. Estela ribbon 12 muestras × 30 ms, alpha `1−i/12`, anti-teleport >100 u.

**MexOrbit**:
- Thrusters ⚠️ (modelo distinto, fiel al Flash 2D): contador de empuje 16→0 con tick de 50 ms (`THRUST_TICK 0.05`, `THRUST_START 16`, `ship_sprite_2d.gd:20-21, 202-217`); el **frame** del sheet de llama es el nivel de empuje (llama crece al volar, se apaga al parar). Anclada por tabla de 32 posiciones por clase (`_build_engines`, `ship_assets.gd:128`), rotada al ángulo de popa (`_set_visual_angle`), detrás del casco. Gate de calidad "engine" ✅.
  - ❌ Idle del jugador: la llama se apaga **del todo** (contador ≥ frames → `visible=false`). DO deja 0.7 en el jugador (tu nave "vive" aunque estés parado) y 0 solo en NPCs. Hoy jugador y NPC se tratan igual.
  - ❌ Sin ×3 por speed-buff (el Travel Mode añade la estela `speedBuffEffect` aparte, `set_travel`, `ship_sprite_2d.gd:320-354`, con fase aleatoria ✅ — cubre parte del rol).
  - El "lerp 20 %/frame" está sustituido por la rampa discreta de 16 pasos (~0.8 s) — más lenta que DO (~0.2 s efectivo) pero continua; aceptable como equivalente 2D.
- Estela ❌: **no hay ribbon**. Lo que hay es humo por bocanadas: `SMOKE_TICK 0.08` (cada 80 ms con empuje alto), sprites `engineSmoke0` a 45 fps soltados en el mundo (`ship_sprite_2d.gd:218-231`) — réplica del 2D. No hay ring buffer 12×30 ms ni alpha decreciente continuo ni anti-teleport (irrelevante sin ribbon; el humo solo se emite moviéndose, no deja latigazos).

**Corrección**: (a) dar a la llama un piso de idle para el héroe/jugadores (p. ej. frame mínimo visible o `scale 0.7` del sprite de llama cuando `!_moving && !es_NPC`); (b) si se quiere la estela DO: `Line2D` por motor con ring buffer de 12 puntos muestreados cada 30 ms en coordenadas de mundo, `width_curve`/gradiente alpha `1−i/12`, color 0x5AC3D8, y colapso del buffer si el salto entre muestras >100 u — mapeo directo de §15 ("ImmediateMesh/MultiMesh con el mismo ring buffer").

---

## Ítem 6 — Slots de motor/láser nombrados · ⚠️

**DO**: submeshes `engine_*` → slots de motor; `laserpoint_<n>` → los disparos salen de los cañones reales incluso con banking.

**MexOrbit**:
- Motores ✅: equivalente completo. `enginePositionClassID` → tablas `{lista: [[x,y]×32]}` por clase (`ship_assets.gd:127-129`), un sprite de llama por lista, posicionado por frame de rotación (`ship_sprite_2d.gd:97-126, 238-247`). Es el "rig radial" horneado: el slot sigue la orientación como en DO.
- Láseres ❌: `projectile_2d.gd:108-134` — `_spawn_laser` sale de **`attacker.position`** (centro de la nave) hacia `target.position`. Los datos existen y están documentados: `docs\naves-y-aliens.md:97-106` describe las `positionsList` (standard/left/right/centerRear/centerFront) como **32 puntos de anclaje de bocas de cañón** con el patrón `<salvo>` de disparos consecutivos — extraídos al catálogo pero **sin consumidor** en el código de disparo.
- El skill `mexorbit-asset-3d` define rigs radiales para assets nuevos (equivalente del slot nombrado), así que el pipeline está listo; falta engancharlo al disparo.

**Corrección**: en `fire_laser/_spawn_laser`, resolver el punto de salida como `attacker.position + anchor[frame_actual]` rotando por la tabla de la nave (igual que las llamas), ciclando el patrón de salvos por disparo consecutivo. Con el ítem 3 resuelto, el anclaje por frame ya "sigue el banking" gratis.

---

## Ítem 7 — Placeholder de carga + desincronización de idle · ➖ / ❌

**DO**: si el AWD no llegó en 1000 ms (2500 la propia), plano con el sprite 2D; animación `idle` con offset aleatorio de hasta 10 s.

**MexOrbit**:
- Placeholder ➖ (no aplica el problema): los assets son PNG locales cargados síncronos (`ship_assets.gd:111-119` `load()` directo con caché) — no hay ventana de carga que cubrir. El fallback permanente para tipos sin arte es el triángulo del prototipo (`entity.gd:1064-1072`, espíritu `replacementShips` ✅) y toda nave aparece con fade alpha 0→1 en 0.3 s (`ship_sprite_2d.gd:88-94`) — no hay "brote" en seco. Si algún día los sheets pasan a `load_threaded_request`, habrá que añadir el placeholder temporal.
- Desincronización ❌ parcial:
  - Aliens `loopPlay`: `_loop_time` arranca en 0 para todos (`ship_sprite_2d.gd:40, 188-190`) → **dos NPCs idénticos animan en fase perfecta**. DO desfasa hasta 10 s. Fix de una línea: `_loop_time = randf() * 10.0` en `setup`.
  - Bobbing: sin fase aleatoria (ver ítem 4).
  - Travel trails ✅ (`"time": randf()`), NPC idle-turn ✅ (timer aleatorio), humo ✅ (emerge del movimiento).

---

## Ítem 8 — Cloak y marcador de selección · ✅ / ⚠️

**DO**: cloak = alpha 0.5 en 0.2 s (drones incluidos); marcador de selección entra 1.5×→1× en 0.3 s.

**MexOrbit**:
- Cloak ✅ **exacto**: `ship_sprite_2d.gd:162-167` alpha 0.5 en 0.2 s ("exacto al TweenLite del original, sin tinte"). Reglas de juego correctas: propia translúcida, ajenas ocultas del mapa pero vivas en minimapa (`entity.gd:179-194`). Nota menor: el comentario de cabecera (`ship_sprite_2d.gd:12`) aún dice "alpha 0.3" — desactualizado respecto al código.
  - ⚠️ Drones: `set_cloaked` modula el `ShipSprite2D`; `DroneRing2D` cuelga de la ENTIDAD (`entity.gd:800-808`), no del ship → los drones de la nave camuflada propia quedan a alpha 1 (DO les aplica el 0.5).
- Marcador ⚠️: aro `ship_border` con frame = lockType y **fade 0.3 s** (`entity.gd:1006-1037`), fiel al Flash 2D (`docs\naves-y-aliens.md:111-112`). No existe el cierre 1.5×→1× de DO 3D. Fix barato con el tween ya existente: partir de `scale·1.5` y tweenar a `scale·1.0` en los mismos 0.3 s en paralelo al alpha — señal de "lock adquirido" mucho más legible en combate.

---

## Veredicto por ítem

| # | Ítem | Veredicto |
|---|---|---|
| 1 | Interpolación lineal dist/speed | ✅ (⚠️ sin recálculo al cambiar speed en vuelo) |
| 2 | Giro suavizado / heading instantáneo | ⚠️ tween LINEAL 0.1 s fijo vs QuadEaseOut d=0.2 |
| 3 | Banking + modo combate | ❌ inexistente en cualquier forma |
| 4 | Hover idle + NPC idle turns | ⚠️ bob 1D síncrono sin fade-out; NPC turns ✅ exactos |
| 5 | Thrusters por estado + ribbon | ⚠️ llama por frames sin idle 0.7; ❌ sin ribbon |
| 6 | Slots motor/láser | ⚠️ motores ✅; láseres desde el centro (datos listos sin usar) |
| 7 | Placeholder + desync idle | ➖ placeholder no aplica; ❌ loops de NPC en fase |
| 8 | Cloak + marcador | ✅ cloak exacto (⚠️ drones); ⚠️ marcador sin cierre 1.5× |

## Top 5 gaps por impacto en la sensación

1. **Banking inexistente** (ítem 3) — la señal de giro más importante de DO; proponer squash `scale.x=cos(roll)` o frames horneados por roll vía el pipeline 3D.
2. **Giro lineal 0.1 s fijo** (ítem 2) — cambiar a ease exponencial d=0.2 (fórmula incremental de §15); giros grandes hoy son robóticos.
3. **Sin recálculo del viaje al cambiar speed** (ítem 1) — desincronía cliente/server tras buffs; fix de 3 líneas en `world.gd` case `"speed"`.
4. **Idle sin vida y sincronizado** (ítems 4+7) — fase aleatoria en bob y `_loop_time`, fade-out 0.5 s al arrancar, thruster idle 0.7 del jugador (ítem 5a).
5. **Láseres desde el centro** (ítem 6) — consumir las tablas de bocas de cañón ya extraídas al catálogo; con banking, ancla por frame.
