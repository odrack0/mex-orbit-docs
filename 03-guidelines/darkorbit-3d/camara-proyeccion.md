# DarkOrbit 3D — Cámara, proyección, renderer y luces (ingeniería inversa)

Fuente: `D:\MexOrbit\Decompiled\spacemap\main\scripts\` (cliente Flash/Stage3D, Away3D 4.x + Starling, decompilado con JPEXS).
Convención de nombres: `§_-XX§ (NombreDeducido)`.

---

## 0. Mapa de clases (anclas verificadas)

| Ofuscado | Deducido | Evidencia |
|---|---|---|
| `net\...\view3D\§_-i3S§.as` | **MapView3D (fachada)** | Crea Stage3DProxy, Starling shareContext, HUD; render loop |
| `net\...\view3D\§_-V16§.as` | **Map3D** | Escena, View3D, proyección/desproyección, zoom |
| `§_-83r§\§_-d1G§.as` | **View3D** (Away3D) | width/height→lens.aspectRatio, filters3d, render |
| `§_-83r§\§_-T1c§.as` | **ObjectContainer3D** | extends `§_-r2u§\§_-p2G§` (Object3D), children, sceneTransform |
| `§_-r2u§\§_-p2G§.as` | **Object3D** | translate/rotate/lookAt/rotateTo |
| `§_-v1N§\§_-B1Z§.as` | **Camera3D** | lens, frustum de 6 planos, project/unproject, `z=-1000` inicial |
| `§_-O2n§\§_-M2§.as` | **LensBase** | near=20, far=3000 por defecto, project con y invertida |
| `§_-O2n§\§_-52g§.as` | **PerspectiveLens** | fieldOfView (default 60), focalLength=1/tan(fov·π/360) |
| `§_-321§\§_-Jw§.as` | **CameraController3D** | input, zonas, drag, aplica §_-Q4t§ a Camera3D |
| `§_-321§\§_-Q4t§.as` | **CameraModel (orbital)** | zoom/tilt/pan/FOV con tweens, posición esférica |
| `§_-321§\§_-03b§.as` | **MathUtils** | `§_-tb§` = delta angular más corto (grados); `§_-U1v§` = esférico→vector |
| `§_-B2U§\§_-h15§.as` | **BindableNumber** | value con clamp [min,max] + señal changed |
| `§_-23a§\§_-n1h§.as` | **DODefaultRenderer** | extends `§_-H2m§\§_-92m§` (DefaultRenderer); skybox + filtro de pasadas |
| `§_-23a§\§_-My§.as` | **DOEntityCollector** | extends `§_-RA§\§_-q1j§` (EntityCollector) **sin cambios** |
| `§_-i22§\DOSkybox.as` | **DOSkybox** | esfera escala 10000 centrada en el lookAt, shader propio |
| `§_-i22§\§_-G2B§.as` | **Background3D** | asset 3D del mapa (LIB_MAPS) + lens flares |
| `net\...\display3D\§_-i7§.as` | **StarDustLayer** | partículas "star_dust_chaotic" en tiles de 2000 u |
| `net\...\display3D\§_-f4D§.as` | **TargetMarker3D** | anillo animado bajo el objetivo fijado |
| `net\...\display3D\LightsManager.as` | — | 1 direccional + point del héroe + pool de 3 efectos |
| `§_-J0§\§_-Q1T§.as` | **LightSettings** | tilt/pan→dirección de la luz |
| `§_-n3Z§\§_-115§.as` | **UberAsset3D** | vista de entidad desde XML (mesh/plane/particles/floating) |
| `§_-3L§\§_-DX§.as` | **ShipDecorator3D** | yaw suavizado + banking + floating |
| `§_-Oj§\§_-73c§.as` | **HeroDecorator3D** | point light del héroe + textura ×2 al hacer zoom |
| `§_-a2H§\§_-J2P§.as` | **GameStage/World** | `static camera:Point`, shake, loop |

---

## 1. Cámara: perspectiva, FOV, near/far, altura y tilt

**Es una cámara en PERSPECTIVA** (`§_-52g§` = PerspectiveLens; no hay lente ortográfica en uso).

En `§_-V16§ (Map3D)` ctor (línea 128): `new §_-52g§(30)` → **FOV vertical = 30°**.

En `§_-Jw§ (CameraController3D).update()` (líneas 230–234), CADA frame:
```as3
this.§_-f3t§.position = this._cam.position;          // posición desde el modelo orbital
this.§_-f3t§.§_-032§(this._cam.§_-1z§, Vector3D.Y_AXIS); // lookAt(target, up=Y)
this.§_-f3t§.lens.§_-I29§ = 10;                       // near = 10
this.§_-f3t§.lens.§_-E2F§ = 80000;                    // far  = 80000
§_-52g§(lens).fieldOfView = this.§_-J41§.§_-Jb§.value; // FOV desde el modelo (default 30)
```
- **near = 10, far = 80000** (unidades de juego; pisa los defaults de LensBase 20/3000).
- **FOV** es un `BindableNumber(30, min 2, max 180)` (`§_-Q4t§.§_-Jb§`); en la práctica solo se restaura a 30 con tween — no encontré gameplay que lo cambie.

### Modelo orbital `§_-Q4t§ (CameraModel).validate()` (líneas 118–140) — la fórmula exacta

```as3
d = §_-V4g§(=1, nunca cambia) * 1740 / zoomFactor;      // §_-Q3H§ = zoomFactor
pos.x = lookAt.x + d·sin(tilt°)·sin(pan°)
pos.y = lookAt.y − d·cos(tilt°)
pos.z = lookAt.z − d·sin(tilt°)·cos(pan°)
```
Constantes (`§_-Q4t§`):
- **Distancia base `§_-L38§` = 1740** unidades de juego (a zoom 1).
- **tilt default `§_-U3B§` = 135°** (135 = mínimo permitido `§_-e4j§`; máximo `§_-H15§` = 179.9 = cenital).
  - Semántica: **elevación sobre el plano = tilt − 90°**. tilt 135° → la cámara mira la nave a **45° sobre el horizonte**; tilt 179.9° → picado total.
- **pan default `§_-66§` = 0°**; en mapas con fondo 3D el controlador fija **pan = 25°** al cargar (`§_-Jw§.§_-H3b§`, línea 180).
- **FOV default `§_-U2H§` = 30°**.

Números derivados (zoom 1, tilt 135): d=1740 → **altura de cámara = 1740·cos45° ≈ 1230.4 u** sobre el plano y **retroceso horizontal ≈ 1230.4 u**. Alto visible en el plano focal ≈ 2·1740·tan15° ≈ **932 u** (sobre el suelo se estira por el tilt: la franja visible en profundidad ≈ 1421 u con estos ángulos).

### Acoplamiento tilt↔zoom (solo mapas con fondo 3D)
`§_-Q4t§.validate()` líneas 128–135: si `§_-D2L§` (mapa 3D) y `Settings3D.§_-t3m§` (true por defecto):
```
tiltEfectivo = tilt − clamp( (zoom−1)/(3−1) · 20, 0, 20 )
```
Es decir, **al hacer zoom-in (1→3) la cámara baja hasta 20° hacia el horizonte** (a zoom 3: tilt 115° → elevación 25°, d=580, altura ≈ 245 u). Efecto cinemático de "acercarse en picado bajo".

### Punto que mira la cámara
`§_-Jw§.update(dt,x,y)` → `_cam.§_-032§(x, 0, -y)`: **lookAt = (heroX, 0, −heroY)**, sin suavizado en la capa 3D (ver §2.5).

---

## 2. Controlador de cámara `§_-Jw§` completo

### 2.1 Seguimiento
- `§_-i3S§.update(x, y, dt)` la llama `§_-J2P§ (GameStage)` línea 641: `this._mapView.update(camera.x, camera.y, dt)`.
- `§_-J2P§.camera` es un `static Point`. Cada tick (líneas 605–609): **`camera.x = int(hero.x); camera.y = int(hero.y)` — seguimiento RÍGIDO, sin lerp ni deadzone** (la suavidad viene de que la posición del héroe ya está interpolada por el modelo de movimiento).
- Cámara cinemática por servidor (`net\§_-y2b§.as`): TweenLite sobre `§_-J2P§.camera` — pan a un punto (duración del servidor), **volver al héroe en 3 s**, o seguir a otra nave (`dynamicProps`). Estados: `§_-a1z§` (FREE/TWEEN/FOLLOW/HERO...).

### 2.2 Zoom
- `§_-Q4t§.zoom setter`: **clamp [1, 3]** (con `§_-t3m§`=true), y **tween del factor `§_-Q3H§` en 0.3 s Quad.easeOut**. La distancia es **d = 1740/zoomFactor** → zoom continuo, no discreto.
- Rueda del ratón (`§_-Jw§.§_-p31§`, sin Alt): `zoom *= 1.2` (acercar) o `*= 0.8` (alejar). `zoomIn()/zoomOut()` públicos hacen lo mismo.
- **El zoom por escala de Sprite del `§_-V16§` está muerto**: `zoom setter` clampa entre `§_-A2z§`(=1, jamás reasignado) y 1 → siempre 1. Todo el zoom real es de cámara.
- El factor de zoom `§_-Q3H§.value` se exporta como `z` en la proyección mundo→pantalla (los elementos HUD lo usan como factor de escala, ver §4).

### 2.3 Zonas de cámara (cameraZone)
- Del XML del mapa (`UberAssetsLib LIB_MAPS → display3D.cameraZone @x @y @scale`): **círculos de radio = `scale`** (`§_-b41§\§_-SA§`: bounds = pos±radio, tipo "CAMERA").
- Chequeo **1 vez por segundo** (acumulador en `update`). Al ENTRAR por primera vez a una zona (se recuerda en `map.storage`): `_cam.zoomIn()` = **tween 1 s Quad.easeOut: zoomFactor→1.5 y pan += 10°**. Al salir: `restoreDefaults(false, true)` = tweens de 1 s: zoom→1, FOV→30, **pan→0 por el camino angular más corto** (`§_-03b§.§_-tb§`), tilt→135.
- Dentro de una zona (o con `§_-t3m§`=false, cámara libre de debug) se habilita **órbita con botón derecho**: `pan −= 0.5·Δmx; tilt += 0.5·Δmy` con tilt clamp [135, 179.9]. Al soltar: restaura FOV/pan/tilt (1 s) sin tocar el zoom.

### 2.4 Inicialización por mapa (`§_-H3b§`)
- Mapa con fondo 3D: `zoom=1` directo, `pan=25°`, engancha al héroe; a los 0.2 s de que su nave esté lista decide zona/estado.
- Mapa con fondo 2D: `restoreDefaults(true)` instantáneo (zoom 1, FOV 30, pan 0, tilt 135) y sin acoplamiento tilt-zoom.

### 2.5 Shake de cámara

> **CORRECCIÓN (ago-2026, verificada jugando DO 3D)**: el shake NO dispara con daño normal. Sus únicos disparadores son el tipo de daño `"I"` (detonaciones tipo mina/kamikaze — `§_-01b§\§_-j3H§.as:79` lo exige) y los efectos cuyo XML declara `shakeScreen="true"` (`§_-H4R§`, `§_-Z1w§`). Ver la corrección en el §3 del doc principal.
`§_-J2P§.shakeScreen()` (líneas 288–293, 610–631): **amplitud inicial 5 u, 40 pasos, 1 paso cada 24 ms (~1 s), amplitud −1 cada 10 pasos**; desplaza el foco `camera.x += amp·cos(paso); camera.y += amp·sin(paso)` (ángulo = índice del paso en radianes → pseudo-aleatorio). Se aplica al punto de mira, no a la cámara 3D directamente.

---

## 3. El "scale 0.0001" — CORRECCIÓN IMPORTANTE: no es scale, es rotateTo

`§_-V16§` línea 137: `this.§_-B4K§.§_-g4u§(0.0001, 0.0001, 0.0001)`.
`§_-g4u§` está en `§_-r2u§\§_-p2G§ (Object3D)` líneas 593–599:
```as3
this._rotationX = param1 * §_-O1Y§.§_-S2T§;  // §_-S2T§ = 0.017453292519943295 = DEG2RAD (§_-g2d§\§_-O1Y§ = MathConsts)
```
→ **`§_-g4u§` = `rotateTo(gradosX, gradosY, gradosZ)`**. El contenedor raíz recibe una **rotación ínfima de 0.0001° por eje** — un workaround clásico de Away3D para sacar la transform del caso identidad (evita bugs de lookAt/decompose con rotación exactamente 0). **NO hay escala de escena.**

**Consecuencia: 1 unidad 3D = 1 unidad de juego.** La cámara está a 1740 u, el far plane a 80000 u cubre el mapa clásico de ~21000×13100 u en diagonal. Los tamaños de naves salen del XML de assets (`scaleX/Y/Z`, `previewScale` — no hay constante en código); las posiciones de entidad se ponen 1:1 en unidades de juego. La precisión del depth buffer se maneja con near=10 (no con escala).

### Mapeo de coordenadas (verificado en 3 sitios)
`§_-o1k§\§_-r1d§` (Entity model) líneas 80–93:
- **3D.x = juego.x; 3D.y = juego.z (altura); 3D.z = −juego.y**
- Igual en `§_-V16§.§_-14v§` (x, alt, −y) y en los lens flares de `§_-G2B§` (x, −z, −y).

---

## 4. Proyección y desproyección exactas

### Mundo→pantalla `§_-V16§.§_-14v§(x, y, alt)` (líneas 267–281)
```as3
v = (x, alt, −y);
n = camera.§_-P4B§(v);            // Camera3D.project → coords normalizadas [−1,1], y ya invertida en LensBase.project (y = −y/w)
out.x = viewportW · (n.x + 1) / 2;
out.y = viewportH · (n.y + 1) / 2;
out.z = zoomFactor (§_-Q3H§.value);   // ¡no es profundidad! es el factor de zoom
```
`§_-B1Z§ (Camera3D).§_-P4B§` = `lens.project(inverseSceneTransform · v)` — idéntico a Away3D `Camera3D.project`.

**Consumidores** (todos posicionan Sprites 2D del display list de Flash sobre el 3D):
- `§_-a2o§\§_-h1d§` (HUDElement3D): `x=int(p.x); y=int(p.y); escala = p.z` → **el HUD escala con el zoom de cámara**.
- `traits\§_-ta§`: proyecta `owner.(x,y,z)+offset` para etiquetas/barras.
- `§_-Wv§\§_-715§`: proyecta 2 extremos de un efecto (rayos/tractor) a pantalla.
- `§_-G2B§.§_-7m§`: proyecta la posición 3D de cada lens flare; el layer de flares `§_-FT§\§_-u2p§` es un Sprite 2D encima.

### Pantalla→mundo `§_-V16§.§_-F4J§(mx, my, alt)` (líneas 283–311) — ray/plano
```as3
o = camera.§_-Q2X§;                                  // scenePosition (global)
p = camera.§_-T11§(2mx/W − 1, 2my/H − 1, 100);        // unproject a profundidad 100 (escena)
dir = p − o;  plano: punto (0, alt, 0), normal (0,1,0);
den = dot(n, dir); si |den| ≤ 1e−6 → null (rayo paralelo);
t = −dot(n, o − punto)/den;  hit = o + t·dir;
return (hit.x, −hit.z, hit.y)                         // de vuelta a coords de juego (x, y, altura)
```
`Camera3D.§_-T11§` = `sceneTransform · lens.unproject(nX, nY, depth)`; `PerspectiveLens.unproject` multiplica (nX,−nY) por la profundidad y aplica la inversa de la matriz de proyección — Away3D estándar.

**Consumidores:**
- **Click-to-move** (`§_-a2H§\§_-I1s§` líneas 493–505): mientras el botón está pulsado, en cada ENTER_FRAME desproyecta el ratón al plano y=0 y, si `dist(héroe, punto) > hero.hitArea.clickRadius + margen`, envía el comando de movimiento. (El "conducir manteniendo pulsado" clásico.)
- **Minimapa** (`§_-V16§.§_-R4T§`, líneas 328–338): cada frame desproyecta las **4 esquinas del viewport** al plano y=0 y pasa el **trapecio** a `MiniMapViewportBounds.setViewport(x1..x4,y1..y4)` — por eso el rectángulo del minimapa en 3D es un trapecio.
- **Hover/picking** (`§_-V16§.§_-z1R§`): primero hit-test 2D del fondo, si no, raycast 3D real (`§_-9W§`/`§_-x1d§` = RaycastPicker) desde la cámara con el rayo desproyectado.

---

## 5. Renderer `§_-n1h§ (DODefaultRenderer)` — §_-23a§

- Extiende `§_-H2m§\§_-92m§` (**DefaultRenderer** de Away3D). Su collector `§_-My§` extiende el EntityCollector estándar **sin ninguna modificación** → culling por frustum y orden estándar de Away3D (opacas agrupadas por material/estado, transparentes por profundidad).
- **`§_-Y2f§` = "renderizar skybox propio"**: `§_-V16§.load()` línea 206 lo pone a `true` si el mapa tiene fondo 3D (`§_-G2B§.§_-F2Q§`). En `draw()` (y en el pre-pass con render target) **dibuja `DOSkybox` ANTES de la escena**.
- `draw()`: `setBlendFactors(ONE, ZERO)` + `setDepthTest(true, LESS_EQUAL)` → dibuja lista opaca (`§_-V2w§`) y lista con blending (`§_-gs§`) con un **filtro de pasadas en dos fases**: cada pasada de material se clasifica con `§_-52Z§(pass)` en bit 1 o bit 2; con render target la fase 1 se dibuja antes (`§_-A4b§`) y el draw principal usa máscara 2; sin target usa 3 (ambas). Al final **`setDepthTest(false, LESS_EQUAL)`** — necesario para devolver el contexto compartido a Starling.
- Sin sombras, sin post-proceso: `§_-V16§.§_-e2C§` construye `filters3d` **siempre vacío** (el `if(Settings3D.effects.max)` tiene el cuerpo vacío — código muerto/recortado).
- Loop por frame (`§_-i3S§.update`, líneas 136–139): `stage3DProxy.clear()` → `starling.render()` (fondo 2D + capa "bg") → `map.render()` (View3D) → `present()`. Perfil Context3D **"baseline"**. Antialias por calidad (`§_-QK§`): **MEDIUM=2, HIGH=8, ULTRA=16, resto 0**.

### DOSkybox (mapas con fondo 3D)
- Geometría "skybox_geometry" **escalada ×10000**, recentrada cada frame en el **punto que mira la cámara** (no en la cámara).
- Shader propio: textura de estrellas (clamp) × 2 muestras desplazadas de una máscara (repeat) que se desplazan con `t/1000/120` (offsets `2t, t+1, −1.5t, t+0.3`) → **parpadeo/twinkle lento de estrellas**. Sin escritura de depth, LESS_EQUAL.

### Capas de la escena (orden de hijos del contenedor raíz `§_-B4K§`, `§_-V16§` ctor)
1. `§_-E1B§ (TargetMarker3D)` — anillo bajo el objetivo (escala del descriptor de la nave ×1.5→×1 en 0.3 s al fijar).
2. **La cámara** (hija del contenedor raíz — hereda la rotación épsilon).
3. `§_-w2U§ (Background3D)` — asset 3D del mapa completo + lens flares.
4. Contenedor de entidades → `MapDisplayLayer3D` (traits → vistas: naves `§_-31O§`, etc.).
5. `§_-838§ (StarDustLayer)` — con efectos ≥ medium: clona un sistema de partículas (`star_dust_chaotic` o `@starfield` del mapa) en **tiles de 2000×2000 u** por todo el mapa, en y∈[0,−300] → polvo estelar con paralaje real.

---

## 6. LightsManager (display3D\LightsManager.as) — completo

**Luces:** 1 DirectionalLight "sol" (`§_-s1a§`), 1 PointLight del héroe (`§_-N4Y§`), **pool de exactamente 3 PointLights de efectos** (`§_-X1A§`×3), y un DirectionalLight nulo (`§_-B4N§`, todo a 0, no entra en los pickers). **Sin sombras** (ningún ShadowMapMethod en uso).

**Dos light pickers** (`§_-x1Y§\§_-W31§` = StaticLightPicker):
- `§_-c2F§` (completo): sol + héroe + efectos visibles — para los materiales de naves/objetos.
- `§_-R4X§` (solo sol): poblado únicamente si `effects.high` — para materiales caros.

**Calidad (`displaySetting3DqualityLights`):** LOW = sin luces; MEDIUM = solo sol + 1 luz de efecto; HIGH = sol + héroe + todas.

**Dirección del sol desde tilt/pan** (`§_-Q1T§.apply` → `§_-03b§.§_-U1v§(tilt, pan, −1)`):
```
dir = −( sin(tilt°)·sin(pan°),  sin(tilt°)·cos(pan°),  cos(tilt°) )
```
Con los defaults del XML del mapa (**tilt=100, pan=35**): dir ≈ **(−0.565, −0.807, +0.174)** — luz casi cenital, ligeramente lateral. Defaults del XML (`§_-V16§.§_-32A§`): color 0xFFFFFF, diffuse 1, specular 0.7, **ambientColor 0xFFA82E, ambient 0.2**, tilt 100, pan 35. (Preset de código `§_-s37§`: 0xA3FFFF, d 0.8, s 1.1, ambient 0xFF855C/0.5 — lo pisa el XML.)

**Luz del héroe** (`§_-Q4c§`): PointLight **azul 0x2E7DFF, diffuse 0.6, specular 1.5, fallOff 450**, radius 0; **parentada a la vista de la nave del héroe** (`§_-73c§.initialize`) — la nave del jugador "brilla" y se distingue.

**Luces de efectos** (`§_-Qz§`): del pool, `moveTo(x,y,z)` + color + TweenMax de diffuse/specular/fallOff, autodesvanecen tras `param8` s (default 0.06). Presets en `Settings3D`: `§_-411§` (0xF7C0C0, d1, s2, fallOff 200 — ¿disparo?), `§_-B9§` (0xDEE4C8, d0, s20, fallOff 400 — ¿explosión?), `§_-e2§` (negro, s3, fallOff 150).

---

## 7. Otros hallazgos jugosos

### 7.1 Sensación de la nave — `§_-3L§\§_-DX§ (ShipDecorator3D).§_-i2o§` (líneas 228–284)
- **Yaw**: objetivo = `ship.rotation.degrees − 90` (offset del modelo). Suavizado: `rotY += deltaMásCorto · QuadEaseOut(dt / 0.2)` → **~0.2 s de respuesta al girar**.
- **Banking (alabeo)**: objetivo = **1× el error de yaw, clamp ±20°**; si es el héroe con objetivo fijado: **−2× el error, clamp ±10°, respuesta 0.08 s** (se inclina al lado contrario, más rápido y más contenido, al combatir). Se aplica como `Rz(−roll)·Ry(yaw)` → decompose → rotateTo.
- **Flotación idle**: amplitudes **pos (5,5,5) u y rot (5,5,5)°, ciclo 2 s** (`§_-DX§.decorate`); oscilador `§_-c1o§`: componentes `(sin·cos, sin², cos·sin)` de la fase — vaivén orgánico no sinusoidal puro. Configurable por XML (`floating @moveX...@cycleLength`); desactivable con `hoverShips`.
- **Cloak**: alpha 0.5, tween 0.2 s.
- **Placeholder**: si el modelo 3D no está listo en 1 s (héroe 2.5 s), monta un **plano con el sprite 2D de la nave** (scaleX=alto, scaleZ=ancho del MovieClip, rotado 90°) — transición 2D→3D invisible para el jugador.
- Héroe (`§_-73c§`): con zoomFactor > 1.3 sube la calidad de textura del mesh ×2.

### 7.2 Zoom del HUD
Los elementos HUD 2D usan el `z` devuelto por la proyección (= zoomFactor) como escala → **el HUD "acompaña" el zoom de la cámara** aunque viva en el display list 2D.

### 7.3 Settings relevantes
- `Settings3D.§_-t3m§` (default **true**) = "cámara restringida": clamps de zoom [1,3] y tilt [135,179.9], órbita solo en zonas. Con false ⇒ cámara libre (debug).
- `Settings3D.render` permite congelar el render (debug).
- Tamaños de textura por categoría de asset (ship 128/256, building 512/1024, planet 1024… según calidad).

### 7.4 Sin clamping en bordes del mapa
No existe ningún clamp de la cámara a los límites del mapa: siempre centrada en el héroe (o en el Point cinemático). Cerca del borde simplemente se ve "fuera" del mapa (el fondo 3D/skybox lo cubre).

### 7.5 Fondos 2D vs 3D
El mismo view 3D sirve mapas con fondo 2D: el bitmap con paralaje (`view2D\backgrounds\§_-Kt§`) se dibuja vía **Starling** (capa "bg" bajo el 3D) y el skybox/fondo 3D se apagan (`§_-Y2f§=false`). La decisión es por mapa (`display3D.@templateId` presente ⇒ 3D).

---

## 8. Receta para replicar la sensación en Godot (resumen operativo)

1. Cámara **perspectiva, FOV vertical 30°**, near 10, far 80000 (en unidades de juego, 1:1).
2. Pivot en la nave (x, 0, −y); cámara a **d = 1740/zoom** con **elevación 45°** (tilt 135) y **azimut 25°** en mapas "3D" (0° si quieres el look plano); `look_at(pivot, Vector3.UP)`.
3. Zoom continuo [1,3] por rueda ×1.2/×0.8, **tween 0.3 s ease-out-quad del factor** (no de la distancia directamente); al hacer zoom-in restar hasta 20° de elevación linealmente.
4. Seguimiento rígido al héroe interpolado + shake aditivo (amp 5→0 u, paso 24 ms, ~1 s).
5. Click-to-move por ray-plane contra y=0, repetido cada frame mientras el botón esté pulsado.
6. Nave: yaw ease-out 0.2 s, banking = error de yaw clamp ±20° (combate héroe: −2×, ±10°, 0.08 s), flotación (5 u, 5°, 2 s).
7. Luz: 1 direccional (dir por tilt 100/pan 35), ambiente cálido 0xFFA82E·0.2, specular 0.7; point azul 0x2E7DFF en la nave propia (fallOff 450); sin sombras.
8. Fondo: skybox de estrellas con twinkle (máscara desplazándose lentamente) + polvo de partículas tileado cada 2000 u en y∈[0,−300] para paralaje.
9. HUD 2D proyectado con la matriz de cámara y escalado por el factor de zoom.
