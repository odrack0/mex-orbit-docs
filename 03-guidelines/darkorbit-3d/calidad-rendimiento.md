# DarkOrbit 3D — Calidad, rendimiento y robustez (reverse-engineering del cliente decompilado)

Fuente: `D:\MexOrbit\Decompiled\spacemap\main\scripts\` (JPEXS). Los nombres `§_-XX§` son los reales del decompilado; el nombre entre paréntesis es el deducido.

## 0. Mapa de clases clave

| Clase ofuscada | Nombre deducido | Ruta |
|---|---|---|
| `§_-J0§\Settings3D` | Settings3D (fachada de calidad 3D) | `§_-J0§\Settings3D.as` |
| `§_-J0§\§_-w2X§` | TextureQualitySettings (por categoría de asset) | `§_-J0§\§_-w2X§.as` |
| `§_-J0§\§_-Q1T§` | LightSettings (bindable) | `§_-J0§\§_-Q1T§.as` |
| `net...settings\Settings` | Settings globales (SharedObject) | `net\bigpoint\darkorbit\settings\Settings.as` |
| `net...settings\BindableSettings` | Setting entero con booleanos derivados low/medium/high/max | ídem |
| `net...settings\Profiler` | Auto-quality por FPS | ídem |
| `net...map\§_-B4r§` | MapViewFactory (elige 2D/3D) | `net\bigpoint\darkorbit\map\§_-B4r§.as` |
| `net...map\§_-C1P§` | IMapView (interfaz común 2D/3D) | `net\bigpoint\darkorbit\map\§_-C1P§.as` |
| `net...view3D\§_-i3S§` | MapView3D (orquestador Stage3D+Starling+Away3D) | `net\bigpoint\darkorbit\map\view3D\§_-i3S§.as` |
| `net...view3D\§_-V16§` | Map3D (escena, cámara, updatables) | `net\bigpoint\darkorbit\map\view3D\§_-V16§.as` |
| `net...view2D\§_-q3t§` | MapView2D (Starling puro) | `net\bigpoint\darkorbit\map\view2D\§_-q3t§.as` |
| `§_-G43§\Stage3DManager` | Stage3DManager (multiton por Stage) | `§_-G43§\Stage3DManager.as` |
| `§_-G43§\§_-d3k§` | Stage3DProxy | `§_-G43§\§_-d3k§.as` |
| `§_-83r§\§_-d1G§` | View3D de Away3D (fork) | `§_-83r§\§_-d1G§.as` |
| `§_-RA§\§_-q1j§` | EntityCollector (frustum culling + numTriangles) | `§_-RA§\§_-q1j§.as` |
| `§_-a2H§\§_-J2P§` | ScreenManager (dueño del frame loop) | `§_-a2H§\§_-J2P§.as` |
| `net...display3D\§_-o2x§` | AssetsManager3D (singleton) | `...view3D\display3D\§_-o2x§.as` |
| `net...display3D\LightsManager` | LightsManager (pool de luces) | ídem |
| `net...display3D\§_-i7§` | Starfield3D (partículas tileadas) | ídem |
| `§_-n3U§\TextureLoader / GeometryLoader / §_-u1u§` | Cachés de texturas / geometría / partículas con "lifetime" | `§_-n3U§\` |
| `§_-X2m§\§_-5B§` | ATFTexture (upload asíncrono) | `§_-X2m§\§_-5B§.as` |
| `§_-jh§\§_-X34§` | MaterialManager (materiales compartidos ref-counted) | `§_-jh§\§_-X34§.as` |
| `§_-321§\§_-eI§` | EffectPool3D (pool por clave) | `§_-321§\§_-eI§.as` |
| `§_-a2H§\§_-13C§` | EffectPool2D (pools de MovieClip precacheados) | `§_-a2H§\§_-13C§.as` |
| `§_-s3u§` (paquete raíz) | RenderStats (contadores globales) | `§_-s3u§.as` |
| `§_-s1i§\Stats` | Panel FPS/MS/MEM (mrdoob-style) | `§_-s1i§\Stats.as` |
| `§_-O2n§\§_-52g§` | PerspectiveLens (FOV default 60; el juego usa 30) | `§_-O2n§\§_-52g§.as` |

---

## 1. Settings3D completo

### 1.1 Settings persistidos (SharedObject "darkorbit", default 3 = ULTRA)
En `Settings.as` (líneas 252-264):

- `FORCE_2D` (bool, key "force2D", default false)
- `displaySetting3DqualityAntialias` — alias `Settings3D.§_-b1E§`
- `displaySetting3DqualityLights` — alias `Settings3D.§_-J34§`
- `displaySetting3DqualityTextures` (mapas normal/specular/glow on-off)
- `displaySetting3DsizeTextures` (resolución de texturas)
- `displaySetting3DqualityEffects` — alias `Settings3D.effects`
- `displaySetting3DtextureFiltering` (smoothing/mipmapping)
- Además `Settings3D.background` = `Settings.qualityBackground` y `Settings3D.zones` = `Settings.qualityPoizone` (compartidos con el cliente 2D).

`BindableSettings`: valor entero NONE(-2^31)/LOW(0)/MEDIUM(1)/HIGH(2)/ULTRA(3) + **booleanos bindables derivados** `low/medium/high/max` calculados como `value >= umbral`, con señal `changed`. Todo chequeo en caliente es un booleano precalculado, nunca comparación de strings, y todo cambio es **en vivo** (señales re-aplican).

### 1.2 Antialias (`§_-QK§` en §_-i3S§, líneas 234-241)
| Tier | Muestras AA |
|---|---|
| NONE/LOW | 0 |
| MEDIUM | 2 |
| HIGH | 8 |
| ULTRA | 16 |
Se aplica reconfigurando el back buffer del proxy en vivo.

### 1.3 Tamaño de texturas por categoría (`Settings3D.§_-F3K§`)
Ocho "perfiles de textura" (`§_-w2X§`), mapeados por categoría (`§_-mp§`):
`ship_very_small`/`drone`→§_-Hd§, `ship_small`→§_-11G§, `ship`→§_-N4W§, `ship_big`→§_-H3l§, `building_small`/`planet_small`→§_-Q2j§, `building`→§_-42l§, `building_big`→§_-a3K§, `planet`→§_-v1R§. Lookup: `Settings3D.§_-A3F§(categoria)` con fallback a `ship` (128).

| Categoría | LOW | MEDIUM | HIGH |
|---|---|---|---|
| drone / ship_very_small | 64 | 64 | 128 |
| ship_small | 64 | 64 | 128 |
| ship | 128 | 128 | 256 |
| ship_big | 256 | 256 | 512 |
| building_small / planet_small | 128 | 256 | 512 |
| building | 256 | 512 | 1024 |
| planet | 256 | 512 | 1024 |
| building_big | 1024 fijo (no lo ajusta ningún tier) | | |

Nota: **la resolución depende de la clase de objeto, no solo del tier** — un dron nunca pasa de 128 aunque el usuario ponga ULTRA. Las URLs de textura ya vienen horneadas por resolución: `<base>_diffuse_256`, `<base>_normal_128` (normal y specular a **mitad** del tamaño del diffuse, glow al tamaño del diffuse con tope 512, mínimo global 128) — ver `§_-X34§.§_-X1g§`.

### 1.4 Mapas por calidad de textura (`§_-i2R§` + `§_-X34§.§_-V3B§`)
| Mapa | Se usa a partir de |
|---|---|
| normalMap (`§_-O4n§`) | HIGH |
| specularMap (`§_-V1T§`) | HIGH (si está off: valores planos `specularLow`/`glossLow` del XML) |
| glowMap (`§_-4Q§`) | MEDIUM |

### 1.5 Filtrado (`§_-P2t§`)
| Propiedad | Se activa a partir de |
|---|---|
| smoothing (filtro bilineal) | MEDIUM |
| mipMapping | HIGH |

### 1.6 Luces (`displaySetting3DqualityLights` → LightsManager.`§_-vz§`)
| Tier | Direccional de mapa | PointLight del héroe | Luces dinámicas (pool) |
|---|---|---|---|
| LOW | off | off | 0 |
| MEDIUM | on | off | 1 |
| HIGH | on | on | todas (pool de 3) |

Dos "light pickers" (`StaticLightPicker`): el completo (`§_-L1w§`, naves) y uno reducido **solo direccional** para partículas (`§_-q2z§`), y este último solo si `effects >= HIGH`. Las partículas nunca ven point lights.

Constantes de luz (Settings3D):
- Luz default de mapa `§_-s37§`/`§_-t1y§`: color 0xA3FFFF, diffuse 0.8, specular 1.1, ambientColor 0xFF855C, ambient 0.5, tilt 100, pan 35 — **sobrescribible por XML del mapa** (`§_-V16§.§_-32A§`: atributos color/diffuse/specular/ambientColor/ambient/tilt/pan; defaults del parser: diffuse 1, specular 0.7, ambient 0.2, ambientColor 16756398 = 0xFFAEAE, color 0xFFFFFF — que en la práctica no se usan: los 6 mapas 3D escriben los siete atributos con los valores del preset).
- Luz del héroe `§_-Q4c§`: 0x2E7DFF, diffuse 0.6, specular 1.5, fallOff 450.
- Luz de láser `§_-e2§`: diffuse 0, specular 3, fallOff 150 (el color viene del arma).
- `§_-411§` (explosión): 0xF7C0C0, diffuse 1, specular 2, fallOff 200.
- `§_-B9§`: 0xDEE4C8, specular 20, fallOff 400.

### 1.7 Efectos (`Settings3D.effects`) — qué recorta cada tier
- **≥ MEDIUM**: starfield 3D de partículas (`§_-i7§`), efecto de impacto en escudo (`§_-U1x§` línea 319, `§_-z23§`).
- **≥ HIGH**: muzzle-flash de láser (sistema de partículas clonado) + luz dinámica por disparo (`§_-U1x§` 146/227), lightPicker en partículas (LightsManager), humo/estelas (`§_-K2v§` 175/192), extras varios (`§_-j3H§`, `§_-bY§`).
- **= ULTRA (max)**: patrones extra de explosión 3D (`§_-bY§` 35), lista de filtros de vista (`§_-V16§.§_-e2C§` — vacía en este build, pero el hook existe).

### 1.8 Condiciones de calidad *data-driven* (`Settings3D.§_-C3E§`)
El XML de assets puede declarar condiciones tipo `"effects.high AND background.max"`; se parsean a bindables con un `ANDBindableBoolean`. El contenido decide su propio gating de calidad sin tocar código.

### 1.9 Clases de material data-driven (`Settings3D.§_-V15§`)
Mapa nombre→clase: `basic`, `ship`, `organic_ship`, `uber_ship`, `uber_organic_ship`, `sector_control_beacon`, `animated_cloud`. Fallback: `ship`.

### 1.10 Flags de debug
- `Settings3D.render` (default true): si está en false el juego **actualiza la simulación pero no renderiza** (headless).
- `Settings3D.§_-mV§` (default false): fuerza recarga de assets (bypassa cachés).
- `Settings3D.§_-t3m§` (default true): control de cámara/zoom del usuario.
- `Settings.show_debug_objects`, `Settings.showHUD/showUI`.
- Config 3D propia en SharedObject separado `"darkorbit.settings3D"` (§_-o2x§).

### 1.11 Autodetección
No hay benchmark de hardware para el 3D: los cuatro settings 3D nacen en ULTRA y el usuario los baja. Lo que sí hay:
- `Settings.driverInfo` setter: `gpuSupport = driverInfo` no contiene "Software"/"Disabled" **y** Flash ≥ 11.8; `has3DCapabilities = gpuSupport`. Si no hay GPU → cliente 2D obligatorio.
- Auto-quality **por FPS** (aplica al pipeline 2D/efectos; ver §5.2).
- El servidor puede empujar `qualityPresetting` y `allowAutoQuality` (`§_-g2A§` 114/344).

---

## 2. Decisión 2D vs 3D

- Interfaz común `§_-C1P§` (IMapView): `load(map)`, `update(camX,camY,dt)`, `resize`, `zoomIn/Out`, proyección mundo↔pantalla (`§_-F4J§`/`§_-14v§`), `dispose`, registro de updatables. `§_-J2P§` (ScreenManager) solo habla con esta interfaz.
- `§_-B4r§` (MapViewFactory).`§_-g1I§(use2D)`: crea `§_-q3t§` (2D Starling) si `use2D || !has3DCapabilities`, si no `§_-i3S§` (3D). Además reporta telemetría `flash.stage3dstats` (driver, profile, hw_acc, use_2D).
- Motivos de 2D: (a) `FORCE_2D` (opción de usuario persistida, cambiable en runtime — `§_-J2P§.§_-A24§` reconstruye la vista); (b) sin GPU (driver Software/Disabled o FP < 11.8); (c) primera vez: pantalla `ClientSelectionView` deja elegir 2D/3D (`§_-P4G§`: usuario nuevo con 3D disponible → 3D; sin 3D → 2D).
- **El cliente 2D también usa Stage3D**: es Starling con atlas ATF (`Settings.useATF`), con el mismo patrón de context-loss. No es un fallback por software, es un pipeline 2D GPU. El fallback real por software lo hace Flash (renderMode AUTO).
- El 2D tiene su propia escalera de auto-quality (AQ_*, ver §5.2); el 3D tiene sus 4 sliders.

---

## 3. Recuperación de context loss (patrón completo)

1. **Prevención de estados podridos**: `Starling.handleLostContext = true` en el constructor de ambas vistas (Starling retiene los datos para restaurar texturas), envuelto en try/catch por si la versión no lo soporta.
2. **Detección por frame** (`§_-i3S§.update` 132-146): solo renderiza si `context3D != null && driverInfo != "Disposed"`. Si el contexto murió, pide uno nuevo **una sola vez** (guard `§_-H1m§`) con `requestContext(false,"baseline")` y sigue actualizando la simulación sin renderizar (ni crash ni spam de requests).
3. **View3D.render** (`§_-d1G§` 587-603): valida `§_-73i§()` del proxy — si `driverInfo == "Disposed"`, anula el contexto, despacha evento DISPOSED, marca el backbuffer como sucio y **retorna sin dibujar**.
4. **Recreación**: el proxy escucha `CONTEXT3D_CREATE` con prioridad 1000; al llegar contexto: `enableErrorChecking` solo si debug, detecta software (`driverInfo.indexOf("Software")==0`), reconfigura backbuffer y despacha `CONTEXT_CREATED` o `CONTEXT_RECREATED` según sea primera vez o pérdida (`§_-e4r§`, 447-464).
5. **Re-subida perezosa**: cada textura/geometría guarda el TextureBase **por slot de proxy** junto con el Context3D con el que se creó (`§_-5B§.§_-W3A§`: si `§_-H3W§[slot] != contextoActual` → re-crea y re-sube). Nada se re-sube en bloque: se re-sube al primer uso.
6. **`dispose(false)` vs `dispose()`**: al cambiar de mapa (`§_-i3S§.load` 92-106 y `§_-d3k§.§_-13q§`) destruyen el contexto adrede para partir de VRAM limpia y piden otro. Usan `context3D.dispose.length == 0 ? dispose() : dispose(false)` — detección por reflexión de la firma del runtime: en FP ≥ 12 `dispose(recreate:Boolean)` existe y pasan `false` para que Flash **no** recree automáticamente (lo piden ellos con su perfil). El mismo patrón está en el 2D (`§_-q3t§.load`).
7. **Ciclo de vida de assets por "lifetime"** (`§_-o2x§`, TextureLoader/GeometryLoader/ParticlesLoader): 0 = por-material (se libera cuando su ref-count cae), 1 = por-mapa, 2 = precarga permanente (lista `preloadLists.pack id="3D"` del XML). `reset(1)` en cada cambio de mapa dispone todo lo ≤1 y conserva la precarga.
8. Upload defensivo: `SingleTextureLoader.loadData` hace el upload dentro de try/catch y, si no hay contexto, despacha `ready` igualmente (la textura se subirá al primer render).

---

## 4. Frame loop

- FPS objetivo explícito: `stage.frameRate = 60` (`§_-M3W§\ApplicationMediator` 22; SWF a 60).
- `§_-J2P§` (ScreenManager): en `ENTER_FRAME` solo hace `stage.invalidate()`; el trabajo real va en el evento **`Event.RENDER`** (`§_-g4M§`, 576-644) — corre inmediatamente antes de que Flash rasterice, así el estado que se dibuja es el más fresco y no se duplica trabajo en frames con resize.
- Medición de dt: `getTimer()` (ms) − timestamp anterior. Se propaga en **ms** a los observers `updateTimer(ms)` (Profiler, etc.) y en **segundos** al gameplay.
- Orden por frame: (1) observers (`§_-Qh§`, vector plano iterado por índice con contador cacheado `§_-k4i§`); (2) cámara (sigue al héroe o a un objetivo, screen-shake acumulando dt con paso de 24ms); (3) `traits` del mapa (`updateObj(dtSeg)`, iterado **en reversa** para tolerar remociones); (4) `_mapView.update(camX, camY, dtSeg)`.
- Dentro de `§_-i3S§.update`: resetea contadores → `_map.update(dt)` (todos los updatables 3D, también en reversa) → si `Settings3D.render` y contexto vivo: `proxy.clear(); starling.render(); map3D.render(); proxy.present();` — **un solo present por frame para 2D+3D**.
- **No hay clamp de dt ni frame-skipping propio**: los movimientos son función del dt real, y el throttling en background lo hace el propio Flash Player (baja el frame rate de pestañas ocultas). Lo que sí hacen: el `Profiler` se des-registra en `Event.DEACTIVATE` y se re-registra en `ACTIVATE` para no medir FPS falsos ni bajar calidad mientras la ventana no tiene foco.
- Resize con debounce de 250ms (`RESIZE_DELAY`), backbuffer con mínimo 50×50, y en perfil baseline clamp a **2048×2048** (`§_-d1G§` 561-568).
- Zoom por tween (TweenLite) sobre la propiedad `zoom`; lente `PerspectiveLens(30°)` (`§_-52g§`, FOV angosto = menos distorsión y frustum más chico).

---

## 5. Estadísticas y auto-quality

### 5.1 Contadores `§_-s3u§` (RenderStats, paquete raíz)
- `textures`: contador global de texturas.
- `§_-U2J§` y `§_-e4M§`: reseteados a 0 al inicio de cada `update` de la vista 3D (`§_-i3S§` 129-130). **No queda ningún sitio que los incremente en el build de release** (el cuerpo de la clase es bytecode ofuscado/muerto): eran contadores de debug (probable draw calls / objetos dibujados) stripped en producción.
- `§_-A1d§`: asignado tras el render con `view.§_-a1W§` = `EntityCollector.§_-A1d§`, que el collector **acumula por renderable** (`§_-q1j§` 223: `this.§_-03N§ += renderable.numTriangles`). Es el **conteo de triángulos renderizados por frame**.
- Panel de debug: `§_-s1i§\Stats.as` — FPS / MS / MEM / MAX con gráfico, y click para subir/bajar `stage.frameRate` (el update está vaciado en release, pero el widget existe). `§_-eI§.§_-y34§` vuelca tamaños de pools al log del juego.

### 5.2 Auto-quality por FPS (`Profiler`)
- Muestrea FPS 1 vez/segundo (colección máx. 100), promedia cada `INTERVAL_LENGTH = 20 s`.
- Si promedio `< AUTO_QUALITY_LOWER_BOUND_FPS = 10` durante ≥1 intervalo y la calidad global no es ya LOW → `autoQualityReduction += 1` (máx `AQ_MAX_REDUCTION = 5`).
- Si promedio `> AUTO_QUALITY_UPPER_BOUND_FPS = 60` → `autoQualityReduction -= 1` (histéresis: se recupera solo cuando sobra rendimiento).
- Escalera de reducción (constantes AQ_* en Settings): nivel ≥1 sin preview del destino de portales; ≥2 sin humo de motores; ≥3 sin map-assets animados; ≥4 explosiones en detalle bajo; ≥5 sin explosiones y sin movimiento del starfield. Cada consumidor compara `autoQualityReduction.value >= AQ_X_LIMIT` localmente (p.ej. `§_-mx§` 270, `§_-Wz§` 48, `§_-o2P§` 26).
- Notificación "low performance" (video tutorial) tras 3/4/6 intervalos consecutivos < 10 FPS, máximo 3 veces (`NOTIFICATION_STEPS`).
- `allowAutoQuality` togglable por el usuario y por el servidor; al apagarlo la reducción vuelve a 0.

---

## 6. Pooling y reuso

1. **Luces**: LightsManager pre-crea 3 PointLights (nunca se instancian en gameplay); ociosas con `radius = fallOff = 0`. Reciclado circular `shift/push` con expiración por `setTimeout` y fade-out por tween (`§_-Qz§`/`§_-d1F§`/`§_-x2E§`). La lista de luces del picker solo se rearma cuando algo cambió (flag dirty `§_-H3W§`), no cada frame.
2. **Pool genérico 3D por clave** (`§_-321§\§_-eI§`): `retrieve(key)` devuelve null hasta que el pool tenga >1 elemento (mantiene 1 "tibio" y fuerza crear mientras tanto), `§_-g16§(obj,key)` devuelve al pool; objetos con interfaz wake/sleep (`§_-K4q§.§_-C§()` / `§_-B3s§()`). Se vacía (con dispose) en cada reset de assets/cambio de mapa. Usuarios: salvas de láser (`§_-q21§`, clave = arma+resKey), clones de muzzle-flash (`"LaserSalvo3D_flash_"+resKey`).
3. **Pools 2D precacheados** (`§_-a2H§\§_-13C§` + config `§_-l1p§`): `poolSize` por efecto viene del XML de recursos; se instancian N MovieClips al inicio y se hace pop/push.
4. **Vector3D scratch**: constantes reutilizadas por instancia (en `§_-V16§`: `§_-84F§`, `§_-P1W§`, `§_-V26§`, `§_-I34§`, `§_-532§`, `§_-T2P§`, `§_-t3S§`, `§_-v2p§`, `§_-jK§`, `§_-n2O§`, `§_-R1P§`) y estáticas compartidas (`§_-o2x§.§_-B18§`, `§_-i12§`, `§_-1K§` en los efectos de láser). Además **todas las funciones de proyección aceptan un out-parameter opcional** (`§_-F4J§(x,y,z,out)`, `§_-14v§(...)`) — cero asignaciones por frame en matemáticas de cámara.
5. **EntityCollector reutilizado**: `clear()` + `traversePartitions` + `§_-GR§()` (cleanUp) cada frame sobre el mismo objeto.
6. Starling trae sus propios pools (Event, Tween, matrices) — se usa tal cual.

---

## 7. Culling

- **Frustum culling estándar de Away3D**: `scene.§_-F47§(entityCollector)` (traversePartitions) con la cámara del collector; solo lo recolectado se dibuja (`§_-d1G§` 608-660). No hay culling por distancia propio para naves: el "spawn radius" lo impone el **servidor** (solo manda entidades cercanas), así que la escena ya llega podada.
- **Starfield tileado con bounds a mano** (`§_-i7§`): el sistema de partículas se clona en tiles de **2000 unidades** cubriendo el mapa, y a cada mesh se le fija el bounding box manualmente a `(0,0,0)-(2000,-300,2000)` → cada tile se frustum-cullea de forma independiente; la "animación" es solo incrementar `animator.time` (GPU-driven, cero CPU por partícula). Todo el starfield desaparece bajo effects<MEDIUM.
- Lente de 30° → frustum angosto (más culling, look más "2.5D").
- Backbuffer clamp 2048×2048 en baseline (límite del profile, y de paso techo de fill-rate).
- Fondo: si el mapa define fondo 3D (`§_-G2B§.§_-F2Q§`) se usa ese; si no, se reutiliza el **fondo parallax 2D del cliente 2D** (`§_-Kt§`) dentro de Starling — el fondo nunca pasa por el pipeline 3D si no hace falta.

---

## 8. Starling: para qué y cómo

- En el cliente 3D, Starling renderiza **solo la capa de fondo 2D** (parallax `§_-Kt§`, capa "bg" agregada al root `§_-c12§`). El HUD (`§_-a2o§.HUD`) y toda la GUI son display list clásica de Flash **encima** del Stage3D (Stage3D siempre queda detrás del stage).
- Configuración exacta (`§_-i3S§.§_-42V§`, 289-297): `new Starling(root, stage, proxy.viewPort, proxy.stage3D, Context3DRenderMode.AUTO, "baseline")`; `shareContext = true`; `enableErrorChecking = false` (errorChecking solo en debug también en el proxy: `§_-er§.active`).
- **Render intercalado con un solo present** (`§_-i3S§.update`): `proxy.clear()` (color+depth) → `starling.render()` (fondo 2D; con shareContext Starling ni limpia ni presenta) → `map3D.render()` (Away3D; en shareContext hace **clear solo de depth** — `§_-d3k§.§_-l4G§`: `clear(0,0,0,1,1,0,DEPTH)` — para dibujar la escena 3D encima del fondo sin borrarlo) → `proxy.present()`.
- En el cliente 2D, Starling es todo el mapa, con atlas de texturas ATF (`resKeyStarling` + `atlasXml` en `§_-B1T§\§_-A34§`; `resKey` devuelve el atlas GPU solo si `Settings.gpuSupport`).

---

## 9. Otros trucos de rendimiento

1. **ATF comprimido + upload asíncrono** (`§_-X2m§\§_-5B§`): `uploadCompressedTextureFromByteArray(data, 0, async=true)` + evento `TEXTURE_READY`; mientras no está lista, `§_-W3A§` devuelve null y el material dibuja sin ella → **cero hitches de subida a GPU**. También JPEG-XR (`§_-636§`) para 2D. `§_-X4L§` reescribe `.png → .atf` cuando hay GPU.
2. **Materiales compartidos con ref-count** (`§_-jh§\§_-X34§`): cache estático por `claseMaterial + uid del perfil de textura + hash de la definición` → todas las naves del mismo tipo comparten un único material (texturas, programas, estado). `§_-A3c§` incrementa contador, `§_-k3r§` decrementa y dispone al llegar a 0.
3. **Cache de programas AGAL** (`§_-G43§\AGALProgram3DCache`): shaders compilados compartidos entre materiales (estándar Away3D, conservado).
4. **Geometrías cacheadas por resKey** (GeometryLoader) con el mismo sistema de lifetimes; recycler recursivo `§_-u3s§` que dispone geometry+material+animator de un árbol completo.
5. **Ref-count con gracia** (`§_-T2s§` + `§_-I4y§(5)`): las texturas de materiales (lifetime 0) no se destruyen al soltarse sino tras una ventana (5) — evita destruir/recargar al alternar naves.
6. **Settings bindables + señales en todo**: cualquier cambio de calidad (AA, tamaño de textura, luces, filtros) se re-aplica en vivo a materiales y luces ya creados; nada requiere reiniciar.
7. Partículas: post-proceso al cargar (`§_-u1u§.§_-i45§`) sustituye el material del `shader_fireball` por `ExplosionMaterial` y asigna lightPicker solo a los meshes marcados en el XML (`lightPickerTargets`).
8. Telemetría de rendimiento real: `flash.stage3dstats` (driver/profile/uso 2D) y `frameRate` en EventStream → decisiones de tiers con datos de la población.
9. Perfil `"baseline"` en todos los requestContext → máxima compatibilidad de hardware 2013.
10. `mouseEnabled = mouseChildren = false` en todas las capas no interactivas (el hit-testing de Flash era caro); el picking 3D usa un raycast propio (`§_-x1d§`) solo bajo demanda.

---

## 10. Guidelines aplicables al cliente Godot de MexOrbit

1. **Tier por categoría de asset, no solo global**: tabla categoría×tier para resolución de texturas/sprites (dron 64-128, nave 128-256, edificio 256-1024). En Godot: variantes de importación o `ResourceLoader` por sufijo.
2. **Booleanos derivados del tier** (`effects_medium`, `effects_high`...) precalculados y con señal `changed`; gating de cada efecto con un booleano, aplicable en vivo.
3. **Auto-quality con histéresis**: promedio de FPS en ventanas de 20 s, baja un escalón bajo 10 FPS, sube solo por encima del target; escalera de recortes ordenada por costo/valor (preview de portales → humo → animaciones de mapa → detalle de explosión → explosiones+starfield). No medir cuando la ventana no tiene foco.
4. **Presupuesto de luces fijo con pool**: N luces pre-creadas apagadas, reciclado circular, luz por disparo/explosión con expiración; lista de luces activas solo se rearma con flag dirty.
5. **Un solo present/frame y capas por clear parcial**: fondo → clear de depth → mundo (en Godot esto lo da el motor, pero la lección es no apilar Viewports que re-presentan).
6. **Pools por clave con warm-spare** para proyectiles/flashes/explosiones, vaciados en cambio de mapa; objetos con wake/sleep en vez de instanciar.
7. **Cero asignaciones en matemáticas por frame**: scratch reutilizado y out-parameters en proyecciones (en GDScript: reutilizar `Vector2/3` no aplica por ser value types, pero sí evitar arrays/Dictionaries temporales y `new` de nodos por frame).
8. **Assets con lifetime 0/1/2** (por-uso / por-mapa / permanente) y un único `reset(nivel)` en la transición de mapa.
9. **Subida de texturas sin hitch**: cargar comprimido, subir asíncrono y dibujar sin el mapa hasta que esté (placeholder), nunca bloquear el frame.
10. **Compartir material entre naves iguales** con ref-count (en Godot: mismo `Material` resource por tipo de nave; cuidado con `material_override` duplicado).
11. **Recuperación de "context loss" ≈ robustez de dispositivo**: validar recursos antes de dibujar, un solo re-request con guard, re-crear perezosamente, y simulación que sigue corriendo aunque el render falle un frame.
12. **Contadores de frame (triángulos/draw calls) reseteados por frame** + panel FPS/MS/MEM oculto (Godot: `Performance.get_monitor` + overlay con toggle).
13. Condiciones de calidad **declaradas en los datos** del asset ("effects.high AND background.max") en vez de hardcodear en código.
14. dt real sin clamp funciona si el gameplay es interpolación servidor-autoritativa (como aquí); si algún sistema integra físicas localmente, ese sí necesita clamp — DarkOrbit no lo necesitaba porque todo se tween-ea hacia estados del servidor.
