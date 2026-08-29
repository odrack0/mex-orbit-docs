# Guidelines 3D de DarkOrbit — ingeniería inversa del cliente

**Qué es esto**: análisis exhaustivo del cliente 3D de DarkOrbit (Flash/Stage3D), obtenido por ingeniería inversa del código decompilado (`D:\MexOrbit\Decompiled\`, JPEXS) y de los assets reales (`MexOrbit.CMS\public\spacemap\3d\`). El objetivo es capturar **cada decisión técnica y estratégica** con la que ese cliente logró el look & feel "2.5D top-down con 3D real", para compararlas contra el cliente Godot de MexOrbit y cerrar la brecha.

Todas las constantes de este documento son **literales del código**, no estimaciones. Los nombres de clase ofuscados (`§_-XX§`) y las rutas de evidencia están en los informes fuente (§16). Corrección aplicada durante la síntesis: el `ambientColor` por defecto del XML de mapa es decimal 16756398 = **0xFFAEAE**.

---

## 1. Arquitectura general

**Stack**: Away3D 4.x (motor 3D) + Starling 1.5.1 (solo el fondo espacial 2D) + display list Flash clásico (HUD y GUI), los tres compartiendo **un único Context3D de Stage3D** con perfil `"baseline"`.

**Render loop por frame** (un solo `present()`):

```
stage3DProxy.clear()          // color 0x000000 + depth
starling.render()             // SOLO el fondo espacial 2D (shareContext: ni limpia ni presenta)
away3d.render()               // clear SOLO de depth → escena 3D encima del fondo
stage3DProxy.present()
```

- El HUD **no** pasa por Stage3D: es display list Flash encima (Stage3D siempre dibuja detrás del stage). Los elementos HUD se posicionan proyectando el mundo 3D a pantalla.
- El frame loop real corre en `Event.RENDER` (con `stage.invalidate()` cada `ENTER_FRAME`): el estado que se dibuja es el más fresco posible y no se duplica trabajo en frames con resize. `stage.frameRate = 60`.
- dt medido con `getTimer()`, **sin clamp ni frame-skipping**: funciona porque todo el gameplay es interpolación hacia estados del servidor (tweens), no integración física local.
- Existe un cliente 2D completo detrás de la misma interfaz (`IMapView`): mismo modelo, mismo HUD, mismo input; solo cambia la capa de vista. La decisión 2D/3D es: opción del usuario (`FORCE_2D`), o driver por software/Flash viejo → 2D obligatorio.

**Ley estratégica #1**: el 3D es *solo una capa de presentación*. Modelo de juego, picking, HUD y protocolo son 100 % lógica 2D; el 3D proyecta. Eso es exactamente lo que hace trivial tener dos clientes y lo que MexOrbit debe conservar.

---

## 2. Mundo y coordenadas

- **Mapeo juego → 3D**: `(x, y)` de juego → `(x3D = x, y3D = altura, z3D = −y)`. La altura de las naves es **siempre 0** (plano del juego). Verificado en 3 sitios independientes.
- **1 unidad 3D = 1 unidad de juego** (mapa clásico ~21000×13100 u). No hay escala de escena: el famoso `0.0001` del contenedor raíz **no es escala, es `rotateTo(0.0001°)`** — una rotación épsilon por eje, workaround clásico de Away3D para sacar la transform del caso identidad (bugs de lookAt/decompose con rotación exactamente 0).
- La precisión de depth se maneja con el near plane (10), no con escala.
- Alturas por tipo de entidad: naves y=0; ores flotan a `y = random(−130, −70)` con tilt aleatorio; star dust en y ∈ [0, −300]; tilemaps de fondo 3D en `y = −3500 + capa·550`.

---

## 3. La cámara (el corazón del 2.5D)

**Perspectiva, nunca ortográfica.** Constantes exactas:

| Parámetro | Valor |
|---|---|
| FOV vertical | **30°** (bindable, clamp 2–180; en la práctica siempre 30) |
| near / far | **10 / 80000** |
| Distancia base al objetivo | **1740 u** (a zoom 1) |
| tilt por defecto | **135°** = la cámara mira la nave a **45° de elevación** (clamp [135, 179.9]; 179.9 = cenital) |
| pan por defecto | **0°** (**25°** en mapas con fondo 3D) |
| zoom | continuo, clamp **[1, 3]**; `distancia = 1740 / zoom` |

**Fórmula orbital** (posición de cámara, recalculada cada frame):

```
d = 1740 / zoom
pos.x = lookAt.x + d·sin(tilt)·sin(pan)
pos.y = lookAt.y − d·cos(tilt)
pos.z = lookAt.z − d·sin(tilt)·cos(pan)
lookAt = (heroX, 0, −heroY);  camera.lookAt(lookAt, up = +Y)
```

A zoom 1 / tilt 135: altura ≈ 1230 u, retroceso horizontal ≈ 1230 u; franja visible sobre el plano ≈ 930–1420 u.

- **Zoom**: rueda ×1.2 (acercar) / ×0.8 (alejar); el *factor* se anima con **tween 0.3 s Quad.easeOut** (no la distancia directamente).
- **Acoplamiento tilt↔zoom** (solo mapas con fondo 3D): `tiltEfectivo = tilt − clamp((zoom−1)/2 · 20, 0, 20)` → al acercar, la cámara baja hasta 20° hacia el horizonte (a zoom 3: elevación 25°, altura ≈ 245 u). Es el efecto cinemático de "picado bajo" al hacer zoom-in.
- **Seguimiento**: RÍGIDO, sin lerp ni deadzone — `camera = int(hero.pos)` cada tick. La suavidad viene de que la posición del héroe ya está interpolada por el modelo. El cast a `int` es deliberado (anti-jitter, ver §11).
- **Shake**: amplitud inicial 5 u, 40 pasos, 1 paso cada 24 ms (~1 s total), amplitud −1 cada 10 pasos; desplaza el *punto de mira* con `amp·(cos(paso), sin(paso))`. Solo cuando golpean al jugador.
- **Zonas de cámara** por mapa (`<cameraZone x y scale>`, círculo de radio `scale`): al entrar la primera vez, tween 1 s a zoom 1.5 y pan +10°; al salir, restore 1 s (zoom 1, FOV 30, pan→0 por el camino angular corto, tilt 135). Dentro de zonas: órbita libre con botón derecho (`pan −= 0.5·Δmx; tilt += 0.5·Δmy`).
- **Cámara cinemática por servidor**: TweenLite sobre el punto de mira — pan a un punto, volver al héroe en 3 s, o seguir a otra nave (modo espectador).
- **Sin clamping en bordes del mapa**: la cámara siempre centrada en el héroe; cerca del borde se ve "fuera" (el fondo lo cubre).

---

## 4. Proyección y desproyección

**Mundo → pantalla** (para HUD, marcadores, flares):

```
v = (x, altura, −y);  n = camera.project(v)          // NDC [−1,1]
px = viewportW·(n.x+1)/2;  py = viewportH·(n.y+1)/2
z devuelto = factor de zoom (1..3)                     // NO es profundidad
```

**Pantalla → mundo** (click-to-move, minimapa): rayo por `camera.unproject(2mx/W−1, 2my/H−1, depth=100)` e **intersección rayo-plano** con el plano `y = altura`, normal (0,1,0); si el rayo es casi paralelo (|den| ≤ 1e−6) devuelve null.

Consumidores clave:
- **Click-to-move**: mientras el botón está pulsado, CADA frame desproyecta el ratón al plano y=0 y, si `dist(héroe, punto) > clickRadius + 45`, envía comando de movimiento con **throttle de 200 ms** (el "conducir manteniendo pulsado" clásico). El destino se clampea al mapa.
- **Minimapa**: cada frame desproyecta las **4 esquinas del viewport** al plano y=0 → el rectángulo de vista en el minimapa es un **trapecio** (firma visual del 3D). Se dibuja solo el 12.5 % de cada lado desde cada esquina, línea 0.7 px 0xCCCCCC alpha 0.5, blendMode INVERT.
- Matemáticas de cámara con **cero asignaciones por frame**: vectores scratch reutilizados y out-parameters en todas las proyecciones.

---

## 5. Naves: la sensación de movimiento ⭐

Esta es la sección más valiosa. **Regla arquitectónica: el modelo es instantáneo y tosco; TODA la suavidad vive en la vista.**

### 5.1 Interpolación de posición (modelo)

- El servidor manda **destino** (no posiciones por frame). `duración = distancia / speed` (speed por nave, del servidor) salvo duración explícita.
- **TweenLite LINEAL** destino→destino; el tween se reutiliza (invalidate+restart) en cada update. Si cambia `speed` en vuelo, se relanza al mismo destino recalculando duración.
- Sin hermite, sin extrapolación. La suavidad visual la ponen el ease de rotación y el banking.
- Heading del modelo: `rotation = atan2(dy, dx) + π` — **asignación instantánea** al recibir destino (y se re-apunta al target atacado escuchando su `position.changed`). Umbral: destino a ≤2 u no re-apunta.

### 5.2 Giro suavizado y banking (la fórmula exacta)

Por frame, con dt en segundos:

```
targetY  = heading° − 90                                  // offset de orientación del mesh
deltaY   = diferenciaAngularCorta(yaw, targetY)           // grados, con signo
yaw      = QuadEaseOut(dt, yaw, deltaY, 0.2)              // "duración" 0.2 s → ~16 %/frame a 60fps

// Banking (roll): el ángulo de alabeo ES la diferencia angular pendiente
si (moviéndose && con objetivo de ataque):
    targetRoll = −deltaY · 2   clamp ±10°   d = 0.08      // se inclina AL REVÉS, más rápido y contenido
si no:
    targetRoll = +deltaY · 1   clamp ±20°   d = 0.2
roll = clamp(QuadEaseOut(dt, roll, diffCorta(roll, targetRoll), d), −max, +max)

aplicación: matriz Rz(−roll) · Ry(yaw) → decompose → euler
```

- `QuadEaseOut(t,b,c,d) = b + c·(t/d)·(2 − t/d)` — suavizado exponencial dependiente de framerate, constante efectiva ≈ 0.1 s.
- El roll **vuelve solo a 0** (cuando deltaY→0, targetRoll→0, el mismo ease lo baja). No hay tween de retorno.
- **No hay pitch** por aceleración/frenado. Solo yaw + roll + oscilación idle.
- Flags por XML del asset: `rotatable=false` (sin yaw — medusas, Hitac), `tilting=false` (sin banking), `floating=false`.

### 5.3 Flotación idle (hover)

- Solo **parada**; al arrancar, tween a 0 en 0.5 s; al parar, se reactiva.
- Amplitudes forzadas para naves: **posición (5,5,5) u y rotación (5,5,5)°**, cycleLength 2 → fase `+= dt/2` (0.5 rad/s, periodo visible ≈ 6.3 s).
- Oscilador Lissajous, no seno puro: `x = sin·cos, y = sin², z = cos·sin` de la fase (fase inicial aleatoria por nave). La componente Y es sin² → la nave flota **solo hacia arriba** 0..5 u.
- Portales: solo rotación ±3°. Ores: flotando a y −130..−70 con spin continuo ±130°/s.

### 5.4 NPCs idle

- Parados, cada **2000 + 5000·rand ms** (2–7 s) giran `±180°·rand` — la vista lo suaviza con el mismo ease de 0.2 s → los aliens "miran alrededor" perezosamente. Se cancela al moverse o al ser atacados.
- Thrusters de NPC idle: escala 0 (apagados); en calidad media los NPC no tienen thrusters.

### 5.5 Placeholder de carga (transición invisible 2D→3D)

Si el AWD no llegó en **1000 ms** (2500 ms la nave propia), se muestra un **plano 3D con el sprite 2D** de la nave (ocultando subclips "laser"/"engine"; scaleX = alto del clip, scaleZ = ancho, rotado 90°). Se sustituye al llegar el mesh. La animación AWD `"idle"` arranca con **offset aleatorio de hasta 10 s** (desincroniza flotas de naves idénticas).

---

## 6. Attachments: slots, thrusters, estelas, drones

### 6.1 Slots nombrados en el mesh

Los AWD traen submeshes-marcador que la carga convierte en puntos de anclaje:
- `engine_*` → slots de motor (no se renderizan).
- `laserpoint_<n>` → bocas de láser: se transforman por `sceneTransform` a offset mundial → **los disparos salen de los cañones reales incluso con banking**.
- `light_*` → filtrados (marcadores de autoría).
- Pods (p. ej. aegis-ship-pod): simplemente un `<mesh>` extra en el descriptor, modelado en su sitio.

### 6.2 Thrusters (llamas de motor)

- Partícula `thruster.zip` clonada en **cada** slot `engine_*`, rotationX=180, tiempo inicial aleatorio.
- Escala objetivo: **moviéndose 1 | idle jugador 0.7 | idle NPC 0**; **×3** con speed-buff (×1.5 sin buff); suavizado 20 %/frame; ocultos bajo escala 0.01.
- Siguen el banking: `rotationY = roll/4`.

### 6.3 Estela de motor (ribbon CPU, NO partículas)

- Ring buffer de **12 muestras cada 30 ms** (~0.36 s de historia) por slot de motor; alpha decae lineal `1 − i/12`; el vertex shader extruye la cinta (varios trails por draw call).
- Color por defecto **0x5AC3D8** (cian), override por nave con `@engineTrailColor`.
- Anti-teleport: salto >100 u → la cinta entera se colapsa a la posición nueva (evita el "latigazo").

### 6.4 Drones

- Formaciones en XML (`game-patterns-profile.xml`): lista de posiciones relativas exactas por número de drones, con variantes 2D/3D.
- El contenedor de drones sigue la nave **incluida su oscilación de hover**.
- **Rotación de la formación con lag propio**: ease 0.3 s (vs 0.2 del casco) → al girar, el anillo de drones "se arrastra" detrás. Posición individual de cada dron: elástico 0.3 s por eje.
- Grupos rotatorios declarativos: `<rotationGroup><tween duration="5" rotationY="-360" repeat="-1"/>` → anillos que orbitan 360° cada 5 s, taladros contrarrotantes.
- Cloak de la nave aplica alpha 0.5 también a sus drones.

---

## 7. Materiales e iluminación

### 7.1 Material de nave

- `ShipMaterial` = TextureMaterial de Away3D + 2 métodos extra: **alphaMask** y **glowMap**.
- **El glow NO es ambient/emissive del material**: es una 2ª textura sumada aditivamente al final del fragment shader, multiplicada por un **factor escalar por-instancia** (`glow`, default 1) — independiente de la iluminación, animable (pulsos senoidales de portales, glow ligado al % de HP).
- Defaults: `specular 1`, `gloss 50` (por XML: `specularity`, `gloss`, y variantes `*Low` cuando el mapa specular está apagado por calidad).
- Canales por convención de nombre: `<base>_diffuse|normal|specular|glow|alpha_<128|256|512>.atf`; `"none"` desactiva un canal; los canales heredan el nombre de `@texture` que hereda de `@geometry`.
- Shaders registrados por nombre (data-driven): `basic`, `ship` (default), `organic_ship`, `uber_ship`, `uber_organic_ship`, `sector_control_beacon`, `animated_cloud`.

### 7.2 Iluminación (presupuesto duro)

- **1 DirectionalLight** ("sol") + **1 PointLight del héroe** + **pool de exactamente 3 PointLights de efectos**. **Sin sombras de ningún tipo** (ni shadow maps ni blobs) y sin post-proceso.
- Sol configurable por XML del mapa; defaults: color 0xFFFFFF, diffuse 1, specular 0.7, **ambientColor 0xFFAEAE, ambient 0.2, tilt 100, pan 35**. Dirección: `dir = −(sin(tilt)·sin(pan), sin(tilt)·cos(pan), cos(tilt))` ≈ (−0.565, −0.807, +0.174) — casi cenital, ligeramente lateral.
- **Luz del héroe**: PointLight **azul 0x2E7DFF**, diffuse 0.6, specular 1.5, **fallOff 450**, parentada a la nave propia → tu nave "brilla" y se distingue en cualquier fondo. (Solo con luces en HIGH.)
- Luces de efectos (pool de 3): presets p. ej. explosión 0xDEE4C8 specular 20 fallOff 400 (~0.1 s), disparo 0xF7C0C0 fallOff 200; tween de fade y expiración automática.
- Por calidad: LOW sin luces; MEDIUM solo sol + 1 luz de efecto; HIGH todo. Las **partículas** solo ven el sol, y solo con efectos ≥ HIGH.
- La lista de luces del picker se reconstruye **solo con flag dirty**, nunca cada frame.

---

## 8. Pipeline de assets

### 8.1 Manifiesto y URLs

- Manifiesto `resources_3d.xml` (+ `resources_3d_particles.xml` para fx): id lógico → `{ruta, tipo, hash MD5}`. 2854 atf + 245 awd.
- URL = `staticHost + basePath + location + name.type + "?__cv=" + hashDelArchivo` → **cache-busting por archivo individual**, no global.

### 8.2 Formatos

- **Mallas**: AWD 2.1 estándar (magic `AWD`, versión 2.1, cuerpo zlib), 10–60 KB por nave (~1–3k triángulos). El cliente también soporta un formato propietario "AWD 99.1" (lzma, texturas embebidas) no usado en este snapshot. OBJ para shapes de fx.
- **Texturas**: ATF **DXT1 con cadena de mips completa precalculada** (ej. 512×512, 10 mips). Subida a GPU **asíncrona** (`uploadCompressedTextureFromByteArray(async=true)` + TEXTURE_READY): hasta que está lista el material dibuja sin ella → **cero hitches de subida**.
- **Parseo AWD con presupuesto de 30 ms por frame** (ParserBase incremental de Away3D): un modelo grande se reparte entre frames.

### 8.3 Resolución de textura = f(categoría × calidad × zoom)

```
resolución = clamp( perfilCategoría.size × multiplicadorZoom, 128, 512 )
```

| Categoría (`tex_settings`) | LOW | MEDIUM | HIGH |
|---|---|---|---|
| drone / ship_very_small / ship_small | 64 | 64 | 128 |
| ship | 128 | 128 | 256 |
| ship_big | 256 | 256 | 512 |
| building_small / planet_small | 128 | 256 | 512 |
| building / planet | 256 | 512 | 1024 |

- **multiplicadorZoom = 2 cuando el zoom de cámara > 1.3** → al acercar la cámara, las naves suben un escalón de resolución en caliente.
- **Normal y specular a MITAD de la resolución** del diffuse; glow a resolución completa con tope 512.
- Toggles por calidad: normal+specular solo en HIGH; glow desde MEDIUM; smoothing desde MEDIUM; mipmapping solo HIGH.
- **La resolución depende de la clase de objeto, no solo del tier global**: un dron nunca pasa de 128 ni en ULTRA.

### 8.4 Caché, refcount y lifetimes

- Un loader-caché por tipo (geometría/texturas/fx) con **promesa por resKey**: pedir dos veces = misma descarga en vuelo.
- **Refcount con periodo de gracia de 5 s**: al llegar a 0 se arma un timer; si nadie lo retoma en 5 s, se libera. Evita thrashing cuando una nave sale y vuelve a entrar.
- **Tres lifetimes**: 0 = transitorio (naves), 1 = de mapa (fx, assets del mapa), 2 = permanente (precarga + skybox). Cambio de mapa = `reset(1)`: purga ≤1, conserva la precarga. El lifetime solo puede *subir*.
- Precarga declarada en XML del servidor (`preloadLists.pack id="3D"`); el juego espera a que termine antes de entrar. Todo lo demás es lazy (la nave se carga al aparecer; mientras, placeholder §5.5).
- **Materiales compartidos refcontados**: clave = clase shader + perfil de calidad + hash MD5 de los parámetros → todas las naves iguales comparten UN material (texturas, programas, estado). Los meshes se clonan compartiendo geometría (vertex buffers una sola vez en GPU).
- Cambio de mapa: el Context3D **se destruye adrede** y se pide otro → VRAM limpia garantizada.

---

## 9. Sistema de efectos (.awp) — partículas GPU declarativas

### 9.1 El modelo mental

Un efecto = un JSON (`.awp` dentro de un `.zip`) con N **capas**; cada capa es UNA mesh (N copias de una shape ensambladas en un solo vertex buffer = **1 draw call por capa**) + una lista de **nodos** GPU que definen el comportamiento por partícula, horneado en atributos de vértice. La animación se avanza a mano (`animator.time += dt`), no hay autoplay.

```
{ particleEvents:[{occurTime, name}], customParameters:{},
  animationDatas:[ { property:{pos/rot/scale/timeOffset/playSpeed},
                     data:{ name, bounds, material, geometry:{assembler:{num, shape}}, nodes:[...] } } ] }
```

### 9.2 Los 18 nodos (equivalencias Godot en §15)

Time (start/duration/loop/delay — siempre el primero) · Velocity · Acceleration · Position (emisor volumétrico) · Billboard (con eje opcional = cilíndrico) · Follow (partículas quedan en mundo; trail de humo) · Scale · Color · Oscillator · RotationalVelocity · Orbit · BezierCurve · SpriteSheet (flipbook) · RotateToHeading (alineada a velocidad; chispas) · **SegmentedColor** (gradiente multi-punto sobre la vida — el nodo estrella) · InitialColor (recolorear assets grises) · SegmentedScale · **UV** (scroll de patrón: lineal o senoidal, `cycle` en s — la vida de los beams).

Tipos de valor: const, random[min,max], curva por anclas (¡interpola sobre el **índice** de partícula i/total, no el tiempo!), esfera/cilindro (emisores y velocidades radiales), ColorTransform, matriz por copia y **celda de atlas aleatoria por copia**.

Shapes: Plane (el sprite típico), Cube, Sphere, Cylinder, o **malla externa .obj** (beams, anillos, shockwaves, crosshairs).

### 9.3 Eventos y parámetros custom (runtime)

- `particleEvents`: marcadores temporales. `end` → autodestrucción; `light*` → **luz dinámica puntual** con parámetros en `customParameters[nombre]` (`{color, radius, duration, fading}`); otros nombres → callbacks (coreografías).
- `customParameters`: `scalable`/`maxScale` (escala al `visualSize` de la nave, ease 0.5 s), `dispose:"scaleDown"` (escala a 0 en 0.5 s al morir), `billboard:1` (todo el contenedor mira a cámara — marcadores UI 3D), `lightPickerTargets` (capas que reciben iluminación).

### 9.4 Recetas destiladas de los .awp reales

- **Beam**: malla de largo fijo (~110 u) + textura patrón repeat + UVNode (cycle 0.3–0.5 s). El estiramiento es runtime: `scaleZ = dist/110·rampa` (rampa 0→1 al aparecer), opcional `scaleX = dist/250`, `rotationY = atan2(dy,dx)+90°`, posición en el origen cada frame. **Chain lightning** = un beam por salto con 0.2 s de retardo + fx de esquina en cada vértice; autodispose 3 s.
- **Explosión** (patrón de 4 capas): chispas rápidas (400×10 u, velocity esférica 100–200, gravedad −100..−500, RotateToHeading, vida 0.1–0.8 s) + nubes medianas (60×120 u, 250–350) + **flash central de 1 frame** (1 quad 250 u, vida 0.25 s, scale 0→1.4) + fireballs lentos (20×65 u, atlas 4×2 celda aleatoria, vida 1.5 s). Muerte de nave: partícula + flash de luz 0.1 s + `ship_debris.zip` en max.
- **Thruster**: 40 quads 8 u, loop 1 s con startTime aleatorio (emisión continua), escape hacia atrás, cian→blanco→cian.
- **Humo de cohete**: FollowNode — el cohete avanza, el humo **queda en el mundo**.
- **Marcadores UI 3D** (target lock): malla crosshair, scale pulsante + spin; entra a 1.5× y cierra a 1× en 0.3 s.
- **Coreografías**: efectos de 11 s en un solo .awp usando el startTime de cada capa como partitura (warp de nave: build-up → clímax con shockwave + luz + evento).

### 9.5 Cómo consiguen EL LOOK (leyes visuales)

1. **Additive en todo**: `blend add` + unshaded es el default (~95 %). Los overlaps SUMAN luz. **No hay bloom**: el "glow" son texturas de gradiente radial suave sumadas.
2. **El fade SIEMPRE por SegmentedColor** (envolvente RGB+alpha multi-punto), nunca por opacidad del material.
3. **Desincronización estadística**: startTime/duration aleatorios por partícula + timeOffset por capa + fase aleatoria por instancia → ningún loop se ve repetido. (Es lo que mata el aspecto "de videojuego barato".)
4. **Anillos y shockwaves = mallas planas en el plano del mapa** (scale 1→45 en 0.3 s), NO billboards.
5. Assets genéricos grises + InitialColor para recoloreo (una familia de texturas sirve para 20 efectos).
6. Luces dinámicas puntuales sincronizadas con el clímax del efecto vía particleEvents.
7. Calidad max: shader burn/dissolve en fireballs (atlas color+máscara, frente de combustión que avanza).

---

## 10. El fondo espacial

Tres mecanismos superpuestos:

### 10.1 Fondo 2D con parallax (Starling; mapas "planos" y base de todos)

- Definido por XML del mapa: capas `{textura/atlas, pFactor (default 10), layer, shift, scale, tiled?, mask?}` + planetas `{x, y, rot, pFactor}` + lensflares.
- **Orden de dibujo** = `1000/pFactor + layer` ascendente (más lejos primero).
- **Scroll**: `pos = int(−cam/pFactor) + viewport/2 + offset`; planetas `int((p − cam)/pFactor) + viewport/2`. **Snap a entero siempre** (anti-jitter).
- Tiles: grid con selección **aleatoria evitando repetir el vecino izquierdo y el de arriba**; máscara opcional (bitmap sampleado al centro de celda → tiles vacíos = fondos con formas); margen de 1 tile alrededor del viewport.
- El fondo NO se escala con el zoom 3D (el zoom real es de cámara).

### 10.2 Fondo 3D por mapa (mini-escena + tilemap a profundidad real)

- Si el mapa declara `<display3D templateId>`: se apaga el fondo Starling y se monta una mini-escena Away3D: `container/mesh/plane/particles/tilemap`, con transforms normales o **coordenadas esféricas** (planetas/soles a distancia R), `<floating>` senoidal, planos billboard para nebulosas/soles.
- **Tilemap 3D** (la joya del parallax): una sola malla estática con un quad por tile a `y = −3500 + capa·550`, y **cada tile con offset vertical aleatorio propio** (−500..−200) → el parallax entre tiles lo produce la propia cámara en perspectiva. No hay fórmula de scroll: es 3D de verdad. Tiles a escala 5×.

### 10.3 Skybox con twinkle procedural

- Esfera ×10000 **recentrada cada frame en el punto que mira la cámara** (no en la cámara), dibujada antes que todo, sin escribir depth.
- Shader: `color = stars(uv) · mask(uv + (2,1)·t) · mask(uv + (−1.5,1)·t)` con `t = segundos/120` → **producto de dos máscaras móviles = titileo pseudoaleatorio de estrellas**. Barato y precioso.

### 10.4 Star dust (paralaje cercano)

- `star_dust*.zip`: 1500 quads de 3 u en volumen 2000×400×2000, deriva lenta, colores variados; **clonado en mosaico cada 2000 u** por todo el mapa (bounds manuales por tile → frustum culling independiente), y ∈ [0, −300]. Elegido por `@starfield` del XML del mapa. Solo con efectos ≥ MEDIUM. Es la capa que "vende" el movimiento y la profundidad al volar.

### 10.5 Lensflares

- Capa 2D encima del mundo, throttle 500 ms. Cadena de N sprites sobre la recta flare↔centro de pantalla extendida ×3 al lado opuesto.
- **Oclusión real**: visible solo si nada lo tapa — primero colliders 2D (alpha > 128 del bitmap), luego **raycast 3D** contra la escena. Fade-in 0.1 s, fade-out inmediato. Con `star=true`, destello giratorio (−9°/s).

---

## 11. HUD, input y picking

- **Patrón trait→vista**: el HUD registra pares (trait del modelo → clase de vista HUD); un listener add/remove crea y destruye vistas. Capas ordenadas (naves=0, iconos=4…).
- **Posicionamiento**: cada frame `p = project(entidad)`; `x = int(p.x); y = int(p.y)` — **snap a píxel = anti-jitter** (combinado con la cámara en enteros y el fondo en enteros: toda la cadena redondea).
- **Los elementos HUD NO escalan con el zoom** (tamaño constante en pantalla). Única excepción: el offset vertical de las barras de la nave PROPIA se multiplica por el zoom (para no solaparse con la nave al acercar la cámara).
- **Picking de entidades: NO es raycast 3D.** Recorre los click-areas ordenados por `clickPriority` descendente, proyecta el centro de cada entidad a pantalla y compara **distancia 2D en píxeles ≤ clickRadius** (radio constante en pantalla). El raycast 3D por triángulos existe pero se usa para oclusión de lensflares y hover cosmético.
- Cadena de prioridad de clicks (gratis por el orden del display list): GUI/ventanas → HUD interactivo → hitLayer ("suelo": click-to-move §4).
- Doble click <500 ms sobre el target = atacar. Rueda = zoom; Alt+rueda = radio de drones (0.5–2, pasos de 0.1).
- Minimapa: `mini = mundo·k`; viewport como trapecio (§4); click → mundo con la inversa.

---

## 12. Calidad, rendimiento y robustez

### 12.1 Settings (4 sliders 3D + globales)

Tier entero LOW/MEDIUM/HIGH/ULTRA con **booleanos derivados precalculados** (`effects.medium`, `effects.high`, `effects.max`…) y señal `changed` → **todo cambio de calidad se aplica en vivo** (materiales, luces, AA), nada requiere reiniciar. Sliders: antialias (0/2/8/16), luces (§7.2), texturas (tamaño + mapas), efectos.

**Qué recorta cada tier de efectos** (gates por sitio de invocación, no budget global):
- ≥ MEDIUM: star dust, efecto de escudo, fx de estado.
- ≥ HIGH: muzzle-flashes, luces dinámicas, thrusters de NPC, estelas, impactos de daño, explosiones completas.
- = ULTRA: debris de naves, shader burn de fireballs.
- En LOW quedan solo los fx de gameplay imprescindibles (beams, explosión básica).

### 12.2 Auto-quality por FPS (con histéresis)

- Muestrea FPS 1/s, promedia ventanas de **20 s**. Promedio < **10 FPS** → +1 nivel de reducción (máx 5); > **60 FPS** → −1 (solo recupera cuando sobra).
- Escalera de recortes ordenada por costo/valor: 1 sin preview de portales → 2 sin humo de motores → 3 sin map-assets animados → 4 explosiones en detalle bajo → 5 sin explosiones ni movimiento de starfield.
- **No mide con la ventana sin foco** (se desengancha en DEACTIVATE).
- Aviso de "low performance" al usuario tras N ventanas malas, máximo 3 veces.

### 12.3 Pooling y cero basura

- Pool genérico por clave con **warm-spare** (mantiene 1 tibio; reusa a partir del 2º) para salvas de láser, muzzle flashes, impactos. Objetos con wake/sleep. Pools vaciados al cambiar de mapa.
- 3 PointLights pre-creadas (nunca se instancian luces en gameplay).
- Vectores scratch + out-parameters en toda la matemática por frame.
- Colecciones iteradas **en reversa** para tolerar remociones durante el update.

### 12.4 Robustez de contexto (≈ device-loss moderno)

- Validar antes de dibujar: si el contexto murió, **un solo re-request** (con guard), y la simulación sigue corriendo sin renderizar (ni crash ni spam).
- Re-subida **perezosa**: cada textura recuerda con qué contexto se creó; al primer uso con contexto nuevo se re-crea y re-sube. Nada se re-sube en bloque.
- `Starling.handleLostContext = true`; errorChecking solo en debug.

### 12.5 Telemetría

- Triángulos por frame acumulados por el collector; panel FPS/MS/MEM oculto; envío de driver/profile/uso-2D/frameRate al servidor → **las decisiones de tiers se toman con datos de la población real**.

---

## 13. Las leyes de oro (síntesis estratégica)

1. **El 3D es presentación pura.** Gameplay, picking, HUD y red son lógica 2D; el 3D proyecta. Nunca acoplar una mecánica a la escena 3D.
2. **Modelo tosco, vista sedosa.** El servidor/modelo asigna posiciones y headings de golpe; la vista aplica ease exponencial (~0.1–0.2 s) + banking derivado del error angular. Toda la "sensación" son 6 constantes: `d_yaw=0.2`, `roll=±20°`, combate `−2×/±10°/0.08`, hover `5u/5°/6.3s`.
3. **Perspectiva FOV 30 a 45°, no ortográfica.** El FOV angosto + tilt 45° + tilt dinámico con el zoom es lo que hace que se sienta "2.5D con profundidad" sin perder la legibilidad top-down.
4. **La cadena entera redondea a píxel** (cámara int, fondo int, HUD int) — así se mata el jitter con cámara rígida.
5. **Additive + unshaded + gradientes suaves, sin bloom.** El look "espacial luminoso" es composición aditiva de texturas radiales, envolventes de color multi-punto y desincronización aleatoria. Ningún post-proceso.
6. **Efectos declarativos (datos, no código)**: un fx = archivo JSON con capas/nodos/eventos; artistas iteran sin recompilar. Los eventos (`end`, `light*`) sincronizan luces y lifecycle con el clímax visual.
7. **Presupuestos duros, no genéricos**: 5 luces totales (1+1+3), 12 muestras de estela, 3 niveles de textura, 1 draw call por capa de fx. Recortes de calidad **por sitio**, con booleanos precalculados, aplicables en vivo.
8. **Cargar sin bloquear jamás**: parseo con presupuesto de 30 ms/frame, subida asíncrona, placeholder 2D a 1 s, idle con offset aleatorio. El jugador nunca ve un hitch ni una nave "T-pose".
9. **Caché con lifetimes (uso/mapa/permanente) + refcount con gracia de 5 s** + materiales/geometría compartidos. Cambio de mapa = purga selectiva + VRAM limpia.
10. **Todo es data-driven por XML**: naves (geometry/texture/scale/visualSize/flags), mapas (fondos, luz, cameraZones, starfield), formaciones de drones, animaciones (`append` = °/s, tweens declarativos), condiciones de calidad ("effects.high AND background.max"). El código es un intérprete.
11. **Auto-quality con histéresis y telemetría**: medir de verdad (ventanas de 20 s, sin foco no cuenta), degradar por escalera de costo/valor, recuperar solo con holgura.
12. **La distinción del héroe es luz, no UI**: point light azul en tu nave + textura ×2 al hacer zoom. Elegante y barato.

---

## 14. Diferencias clave a auditar en el cliente Godot de MexOrbit

Checklist de comparación (cada punto = una sesión de auditoría contra `GodotClient`):

- [ ] Cámara: ¿perspectiva 30°/45° con distancia 1740/zoom y tween 0.3 s, o algo ad-hoc?
- [ ] ¿El banking deriva del error angular con las constantes de §5.2? ¿Existe el modo combate invertido?
- [ ] ¿Interpolación lineal destino→destino con duración = dist/speed (no snapping ni extrapolación)?
- [ ] ¿Hover idle Lissajous con fade al arrancar? ¿NPCs giran en idle?
- [ ] ¿Slots de motor/láser en los modelos (Meshy/Blender) como nodos nombrados?
- [ ] ¿Estela ribbon 12×30 ms? ¿Thrusters con escala por estado?
- [ ] ¿FX aditivos unshaded con color ramp como envolvente y aleatorización de fase?
- [ ] ¿Beams = malla fija estirada + UV scroll?
- [ ] ¿Snap a píxel en cámara/HUD? ¿Picking 2D por radio proyectado con prioridades?
- [ ] ¿Presupuesto de luces? ¿Glow emissivo por canal ligado a HP?
- [ ] ¿Tiers de calidad con booleanos y gates por sitio? ¿Auto-quality?
- [ ] ¿Carga asíncrona con placeholder y caché por lifetime?
- [ ] ¿Star dust en mosaico + skybox twinkle + parallax por capas?

## 15. Mapeo técnica → Godot (referencia rápida)

| Técnica DarkOrbit | Equivalente Godot |
|---|---|
| Capa .awp (N copias + nodos GPU) | `GPUParticles3D` + `ParticleProcessMaterial` (amount=num, lifetime=duration, randomness para startTime) |
| Efecto multi-capa + particleEvents | Escena .tscn con varios GPUParticles3D + `AnimationPlayer` (los eventos = keyframes que llaman métodos/luces) |
| blend add + unshaded | `StandardMaterial3D`: BLEND_ADD + UNSHADED |
| SegmentedColor / SegmentedScale | `color_ramp` (GradientTexture1D) / `scale_curve` (CurveTexture) — mapeo 1:1 |
| Billboard / RotateToHeading | `billboard_mode = PARTICLES` / `align_y = true` |
| Velocity/Position esfera-cilindro | `emission_shape` + `initial_velocity_min/max` + `spread` |
| SpriteSheet / celda por copia | `particles_anim_h/v_frames` + `anim_offset` random |
| UVNode (beam) | `MeshInstance3D` + shader `uv.x += TIME/cycle`, textura repeat |
| Beam estirado | `look_at` en el plano + `scale.z = dist/largoMalla` + Tween de rampa |
| FollowNode (humo que queda) | GPUParticles3D con `local_coords = false` |
| Estela ribbon | ImmediateMesh/MultiMesh con el mismo ring buffer (12×30 ms, alpha 1−i/12) |
| Eventos light* | `OmniLight3D` + Tween de energía (duration/fading del JSON) |
| Glow map + factor | `emission_texture` + `emission_energy` animable |
| ATF DXT1 + mips | Importar con compresión VRAM + mipmaps (misma idea) |
| Parseo 30 ms/frame + upload async | `ResourceLoader.load_threaded_request` |
| Skybox twinkle | Sky shader: `stars * mask(uv+v1·t) * mask(uv+v2·t)`, v1=(2,1)/120, v2=(−1.5,1)/120 |
| Auto-quality | Timer 1 s + ventana 20 s sobre `Performance.TIME_FPS`, escalera de flags |
| QuadEaseOut incremental | `pos += diff * min(1.0, (dt/d)*(2.0 - dt/d))` (o `1 − exp(−dt/τ)` con τ≈0.1) |

---

## 16. Fuentes y trazabilidad

Informes de excavación completos (con mapa de clases ofuscadas → deducidas, rutas de archivo exactas y líneas), generados el 2026-08-28, en [darkorbit-3d/](darkorbit-3d/):

- [camara-proyeccion.md](darkorbit-3d/camara-proyeccion.md) — cámara, lentes, renderer, luces
- [naves-entidades.md](darkorbit-3d/naves-entidades.md) — descriptores, movimiento, banking, attachments, drones, fx por entidad
- [pipeline-assets.md](darkorbit-3d/pipeline-assets.md) — manifiesto, AWD/ATF, materiales, caché/lifetimes
- [particulas-fx.md](darkorbit-3d/particulas-fx.md) — formato .awp completo, 15 efectos analizados, runtime
- [calidad-rendimiento.md](darkorbit-3d/calidad-rendimiento.md) — settings, auto-quality, pooling, context loss, frame loop
- [fondo-hud-integracion.md](darkorbit-3d/fondo-hud-integracion.md) — parallax, tilemap 3D, skybox, HUD, input, minimapa

Código fuente: `D:\MexOrbit\Decompiled\spacemap\main\scripts\` (núcleo 3D en `net\bigpoint\darkorbit\map\view3D\` y ~40 paquetes ofuscados). Assets: `MexOrbit.CMS\public\spacemap\3d\` (322 meshes AWD, 4401 texturas ATF, 663 fx). Stack identificado: Away3D 4.x + Starling 1.5.1 + Stage3D "baseline", TweenLite/TweenMax.
