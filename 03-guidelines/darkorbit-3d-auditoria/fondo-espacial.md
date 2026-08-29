# Auditoría: fondo espacial del cliente Godot vs guidelines DarkOrbit 3D

Fecha: 2026-08-29. Alcance: ítems 1–8 del checklist §14 (fondo espacial) de
`mex-orbit-docs\03-guidelines\darkorbit-3d-guidelines.md` (§10) y detalle en
`darkorbit-3d\fondo-hud-integracion.md` (§2–4) y `darkorbit-3d\calidad-rendimiento.md`.

Código auditado (solo lectura, sin cambios):

- `C:\Source\MexOrbit\MexOrbit.GodotClient\game\map\map_background.gd`
- `C:\Source\MexOrbit\MexOrbit.GodotClient\game\map\map_assets.gd`
- `C:\Source\MexOrbit\MexOrbit.GodotClient\game\map\map_tile_layer.gd`
- `C:\Source\MexOrbit\MexOrbit.GodotClient\game\map\starfield_2d.gd`
- `C:\Source\MexOrbit\MexOrbit.GodotClient\game\map\lensflare_2d.gd`
- `C:\Source\MexOrbit\MexOrbit.GodotClient\game\map\main_background.gdshader`
- `C:\Source\MexOrbit\MexOrbit.GodotClient\game\scenes\world\world.gd` (integración)
- `C:\Source\MexOrbit\MexOrbit.GodotClient\assets\spacemap\` (maps-config.xml, patterns.json, texturas)

Contexto: el cliente MexOrbit es 2D top-down; la vara correcta es el fondo Starling 2D
de DO (§10.1/§2 del doc de integración), con los mecanismos 3D (skybox twinkle, star
dust volumétrico) como referencia aspiracional. El fondo está notablemente bien
portado — es una réplica consciente del cliente Flash, con comentarios que citan las
fórmulas originales — pero hay dos desviaciones de fórmula con efecto visible.

## Resumen de veredictos

| # | Ítem | Veredicto |
|---|------|-----------|
| 1 | Parallax por capas con factores por capa | ⚠️ fórmula de posición ✅, **orden de dibujo ❌** |
| 2 | Snap a entero (anti-jitter) | ❌ ausente en toda la cadena |
| 3 | Star dust que "venda" el movimiento | ⚠️ existe (réplica del 2D Flash), sin color por mapa |
| 4 | Twinkle de estrellas | ➖ fiel al cliente 2D (estático); mejora opcional |
| 5 | Fondo definido por datos | ✅ maps-config.xml + patterns.json, igual que el CMS |
| 6 | Tiles aleatorios sin repetir vecinos + máscara | ✅ (nota de rendimiento) |
| 7 | Planetas con pFactor propio y fade-in | ⚠️ pFactor ✅, fade-in ❌ |
| 8 | Lensflares con oclusión | ✅ (oclusión aproximada por círculo; capa bajo el mundo) |

---

## 1. Parallax por capas — ⚠️

**Fórmula de posición: correcta.** `map_background.gd:96-98`:

```gdscript
node.position = -center / float(entry.p_factor) * zoom + screen_center \
    + (entry.offset as Vector2) * zoom
```

Lado a lado con DO (fondo-hud-integracion.md §2.3):

| Aspecto | DarkOrbit | Godot | Veredicto |
|---|---|---|---|
| Scroll fondo | `x = int(−camX/pF + px + off)` | `−center/pF·zoom + vp/2 + off·zoom` | ✅ salvo el `int()` (ítem 2) |
| Planetas | `(p − cam)/pF + px` | mismo resultado: offset precalculado `pos/pF` en `map_background.gd:218` | ✅ |
| pFactor default | 10 | 10 (`map_assets.gd:397,416,428`) | ✅ |
| Coordenadas ×10 de planetas/flares | sí | sí (`map_assets.gd:413,426`) | ✅ |
| Tamaño capa tileada | `mapW/pF·scale`, recentrado del excedente | idéntico (`map_background.gd:170,193-195`) | ✅ |
| scaleFactor de mapas grandes | sí | sí (`map_background.gd:53`) | ✅ |
| Nº de capas | por XML del mapa | por XML del mapa (mismo archivo) | ✅ |
| **Orden de dibujo** | **`depth = 1000/pFactor + layer` ascendente** | **`z_index = layer` a secas** | **❌** |

El `zoom` aquí es la escala ventana/1280 del render 2D (`world.gd:850-851`), equivalente
al `px = viewportW/(2·scale)` del original: correcto.

**El gap: orden de dibujo.** DO ordena por `1000/pFactor + layer` (el `layer` solo
desempata); el cliente Godot usa solo `layer`:

- `map_background.gd:137` (fondo): `sprite.z_index = int(bg.layer)`
- `map_background.gd:184` (mosaico): `tiles.z_index = int(bg.layer)`
- `map_background.gd:215` (planeta): `sprite.z_index = int(planet.layer)`

Consecuencia real, no teórica — mapa 1-1 de `assets\spacemap\maps-config.xml`:

| Elemento | pFactor | layer | depth DO (1000/pF+l) | z Godot |
|---|---|---|---|---|
| bg 2001 | 10 | 0 | 100 | 0 |
| bg 1 (main) | 10 | 1 | 101 | 1 |
| planeta 114 | 9 | 2 | 113 | 2 |
| bg 2013 | 6 | 3 | 170 | 3 |
| planeta 116 | 6 | 6 | 173 | 6 |
| planeta 115 | 5 | 5 | 205 | 5 |
| **bg 2024 (nebulosa cercana)** | **3** | **4** | **337 → encima de los planetas** | **4 → debajo de 115 y 116** |

En DO la nebulosa de pFactor 3 (la capa más cercana) tapa a los planetas lejanos al
pasar por delante; en Godot los planetas 115/116 se dibujan encima de ella. Se invierte
la profundidad percibida justo en las capas que más se mueven.

**Corrección**: `z_index = int(round(1000.0 / p_factor)) + int(layer)` en los tres
puntos (o mejor, un solo cómputo en `_add`), replicando `depth = 1000/parallaxFactor + layer`
con orden ascendente. Ojo con el rango de z_index de Godot (±4096): 1000/0.001 desborda;
basta con clampear (los pFactor reales del XML son ≥ 1).

## 2. Snap a entero — ❌

DO castea a entero en TODA la cadena (fondo-hud-integracion.md §2.3, §6):
`view.x = int(−camX/pFactor + px + offsetX)`, cámara `camera.x = int(hero.x)`, HUD
`x = int(p.x)`. Es su anti-jitter deliberado (guidelines §10.1: "Snap a entero siempre").

En el cliente Godot no hay ningún redondeo:

- `map_background.gd:96-98` — posición de capa en float puro.
- `world.gd:856` — `_camera.position = hero.position` (posición interpolada, float).
- El starfield tampoco redondea (`starfield_2d.gd:69`, menor: son puntos de 1px).

Con la cámara siguiendo a la nave interpolada, cada capa aterriza en subpíxel distinto
cada frame → shimmer en los bordes de tiles y "nado" del fondo respecto a las naves,
sobre todo con filtrado bilineal y a velocidad de crucero.

**Corrección** (respetando el espacio lógico 1280 del render 2D): redondear la posición
final de pantalla de cada capa, p. ej. en `update_parallax`:

```gdscript
node.position = (-center / float(entry.p_factor) * zoom + screen_center
    + (entry.offset as Vector2) * zoom).floor()
```

y valorar el snap de la cámara lógica (`int(hero.x)`) como hace DO — con cuidado de no
introducir escalones en el propio movimiento de las naves (DO redondea la cámara, no
las entidades).

## 3. Star dust — ⚠️

Existe y corre siempre: `Starfield2D` (`starfield_2d.gd`), montado incondicionalmente en
`map_background.gd:35-37` y avanzado por frame en `update_parallax` (`map_background.gd:89-90`),
incluso en mapas sin assets. Es una réplica documentada del starfield del **cliente 2D**
Flash, no del star dust 3D:

| Aspecto | DO 3D (§10.4, calidad-rendimiento.md §7) | Godot |
|---|---|---|
| Naturaleza | 1500 quads de 3u en volumen 2000×400×2000, clonado en tiles de 2000u, profundidad real y ∈ [0,−300] | 300 puntos de 1px lógico en coordenadas de PANTALLA (`starfield_2d.gd:20`) |
| Parallax | real (cámara en perspectiva) | fake: velocidad = 30 × delta de cámara, multiplicador de profundidad 0.5–3.5 por partícula (`starfield_2d.gd:22,51`) |
| Reposo | deriva lenta | deriva idle (+8,+4) px/s (`starfield_2d.gd:21`) |
| Colores | variados, elegidos por `@starfield` del XML del mapa | gris 17–255 proporcional a la profundidad (`starfield_2d.gd:48`) |
| Gate de calidad | solo con efectos ≥ MEDIUM | siempre (fiel al 2D, comentado en `map_background.gd:25-27`) |

Para un cliente 2D la réplica es la correcta y la lógica (idle/vuelo/salto de mapa) está
calcada. Dos flecos:

1. El atributo `starfield="star_dust_colors_cyan"` que SÍ está en `maps-config.xml`
   (p. ej. mapas 666 y 1) **no se parsea** en `map_assets.gd:_read_maps_config` — se
   pierde el tinte por mapa que DO usa para dar identidad a cada zona.
2. No usa `CPUParticles2D` sino `_draw` manual; a 300 puntos es irrelevante, pero si se
   quisiera acercar al look 3D (más densidad, quads con leve variación de tamaño/color),
   la vía barata es subir COUNT y tintar con el atributo del XML.

## 4. Twinkle de estrellas — ➖ (con mejora opcional)

No hay ningún shader de titileo; las estrellas van horneadas en las texturas de fondo y
tiles. Esto es **fiel al cliente 2D**: el doc lo dice explícitamente
(fondo-hud-integracion.md §2.4: "Las estrellas del fondo 2D NO son procedurales… las
procedurales-animadas están en el skybox 3D"). El único shader de fondo es
`main_background.gdshader` (blend add + fade de bordes), que replica el `blendMode="add"`
del descriptor del mapa.

Mejora opcional "premium" si se quiere vida en el cielo sin tocar assets: aplicar al
fondo principal (o a una capa de estrellas dedicada) la fórmula del skybox DO
(guidelines §10.3, receta §11.5):

```
color = stars(uv) * mask(uv + vec2(2,1)*t) * mask(uv + vec2(-1.5,1)*t)   // t = TIME/120
```

Producto de dos máscaras móviles = titileo pseudoaleatorio, un sampler extra y cero CPU.

## 5. Fondo definido por datos — ✅

Cadena completa y idéntica a la del cliente Flash (`map_assets.gd:4-16`):

- `assets/spacemap/maps-config.xml` — **el mismo archivo que sirve el CMS** al cliente
  original; se parsea en runtime (`map_assets.gd:356-431`) precisamente para poder
  reemplazarlo tal cual. Defaults correctos al parsear: pFactor 10, layer 0, shift 0,
  scale 1, maskID −1 (`map_assets.gd:397-402`).
- `patterns.json` (generado de profile.xml del SWF) — typeID → resKey/tile/atlas.
- El GameServer solo manda el mapId; todo lo visual se resuelve en cliente, como DO.
- backgrounds, planets y lensflares por mapa; scaleFactor y override de minimapa incluidos.

Fleco: no se parsean `starfield` (ítem 3) ni `isMain` implícito cuando hay un solo
`<background>` (DO §2.1); hoy todos los mapas del XML lo declaran explícito, así que no
tiene efecto.

## 6. Tiles aleatorios + máscara — ✅

`map_tile_layer.gd` replica el TileMapBuilder:

- Grid `round(layer_size / tile_size)` (`map_tile_layer.gd:29-30`) — igual que DO.
- **Selección aleatoria evitando el vecino izquierdo y el de arriba** con re-roll
  (guard 8 intentos, solo si hay >2 subtexturas) (`map_tile_layer.gd:61-66`).
- **Máscara sampleada al centro de celda**; alpha 0 → celda vacía
  (`map_tile_layer.gd:50-60`), con warning si falta el asset (`map_background.gd:181`).
- Semilla estable `map_id*1000 + typeID` (`map_background.gd:186`): mosaico idéntico
  entre sesiones y distinto por capa — mejora razonable sobre DO (que re-randomiza).

Diferencia de método, no de resultado visual: DO solo materializa `viewport + 2 tiles`
y recicla al hacer scroll (§2.4); el cliente Godot construye la capa ENTERA como un solo
CanvasItem con un `draw_texture_rect_region` por celda (`map_tile_layer.gd:81-86`). Para
la capa de pFactor 3 y scale 1.8 de 1-1 eso son miles de comandos re-procesados por
frame aunque el 90% caiga fuera de pantalla (Godot no recorta comandos individuales
dentro de un item). Si el profiler lo señala, la corrección DO es la ventana
`viewport + 2·tile` con reciclado, o trocear la capa en varios CanvasItems para que el
culling de items actúe.

## 7. Planetas: pFactor propio y fade-in — ⚠️

- pFactor propio ✅: `map_background.gd:205-222`, con la fórmula del cliente
  (`(planeta − cam)/pFactor + centro`) implementada como offset `pos/pFactor` — comentario
  correcto en `map_background.gd:200-204`. Rotación y radio de oclusor incluidos.
- **Fade-in ❌**: DO funde los planetas 0.5 s y los fondos/tiles 1 s al montar el mapa
  (fondo-hud-integracion.md §2.3-2.5). En Godot todo aparece de golpe al `build()`. Es un
  pop visible en cada cambio de mapa.

**Corrección**: en `_add()` un tween compartido `modulate.a 0→1` (0.5 s planetas, 1 s
fondos/tiles), como ya hace `Lensflare2D` con su fundido de 0.1 s.

## 8. Lensflares — ✅

`lensflare_2d.gd` + `map_background.gd:99-111,227-236`:

| Aspecto | DO (§10.5, §2.6) | Godot |
|---|---|---|
| Cadena de lentes | `lens_i = (centro − flare)·3/N·i` | idéntico (`lensflare_2d.gd:67`) | 
| Estrella con `star=true` | MovieClip girando −9°/s | 15 frames a 60 fps, −9°/s (`lensflare_2d.gd:35,64`) |
| Fades | in 0.1 s, out inmediato | idéntico (`lensflare_2d.gd:74-85`) |
| Oclusión | bitmap collider (alpha>128) + raycast 3D | círculo de radio del catálogo (`map_background.gd:106-110`), aproximación documentada |
| Throttle | 500 ms | por frame (más caro pero trivial a esta escala) |
| Capa | sprite Flash ENCIMA del mundo, bajo el HUD | z_index 90 dentro del CanvasLayer −10, **debajo del mundo** (`map_background.gd:232`) |
| Gate de calidad | solo qualityBackground HIGH | `Quality.level("effect") > 0` (`map_background.gd:65`) |

El único matiz con efecto visible es la capa: en DO el destello y su cadena cruzan POR
ENCIMA de las naves (es parte del look "premium"); aquí quedan detrás de todo el mundo.
Corrección barata: mover los flares a un CanvasLayer propio entre mundo y HUD (o al
mundo con z alto), conservando la misma fórmula de cadena.

Elemento extra que DO no tiene en 2D: el shader aditivo con fade de bordes del fondo
principal (`main_background.gdshader`) — decisión correcta y documentada.

---

## Los 5 gaps por impacto en la sensación de volar

1. **Sin snap a entero en toda la cadena de scroll** (`map_background.gd:96`,
   `world.gd:856`). Es EL anti-jitter de DO (guidelines §10.1 y §11: "toda la cadena
   redondea"). Shimmer subpíxel constante en tiles y bordes al volar — afecta cada
   segundo de juego. Fix: `.floor()` de la posición final de pantalla por capa (y
   valorar `int()` en la cámara lógica).
2. **Orden de dibujo sin el término `1000/pFactor`** (`map_background.gd:137,184,215`).
   Invierte la profundidad entre capas cercanas y planetas lejanos — demostrado en el
   mapa 1-1 (la nebulosa pFactor 3 queda tras planetas pFactor 5-6). Rompe la ilusión de
   paralaje justo en las capas que más se mueven. Fix: `z_index = 1000/pFactor + layer`.
3. **Star dust sin color por mapa**: el atributo `starfield` del XML se ignora
   (`map_assets.gd:_read_maps_config`) y las 300 partículas son siempre grises. Es la
   capa que "vende" el movimiento; tintarla por mapa (cyan en 1-1, etc.) y/o subir la
   densidad la acerca al carácter del 3D con costo casi nulo.
4. **Mosaico completo dibujado siempre** (`map_tile_layer.gd:81-86`): miles de comandos
   por frame frente a la ventana `viewport+2 tiles` de DO. No se ve, pero se siente si
   baja el framerate en mapas con capas grandes; medir antes de tocar.
5. **Sin fade-in de planetas/fondos** (0.5 s / 1 s en DO) y **twinkle ausente**
   (opcional): pop al entrar a cada mapa y cielo estático. Ambos son pinceladas
   "premium" baratas: tween de alpha en `_add()` y el shader de doble máscara móvil
   (§10.3) sobre el fondo principal.
