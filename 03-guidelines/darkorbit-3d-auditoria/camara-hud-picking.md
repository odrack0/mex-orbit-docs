# Auditoría: cámara, snap a píxel, picking y HUD (checklist §14, ítems 1–4)

**Referencia**: `mex-orbit-docs\03-guidelines\darkorbit-3d-guidelines.md` §3 (cámara), §4 (proyección), §11 (HUD/input/picking).
**Código auditado**: `C:\Source\MexOrbit\MexOrbit.GodotClient\` — principalmente `game\scenes\world\world.gd`, `game\scenes\entities\entity.gd`, `game\map\map_background.gd`, `game\ships\ship_sprite_2d.gd`.
**Criterio**: el cliente es 2D top-down (sprites horneados); se juzga el EQUIVALENTE en Camera2D/CanvasLayer, no el 3D literal.

---

## Ítem 1 — Cámara

### 1a. Zoom continuo con límites y tween — ⚠️ PARCIAL

| Aspecto | DarkOrbit | Godot | Evidencia |
|---|---|---|---|
| Paso de rueda | ×1.2 acercar / ×0.8 alejar | ×1.1 / ÷1.1 (≈×0.909) | `world.gd:1044-1046` y teclas en `world.gd:2513-2516` |
| Animación | tween **0.3 s Quad.easeOut** sobre el factor | **ninguna** — asignación instantánea | `world.gd:1044` |
| Clamp | **[1, 3]** | **[0.1, 3]** | `world.gd:1044` |
| Zoom base | 1 | ancho de ventana / 1280 (espacio lógico del Flash 2D), recalculado al redimensionar | `world.gd:850-851`, `263-270` |

- El zoom es continuo y con límites ✅, pero **sin tween**: cada golpe de rueda salta en seco. En Camera2D el equivalente es idéntico al DO: tweenear el factor con `Tween` 0.3 s `TRANS_QUAD`/`EASE_OUT`.
- El clamp inferior **0.1** permite alejar 10× el espacio lógico — DO nunca deja alejar por debajo de zoom 1 (verías demasiado mapa y las naves quedan ilegibles). Gap funcional además de sensorial.
- **Corrección**: pasos ×1.2/×0.8 sobre un `_zoom_factor` lógico con clamp [1, 3] (multiplicado por el zoom base ventana/1280), animado con tween 0.3 s Quad.easeOut.

### 1b. Seguimiento rígido al héroe interpolado — ✅ EQUIVALENTE

- `world.gd:856`: `_camera.position = hero.position` cada frame — rígido, sin lerp ni deadzone, igual que DO (§3: "camera = int(hero.pos)").
- La suavidad viene del modelo, como en DO: el héroe interpola **linealmente destino→destino** con `duración = dist/speed` (`world.gd:1178-1180` calcula `time_ms`; `entity.gd:896-903` hace `_origin.lerp(_target, t)` con reloj en ms).
- Falta solo el cast a entero (ver ítem 2).

### 1c. Shake de cámara al ser golpeado — ❌ AUSENTE

- No existe shake de cámara en todo el proyecto. El único "shake" es el del casco en Travel Mode del Citadel: `ship_sprite_2d.gd:342-344` (yoyo ±2 px cada 0.075 s sobre `_body`, no la cámara, y no ligado a recibir daño).
- El punto de enganche existe: `world.gd:1511-1514` ya distingue el caso "te golpean a ti" (`attack_hit` con `attacker_id != my_id` y víctima = héroe).
- **Corrección (constantes DO §3)**: desplazar el punto de mira de la cámara con `amp·(cos(paso), sin(paso))`, amplitud inicial **5 u**, **40 pasos de 24 ms** (~1 s), amplitud −1 cada 10 pasos; **solo** cuando el golpeado es el héroe. En Camera2D: un `offset` sumado a la posición rígida.

### 1d. Clamp en bordes del mapa — ✅ EQUIVALENTE

- La cámara no se clampea (Camera2D sin `limit_*`) y el movimiento tampoco: `world.gd:1174-1176` — "Sin clamp: el original navega como si el mapa fuera infinito — el freno real es la radiación". Coincide con DO §3: "Sin clamping en bordes del mapa".
- Matiz: DO sí clampea el **destino** del click-to-move al mapa (§4) aunque no la cámara; el cliente Godot no clampea ninguno y delega en la radiación del server. Diferencia menor y deliberada (comentada en el código).

### 1e. Perspectiva FOV 30 / tilt 45 / distancia 1740/zoom / acople tilt↔zoom — ➖ N/A POR ARQUITECTURA

- El cliente es 2D top-down con sprites horneados desde los modelos 3D; el "picado a 45°" es dirección de arte horneada en el sprite, no un parámetro runtime. Las zonas de cámara, la órbita con botón derecho y el picado dinámico al hacer zoom-in no tienen equivalente 2D directo. Explícitamente N/A.

**Veredicto ítem 1: ⚠️ PARCIAL** (seguimiento ✅, sin-clamp ✅, zoom ⚠️ sin tween y con clamp malo, shake ❌, proyección ➖).

---

## Ítem 2 — Snap a píxel en toda la cadena — ❌ AUSENTE

La ley DO (§13.4): "La cadena entera redondea a píxel (cámara int, fondo int, HUD int) — así se mata el jitter con cámara rígida".

| Eslabón | DarkOrbit | Godot | Evidencia |
|---|---|---|---|
| Cámara | `int(hero.pos)` | `hero.position` sin redondear | `world.gd:856` |
| Fondo parallax | `pos = int(−cam/pFactor) + viewport/2` | `-center / p_factor * zoom + screen_center` sin floor | `map_background.gd:96-98` |
| HUD sobre entidades | `x = int(p.x); y = int(p.y)` por frame | barras/nombres dibujados en espacio de mundo, sin redondeo | `entity.gd:954-973` |
| Project settings | — | sin `rendering/2d/snap/snap_2d_transforms_to_pixel` ni `snap_2d_vertices_to_pixel` | `project.godot` (grep sin resultados) |

- Los únicos redondeos encontrados son cosméticos y aislados: iconos del minimapa (`world.gd:2111`, `2121` con `.floor()`) y el tamaño del minimapa (`world.gd:942`).
- Matiz honesto: con filtrado lineal y sprites de alta resolución el sub-píxel se nota menos que en Flash con bitmaps 1:1; aun así, con cámara rígida + zoom ≠ 1.0 el shimmering de texto (nombres, números de combate) y de las barras de 5 px de alto es visible.
- **Corrección mínima**: activar `rendering/2d/snap/snap_2d_transforms_to_pixel` y redondear `_camera.position` y la posición de cada capa de parallax. Ojo: con `zoom` fraccional (ventana/1280) el snap de cámara debe hacerse en espacio de pantalla, no de mundo.

**Veredicto ítem 2: ❌ AUSENTE.**

---

## Ítem 3 — Picking 2D y click-to-move

### 3a. Radio proyectado constante en pantalla — ✅ EQUIVALENTE (con matiz)

- `world.gd:9` `CLICK_RADIUS := 34.0` y `world.gd:1061` `min_radius := CLICK_RADIUS / _camera.zoom.x` — el mínimo de 34 px es **constante en pantalla**, exactamente la técnica DO (§11: distancia 2D en píxeles ≤ clickRadius).
- Además cada entidad aporta su `clickRadius` real del catálogo (`entity.gd:141`, cajas 25 en `entity.gd:621`, estaciones 50 en `entity.gd:587`) y su `click_center()` con offset de pivote (`entity.gd:874-875`), fiel al original.
- Matiz: el radio del catálogo se compara en **unidades de mundo** (`world.gd:1067`: `maxf(e.click_radius(), min_radius)`), así que al acercar el zoom el área efectiva de una nave crece en pantalla; en DO el clickRadius es px de pantalla puros. Diferencia menor, favorece al jugador.

### 3b. clickPriority — ⚠️ PARCIAL

- DO recorre los click-areas **ordenados por clickPriority descendente**; el Godot elige la entidad **más cercana al cursor** entre todas las que pasan el radio (`world.gd:1058-1070`, `best_dist`). Sin prioridad por tipo: una caja y un NPC solapados se resuelven por distancia, no por importancia.
- **Corrección**: desempatar por prioridad de tipo (bubble > nave enemiga > caja > asset) antes que por distancia, o añadir un campo `click_priority` al catálogo.

### 3c. Cadena GUI → HUD → suelo — ✅ EQUIVALENTE

- Gratis por la arquitectura Godot, igual que DO la tenía gratis por el display list: las ventanas/botones (Controls en CanvasLayer) consumen el evento antes de que llegue a `_unhandled_input` (`world.gd:1035`); dentro del mundo, los click bubbles tienen prioridad explícita (`world.gd:1110-1124`), luego entidades (`world.gd:1126-1144`), y el vacío es click-to-move (`world.gd:1147`).

### 3d. Click-to-move mantenido con throttle — ⚠️ PARCIAL

| Aspecto | DarkOrbit | Godot | Evidencia |
|---|---|---|---|
| Repetición mientras presionado | sí, cada frame evalúa | sí (`_hold_move` + `_process_hold_move`) | `world.gd:1148-1149`, `1238-1251` |
| Throttle | **200 ms** | **250 ms** (`HOLD_RESEND_SEC := 0.25`) | `world.gd:13` |
| Umbral | `dist(héroe, punto) > clickRadius + 45` | `dist(destino_nuevo, último_enviado) ≥ 60` (`HOLD_MIN_DELTA`) | `world.gd:15`, `1250` |
| Clamp del destino al mapa | sí | no (deliberado, radiación) | `world.gd:1174-1176` |

- La mecánica central (conducir manteniendo pulsado, con la cámara siguiendo → la nave persigue al cursor) está y el comentario de `world.gd:1235-1237` demuestra que se portó a conciencia.
- El umbral tiene **otra semántica**: DO mide héroe→cursor (deja de enviar cuando el cursor está casi encima de la nave — evita el baile en parada); Godot mide destino→último destino enviado. En la práctica el efecto de reposo es parecido (el cursor quieto sobre la nave produce destinos que difieren <60 u), pero no impide reenvíos cuando arrastras el cursor en círculos pequeños alrededor de la nave.
- **Corrección**: throttle a 200 ms y añadir la guarda DO: no enviar si `hero.position.distance_to(cursor) <= hero.click_radius() + 45`.

### 3e. Doble click <500 ms = atacar — ❌ AUSENTE

- No hay ningún manejo de `double_click` en el mundo (el único del proyecto es de la ventana de equipamiento, `equipment_slot.gd:289`). El click sobre una nave solo selecciona (`world.gd:1141` `select_ship`); atacar exige la tecla/acción 2 (toggle de láser, `world.gd:2495-2500`) o la barra.
- La infraestructura ya existe y queda huérfana: el ajuste "Doble click para lanzar el ataque" está en `settings_window.gd:50` y viaja por protocolo (`incoming.gd:942` `doubleclick_attack`, `outgoing.gd:169`), y el texto `ttip_double_click_to_fire` está en `messages.json:854`.
- **Corrección**: en `_handle_click`, si la entidad es la ya seleccionada y `event.double_click` (o timestamp <500 ms desde el click anterior sobre la misma), disparar la acción 2 (start_laser), condicionado al setting.

**Veredicto ítem 3: ⚠️ PARCIAL** (radio proyectado ✅, cadena de prioridad ✅, hold-move ⚠️ constantes desviadas, clickPriority ⚠️, doble click ❌).

---

## Ítem 4 — HUD

### 4a. Elementos de tamaño constante en pantalla — ⚠️ PARCIAL

- **Ventanas/paneles** (minimapa, chat, barras de acción, stats): viven en un `CanvasLayer` (`world.gd:279-280`) → tamaño constante, no les afecta el zoom. ✅
- **HUD sobre entidades** (barras de vida 52×5 px, nombres, iconos, números de combate): son hijos del `WorldEntity` (Node2D en espacio de mundo) — `entity.gd:88` (Label), `entity.gd:954-973` (barras con `draw_rect`), `world.gd:971-974` (FloatingText2D anclado a la nave) — y por tanto **escalan con el zoom de la cámara**. En DO §11 "los elementos HUD NO escalan con el zoom": a zoom 3 las barras DO siguen midiendo lo mismo en pantalla; aquí se ven 3× más grandes (y 10× más pequeñas al zoom 0.1 actual).
- **Corrección**: compensar con `scale = 1/zoom` en el sub-nodo de HUD de cada entidad (y entonces sí aplicar la excepción DO: el offset vertical de las barras del héroe multiplicado por el zoom), o mover barras/nombres a una capa de pantalla posicionada por proyección.

### 4b. Barras/nombres siguen a las entidades — ✅ EQUIVALENTE

- Por jerarquía de nodos (el equivalente 2D de "proyectar cada frame"): las barras se dibujan en el `_draw` de la entidad (`entity.gd:1056-1057`), el nombre se recoloca respecto a las barras (`entity.gd:943`), los números de combate van anclados a la nave para seguirla si se mueve (`world.gd:969-975`). El patrón trait→vista de DO se traduce razonablemente a "el fx/label es hijo de la entidad".

### 4c. Minimapa con viewport visible — ⚠️ PARCIAL

- El minimapa es fiel en casi todo: escala `mini = mundo·k` (`world.gd:2086` `scale_v := size / map_limits`), click→mundo con la inversa (`world.gd:2167`), iconos reales, rutas, pings, recon del Spearhead, escalera de tamaños del original (`world.gd:911` `MINIMAP_STEPS`).
- Pero **no dibuja el rectángulo del área visible** de la cámara (`_draw_minimap`, `world.gd:2072-2151`: no hay traza del viewport). En DO es el trapecio (§4); en 2D el equivalente correcto es el rect `viewport/zoom` centrado en la cámara — la firma DO exacta sería dibujar solo el 12.5 % de cada lado desde cada esquina, línea 0.7 px, 0xCCCCCC, alpha 0.5.
- **Corrección**: en `_draw_minimap`, `Rect2` centrado en `_camera.position * scale_v` de tamaño `viewport_size / (zoom * scale_inversa)`, esquinas al estilo DO.

**Veredicto ítem 4: ⚠️ PARCIAL** (seguimiento ✅, tamaño constante ❌ en el HUD de entidades, minimapa sin rect de viewport).

---

## Resumen de veredictos

| # | Ítem | Veredicto |
|---|---|---|
| 1 | Cámara (zoom/tween, seguimiento, shake, clamp) | ⚠️ parcial — seguimiento rígido ✅, sin-clamp ✅, zoom sin tween y clamp [0.1,3] ⚠️, shake ❌, FOV/tilt ➖ N/A |
| 2 | Snap a píxel en toda la cadena | ❌ ausente |
| 3 | Picking + click-to-move + doble click | ⚠️ parcial — radio proyectado ✅, cadena GUI→HUD→suelo ✅, throttle/umbral desviados ⚠️, clickPriority ⚠️, doble click ❌ |
| 4 | HUD (tamaño constante, seguimiento, minimapa) | ⚠️ parcial — seguimiento ✅, escala con zoom ❌, sin rect de viewport ⚠️ |

## Top 5 gaps por impacto en la sensación

1. **Zoom sin tween y con clamp roto** (`world.gd:1044-1046`): cada golpe de rueda salta en seco y se puede alejar hasta 0.1 (10× el espacio lógico, rompe la escala del juego). Fix: pasos ×1.2/×0.8, clamp lógico [1,3] sobre el zoom base, tween 0.3 s Quad.easeOut. Es el gesto de cámara más frecuente del juego.
2. **HUD de entidades escala con el zoom** (`entity.gd:954-973`, `entity.gd:88`): barras y nombres crecen/encogen con la rueda; en DO son constantes en pantalla (legibilidad a cualquier zoom). Fix: `scale = 1/zoom` en el sub-árbol de HUD de cada entidad.
3. **Doble click <500 ms para atacar ausente** (`world.gd:1103-1144`): es el gesto de combate canónico de DO y el setting ya existe y viaja por protocolo (`settings_window.gd:50`, `incoming.gd:942`) — está prometido al jugador y no hace nada.
4. **Shake de cámara al ser golpeado ausente** (todo el proyecto): recibir daño no se siente en el cuerpo. Fix con constantes DO: amp 5→0, 40 pasos de 24 ms (~1 s), −1 cada 10 pasos, sobre el punto de mira, solo al héroe (el evento ya se distingue en `world.gd:1511-1514`).
5. **Snap a píxel ausente en toda la cadena** (`world.gd:856`, `map_background.gd:96`): con cámara rígida y zoom fraccional (ventana/1280), texto y barras de 5 px hacen shimmering al volar. Fix: `snap_2d_transforms_to_pixel` + redondeo de cámara y parallax en espacio de pantalla.

Menores (fuera del top 5): throttle 250→200 ms y guarda `clickRadius+45` del hold-move (`world.gd:13,15`), clickPriority en el picking (`world.gd:1058-1070`), rect del viewport en el minimapa (`world.gd:2072`).
