# Auditoría del cliente Godot contra el checklist §14 de los guidelines 3D

**Fecha**: 2026-08-29. **Método**: 5 auditorías paralelas (cámara/HUD/picking, movimiento/sensación, FX, assets/calidad/carga, fondo) comparando constante por constante el código Godot contra [darkorbit-3d-guidelines.md](../darkorbit-3d-guidelines.md).

**Alcance auditado**: el prototipo `MexOrbit.GodotClient` (port fiel del cliente Flash 2D) y, donde aplica el pipeline 3D, el cliente v1 (`mex-orbit-client`: entity_node.gd/quality.gd/asset_defs.gd). Cada informe marca a cuál se refiere. Como el v1 porta leyendo el prototipo, los gaps de sensación aplican a la línea que avance.

## Fotografía general

**Lo que ya está clavado** (fiel a DO, con constantes exactas): interpolación lineal destino→destino con dist/speed · seguimiento rígido de cámara sin clamp en bordes · impactos de daño (offset+rotación aleatorios, hasta el descuido de `Point.polar` replicado) · cloak 0.5/0.2 s · giros idle de NPC 2–7 s/±180° · fondo data-driven con tiles anti-vecino y máscara · lensflares con cadena ×3 y oclusión · luz del héroe 0x2E7DFF y sol portados al v1 · gates de calidad por sitio con señal en caliente · robustez de assets (todo degrada con warning) · sin post-proceso.

**El patrón de los gaps**: el prototipo es un port fiel del cliente **2D** de Flash. Casi todo lo que falta es exactamente lo que el cliente **3D** añadió encima del 2D — la capa de "sensación" (banking, eases, aditivos, shake, multi-capa). Es la brecha esperada, y está perfectamente delimitada.

## Brechas priorizadas

### Tier 1 — Sensación núcleo (lo que más se siente, en orden)

| # | Gap | Constantes DO | Dónde |
|---|---|---|---|
| 1 | **Banking ausente** (la señal de giro #1) | roll = error angular, clamp ±20° d=0.2; combate −2×, ±10°, d=0.08 | ship_sprite_2d.gd — traducción 2D: squash `scale.x = cos(roll)` o frames horneados por roll (pipeline asset-3d) |
| 2 | **Giro visual lineal 0.1 s** (robótico en giros grandes) | QuadEaseOut d=0.2 incremental (§15 del guideline) | ship_sprite_2d.gd:140-150 |
| 3 | **Zoom sin tween** + pasos ×1.1 + clamp [0.1,3] | ×1.2/×0.8, clamp [1,3], tween 0.3 s Quad.easeOut | world.gd:1044-1046 |
| 4 | ~~Sin shake de cámara al recibir daño~~ **RETIRADO (ago-2026)**: verificado jugando DO 3D, el original NO sacude con daño normal — solo con el tipo "I" (minas/kamikaze) y efectos con `shakeScreen="true"` en su XML. Este hallazgo era una generalización errónea del análisis; se portó, se sintió mal, y se quitó. Ver la corrección en §3 del guideline | — | — |
| 5 | **Sin snap a píxel en toda la cadena** (shimmer subpíxel) | DO redondea cámara + fondo + HUD a entero | world.gd:856, map_background.gd:96-98, entity.gd HUD |

### Tier 2 — Identidad visual (el look del 3D)

| # | Gap | Receta DO | Dónde |
|---|---|---|---|
| 6 | **Cero blend aditivo en FX** (único add: el fondo) | CanvasItemMaterial BLEND_ADD compartido en FX luminosos — la ley del 95 % | ~13 clases de fx en game\ |
| 7 | **Loops de FX en fase perfecta** (auras/flotas pulsan sincronizadas) | fase aleatoria por instancia (startTime/loop_time) | sheet_anim_2d.gd — fix de 1 línea |
| 8 | **Explosión = 1 flipbook** para todo | multi-capa: chispas + nubes + flash 1-frame + fireballs atlas + debris, escalada al clickRadius | world.gd:1289 |
| 9 | **Láseres desde el centro de la nave** | bocas de cañón — las tablas YA están extraídas en docs\naves-y-aliens.md:97-106 sin consumidor (mismo mecanismo que las llamas de motor) | projectile_2d.gd:115 |
| 10 | **Orden de capas del fondo sin profundidad** — nebulosa cercana queda DETRÁS de planetas lejanos (bug real, demostrado en mapa 1-1) | orden = `1000/pFactor + layer` ascendente | map_background.gd (hoy z_index = layer) |
| 11 | **HUD de entidades escala con el zoom** | tamaño constante en pantalla (solo el offset de barras propias × zoom) | entity.gd:954-973 |

### Tier 3 — Corrección y bugs de sincronía

| # | Gap | Fix | Dónde |
|---|---|---|---|
| 12 | **SET_SPEED no recalcula el viaje en curso** → desincronía tras buffs en vuelo | relanzar goto al mismo destino (~3 líneas) | world.gd:1369-1370 |
| 13 | **Doble click <500 ms = atacar** prometido por el setting y sin implementar | gesto canónico DO | settings_window.gd:50, incoming.gd:942 |
| 14 | **`_sheet_anim` reconstruye SpriteFrames por impacto** (hasta 14×/entidad en combate) | caché estática (1 línea) | v1 entity_node.gd |
| 15 | **Beams sin UV scroll** (draw_line plano) | malla/quad estirado + `uv.x += TIME/cycle` (0.3–0.5 s) | AbilityBeam2D |

### Tier 4 — Infraestructura pendiente (v1)

| # | Gap | Referencia |
|---|---|---|
| 16 | **Carga síncrona del GLB en el frame del spawn** (hitch) — `load_threaded_request` + el PNG de media como placeholder (el truco §5.5 de DO sale gratis del pipeline) | guidelines §8 |
| 17 | **Auto-quality por FPS inexistente** — ventana 20 s, baja <10, sube >60, escalera de 5 recortes, no medir sin foco; no pisar el preset por cuenta | guidelines §12.2 |
| 18 | **Glow no ligado al HP** — el mecanismo por instancia ya existe (emission_energy_multiplier); falta multiplicar por `_hp_pct` | guidelines §7.1 |
| 19 | **Materiales duplicados por instancia + supersampling ×2 constante + viewport-por-entidad sin medir con población** — palancas: instance uniforms, factor por zoom (>1.3 → ×2) | guidelines §8.3 |

### Tier 5 — Pulido (baratos, cuando toque)

- Hover idle: amplitud 5u/5°, periodo ~6.3 s, **fase aleatoria** y fade-out 0.5 s al arrancar (hoy: 3 px, 2.4 s, síncrono, corta en seco).
- Llama de motor idle del jugador a 0.7 (hoy se apaga del todo); ×3 con speed-buff; estela ribbon 12×30 ms.
- Star dust: color por `@starfield` del mapa (hoy siempre gris).
- Fade-in de fondos/planetas al entrar al mapa (0.5–1 s; hoy pop).
- Twinkle de estrellas: `stars·mask(uv+(2,1)t/120)·mask(uv+(−1.5,1)t/120)`.
- Marcador de selección con cierre 1.5×→1× en 0.3 s; drones heredando alpha del cloak.
- Minimapa: dibujar el rect del viewport visible.
- Mosaico del fondo dibujado entero por frame → medir; si duele, ventana viewport+2 con reciclado.
- Desync de animaciones idle de NPCs idénticos (loop_time aleatorio).

## Informes por área

- [camara-hud-picking.md](camara-hud-picking.md)
- [movimiento-sensacion.md](movimiento-sensacion.md)
- [efectos-fx.md](efectos-fx.md)
- [assets-calidad-carga.md](assets-calidad-carga.md)
- [fondo-espacial.md](fondo-espacial.md)
