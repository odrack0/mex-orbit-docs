# DarkOrbit 3D — Integración 2D/3D: fondo espacial, parallax, HUD y clicks

Análisis del cliente decompilado (`D:\MexOrbit\Decompiled\spacemap\main\scripts\`).
Convención: **clase ofuscada → nombre deducido** (el nombre deducido sale de strings literales del propio código cuando existen, p. ej. `"BackgroundLayerView2D"`, `"ScreenManager is a Singleton"`).

## Índice de clases (mapa de nombres)

| Ofuscada | Nombre deducido | Ruta |
|---|---|---|
| `§_-i3S§` | **Map3DStarlingProxy** (vista raíz del modo 3D, dueña del bucle de render) | `net\bigpoint\darkorbit\map\view3D\§_-i3S§.as` |
| `§_-V16§` | **Map3D** (escena Away3D + fondo + proyecciones) | `net\bigpoint\darkorbit\map\view3D\§_-V16§.as` |
| `§_-Kt§` | **BackgroundLayerView2D** (string literal en el código) | `...\map\view2D\backgrounds\view\§_-Kt§.as` |
| `§_-j4r§` | BackgroundElement (base de todo elemento de fondo) | ídem |
| `§_-MQ§` | TiledBackground (fondo tileado Starling) | ídem |
| `§_-z1f§` | StaticBackground (fondo de 1–4 imágenes 2048px) | ídem |
| `§_-s27§` | BasicElementTile (string literal "BasicElementTile") | ídem |
| `§_-c3G§` | PlanetView (planeta/decoración con parallax propio) | ídem |
| `§_-t1f§` | TraitBackgroundView (fondos-asset animados con decoradores) | `§_-023§\§_-t1f§.as` |
| `§_-z2o§` | BackgroundVO (base: parallaxFactor + depth) | `§_-82X§\§_-z2o§.as` |
| `§_-t3L§` | BackgroundDataVO | `§_-82X§\§_-t3L§.as` |
| `§_-N3K§` | PlanetVO | `§_-82X§\§_-N3K§.as` |
| `§_-B4W§` | BackgroundPattern (toString literal: "BackgroundPattern") | `§_-B1T§\§_-B4W§.as` |
| `§_-l1p§` | PatternsLibrary (parsea el XML global de patterns) | `§_-B1T§\§_-l1p§.as` |
| `§_-G2B§` | **Background3D** (contenedor del fondo 3D por mapa) | `§_-i22§\§_-G2B§.as` |
| `DOSkybox` | DOSkybox (¡no ofuscada!) + `DOSkyBoxMaterialPass` | `§_-i22§\DOSkybox.as` |
| `§_-j4F§` | TileMap3D (tilemap de quads a distintas profundidades) | `§_-i22§\§_-j4F§.as` |
| `§_-z2S§` | AtlasSubTextureParser | `§_-i22§\§_-z2S§.as` |
| `§_-115§` | UberAsset3D (escena display3D desde XML) | `§_-n3Z§\§_-115§.as` |
| `§_-x38§` | Plane3D (plano texturizado, billboard por defecto) | `§_-n3Z§\§_-x38§.as` |
| `§_-Jw§` | CameraController | `§_-321§\§_-Jw§.as` |
| `§_-Q4t§` | CameraState (zoom/pan/tilt/fov) | `§_-321§\§_-Q4t§.as` |
| `§_-n1h§` | DORenderer (renderer Away3D custom, dibuja el skybox) | `§_-23a§\§_-n1h§.as` |
| `§_-u2p§` / `§_-51Z§` / `§_-d2L§` | LensFlareLayer / LensFlareView / LensFlareVO | `§_-FT§\` |
| `§_-f4P§` | ILensFlareProvider (interfaz: `§_-Zy§` lista de flares + `§_-7m§` proyección) | `§_-FT§\§_-f4P§.as` |
| `HUD` | HUD (¡no ofuscada!) + `InternalDisplayLayer` | `§_-a2o§\HUD.as` |
| `§_-h1d§` | HUDElement (base: proyecta y ancla a píxel) | `§_-a2o§\§_-h1d§.as` |
| `§_-N2Y§` | HUDTextLabel | `§_-a2o§\§_-N2Y§.as` |
| `§_-k1j§` | ClickableMapIconView (portales, etc.) | `§_-Py§\§_-k1j§.as` |
| `§_-KV§` | ShipHUDView (barras HP/escudo + nombre) | `§_-92P§\§_-KV§.as` |
| `§_-g4Q§` | SelectionRingView | `§_-92P§\§_-g4Q§.as` |
| `§_-J2P§` | **ScreenManager** (string literal; singleton, dueño de `hitLayer` y de la cámara 2D lógica) | `§_-a2H§\§_-J2P§.as` |
| `§_-I1s§` | MapInputController (click-to-move, teclado, doble click) | `§_-a2H§\§_-I1s§.as` |
| `§_-D3V§` | SelectionManager (picking de entidades) | `§_-B1Q§\§_-D3V§.as` |
| `§_-ta§` | ClickAreaTrait (clickRadius, clickPriority, mouseOver/Out) | `...\map\model\traits\§_-ta§.as` |
| `§_-c2e§` | ScreenEffectsFactory (efectos overlay sobre el HUD) | `§_-S2p§\§_-c2e§.as` |
| `§_-c12§` | StarlingSpriteAdapter (root de Starling + wrapper IDisplayObject) | `...\mvc\display\§_-c12§.as` |
| `§_-r1X§` | ZoneBackgroundManager ("poizone": fondo de zonas tóxicas con máscara) | `§_-P2o§\§_-r1X§.as` |
| `§_-q3t§` / `§_-d21§` | Map2DStarlingProxy / Map2D (cliente 2D fallback) | `...\map\view2D\` |
| `§_-Nr§` | MiniMap (usa recurso "minimap") | `§_-H2N§\§_-Nr§.as` |
| `§_-a2F§` | Vector3-like util (x,y,z) | `com.bigpoint.utils` |

---

## 1. Arquitectura de render — un solo Context3D

`§_-i3S§` (Map3DStarlingProxy) es dueña del `Stage3DProxy` de Away3D y crea Starling **compartiendo contexto**:

```actionscript
// §_-i3S§.§_-42V§()
this._map = new §_-V16§(this, this._stage3DProxy, ...);
this.§_-x1z§ = new Starling(§_-c12§, stage, _stage3DProxy.viewPort, _stage3DProxy.stage3D,
                            Context3DRenderMode.AUTO, "baseline");
this.§_-x1z§.shareContext = true;

// §_-i3S§.update() — ORDEN POR FRAME:
this._map.update(dt, camX, camY);      // lógica + fondo 2D + minimapa
this._stage3DProxy.clear();            // 1. clear (color 0x000000)
this.§_-x1z§.render();                 // 2. Starling: SOLO el fondo espacial 2D
this._map.render();                    // 3. Away3D: skybox + escena 3D
this._stage3DProxy.present();          // 4. present
```

- Perfil Stage3D: `"baseline"`. `Starling.handleLostContext = true`, `enableErrorChecking = false`.
- Antialias del proxy según calidad: **MEDIUM=2, HIGH=8, ULTRA=16** (`§_-QK§()`).
- El **HUD es display list clásico de Flash** encima del Stage3D: en `§_-6Z§()` se hace `addChild(this._map)` (sprite Away3D View) y después `addChild(this.§_-N3y§ = new HUD(this))`.
- Quién llama a `update`: `ScreenManager.§_-g4M§` en `Event.RENDER` (con `stage.invalidate()` cada ENTER_FRAME). La cámara lógica es `ScreenManager.camera:Point`, que sigue al héroe con `camera.x = int(hero.x)` (o a otra nave en modo espectador), más el screen-shake (`camera += amplitud*cos/sin`, amplitud 5→0 en pasos cada 24 ms, 40 ticks).

En la escena Away3D todo cuelga de un contenedor raíz `§_-B4K§` con `scale(0.0001, 0.0001, 0.0001)` — el mundo 3D interno trabaja en unidades ×10000. Mapeo de ejes juego→3D: **x3D = x, z3D = −y, y3D = altura** (visible en `§_-14v§` y `§_-F4J§`).

## 2. El fondo espacial 2D (Starling) — `§_-Kt§` BackgroundLayerView2D

Instanciado en Map3D y agregado al **root de Starling** (lo ÚNICO que Starling renderiza en modo 3D). Se recarga al cambiar de mapa leyendo el **XML del mapa** (`this._map.§_-d1k§`):

### 2.1 Definición por XML del mapa

```xml
<backgrounds>
  <background typeID=".." isMain=".." pFactor=".." layer=".." shiftX=".." shiftY=".."
              scale=".." maskID=".." region="x,y,w,h | w,h"/>
</backgrounds>
<planets>
  <planet typeID=".." x=".." y=".." rotation=".." pFactor=".." layer=".."/>
</planets>
<lensflares>
  <lensflare id=".." typeID=".." x=".." y=".." pFactor=".." star="true|false"/>
</lensflares>
```

Defaults al parsear (`§_-Kt§.reload()`): **pFactor = 10**, layer = 0, shiftX/Y = 0, scale = 1, maskID = −1. Si hay un solo `<background>` es `isMain` implícito. Las coordenadas de planetas y lensflares del XML se multiplican **×10** (el XML está en la escala del minimapa/servidor; el mundo interno es ×10).

El `typeID` indexa en `PatternsLibrary.§_-022§` un `BackgroundPattern` que viene de otro XML global:

```xml
<patterns><backgrounds>
  <background type=".." content="resource" resKey=".." resKeyStarling=".." isTiled=".."
              atlasXml=".." tileWidth=".." tileHeight=".." width=".." height=".."
              order="ASC|DESC" offsetX=".." offsetY=".." showInStarlingMode=".."
              geometry=".." deepTiles=".." textureDiff/Normal/Specular/AlphaMask=".."
              ySpread=".." additionalTilesInGrid=".." enlargeInnerTilesFactor=".."/>
</backgrounds></patterns>
```

(los últimos atributos alimentan `§_-J9§`, la variante con geometría 3D; también hay `content="custom"` con solo width/height/color plano).

### 2.2 Capas y orden

Cada elemento (fondo, planeta, trait) tiene un VO `§_-z2o§` con:

```
depth = 1000 / parallaxFactor + layer
```

y la lista se ordena ascendente por `depth` (`§_-j4r§.compare`). Es decir: **a mayor pFactor (más lejos) se dibuja antes/más al fondo**; `layer` desempata dentro del mismo pFactor. **No hay un número fijo de capas**: cada mapa define las suyas en su XML (típicamente 1 fondo principal pFactor 10 + planetas con pFactor entre ~2 y ~8 + lensflares).

### 2.3 Fórmula de parallax (exacta)

`Map3D.update()` llama cada frame:

```actionscript
// scale = Map3D._scale (zoom de render, ≤1; normalmente 1)
§_-U2d§.update(camX, camY, Math.round(viewportW / (2*scale)), Math.round(viewportH / (2*scale)));
```

y cada elemento se posiciona así (px = mitad del viewport en X, py = mitad en Y):

- **Fondo estático / tileado** (`§_-z1f§`, `§_-MQ§`):
  `view.x = int(−camX / pFactor + px + offsetX)`
  `view.y = int(−camY / pFactor + py + offsetY)`
  con `offset = shiftX/Y + pattern.offsetX/Y` (y para tileados con scale≠1: `offsetX = −round((w − w/scale)/2) + shiftX`).
- **Planeta** (`§_-c3G§`):
  `view.x = (planet.x − camX) / pFactor + px` (ídem Y). El sprite queda centrado (`bitmap.x = −w/2`), respeta `rotation`, hace fade-in 0.5 s.
- **Trait-fondo** (`§_-t1f§`): misma fórmula que planeta con la posición del owner.
- **Lensflare 2D** (`§_-Kt§.§_-7m§`):
  `screen.x = (fx*10 − camX) / pFactor + px` (pFactor viene en `pos.z`).

Nota clave: el término `px = viewportW/(2*scale)` significa que **el punto del fondo bajo el centro de la pantalla es camPos/pFactor** — parallax clásico anclado a cámara, independiente del zoom 3D (el zoom 3D real lo hace la cámara Away3D; el fondo Starling NO se escala con ese zoom, solo con el render-scale del sprite Map3D).

### 2.4 Tamaño del fondo y tiles (`§_-MQ§` TiledBackground)

Tamaño total del fondo tileado:

```
si pattern.width/height > 0:      ese tamaño fijo
si pattern.order == ASC|DESC:     mapW/10 * scale  ×  mapH/10 * scale
si no:                            mapW / pFactor * scale * D4C  ×  mapH / pFactor * scale * D4C
```

(mapW/mapH = tamaño del mapa en unidades de mundo; D4C = multiplicador extra, default 1).

Construcción (usa el `TileMapBuilder` de `net.bigpoint.as3toolbox.starling.mapfactory`, con `OversizedTilesModifier`):

- Grid de `round(width/tileWidth) × round(height/tileHeight)` celdas.
- Viewport del TileMap = `viewport + 2*tile` en cada eje, con `tileMap.x/y = −tileW/−tileH` (una fila/columna extra alrededor para el scroll).
- **Selección de tile aleatoria evitando repetir el vecino izquierdo y el de arriba** (while re-roll), salvo `order="ASC"/"DESC"` (secuencial cíclico 1..N) o el flag `§_-R16§` (aleatorio puro).
- **Máscara opcional** (`maskID` → bitmap `"mask"`): se samplea el centro de cada celda; si el píxel es 0x00000000 el tile queda vacío (tileID = −1). Así se hacen fondos con "agujeros"/formas.
- Scroll: `tileMap.setPosition(int(−camX/pFactor + px + offsetX), ...)` — el TileMap recicla tiles internamente.
- Texturas: atlas Starling (`resKeyStarling` + `atlasXml`), con variante `_atf` comprimida si existe. Fade-in 1 s (TweenLite alpha 0→1).
- Calidad: el fondo tileado solo con `qualityBackground == HIGH` (los "poizone" desde MEDIUM); el estático `isMain` sobrevive en calidad media; planetas requieren `>= GOOD`.

**Las "estrellas" del fondo 2D NO son procedurales**: son parte de las texturas de tiles/fondo. Lo procedural es la *distribución* de tiles. (Las estrellas procedurales-animadas están en el skybox 3D, §4.2.)

### 2.5 Fondo estático (`§_-z1f§` StaticBackground)

Si no es tileable (o no hay GPU): busca `resKey_01.._04` (4 cuadrantes de **2048×2048** en (0,0),(2048,0),(0,2048),(2048,2048)) o un único `resKey`. Cada tile (`§_-s27§` BasicElementTile) es una `starling.display.Image` (o Bitmap sin GPU), con `region` opcional para recortar, fade-in 1 s, restore de contexto vía `CONTEXT3D_CREATE`.

### 2.6 Lensflares (`§_-u2p§` capa, `§_-51Z§` vista)

- La capa vive **fuera de Starling**, como sprite Flash dentro de Map3D (`addChild(§_-J38§)`); solo visible con `qualityBackground.high`.
- Actualiza cada ~500 ms (throttle `§_-W1K§ = 500`).
- Posición: pide al proveedor activo (`§_-B4E§` = fondo 2D o fondo 3D) que proyecte el flare a pantalla (`§_-7m§`).
- **Oclusión**: visible solo si está dentro del viewport y ningún `§_-z1R§(x,y)` da true. `Map3D.§_-z1R§` primero consulta los colliders 2D del fondo (bitmaps `_collider` de planetas/traits, alpha > 128 en `getPixel32`) y luego hace **raycast 3D** contra la escena Away3D (`§_-x1d§` picker con rayo cámara→punto). Fade-in 0.1 s / fade-out inmediato.
- Layout del flare: los N sprites `lens0..lensN` se colocan en la recta que pasa por el centro de pantalla: `lens_i.pos = i * (−(flare − centro) * 3 / N)` — es decir, la cadena de reflejos cruza el centro hacia el lado opuesto, hasta 3× la distancia. Si `star=true` hay un MovieClip "star" girando (−0.15°/frame ≈ −9°/s a 60fps) y un destello "lensFlash" (alpha 0→0.75 en 0.2 s → 0 en 3 s) al reaparecer.
- En mapas con fondo 3D, los lensflares salen del XML `display3D` (`<lensFlare x y z star blendMode>`) con mapeo de ejes `(x, −z, −y)` y se proyectan con la cámara 3D real (`mapView.§_-14v§`), con `blendMode` opcional.

## 3. El "fondo 3D" `§_-G2B§` (Background3D) — qué es

Respuesta corta: **no es solo un skybox; es una mini-escena Away3D por mapa** (contenedor `ObjectContainer3D`) **más** un skybox estrellado global que activa el renderer.

- Se define por mapa: si el XML del mapa tiene `<display3D templateId="...">`, se busca el descriptor en `UberAssetsLib` (`LIB_MAPS`) y se instancia `§_-115§` (UberAsset3D) con `display3D[0]`.
- En `Map3D.load`:
  ```actionscript
  §_-w2U§.map = mapa;                       // Background3D
  §_-U2d§.map = §_-w2U§.§_-F2Q§ ? null : mapa;  // si hay fondo 3D → APAGA el fondo Starling
  §_-yh§.§_-Y2f§ = §_-w2U§.§_-F2Q§;             // flag del renderer → activa DOSkybox
  §_-J38§.§_-B4E§ = hasBg3D ? fondo3D : fondo2D; // proveedor de lensflares
  ```
  (`§_-F2Q§` = "tiene fondo 3D" = el descriptor existía).
- El mismo `<display3D>` también define la **luz del mapa** (`<light color diffuse specular ambientColor ambient tilt pan>`; defaults del parser: color 0xFFFFFF, diffuse 1, specular 0.7, ambientColor 16756398 = 0xFFAEAE, ambient 0.2, tilt 100, pan 35 — pero los 6 mapas 3D los pisan todos con 0xA3FFFF/0.8/1.1/0xFF855C/0.5) y las **cameraZone** (§5).

### 3.1 Contenido del XML display3D (`§_-115§` UberAsset3D)

Nodos soportados (`parseDisplayChildren`): `container`, `mesh`, `plane`, `particles`, `pivot`, `display3D` (sub-asset por templateId), `tilemap`. Atributos de transform por nodo: `x,y,z`, `rotationX/Y/Z`, `scaleX/Y/Z`, y **coordenadas esféricas** `sphericalR/sphericalTheta/sphericalPhi` (colocan planetas/soles alrededor del origen a distancia R). Además `<floating moveX/Y/Z rotationX/Y/Z cycleLength>` para vaivén senoidal del asset (default ciclo 2 s, fase aleatoria).

- **`plane`** (`§_-x38§`): quad texturizado (imagen, ATF o MovieClip animado a N fps), `billboard` por defecto (se orienta a la posición de la cámara cada frame), con `alphaBlending/bothSides/blendMode`. Con esto montan nebulosas, soles y cartelones lejanos.
- **`tilemap`** (`§_-j4F§` TileMap3D): la joya del parallax real. Construye **una sola malla estática** con un quad por tile:
  - Base: `y = −3500 + layer*550` (debajo del plano de juego y=0; recuerda la escala global 0.0001).
  - **Cada tile recibe un offset vertical aleatorio propio**: `yTile = min(−500 + rand()*1000, −200)` → tiles repartidos entre −500 y −200 relativo a la base. Al estar a **distinta profundidad real**, el parallax entre tiles lo produce la propia cámara en perspectiva (no hay fórmula de scroll: es 3D de verdad).
  - `tileScale` default **5** (los tiles son 5× el tamaño 2D), `mapScale` default 1 con centrado del excedente.
  - Misma lógica de distribución que el 2D: máscara por `maskID`, orden ASC/DESC o aleatorio sin repetir vecinos.
  - Material: textura ATF, `smooth`, sin mipmaps, `alphaBlending`, sin premultiplicar. Fade-in 1 s.
- **`§_-r1X§`** (ZoneBackgroundManager): para zonas ("poizone"): genera un plano con textura de fondo + **máscara dibujada con la geometría de las zonas** (rects/círculos, incl. invertidas) y lo agrega a la escena 3D; se degrada a `simpleBackgroundID` en calidad baja.

### 3.2 El skybox `DOSkybox` (estrellas procedural-animadas)

Renderizado por el renderer custom `§_-n1h§` **antes que todo lo demás** cuando `§_-Y2f§ = true` (con `depthCompare LESS_EQUAL`; el skybox se mueve con la cámara: `mesh.moveTo(camTargetPos)`).

- Geometría: recurso `"skybox_geometry"` escalado ×10000.
- Texturas: `"skybox_stars"` (clamp) + `"skybox_mask"` (repeat).
- Shader (AGAL) — parpadeo de estrellas por doble máscara desplazada:
  ```
  t = getTimer()/1000 / 120        // timeScale = 120
  fc0 = [2t, t+1, −1.5t, t+0.3]
  m1 = mask(uv + fc0.xy)           // se desplaza a velocidad (2, 1)·t
  m2 = mask(uv + fc0.zw)           // se desplaza a velocidad (−1.5, 1)·t
  color = m1 * m2 * stars(uv)      // producto de dos máscaras móviles = titileo pseudoaleatorio
  ```
  Réplica en Godot: shader de cielo (o quad de fondo) con `stars_tex * mask(uv+v1*t) * mask(uv+v2*t)`, v1=(2,1)/120, v2=(−1.5,1)/120 UV/seg.

## 4. Cámara 3D (`§_-Jw§` + `§_-Q4t§`) — constantes exactas

- **Distancia**: `dist = distFactor * 1740 / zoom` (distFactor `§_-V4g§` normalmente 1; **1740** es la constante base).
- **zoom** (`§_-Q3H§`): clamp **[1, 3]** en mapas 3D (sin límite superior… mínimo 1; en modo "clásico" clamp [0.01, ∞) vía `§_-a2b§`). Rueda del ratón: ×1.2 / ×0.8 por tick; tween 0.3 s Quad.easeOut.
- **tilt** default **135°**, clamp [135, 179.9] (135° ⇒ cámara a 45° de elevación: `y = −dist*cos(135°) ≈ 0.707*dist` sobre el plano). En mapas 3D con zoom, el tilt efectivo baja hasta −20°: `tilt −= clamp((zoom−1)/2 * 20, 0, 20)` (a más zoom, cámara más rasante).
- **pan** default 0° (25° en mapas con fondo 3D). Right-drag: `pan −= 0.5*dx; tilt += 0.5*dy` (solo dentro de cameraZones o en modo libre).
- **FOV** default **30°** (clamp 2..180). **near = 10, far = 80000**.
- Posición (esférica): `pos = target + dist*(sin(tilt)sin(pan), −cos(tilt), −sin(tilt)cos(pan))`, lookAt(target, Y_AXIS). El target es `(camX, 0, −camY)`.
- **cameraZone** por mapa (`<cameraZone x y scale>` en display3D): al entrar el héroe (círculo de radio scale), tween 1 s a zoom 1.5 y pan +10°; al salir, restore a defaults (zoom 1, fov 30, tilt 135, pan al múltiplo de 360 más cercano). Solo la primera vez por zona (se marca en `map.storage`).
- Además `Map3D.zoom` (≤1) es un **render-scale**: escala el sprite del View3D y agranda el viewport lógico (1/zoom) — downscale de resolución para rendimiento, no zoom de juego.

## 5. Proyecciones mundo↔pantalla

```actionscript
// §_-V16§.§_-14v§  (mundo → pantalla)  [x,y mundo; z = altura]
v3 = (x, altura, −y);  p = camera.project(v3)      // NDC
out.x = viewportW * (p.x + 1) / 2
out.y = viewportH * (p.y + 1) / 2
out.z = cameraState.zoom (§_-Q3H§.value, 1..3)      // "escala" que reciben los HUD

// §_-V16§.§_-F4J§  (pantalla → mundo): rayo por unproject(2*mx/w−1, 2*my/h−1, 100)
// intersección con el plano y = altura, normal (0,1,0):
// t = −dot(n, camPos−p0) / dot(n, dir);  out = (hit.x, −hit.z, hit.y)
```

En 2D (`§_-d21§`): `screen = (world − cam) * zoom + halfViewport`, `z` siempre 1; inversa `world = cam + (screen − half)/zoom`. El zoom 2D es `viewportW/1280` (¡el zoom 2D solo compensa el tamaño de ventana contra un diseño de 1280 px!).

## 6. HUD sobre el mundo 3D (`§_-a2o§\HUD.as`)

- `HUD extends LayeredSprite`, creado como `new HUD(vista3D)` y `addChild` **encima** del View3D (display list Flash). `Settings.showHUD` alterna su `visible`.
- Patrón **trait → vista**: el HUD registra pares (clase de trait del modelo, clase de vista): `§_-F41§→§_-k1j§` (iconos clickeables de mapa), `§_-g3j§→§_-KV§` (barras de nave), `§_-o1d§→§_-g4Q§` (anillo de selección), `§_-T1Q§→§_-N2Y§` (labels de texto); debug: `§_-ta§/§_-F41§→§_-O17§`. `InternalDisplayLayer` escucha add/remove de traits del mapa y crea/destruye vistas; cada vista declara `layer` (LayeredSprite ordena por capa: barcos=0, iconos=4…).
- **Posicionamiento** (base `§_-h1d§` HUDElement, cada frame vía updatables del mapView):
  ```actionscript
  mapView.§_-14v§(pos.x, pos.y, pos.z, tmp);
  x = int(tmp.x);  y = int(tmp.y);      // ← SNAPPING A PÍXEL (int-cast) = anti-jitter
  this.§_-81u§ = tmp.z;                 // zoom 1..3 disponible para la vista
  ```
- **Los elementos HUD NO se escalan con el zoom**: tamaño constante en pantalla (texto, barras, iconos). El único uso del z/zoom es en `§_-KV§` (ShipHUDView): el **offset vertical** de las barras/nombre de la **nave propia** se multiplica por el zoom (`clickRadius * zoom` para que no se solapen con la nave al acercarse la cámara); para las demás naves el factor es 1. `§_-N2Y§` centra el texto (`x − w/2`).
- Anti-jitter adicional: la cámara lógica ya está en enteros (`camera.x = int(hero.x)`), y el fondo 2D también castea a int su scroll.
- Los **efectos de pantalla** (`§_-c2e§` ScreenEffectsFactory, `§_-418§ = new §_-c2e§(vista, hud)`) instancian vistas de efectos (4 tipos mapeados de `§_-p2q§` → clases en `§_-Wv§`: overlays tipo daño/curación/EMP…) y las agregan **al contenedor del HUD**, encima del mundo.

## 7. Click handling y prioridades

`ScreenManager.hitLayer` = sprite transparente (rect 100×100 estirado al viewport) agregado con `addChildAt(..., 0)` — **debajo de todo** en el display list. La cadena de prioridad la da Flash gratis:

1. **GUI/ventanas** (capas superiores del ScreenManager, `stage.addChild`) capturan primero.
2. **HUD**: los elementos interactivos (p. ej. `§_-k1j§` iconos de mapa) capturan su propio mouse.
3. **hitLayer** (el "suelo"): recibe lo que nadie más capturó. `§_-I1s§` (MapInputController) escucha `MOUSE_DOWN/MOVE/OUT/WHEEL` en el hitLayer y `MOUSE_UP/KEY_*` en el stage.

En `MOUSE_DOWN` (`§_-G2T§`):
- Doble click (<500 ms) con `doubleclickAttackEnabled` → `SelectionManager.§_-V7§` = atacar el target actual si el click cae en su radio.
- Modo grupo activo → unproyecta y manda ping de grupo.
- **Picking de entidades** (`SelectionManager.§_-A1a§`): **NO es raycast 3D** — recorre los `ClickAreaTrait` ordenados por `clickPriority` descendente, proyecta el centro de cada entidad a pantalla (`§_-o1U§` → `§_-14v§` + offsets `§_-c2W§`/`§_-H1o§`) y compara **distancia 2D en píxeles ≤ clickRadius** (radio constante en pantalla, no escala con zoom). Si acierta: selecciona/ataca (`TRY_TO_SELECT_MAPASSET`) y consume el click.
- Si nada acierta → **click-to-move**: oculta el marker, y mientras el botón siga presionado, cada frame (`ENTER_FRAME`) unproyecta el mouse al plano y=0 (`§_-F4J§`) y si `dist(hero, destino) > clickRadius + 45` manda `§_-E1M§` (throttle de **200 ms** entre comandos de movimiento al servidor; el destino se ajusta con `map.§_-K3h§` clamp a mapa). En `MOUSE_UP` se corta el enter-frame.
- Hover (`MOUSE_MOVE`): `SelectionManager.§_-z2I§` con la misma prueba de distancia; pone cursor de mano y dispara `mouseOver/mouseOut` del trait (cachea el último para salir barato).
- Rueda: zoom cámara (en el CameraController, tambien via hitLayer); con ALT ajusta el radio de drones (0.5..2 en pasos de 0.1).
- Right-drag (stage, captura con prioridad 1000): rotación de cámara (§4).
- El raycast 3D (`Map3D.§_-z1R§`) existe pero se usa para **oclusión de lensflares**, no para picking de naves.

## 8. Minimapa

- **Escala** (`§_-Nr§` MiniMap): `minimapPx = world / (P1k * 10)` donde `P1k = scaleFactorUsuario * map.scaleFactor`; el trait genérico usa `_scale = 1/(P1k*10)` y hace `icon.x = pos.x * _scale`. Tamaño del minimapa: `mapW/(P1k*10)`. Click en minimapa → `world = mouse * P1k * 10` (dispatch `clicked`).
- **Trapecio del viewport** (`MiniMapViewportBounds`, entidad del mapa con trait propio): Map3D calcula por frame las 4 esquinas del viewport unproyectadas al plano y=0 (`§_-F4J§(0,0) / (w,0) / (w,h) / (0,h)`) → `setViewport(x1..y4)`. Con la cámara inclinada resulta el **trapecio** característico. En 2D es el rect exacto (`cam ± half`).
- Dibujo: NO cierra el polígono; dibuja **solo las esquinas** — desde cada vértice, dos segmentos hacia los vecinos con longitud **12.5%** (`_loc2_ = 0.125`) del lado. `lineStyle(0.7, 0xCCCCCC, 0.5)` y `blendMode = INVERT` (se ve sobre cualquier fondo). Las coords se multiplican por el `_scale` del minimapa.

## 9. Starling: versión y configuración

- **Starling 1.5.1** (`starling.core.Starling.VERSION`).
- Root class: `§_-c12§` (Sprite Starling con API de compatibilidad IDisplayObject; expone mouseX/Y vía TouchPhase.HOVER cuando se activa).
- Modo 3D: `shareContext = true`, viewport = el del Stage3DProxy, perfil "baseline", sin error checking; `stage.stageWidth/Height` se sincronizan al viewport. **Solo contiene el contenedor "bg"** (el fondo espacial). Nada más se renderiza en Starling en modo 3D (lensflares y HUD son Flash; efectos son Flash sobre el HUD).
- Modo 2D: Starling normal (no compartido, "auto"), y ahí dentro va **todo el mundo 2D** (`_map.§_-R1V§`: fondo + entidades), con HUD Flash encima igual.
- Texturas: preferencia ATF (`resKey_atf`) si `Settings.useATF`; `ConcreteTexture.onRestore` re-carga tras context-loss; `TexturesCache.§_-03P§` re-sube data a texturas vivas.

## 10. Contraste con el cliente 2D (qué gana el 3D)

El 2D (`§_-q3t§`/`§_-d21§`) usa la MISMA infraestructura de fondo (`§_-Kt§` con idéntica fórmula `update(camX, camY, halfW/zoom, halfH/zoom)`), mismos lensflares, mismo HUD y mismo input. Diferencias:

| | 2D | 3D |
|---|---|---|
| Mundo | sprites Starling planos, `screen=(world−cam)*zoom+half` | mallas Away3D, cámara en perspectiva |
| Zoom | fijo: `viewportW/1280` (solo adapta resolución) | real: distancia 1740/zoom, tilt dinámico, cameraZones |
| Parallax | solo el del fondo (división por pFactor) | fondo 2D igual + tilemaps 3D a profundidad real + skybox + billboards |
| Viewport en minimapa | rectángulo | trapecio (unproject de 4 esquinas) |
| z del HUD | siempre 1 | zoom 1..3 (solo se usa para offset de barras propias) |
| Picking | mismo (distancia 2D a centros proyectados) | mismo |

Conclusión: el 3D "gana" cámara perspectiva con zoom/rotación, profundidad real en fondos y oclusión 3D de lensflares — pero **mantiene el gameplay y el HUD 100% en lógica 2D proyectada**, que es exactamente el modelo cómodo para replicar en Godot.

## 11. Receta de réplica en Godot (mapeo directo)

1. **Orden de render**: fondo (CanvasLayer o quad al fondo) → mundo → HUD (CanvasLayer). En Godot esto es gratis con CanvasLayers; no hace falta compartir contexto.
2. **Fondo por mapa (datos)**: lista de capas `{textura/atlas, pFactor (default 10), layer, shift, scale, tiled?, tileW/H, order, mask}` + planetas `{tex, x, y, rot, pFactor, layer}` + lensflares `{tex, x, y, pFactor|z, star}`. Orden de dibujo: `1000/pFactor + layer` ascendente.
3. **Scroll**: por capa, `pos_pantalla = floor(−cam/pFactor) + viewport/2 + offset`; planetas `floor((p − cam)/pFactor) + viewport/2`. Snap a entero SIEMPRE (es su anti-jitter).
4. **Tiles**: grid `round(size/tile)`; aleatorio sin repetir vecino izq/arriba; máscara opcional sampleada al centro de celda; margen de 1 tile alrededor del viewport.
5. **Estrellas animadas**: shader `stars(uv) * mask(uv + (2,1)t/120) * mask(uv + (−1.5,1)t/120)`.
6. **Cámara 3D** (si aplica): dist 1740/zoom, zoom 1..3 (wheel ×1.2/×0.8, tween 0.3 s), tilt 135° (−hasta 20° con zoom), FOV 30°, pan 0.
7. **HUD**: cada frame `screen = project(entidad)`, `pos = Vector2(int(x), int(y))`; tamaño constante; solo el offset vertical de las barras propias × zoom.
8. **Clicks**: 1º UI, 2º HUD, 3º mundo. Picking = distancia 2D al centro proyectado ≤ clickRadius (px constantes), prioridad por clickPriority; si falla → click-to-move con unproject a plano, umbral `clickRadius+45`, throttle 200 ms, repetición mientras se mantenga presionado.
9. **Minimapa**: `mini = world * k`; viewport como 4 esquinas unproyectadas, dibujando solo el 12.5% de cada lado desde cada esquina, línea fina clara semitransparente.
10. **Lensflares**: cadena de N sprites sobre la recta flare↔centro extendida ×3, oclusión por raycast/collider, fade 0.1 s in / instantáneo out, throttle 500 ms.
