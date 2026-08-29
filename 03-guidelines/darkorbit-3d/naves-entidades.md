# Naves y entidades en 3D — cómo se ven y se MUEVEN (cliente DarkOrbit decompilado)

Fuente: `D:\MexOrbit\Decompiled\spacemap\main\scripts\` (AS3 + Away3D 4.x, ofuscado por JPEXS).
Assets: `C:\Source\MexOrbit\MexOrbit.CMS\public\spacemap\3d\` y XML embebidos en `D:\MexOrbit\Decompiled\spacemap\main\binaryData\`.

Convención del informe: `§_-XX§ (NombreDeducido)`. Todas las constantes son literales del código, no estimaciones.

---

## 0. Mapa de clases (ofuscada → deducida)

| Clase | Paquete | Nombre deducido | Rol |
|---|---|---|---|
| `§_-834§` | map.model.ship | **ShipModel** | Modelo lógico de nave (posición, rotación, speed, cloak, NPC) |
| `§_-Ac§` | map.model.ship | **ShipFactory/Controller** | Crea/destruye naves y PETs desde paquetes de red |
| `§_-r1d§` | §_-o1k§ | **Entity** | Entidad base con traits; define el mapeo de coordenadas |
| `§_-q1E§` | map.model.traits | **DisplayTrait** | Trait de visualización; guarda el XML descriptor y `visualSize` |
| `§_-31O§` | view3D.display3D | **EntityView3D** | Vista 3D de entidad; instancia decoradores |
| `§_-115§` | §_-n3Z§ | **UberAssetView3D** | Vista base: parsea el XML display3D (mesh/container/particles/pivot), flotación |
| `§_-DX§` | §_-3L§ | **ShipDecorator** | ⭐ Movimiento y sensación: giro suavizado, banking, placeholder |
| `§_-K2v§` | §_-G1y§ | **ThrusterDecorator** | Partículas de motor + trail + speed buff |
| `§_-Pd§` | §_-G1y§ | **EngineTrailRibbon** | Cinta de estela (ring buffer de 12 puntos) |
| `§_-ZC§` | §_-3L§ | **LaserSlotResolver** | Resuelve slots `laserpoint_*` del mesh para disparos |
| `§_-a11§` | §_-aj§ | **DronesDecorator** | Formaciones de drones + grupos rotatorios |
| `§_-C4g§` | §_-aj§ | **DroneView** | Vista de dron individual (lag de seguimiento) |
| `§_-73c§` / `§_-r16§` | §_-Oj§ | **HeroDecorator / PetDecorator** | Luz del héroe + textura 2× al hacer zoom |
| `§_-t3E§` | §_-Oj§ | **CollectableDecorator** | Animaciones spawn/collect/dispose de cajas/ores |
| `§_-717§` / `§_-P4X§` | §_-jh§ | **PrefabView3D / ShipMesh** | Mesh con submeshes; extrae slots `engine_*`, `laserpoint_*` |
| `§_-LC§` | §_-jh§ | **MaterialDescriptor** | geometry/diffuse/normal/specular/glow/gloss… desde XML |
| `§_-ig§` | §_-n3Z§ | **MeshView** | Wrapper de mesh con propiedad `glow` |
| `§_-T4L§` | §_-b1Q§ | **AnimationStateManager** | Animaciones por id + `background_animation` + estados |
| `§_-Qc§` / `§_-Yh§` / `§_-78§` / `§_-94M§` | §_-b1Q§ | **UberAnimation / TweenParser / AppendUpdater / GlowPulse** | Sistema de animación declarativo XML |
| `§_-c1o§` | §_-321§ | **FloatOscillator** | Oscilador Lissajous de flotación idle |
| `§_-03b§` | §_-321§ | **MathUtils** | `§_-tb§` = diferencia angular con signo (grados, camino corto) |
| `§_-Jw§` / `§_-Q4t§` | §_-321§ | **CameraController / CameraRig** | (lo cubre otro agente; aquí solo lo que toca a entidades) |
| `§_-V16§` | map.view3D | **Map3DView** | Raíz de escena (escala 0.0001), luz de mapa, updatables |
| `§_-i7§` | view3D.display3D | **StarfieldLayer** | Star dust en mosaico de 2000×2000 |
| `§_-f4D§` | view3D.display3D | **TargetMarker3D** | Anillo de selección de objetivo |
| `§_-x1d§` | view3D.display3D | **EntityRaycaster** | Picking por triángulos sobre meshes de naves |
| `LightsManager` | view3D.display3D | — | Sol direccional, luz del héroe, pool de flashes |
| `§_-a24§` | §_-X40§ | **Effects3DFactory** | Registro efecto→clase 3D (70+ efectos) |
| `§_-bY§` / `§_-j3H§` / `§_-42y§` / `§_-r1f§` | §_-01b§/§_-n3Z§ | **ExplosionEffect3D / DamageImpact3D / AttachedEffect3D / ParticleEffectBase** | Efectos por entidad |
| `UberAssetsLib` | mvc.common.model.assets | — | Tabla id→descriptor XML (lib_ships, lib_drones…) |
| `§_-j2P§` | §_-s2S§ | **FollowTargetParticleNode** | Nodo de animador de partículas que sigue posición/rotación de un objeto (paquete §_-s2S§ = nodos custom de partículas Away3D, no la capa de vista) |

---

## 1. De entidad lógica a vista 3D (id → asset)

### 1.1 Tabla id→asset: XML embebidos (UberAssetsLib)

- Librerías XML: `lib_ships`, `lib_drones`, `lib_portals`, `lib_collectables`, `lib_battlestation`, `lib_maps`, `lib_assets` (default). Embebidas en el SWF y recargables desde `../../config/data/<lib>.xml`. **Volcadas en**: `D:\MexOrbit\Decompiled\spacemap\main\binaryData\187_...LIB_SHIPS_XML.bin` (y 179/180/181/183/190).
- Cada `<asset id="ship_goliath">` (jugadores por nombre, **NPCs por id numérico** "27", "76"…) tiene `<display2D>` (sprite legado) y `<display3D>`.
- Selección con facción: `ShipModel.§_-vQ§` busca `getDescriptor(patternKey + "_" + factionID, LIB_SHIPS)` y cae a `getDescriptor(patternKey)` — variantes por facción vía sufijo.
- El descriptor genera traits: display, click_area (radio), miniMapIcon, hp, attack_target (lockType, explosión), attacker (láser/cohetes).

### 1.2 Formato del descriptor 3D (ejemplos reales)

```xml
<asset id="ship_goliath" comment="Goliath">
  <display3D tex_settings="ship" geometry="goliath" scale="0.7" texture="goliath" visualSize="1.4"/>
</asset>
<asset id="ship_liberator">
  <display3D engineTrailColor="0x5ac3d8" tex_settings="ship_small" geometry="liberator" scale="0.60" visualSize="0.85"/>
</asset>
<!-- Skins/designs: mismo geometry, distinto diffuse -->
<display3D geometry="goliath" texture="goliath" diffuse="goliath-exalted" visualSize="1.4"/>
<!-- Attachments (pods) = mesh extra en el mismo descriptor -->
<asset id="ship_aegis">
  <display3D visualSize="1.4">
    <mesh geometry="aegis" texture="aegis" scale="0.8"/>
    <mesh id="pod" geometry="aegis-ship-pod" texture="aegis" scale="0.8"/>
  </display3D>
</asset>
```

- `geometry="X"` → archivo `X.awd` en `3d/meshes` (mapeo en `resources_3d.xml`: `<file type="awd" name="phoenix" location="3d_meshes">`).
- Texturas ATF por canal y tamaño: `phoenix_diffuse_128/256/512`, `*_normal_*`, `*_specular_*`, `*_glow_*`. En `§_-LC§ (MaterialDescriptor)`: si no se especifica canal, hereda el nombre de `@texture` (que a su vez hereda de `@geometry`); `"none"` desactiva el canal.
- `tex_settings` elige categoría de resolución (`Settings3D`): ship=128 (256 en high), ship_small/very_small/drone=64 (128), ship_big=256 (512), building=512 (1024), planet=1024.
- `scale` = escala del mesh; `visualSize` = tamaño lógico usado para escalar EFECTOS adjuntos y el anillo de selección (ej. Goliath 1.4, Phoenix 0.6, Hitac 2.5).
- Material por defecto shader `"ship"` (`§_-12c§`); otros: `uber_ship` (UberShipMaterial, jefes "Uber" con `glow="none"`), `organic_ship`, `basic`. Parámetros: `specularity` (def. 1), `gloss` (def. 50), `alphaBlending`, `blendMode`, `materialBothSides`, `zOffset`.
- Flags de comportamiento en `<display3D>`: `rotatable`, `tilting`, `floating` (los tres por defecto `true`). NPCs "estáticos" (Sibelonit, Icy, Hitac, spaceballs) usan `rotatable="false"` (+ a veces `tilting="false" floating="false"`).

### 1.3 Placeholder durante la carga

`§_-DX§.§_-N4a§` (ShipDecorator): si a los **1000 ms** (nave ajena) o **2500 ms** (nave propia) el AWD no está listo, crea un **plano 3D con el sprite 2D** de la librería `replacementShips` (display2D.@srcKey), ocultando los hijos del MovieClip cuyo nombre contenga `laser` o `engine`. `scaleX = alto del clip`, `scaleZ = ancho del clip`, plano rotado 90° en Y. Se elimina al llegar el mesh real. Al cargar el mesh se auto-reproduce la animación AWD `"idle"` con offset aleatorio `Math.random()*10000` ms (desincroniza entidades idénticas).

---

## 2. MOVIMIENTO Y SENSACIÓN (lo crítico)

### 2.0 Mapeo de coordenadas y escala

- `Entity (§_-r1d§)`: `§_-03J§ = position.x`, `§_-t1g§ = position.z` (altura, siempre 0 en naves), `§_-W2T§ = -position.y`. La vista hace `moveTo(x, altura, -y)` → confirma **(x, y) juego → (x, h, −y) 3D**.
- `Map3DView (§_-V16§)`: contenedor raíz `§_-g4u§(0.0001, 0.0001, 0.0001)` — toda la escena escalada ×0.0001.
- Cámara (resumen mínimo): distancia base **1740** unidades / zoom (rango 1–3), FOV **30°**, tilt por defecto **135°** (≈45° de picado), near 10 / far 80000. Zoom animado 0.3 s Quad.easeOut; pasos de rueda ×1.2 / ×0.8.

### 2.1 Interpolación de posición (modelo)

`ShipModel.goto(x, y, …)` — la posición NO se recibe por frame; el servidor manda destino (+opcional duración):

- `duración = distancia / speed` (speed en unidades/seg, viene del servidor por nave) salvo que el paquete traiga duración explícita.
- **TweenLite lineal** (`Linear.easeNone`) sobre `position` hasta el destino; el tween se **reutiliza** (invalidate+restart) en cada update.
- Si cambia `speed` en pleno vuelo se relanza `goto` al mismo destino (recalcula duración).
- No hay hermite ni extrapolación: **lerp lineal puro a nivel de modelo**; la suavidad visual la ponen el ease de rotación y el banking de la vista.
- Umbral: si el destino está a ≤2 unidades en x e y, no re-apunta la rotación (`§_-032§`).

### 2.2 Rotación del modelo (instantánea)

- `ShipModel.§_-032§`: `rotation.radians = atan2(dy, dx) + π` — **asignación instantánea** al recibir destino o al moverse el objetivo atacado (escucha `position.changed` del target y del propio barco mientras hay target).
- El suavizado es 100 % responsabilidad de la vista.

### 2.3 Suavizado de giro y BANKING (ShipDecorator `§_-DX§.§_-i2o§`) ⭐

Por frame, con `dt` en segundos (`updateObj(dt)`):

```
targetY  = ship.rotation.degrees - 90        // offset de orientación del mesh
deltaY   = shortestAngleDiff(_rotationY, targetY)          // §_-03b§.§_-tb§, en grados, con signo
_rotationY = Quad.easeOut(dt, _rotationY, deltaY, 0.2)     // "duración" 0.2 s

// Banking (roll):
si (en movimiento && con objetivo de ataque):
    maxTilt = 10;  d = 0.08;  targetZ = -deltaY * 2
si no:
    maxTilt = 20;  d = 0.2;   targetZ = +deltaY * 1
deltaZ   = shortestAngleDiff(_rotationZ, targetZ)
_rotationZ = clamp(Quad.easeOut(dt, _rotationZ, deltaZ, d), -maxTilt, +maxTilt)
```

- `Quad.easeOut(t,b,c,d) = b + c·(t/d)·(2 − t/d)`. Con dt por frame: fracción avanzada ≈ `(dt/d)·(2 − dt/d)` → a 60 fps con d=0.2 avanza ~16 %/frame; a 30 fps ~30 %/frame (suavizado exponencial dependiente de frame-rate, constante efectiva ≈ 0.1 s).
- **El ángulo de banking ES la diferencia angular pendiente**: girar 90° instantáneos produce un roll transitorio que el clamp limita a **±20°** (crucero) o **±10°** (atacando, con signo invertido y ganancia ×2 — la nave "se abre" hacia el target y responde más rápido, d=0.08).
- El roll **vuelve solo a 0**: cuando la nave deja de girar, deltaY→0, targetZ→0 y el mismo ease lo devuelve a nivel. No hay tween de retorno separado.
- Aplicación: matriz `rotZ(−_rotationZ)` luego `rotY(_rotationY)`, descompuesta a euler. Con `dt=0` (primer frame) se aplica sin suavizar.
- **No hay pitch** por aceleración/frenado. Solo yaw + roll (+ oscilación idle, ver 2.5).
- Flags XML: `rotatable=false` → sin yaw (NPCs tipo Sibelonit/medusa que "miran" siempre igual); `tilting=false` → sin banking.
- El tilt se propaga: `ThrusterDecorator.tilt = −_rotationZ` (las llamas rotan `tilt/4` en Y).

### 2.4 Giro idle de NPCs

`ShipModel.isNPC=true`: al detenerse programa un timer de **2000 + 5000·rand ms** (2–7 s) que hace `rotation.degrees += (rand − 0.5) · 360` (giro aleatorio de hasta ±180°) y se reprograma. La vista lo suaviza con el ease de 0.2 s → los aliens quietos "miran alrededor" perezosamente. Se cancela al empezar a moverse o al tener atacante fijado (`§_-Yk§`).

### 2.5 Flotación idle (hover/bobbing)

- Ships **sin** nodo `<floating>` en XML; `ShipDecorator.decorate()` fuerza amplitudes: **movimiento (5,5,5) unidades** y **rotación (5,5,5) grados** (`§_-jd§`/`§_-k25§`).
- Solo flota **parada** (al iniciar movimiento se para el hover con tween a 0 en 0.5 s; al detenerse se reactiva). Controlado por `Settings.hoverShips` y `@floating`.
- Oscilador `§_-c1o§ (FloatOscillator)`: fase `+= dt / cycleLength` (cycleLength por defecto **2** → 0.5 rad/s), y produce `x = sin·cos`, `y = sin²`, `z = cos·sin` (osciladores separados para posición y rotación, misma fase inicial aleatoria). Periodo visible ≈ **6.3 s**; la componente Y es sin² → la nave flota **solo hacia arriba** 0..5 unidades.
- Portales: `<floating rotationX="3" rotationY="3" rotationZ="3"/>` (solo rotación ±3°, gates galaxy ±1°). Ores: `floating="true"` con offset base `y="random(-130,-70)"` y `rotationX/Y="random(-30,-20)"`.

### 2.6 Altura Y de las naves

- Todas las naves nacen y viven a **z lógico 0** → **y=0 en 3D** (`position.setTo(x, y, 0)`); no hay offsets ni jitter por entidad para z-fighting en naves. La separación visual la dan: hover (0..5 solo en idle), `zOffset` de material por descriptor, y offsets por tipo (ores −130..−70; algunos efectos/pivots con y propio, p.ej. partículas de Hitac `y="20"`).

---

## 3. Attachments

### 3.1 Slots nombrados en el AWD (ShipMesh `§_-P4X§.§_-m2V§`)

Al clonar los submeshes del AWD, por **nombre de submesh**:
- `engine_*` → slot de motor (posición extraída; el submesh no se renderiza). Vector `§_-g3c§`.
- `laserpoint_<n>` → slot de láser accesible por nombre (`§_-B25§("<n>")`).
- `light_laser*` y `light_position*` → filtrados (marcadores, no se renderizan).

`LaserSlotResolver (§_-ZC§)` transforma el slot por `sceneTransform` del mesh y devuelve el **offset mundial respecto a la nave** para origen de disparos (los láseres salen de los cañones reales incluso con banking).

### 3.2 Thrusters (ThrusterDecorator `§_-K2v§`)

- Partícula `thruster.zip` clonada en **cada** slot `engine_*`, `rotationX=180`, tiempo inicial aleatorio (desincronizadas).
- Escala objetivo por frame: `moviendo=1 | idle jugador=0.7 | idle NPC=0`, **×3** con speed-buff, **×1.5** sin él; suavizado `Linear.easeNone(0.1, actual, objetivo−actual, 0.5)` = 20 % por frame; se ocultan bajo escala 0.01.
- `rotationY = tilt/4` (siguen el banking).
- Speed buff añade `speed_buff.zip` en los mismos slots.
- Calidad: high = todas las naves; medium = solo jugadores (NPCs sin thruster); low = nada. Ídem para el trail.

### 3.3 Estela (EngineTrailRibbon `§_-Pd§`)

- Cinta por slots de motor: ring buffer de **12 puntos** (`EngineTrailMaterial.§_-P1X§ = 12`) muestreado cada **0.03 s** → ~0.36 s de historia; alpha decae linealmente `1 − i/12`; ancho factor 3.
- Color por defecto **0x5AC3D8** (cian), sobreescribible por nave con `display3D@engineTrailColor` (ej. 0xAD3333 rojo).
- Anti-teleport: si la nave saltó >100 unidades en un muestreo, la cinta entera se colapsa a la posición actual.

### 3.4 Pods y módulos

- **Pods** (aegis-ship-pod): simplemente un `<mesh id="pod">` extra en el descriptor, modelado en su sitio (sin joint: coordenadas propias del AWD).
- **al-/am-laser-module.awd**: módulos de la **estación de clan (CBS)** — entidades propias en `lib_battlestation` (mesh módulo + mesh `moduleplatform` cbs-module-platform-lvl1/2/3), no attachments de naves.

### 3.5 Drones (DronesDecorator `§_-a11§` + DroneView `§_-C4g§` + game-patterns-profile.xml)

- Formaciones en `binaryData\game-patterns-profile.xml` → `<droneFormations><formation id name>`: `positionsList data="x,y[,z];…"` (coordenadas relativas a la nave, unidades de juego), `usedPositions` por nº de drones, `<scale>`. Variantes `<display3D>` y `<display2D>` (fallback `Settings.show2DFormation`). Ej. estándar: back group a ±150–190, right/left a ±112–188.
- El contenedor de drones sigue la posición de la nave **incluyendo su oscilación de hover** (misma fase, ejes intercambiados).
- Rotación de formación suavizada **aparte** del barco: `Quad.easeOut(dt, ang, shortestDiff, 0.3)` (más lenta que el casco: 0.3 vs 0.2) → al girar, el anillo de drones "se arrastra" detrás. Cada dron mira `rotY = angFormación − 90`.
- Posición individual del dron: por eje, `Quad.easeOut(dt, pos, target − pos, 0.3)` → elástico de 0.3 s hacia su slot.
- **Grupos rotatorios 3D** (`<rotationGroup droneIds="…"><tween duration="5" rotationX="360" rotationY="-360" repeat="-1" ease="Linear.easeNone"/></rotationGroup>`): p. ej. "Protection Ring 3D" gira el anillo completo 360° cada **5 s**; "Drill 3D" tres grupos contrarrotando a 5 s. Ejecutados por `§_-D33§ (TweenRunner)` sobre un contenedor padre.
- Escala de formación dinámica del modelo (`drones.§_-6j§`), cloak de la nave aplica alpha 0.5 a los drones (tween 0.2 s).

---

## 4. Efectos por entidad

### 4.1 Registro

`Effects3DFactory (§_-a24§)`: mapa efecto-modelo → clase 3D (70+). Claves: explosión→`§_-bY§`, DAMAGE→`§_-j3H§`, LASER, ROCKET, EMP, INVINCIBILITY, SKULL, RepairRobotEffect3D, etc.

### 4.2 Base de partículas (`§_-r1f§` ParticleEffectBase + `§_-42y§` AttachedEffect3D)

- Efectos = archivos `.zip` de partículas (carpeta fx). Se clonan de un prototipo cacheado.
- **Scale-in**: `Quad.easeInOut` hasta escala objetivo en **0.5 s**; el objetivo se multiplica por el `visualSize` de la entidad. Dispose `"scaleDown"`: escala→0 en 0.5 s.
- Eventos con nombre dentro de la animación de partículas: `"end"` (autodispose), `"flash"`, y `"light*"` → **flash de luz puntual** (radio def. 100, duración 0.5 s, fade 0.3 s, color del evento, specular 20) vía pool de 3 PointLights del LightsManager.
- `billboard=1` en el prototipo → el efecto mira a cámara cada frame.

### 4.3 Explosión de muerte (`§_-bY§` ExplosionEffect3D)

- La factory de naves (`§_-Ac§.§_-r36§`) al destruir con explosión lanza `§_-o3§` en la posición + opcional shockwave (`§_-gT§`, id según patrón).
- Solo con efectos **high**: partícula `<resKey de la explosión>.zip` en la posición de muerte, fade 0.5; sonido posicional.
- Evento `"flash"` de la partícula → luz puntual: color **0xDEE4C8**, fallOff 400 (preset `Settings3D.§_-B9§`), encendida ~0.1 s.
- Con efectos **max** + `qualityExplosion`: partícula extra **`ship_debris.zip`** (solo naves estándar).

### 4.4 Impactos de daño (`§_-j3H§` DamageImpact3D)

- Solo con efectos high. Láser/ECI/Singularity → `impact_hull_laser.zip`; rocket/mina → `impact_hull_rocket.zip`.
- Posición: offset aleatorio dentro de `clickRadius·rand·0.5` alrededor del centro; `rotationY` aleatoria 0–360. Sin flash rojo del material: el "daño" son partículas de impacto + sonido.
- Si el golpeado es el jugador: **shakeScreen()** + sonido 7.

### 4.5 Cloaking

- `ShipDecorator.§_-d4v§`: TweenLite **0.2 s** a `alpha 0.5` (cloaked) / `alpha 1` (visible). El alpha cambia el material a variante alphaBlending.
- A nivel de modelo (`ShipModel.§_-f4x§`): las naves ENEMIGAS cloaked pierden el DisplayTrait completo (desaparecen del 3D); el alpha 0.5 solo se ve en tu nave (y las que sigues viendo).

### 4.6 Spawn/despawn

- Naves: **sin animación de spawn** — aparecen (o su placeholder) directamente; los NPC "spawnean" a menudo bajo su explosión previa. Despawn sin explosión = remoción directa.
- Colectables (cajas/ores): animaciones XML `spawn` (set scale=0 → tween scale=1 en 1 s), `dispose` (scale→0 en 1 s), `collect` (delay 0.5, y→0 y scale→0 en 1 s). `CollectableDecorator` cambia `disposeAnimation` a `"collect"` cuando el modelo marca recogida.
- Portales: animación `jump` con partículas `gate_*.zip`; el asset tiene animación de spawn 2D (spawn_animation) en display2D.

### 4.7 Glow pulsante y animaciones declarativas

- `<glow duration="5" minValue="0" maxValue="1"/>`: seno con fase aleatoria sobre el uniform `glow` del material (def. min 0, max 1, periodo 6 s). Usado por portales (5 s) y NPCs.
- `<background_animation><append target="ore" rotationZ="randomPick(-130,130)"/></background_animation>`: **`append` = grados POR SEGUNDO** (`§_-78§`: `prop += valor·dt`). Ores giran ±130°/s; spaceballs/asteroides `rotationY="randomPick(360,-360)"` = 1 vuelta/s con sentido aleatorio.
- `<tween duration ease repeat yoyo>` = TweenLite declarativo; funciones `random(a,b)` y `randomPick(a,b,…)` en cualquier atributo.

### 4.8 Marcador de selección (`§_-f4D§` TargetMarker3D)

- Partícula por lockType: 1=`abstract_ui_mark_target_red.zip`, 2=`gray_light`, 3=`purple`, 4=`gray_dark` (def. rojo).
- Al seleccionar: aparece a escala `visualSize·1.5` y **tween a visualSize en 0.3 s** (efecto de "cierre" sobre el objetivo). Sigue la posición del target cada frame.

---

## 5. Selección y hover (picking)

- Doble sistema: el gameplay usa el trait 2D `click_area` (`clickRadius` del patrón, proyección a pantalla). El **raycaster 3D** (`§_-x1d§` EntityRaycaster) se usa como test de "¿el puntero toca algo?" (`§_-V16§.§_-z1R§`).
- Raycast: `camera.unproject` → recorre colisiones de escena y filtra **solo submeshes cuyo parent es `§_-717§` (PrefabView3D)** — es decir, solo meshes de entidades; ordena por distancia.
- Colisionador por submesh: `§_-b1i§.§_-K1m§` = collider AS3 **por triángulos** (ray-triangle, primer impacto, findClosest=false). No bounding-box: el hover es preciso a la silueta.

## 6. NPCs vs jugadores — diferencias de pipeline

| Aspecto | Jugador | NPC |
|---|---|---|
| Clave de asset | `ship_<nombre>` (+`_facción`) | id numérico ("27") |
| Giro idle | no | ±180° aleatorio cada 2–7 s |
| Thrusters idle | escala 0.7 | escala 0 (apagados) |
| Thrusters (calidad medium) | sí | **no** |
| Placeholder timeout | 2500 ms (héroe) | 1000 ms |
| rotatable/tilting | siempre true | frecuentemente false (jefes, medusas, Hitac) |
| Extra héroe | PointLight azul 0x2E7DFF (fallOff 450, diffuse 0.6, specular 1.5; solo luces HIGH) + textura ×2 si zoom>1.3 | — |
| Sonido motor | loop posicional al moverse (solo héroe) | — |

Asteroides/estaciones/props: sin pipeline especial de naves; usan `background_animation` (`append rotationY` = vueltas continuas) y `floating` del XML. Las estaciones/battlestation son assets `building*` con módulos como entidades hijas.

## 7. Sombras / proyección al plano

**No existen.** Ni shadow maps ni blob shadows (grep "shadow" en materiales y view3D: 0 resultados; la lista de postefectos a calidad max está vacía). La lectura de profundidad la dan la luz direccional (tilt 100, pan 35 por defecto, override por XML del mapa) y el fondo.

## 8. Las clases preguntadas del enunciado

- `§_-i7§` = **StarfieldLayer**: clona la partícula `star_dust_chaotic.zip` (o `@starfield` del mapa) en una rejilla de celdas de **2000×2000** unidades cubriendo el mapa (bounds forzados y=0..−300); su `updateObj` solo avanza el tiempo de los animadores. Solo con efectos ≥ medium.
- `§_-f4D§` = **TargetMarker3D** (ver 4.8).
- `§_-31O§` = **EntityView3D**; `§_-x1d§` = **EntityRaycaster**; `MapDisplayLayer3D` = fábrica trait→vista (DisplayTrait→EntityView3D, más zonas).
- Paquete `§_-s2S§` = **nodos de animación de partículas custom** (extensiones del sistema de partículas de Away3D). `§_-j2P§` es un nodo "follow target": inyecta por vértice la posición/rotación de un objeto (con interpolación lineal si `smooth`) — es como los thrusters/estelas siguen a la nave dentro de un solo sistema de partículas. No es la capa de vista de entidades (esa es `§_-3L§`/`§_-Oj§`/`§_-n3Z§`).

---

## 9. Receta para Godot (traducción directa)

1. **Modelo**: posición lineal destino→destino a `speed` u/s; heading = atan2 instantáneo.
2. **Vista**: `yaw += diffAngle(yaw, heading−90°) · easeQuadOut(dt/0.2)`; roll objetivo = diff pendiente (clamp ±20°, o ±10° invertido ×2 con d=0.08 al atacar); en 2.5D top-down el roll se traduce a skew/escala del sprite u orientación del mesh.
3. **Idle**: hover ±5 u / ±5° (osciladores sin·cos, sin², periodo ~6.3 s, solo al estar parado, fade-out 0.5 s al arrancar); NPCs giran ±180° cada 2–7 s.
4. **Thrusters**: escala 0/0.7/1 (×1.5 o ×3 con buff), lerp 20 %/frame; estela 12 muestras × 30 ms con alpha lineal, color 0x5AC3D8.
5. **Feedback**: cloak alpha 0.5 en 0.2 s; marcador de selección 1.5→1 en 0.3 s; explosión = partícula + flash de luz 0.1 s + debris; impactos con offset aleatorio en el radio; shake solo cuando te pegan a ti.
6. **Drones**: slots de formación por tabla, rotación de formación con lag 0.3 s, posición individual con elástico 0.3 s.
