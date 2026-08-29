# Auditoría de efectos visuales — Cliente Godot vs. guidelines DO-3D

**Alcance**: ítems 1–8 del checklist §14 relativos a FX (sistema .awp, §9 y §15 de
`mex-orbit-v1\mex-orbit-docs\03-guidelines\darkorbit-3d-guidelines.md`, detalle en
`03-guidelines\darkorbit-3d\particulas-fx.md`).
**Código auditado**: `C:\Source\MexOrbit\MexOrbit.GodotClient\` (2026-08-29). Solo lectura.

## Contexto que condiciona todo el veredicto

El cliente Godot es un **port fiel del cliente 2D Flash** (sprite sheets del SWF vía
`ShipAssets`/`MapAssets`), no del cliente 3D. **No existe ni un solo nodo de partículas en
el proyecto**: cero `GPUParticles2D`/`CPUParticles2D`, cero `CanvasItemMaterial`, cero
`.tres`, y las únicas escenas son `login_screen.tscn` y `world.tscn` (sin FX dentro). Todos
los efectos son ~13 clases GDScript (`game\ships\*_fx_2d.gd`, `sheet_anim_2d.gd`,
`projectile_2d.gd`, `emp_wave_2d.gd`…) que combinan flipbooks de sprite sheet, `draw_*` y
Tweens. Muchos ❌ de abajo son fidelidad consciente al 2D, no descuidos — pero contra las
leyes visuales del 3D (el objetivo declarado del guideline) la brecha es esa.

---

## Ítem 1 — FX aditivos + unshaded, envolvente por color ramp, desincronización — ❌

| Ley DO | Cliente Godot |
|---|---|
| `blend add` + unshaded en ~95 % de los fx; los overlaps suman luz | El **único** `blend_add` del proyecto es el fondo del mapa (`game\map\main_background.gdshader:6`, aplicado en `map_background.gd:139`). Todos los FX son `Sprite2D`/`draw_*` con mezcla normal (mix). |
| Fade SIEMPRE por SegmentedColor (rampa RGB+alpha multi-punto), nunca por opacidad del material | Todos los fades son `modulate:a` lineal por Tween: `ring_pulse_fx_2d.gd:77`, `jam_burst_fx_2d.gd:44`, `entity.gd:640,696,752,1028`; o alpha calculado a mano: `ability_beam_2d.gd:41`, `emp_wave_2d.gd:34`. No hay ningún `Gradient`/`GradientTexture1D` en el código de juego. |
| Desincronización estadística: startTime/lifetime aleatorios por partícula, fase aleatoria por instancia | Parcial y puntual: offsets escalonados fijos en JamBurst (`RING_DELAYS`, `jam_burst_fx_2d.gd:10`) y SAB (60/440/500 ms, `projectile_2d.gd:102`). Pero los bucles (`markTarget`, `shield1`, `invincibilityShield`, `ela0`) arrancan todos en `_time = 0` — `SheetAnim2D.spawn` (`sheet_anim_2d.gd:22-46`) no tiene offset de fase: dos naves con el mismo aura pulsan sincronizadas. |

**Receta DO a aplicar**: un `CanvasItemMaterial` compartido con `blend_mode = BLEND_MODE_ADD`
para toda la capa de FX luminosos (láseres, anillos, flashes, beams, motores); envolventes con
`GradientTexture1D` (equivalente 1:1 de SegmentedColor, §15) en vez de tween lineal de alpha;
y `_time = randf() * frames / fps` opcional en los `SheetAnim2D` en loop.

## Ítem 2 — Beams = geometría estirada + UV scroll; disparos desde bocas de láser — ❌

| Ley DO | Cliente Godot |
|---|---|
| Beam = malla de largo fijo + textura patrón repeat + UV scroll (cycle 0.3–0.5 s) + rampa de aparición; estirado runtime `scale = dist/largo` | Los láseres de combate son **proyectiles voladores** (sprite que viaja de atacante a objetivo en 0.15 s, `projectile_2d.gd:76-134`) — réplica del cliente 2D, no beams. El único beam real es `AbilityBeam2D` (reparación Aegis): sí se estira entre origen y destino y sigue a ambas naves (`ability_beam_2d.gd:38-48`), pero es `draw_line` ×3 + 8 círculos viajando — sin textura patrón, sin UV scroll, sin rampa de aparición (aparece a alpha pleno; solo tiene fade-out al 70 %). El beam recolector es un sprite (`entity.gd:666,695-697`). |
| Disparos salen de `laserpoint_<n>` reales (siguen el banking) | Los disparos salen de `attacker.position` (centro de la entidad, `projectile_2d.gd:115`). No existe "laserpoint" en el proyecto (grep sin resultados). Curiosamente los **motores sí** tienen puntos por nave (`enginePositionClassID`, `ship_sprite_2d.gd:97-126`) — el patrón de datos ya existe, falta el equivalente para cañones. |

**Receta DO**: para beams continuos (Aegis, futuro chain lightning): `Line2D`/quad con shader
canvas_item `uv.x += TIME / cycle` (formula 1; `sin` para vaivén), textura repeat, blend add,
`scale.x = dist / largo_base * rampa` con rampa 0→1 tweeneada. Para los cañones: tabla
`laserPoints` por nave junto a `engine_positions` y disparar desde ahí.

## Ítem 3 — Explosiones multi-capa + debris — ❌

| Ley DO | Cliente Godot |
|---|---|
| 4 capas: 400 chispas rápidas (velocity esférica 100–200, gravedad, RotateToHeading, vida 0.1–0.8 s) + 60 nubes (250–350) + flash central 1 quad 250 u vida 0.25 s scale 0→1.4 + 20 fireballs con atlas 4×2 celda aleatoria. Muerte de nave: + flash de luz 0.1 s + `ship_debris` en max | Explosión = **un solo flipbook** `explosionN` en la posición de la víctima (`world.gd:1289-1291`, sheet por `explodeTypeID` en `entity.gd:142`). Minas: flipbook + para la EMP un anillo dibujado a mano (`emp_wave_2d.gd`, aproximación declarada en su docstring). Smartbomb: flipbook `smartbomb1` (`world.gd:1943-1944`). Sin chispas, sin flash de 1 frame, sin fireballs, **sin debris**, sin luz. |

**Receta DO**: escena `explosion.tscn` con 3–4 `GPUParticles2D` (equivalencias §15: chispas =
`amount 400`, `spread 180`, `initial_velocity 100-200`, `gravity`, `align_y`; flash = 1 sprite
scale 0→1.4 en 0.25 s; fireballs = `particles_anim_h/v_frames` con `anim_offset` aleatorio
sobre el atlas) + `one_shot`, todo additive. El flipbook actual puede quedarse como la capa
"fireball" mientras tanto.

## Ítem 4 — Escala por visualSize con scale-in 0.5 s; dispose scaleDown — ⚠️

| Ley DO | Cliente Godot |
|---|---|
| `scalable`/`maxScale`: el fx se escala al visualSize de la nave con ease Quad 0.5 s | Parcial, instantáneo: heal burst escala a `click_radius * 2.4 / cellW` (`entity.gd:483-486`); impacto de escudo a `click_radius / 100` (`entity.gd:522-523`); aro de selección a `click_radius * 2 / native` (`entity.gd:1017-1019`). Pero la **explosión no escala** (igual para un Streuner que para un jefe, `world.gd:1290`), y ningún fx tiene scale-in con ease de 0.5 s. |
| `dispose:"scaleDown"`: al morir, escala a 0 en 0.5 s | No existe. Los fx en loop mueren en seco con `queue_free()` (`entity.gd:369,536`); otros con fade de alpha. |

**Receta DO**: pasar `click_radius` (el visualSize de facto) al spawn de explosión/impactos y
tweenear `scale` 0→objetivo 0.5 s Quad-out al nacer; en los loops (marcador, auras), tween de
escala a 0 en 0.5 s antes de liberar.

## Ítem 5 — Impactos con offset y rotación aleatorios, sin flash rojo; shake solo al héroe — ✅ / ➖

| Ley DO | Cliente Godot |
|---|---|
| Impacto de daño: offset aleatorio dentro del radio + rotación aleatoria, suelto en el mundo, sin teñir el sprite de la nave | ✅ Réplica exacta: `show_hull_impact` (`entity.gd:493-509`) — punto aleatorio en el disco de `clickRadius/2`, `rotation_degrees = randf()*360`, spawn en el **padre** (no sigue a la nave), 0.5 s, cap de 5 (incluso replica el descuido `Point.polar` del original, comentario en `entity.gd:501-502`). Escudo: en la circunferencia del lado del atacante, hijo de la entidad, cap 9 (`entity.gd:515-525`). No hay flash rojo del sprite (el único tinte del casco es el gris de LSH/lock, `entity.gd:1000-1003` — semántica distinta, correcto). |
| Shake de cámara SOLO cuando golpean al jugador (amp 5 u, 40 pasos × 24 ms, −1 cada 10) | ➖ **No hay shake de cámara en absoluto** (grep: el único "shake" es el temblor del casco en Travel Mode, `ship_sprite_2d.gd:342-344`). No hay shake indebido — pero falta el feedback táctil de recibir daño. |

**Receta DO**: shake del punto de mira de `_camera` (`world.gd:263,856`) al recibir impacto el
héroe: amplitud 5 px, 40 pasos a 24 ms, amplitud −1 cada 10 pasos, offset `amp·(cos i, sin i)`.

## Ítem 6 — Efectos declarativos (datos) vs. hardcodeados — ❌

| Ley DO | Cliente Godot |
|---|---|
| Un fx = archivo de datos (.awp: capas + nodos + eventos); en Godot, el `.tscn` ES el .awp (§15) | Todos los fx son **clases GDScript** con constantes embebidas: `EmpWave2D` (radio 275, color en `emp_wave_2d.gd:9-11`), `JamBurstFx2D` (delays/tiempos, `:10-13`), `AbilityBeam2D`, `RingPulseFx2D` (este al menos está parametrizado por args, `ring_pulse_fx_2d.gd:23-25`, y sus presets viven en `entity.gd:313-318`)… Cero escenas de fx, cero recursos. Lo que sí es data-driven: los catálogos de sheets y tablas de láser/sonido (`projectile_2d.gd:25-54`, `ShipAssets`/`MapAssets`) — datos de assets, no de comportamiento. |

**Receta DO**: cuando entren los fx de partículas, hacerlos `.tscn` autocontenidos
(GPUParticles2D + AnimationPlayer para los "particleEvents": encender luz/flash en t exactos,
`end` = fin de la animación) y precargar el `PackedScene` como template (+pool para
impactos/muzzle, §3 de particulas-fx.md).

## Ítem 7 — Luces dinámicas con presupuesto; marcador de target — ⚠️

| Ley DO | Cliente Godot |
|---|---|
| Eventos `light*` → luz puntual dinámica {color, radius, duration, fading}; pool duro de 3 luces de efectos + luz azul del héroe; gates por calidad | ❌ **Cero luces**: no hay `PointLight2D`/`Light2D` ni equivalente de glow en todo el proyecto (grep sin resultados). Explosiones, EMP y warp no iluminan nada. Tampoco existe la "luz del héroe" (la distinción del héroe es solo el HUD). |
| Marcador de target: crosshair con scale pulsante + spin, entra 1.5×→1× en 0.3 s | ⚠️ Existe y funciona, estilo 2D: `markTarget` en bucle sobre la nave (`entity.gd:365-370`) + aro de selección `ship_border` escalado al clickRadius con fundido 0.3 s (`entity.gd:1006-1037`, mismo 0.3 s que el original). Sin pulso de escala ni spin ni entrada 1.5×. |
| | ✔ Nota a favor: el **presupuesto/gate por sitio** sí está calcado — `Quality.level(clave)` consultado en cada punto de spawn (`quality.gd:19-27`; `world.gd:1289,1943,2055`; `projectile_2d.gd:243-244`; `ship_sprite_2d.gd:211`), la misma filosofía DO de recortes por sitio sin budget global. |

**Receta DO**: 2–3 `PointLight2D` pooleadas (nunca instanciar en gameplay) con Tween de
energía según {duration, fading}, encendidas por explosiones/disparos/warp, gated a calidad
alta; una fija azul tenue en el héroe. Requiere pasar los sprites afectados a un canvas con
luz o usar glow aditivo simulado (sprite radial add) si no se quiere tocar el pipeline.

## Ítem 8 — Sin post-proceso/bloom — ✅

| Ley DO | Cliente Godot |
|---|---|
| Ningún post-proceso; el "glow" es composición aditiva de gradientes radiales | ✅ Renderer `gl_compatibility` (`project.godot:31-32`), sin `WorldEnvironment`, sin glow/bloom, sin CanvasModulate (grep). El único shader es el aditivo del fondo. |
| | ⚠️ Matiz: DO cumple esta ley **porque** compensa con additive por todas partes (ítem 1). El cliente hoy no tiene ni bloom ni additive, así que los FX no "emiten luz" — se ve plano. La ley se cumple; la contraparte que la hace funcionar, no. |

---

## Veredicto por ítem

| # | Ítem | Veredicto |
|---|---|---|
| 1 | Additive + unshaded + color ramp + desincronización | ❌ |
| 2 | Beams estirados con UV scroll; bocas de láser | ❌ |
| 3 | Explosión multi-capa + debris | ❌ |
| 4 | Escala por visualSize + scale-in/scaleDown | ⚠️ |
| 5 | Impactos aleatorios sin flash rojo / shake solo al héroe | ✅ impactos · ➖ shake (no existe) |
| 6 | Efectos declarativos | ❌ |
| 7 | Luces dinámicas + marcador de target | ❌ luces · ⚠️ marcador |
| 8 | Sin post-proceso/bloom | ✅ |

## Los 5 gaps por impacto visual (ordenados)

1. **Blend aditivo en los FX luminosos** — la ley del 95 % de DO y el gap de mejor
   relación impacto/costo: UN `CanvasItemMaterial` BLEND_MODE_ADD compartido aplicado a
   láseres, anillos, flashes, beams y llamas de motor transforma el look de "pegatinas
   opacas" a "luz que suma". Hoy solo el fondo es aditivo.
2. **Explosión multi-capa** — sustituir/complementar el flipbook único con la receta de 4
   capas (chispas GPUParticles2D + flash 1-frame + fireballs de atlas con celda aleatoria),
   escalada al clickRadius de la víctima. Es el efecto más visto del juego y hoy es idéntico
   para un Streuner y un jefe.
3. **Desincronización estadística** — offset de fase aleatorio en todos los loops
   (`SheetAnim2D`) y jitter de lifetime cuando entren partículas: es, cita del guideline, "lo
   que mata el aspecto de videojuego barato" en flotas y auras sincronizadas.
4. **Beams con UV scroll + bocas de láser** — el beam del Aegis (y cualquier rayo futuro)
   con textura patrón + `uv.x += TIME/cycle` y rampa de aparición en vez de draw_line; y tabla
   de `laserPoints` por nave (el patrón ya existe para motores) para que los disparos salgan
   de los cañones.
5. **Shake de cámara al recibir daño** — no existe ningún feedback de cámara al ser
   golpeado; la receta DO exacta (5 u, 40 pasos × 24 ms, decae −1 cada 10) es trivial sobre la
   `Camera2D` de `world.gd` y solo para el héroe.

**Fortalezas confirmadas**: impactos de daño clavados al original (offset+rotación aleatorios,
caps 5/9, hasta los descuidos replicados); gates de calidad por sitio con la misma filosofía
DO; sin post-proceso; marcador y aro de selección funcionales con sus tiempos originales.
