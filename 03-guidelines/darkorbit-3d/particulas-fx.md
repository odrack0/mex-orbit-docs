# Sistema de efectos de partículas del cliente 3D de DarkOrbit (formato .awp)

Ingeniería inversa del cliente decompilado (`D:\MexOrbit\Decompiled\spacemap\main\scripts\`) y de los assets
(`C:\Source\MexOrbit\MexOrbit.CMS\public\spacemap\3d\fx\*.zip`). Extraídos y analizados 15 .awp representativos en
`...\scratchpad\fx_extract\` (hay `.pretty.json` junto a cada `.awp`).

Clases clave (rutas exactas, paquetes ofuscados):

| Rol | Archivo |
|---|---|
| Parser del .awp (tipo "awp", raíz JSON) | `§_-F1w§\§_-PT§.as` |
| Parser de cada animationData (tipo "pam") | `§_-F1w§\ParticleAnimationParser.as` |
| Registro id→clase de SubParsers | `§_-b2D§\§_-D3Y§.as` (ids string en `§_-b2D§\§_-q1X§.as`) |
| Loader/caché de efectos (AssetsManager3D.§_-m3a§) | `§_-n3U§\§_-u1u§.as` (método `§_-c2T§(url, lifetime, cache)`) |
| Contenedor runtime del efecto (template clonable) | `§_-k4J§\§_-JU§.as` |
| Animator compuesto + eventos | `away3d\animators\§_-71i§.as`, evento `§_-p3c§\§_-9Y§.as` |
| Wrapper de instancia (play/scale/dispose/eventos) | `§_-n3Z§\§_-r1f§.as` |
| Wrapper anclado a entidad (posición viva) | `§_-01b§\§_-42y§.as` |
| Wrapper de BEAM (estirar entre 2 puntos) | `§_-01b§\§_-w4§.as` |
| Chain lightning (beam por salto + esquinas) | `§_-01b§\§_-U1q§.as` |
| Pool de instancias | `§_-321§\§_-eI§.as` |
| Trail de motor (ribbon CPU, NO .awp) | `§_-G1y§\§_-Pd§.as` + `§_-w2S§\EngineTrailMaterial.as` |
| Shader de bola de fuego (calidad max) | `§_-w2S§\ExplosionMaterial.as` |
| Starfield ambiental por mapa | `net\bigpoint\darkorbit\map\view3D\display3D\§_-i7§.as` |
| Nodo UV GPU (scroll de patrón) | `§_-K1W§\§_-914§.as` |

---

## 1. Formato .awp COMPLETO

Un `.zip` de fx contiene un `.awp` (JSON) y, a veces, recursos (`.obj`, `.png`). Las texturas/geometrías externas se
resuelven vía `xml/resources_3d_particles.xml` (FileCollection aparte de la de modelos).

### 1.1 Raíz

```json
{
  "particleEvents":  [ {"occurTime": 1.25, "name": "end"}, ... ],   // opcional
  "customParameters": { ... },                                        // opcional, libre
  "animationDatas": [
    {
      "embed": true,                       // si false: "url" a otro .pam externo
      "property": { "id": "InstancePropertySubParser", "data": {...} },  // transform del emisor
      "data": {                            // ===== un "sistema" (una mesh de partículas) =====
        "name": "chain_bolt",              // opcional; usado por lightPickerTargets y shader_fireball
        "bounds": 100,                     // radio de la esfera de culling
        "shareAnimationGeometry": true,    // compartir geometría animada entre clones
        "material": { "id": "...MaterialSubParser", "data": {...} },
        "geometry": { "embed": true, "data": { "assembler": { "id": "SingleGeometrySubParser", "data": {...} } } },
        "nodes": [ { "id": "Particle...NodeSubParser", "data": {...} }, ... ],  // nodes[0] SIEMPRE es TimeNode
        "globalValues": [ ... ]            // existe en el parser pero el registro está VACÍO (sin uso)
      }
    }
  ]
}
```

Un efecto = N `animationDatas` (capas). Cada capa es UNA mesh Stage3D con su animación GPU propia
(1 draw call por capa). Ej.: `warp_ship` tiene 7 capas; `explosion3` tiene 4.

### 1.2 InstancePropertySubParser (transform y timing de cada capa)

`position` (3D), `rotation` (3D, grados euler), `scale` (3D), `timeOffset` (seg, puede ser negativo — adelanta la
animación), `playSpeed` (multiplicador). Todos aceptan cualquier value-subparser (const o random).

### 1.3 Nodos soportados (registro completo, `§_-D3Y§.§_-g1w§`)

Los ids son literales sin ofuscar. Semántica = nodos de partículas de Away3D 4.x (LocalStatic: atributo por
partícula horneado en vértices; los "uses*" eligen variante del shader AGAL).

| id (SubParser) | Parámetros en `data` | Semántica |
|---|---|---|
| `ParticleTimeNodeSubParser` | `usesDuration`, `usesLooping`, `usesDelay` (bool); `startTime`, `duration`, `delay` (1D) | Obligatorio y primero. startTime = retraso de nacimiento por partícula; duration = vida; looping repite vida; delay = pausa entre loops. Construye el ParticleAnimationSet. |
| `ParticleVelocityNodeSubParser` | `velocity` (3D) | Velocidad constante por partícula (u/seg). |
| `ParticleAccelerationNodeSubParser` | `acceleration` (3D) | Aceleración constante (½at²). |
| `ParticlePositionNodeSubParser` | `position` (3D) | Posición inicial por partícula (emisor volumétrico). |
| `ParticleBillboardNodeSubParser` | `usesAxis` (bool); `axisX/axisY/axisZ` si true | Billboard GPU hacia cámara; con eje = billboard restringido (cilíndrico). |
| `ParticleFollowNodeSubParser` | `usesPosition`, `usesRotation` (bool) | Partículas quedan en espacio mundo; el punto de emisión sigue al objeto (trail). Runtime: `FollowParticleContainer` escribe cada frame la posición del target en el atributo de las partículas que nacen. |
| `ParticleScaleNodeSubParser` | `usesCycle`, `usesPhase` (bool); `scale` (4D: x=min,y=max,z=cycleDur,w=fase) | Sin cycle: interpola escala min→max a lo largo de la vida. Con cycle: oscila (pulso). |
| `ParticleColorNodeSubParser` | `usesCycle`, `usesPhase`, `cycleDuration`, `cyclePhase`, `startColor`, `endColor` (ColorTransform) | Interpolación de color inicio→fin (o cíclica). |
| `ParticleOscillatorNodeSubParser` | `oscillator` (4D: xyz=amplitud, w=periodo) | Oscilación senoidal de posición. |
| `ParticleRotationalVelocityNodeSubParser` | `rotation` (4D: xyz=eje, w=vueltas/ciclo) | Rotación continua alrededor de un eje. |
| `ParticleOrbitNodeSubParser` | `usesCycle`, `usesPhase`; `orbit` (4D: radio/ciclo/fase), `eulers` (3D) | Órbita circular alrededor del origen, plano inclinable por eulers. |
| `ParticleBezierCurveNodeSubParser` | `control` (3D), `end` (3D) | Trayectoria Bézier cuadrática origen→end con punto de control. |
| `ParticleSpriteSheetNodeSubParser` | `numColumns`, `numRows`, `total`, `usesCycle`, `usesPhase`, `scale` (4D) | Animación de flipbook UV. |
| `ParticleRotateToHeadingNodeSubParser` | (sin parámetros) | Orienta la partícula a su vector velocidad (chispas alargadas). Se combina con Billboard. |
| `ParticleSegmentedColorNodeSubParser` | `usesMultiplier`, `usesOffset`; `startColor`, `endColor`, `segmentPoints:[{life:0..1, color}]` | **El nodo estrella**: gradiente de color/alpha multi-punto sobre la vida (hasta 4 segmentos). Vidas duplicadas se separan -1e-5. |
| `ParticleInitialColorNodeSubParser` | `color` (ColorTransform) | Tinte constante (permite recolorear un fx genérico gris). |
| `ParticleSegmentedScaleNodeSubParser` | `startScale`, `endScale`, `segmentPoints` | Igual que SegmentedColor pero para escala. |
| `ParticleUVNodeSubParser` | `axis` ("x"/"y"), `formula` (1=lineal, 2=seno), `cycle` (1D seg), `scale` (1D opcional) | **Scroll de UV en shader**: `uv.axis = uv.axis*scale + t/cycle` (formula 1) o `+ sin(t·2π/cycle)` (formula 2). Es lo que da vida a los beams. Confirmado en AGAL de `§_-K1W§\§_-914§.as`. |

### 1.4 Tipos de valor

| id | data | Nota |
|---|---|---|
| `OneDConstValueSubParser` | `{value}` | |
| `OneDRandomVauleSubParser` (sic, con typo) | `{min,max}` | Uniforme por partícula. |
| `OneDCurveValueSubParser` | `{anchorDatas:[{x,y,type}]}` | ¡Interpola sobre el **índice** de partícula (i/total), no sobre el tiempo! type: 0=LINEAR, 1=CONST (2=BEZIER definido pero no implementado). |
| `ThreeDConstValueSubParser` | `{x,y,z}` | |
| `ThreeDCompositeValueSubParser` | `{x,y,z}` (cada uno un 1D) + `isometric` | Cada componente const/random/curve independiente. |
| `ThreeDSphereValueSubParser` | `{innerRadius,outerRadius,centerX/Y/Z}` | Dirección aleatoria en cascarón esférico (velocidad radial de explosión / emisor esférico). |
| `ThreeDCylinderValueSubParser` | `{innerRadius,outerRadius,height,centerX/Y/Z,dX,dY,dZ}` | Anillo/cilindro con eje d. |
| `FourDCompositeWithOneDValueSubParser` | `{x,y,z,w}` (cada uno 1D) | Para scale/orbit (xyz + w). |
| `FourDCompositeWithThreeDValueSubParser` | `{x:(3D),w:(1D)}` | Para ejes de rotación (xyz juntos + w). |
| `CompositeColorValueSubParser` | `{redMultiplierValue, greenMultiplierValue, blueMultiplierValue, alphaMultiplierValue, redOffsetValue, ..., alphaOffsetValue}` (cada uno 1D) | ColorTransform con componentes random/curve. |
| `ConstColorValueSubParser` | claves cortas `{mr,mg,mb,ma,or,og,ob,oa}` | Usado dentro de `segmentPoints` (aunque el JSON diga id Composite, el SegmentedColor lo parsea como Const con claves cortas). Multiplicadores 0..1, offsets 0..255. |
| `Matrix3DCompositeValueSubParser` | `{transforms:[...]}` | vertexTransform por copia de geometría. |
| `Matrix2DUVCompositeValueSubParser` | `{numColumns,numRows,selectedValue}` | Selección de celda de atlas POR COPIA de geometría (p.ej. `uvGrid[4x2 sel=rnd[0..4]]` en explosion3 — cada quad usa un frame distinto del atlas). |
| `LuaExtractSubParser`, `LuaGeneratorSubParser` | — | Definidos como id pero SIN clase registrada (herencia del editor, no soportados). |

### 1.5 Geometría (assembler)

Único assembler registrado: `SingleGeometrySubParser`:
- `num`: **número de partículas** (copias de la shape ensambladas en una sola geometría; 1 vertex buffer, 1 draw call).
- `shape`: uno de:
  - `PlaneShapeSubParser` `{width,height}` — quad (el caso típico de sprite).
  - `CubeShapeSubParser` `{width,height,depth}`
  - `SphereShapeSubParser` `{radius,segmentsW,segmentsH}`
  - `CylinderShapeSubParser` `{topRadius,bottomRadius,height,segmentsW,topClosed,bottomClosed}`
  - `ExternalShapeSubParser` `{url:"*.obj"}` — malla arbitraria (beams, anillos, shockwaves, spikes).
- `vertexTransform` (Matrix3D por copia, opcional) y `uvTransformValue` (celda de atlas por copia, opcional).

### 1.6 Material

- `TextureMaterialSubParser`: `{url, repeat, smooth, alphaBlending, alphaThreshold, blendMode, bothSide}`.
  En la práctica: **`blendMode:"add"` + `alphaBlending:true` en ~95% de los efectos**; `normal` solo cuando el fx debe
  oscurecer (agujero negro, humo del atlas de explosión).
- `ColorMaterialSubParser`: `{color, alpha}` (raro).
- Sin iluminación por defecto. Excepción: capas cuyo `name` empieza con `light_` o listadas en
  `customParameters.lightPickerTargets` (o `["*"]`) reciben el lightPicker de la escena (`§_-u1u§.§_-i45§`).

### 1.7 particleEvents y customParameters (semántica REAL en runtime)

`particleEvents` = marcadores con nombre en la línea de tiempo del efecto. El animator compuesto (`§_-71i§.set time`)
dispara un evento cuando `time` cruza `occurTime` (en segundos). En `§_-r1f§.§_-K3S§`:
- **`end`** → autodestrucción del efecto (todos los fx de un disparo lo tienen; los loops no).
- **`light*`** (ej. `light1@0s` en warp_ship) → crea una LUZ DINÁMICA puntual vía LightsManager, con parámetros
  leídos de `customParameters[nombre] = {color:"#7FC4F5", radius:200, duration:2, fading:0.3}`.
- cualquier otro nombre (`explosion@3.073s` en warp_ship) → callback registrado por el código del wrapper.

`customParameters` conocidos (leídos en `§_-r1f§`/`§_-u1u§`):
- `scalable:1` (y typo `scaleable`) — el efecto se escala al tamaño de la nave objetivo (con ease Quad 0.5 s).
- `maxScale:N` — tope de esa escala.
- `dispose:"scaleDown"` — al morir, en vez de desaparecer, escala a 0 en 0.5 s.
- `billboard:1` — TODO el contenedor hace lookAt a la cámara cada frame (marcadores UI 3D).
- `lightPickerTargets:["*"]|[names]` — capas que reciben iluminación de escena.
- `lightN:{color,radius,duration,fading}` — datos para los eventos `lightN`.

---

## 2. Efectos representativos analizados (digest completo en `fx_extract\`)

### Beam (`beam_chain_bolt`) — receta mínima de un rayo
1 capa, `num=1`, malla externa `fx_chain_bolt.obj`, textura patrón `lightning_chain_beam.png` con `repeat:true`,
blend add. Nodos: Time(loop 1 s) + **UVNode(axis x, formula 1, cycle 0.5)** → el patrón fluye a lo largo del beam.
Nada más: el estiramiento entre naves es 100% runtime (ver §3). `beam_heal` es igual con `fx_beam_generic.obj` +
`health_charge_beam_pattern.png`.

### Explosión (`explosion3`) — 4 capas, evento `end@1.25s`
- `glibber_sparks`: 400 quads 10×10 `spark.png` add; vida rnd 0.1–0.8 s, startTime rnd 0–0.2; **Velocity esfera
  100–200** + Position esfera 0–100 + gravedad (Accel y rnd −100..−500) + Billboard + RotateToHeading
  (chispas estiradas en su dirección) + Scale 1→0 + SegmentedColor blanco→0.6.
- `glibber_cloud_expanding`: 60 quads 120×30 `burning_debris_trail.png`, velocity esfera 250–350.
- `star_explosion`: 1 quad 250×250 `fire_orb.png`, vida 0.25 s, Scale 0→1.4 (flash central).
- `shader_fireball_01`: 20 quads 65×65 con **atlas 4×2 `explosion_atlas.png` (celda aleatoria por copia)**, blend
  normal, vida 1.5 s, Scale 0.3→1, RotationalVelocity z rnd −1..1 (w=8). En calidad max su material se sustituye por
  `ExplosionMaterial` (shader burn/dissolve, ver §5).
Patrón general de explosión: chispas rápidas + nubes medianas + flash de 1 frame + fireballs lentos, todo con
gravedad ligera hacia −y y colores por SegmentedColor.

### Motor (`thruster`) — 40 quads 8×8 `simple_gradient.png` add
Vida 1 s loop, startTime rnd 0–1 (emisión continua), Velocity z 5–6 + Accel z 15–20 (escape hacia atrás),
Scale 1→0.2, SegmentedColor cian→blanco→cian. Se ancla detrás de la nave. (El trail largo del motor NO es esto —
es un ribbon CPU aparte, §4.)

### Trail de cohete (`rocket_trail_1`) — humo que queda atrás
50 quads 15×15, atlas 3×3 de humo (celda aleatoria por copia), vida rnd 0.01–0.7 loop, Scale 1→0.1, Billboard y
**ParticleFollowNode(usesPosition)**: el cohete se mueve, las partículas nacen donde está y SE QUEDAN en el mundo.

### Ambiente (`black_light_black_hole`) — agujero negro
3 mallas externas (spikes ×5, glow ×5, main ×1) con gradiente `space_time_rift_gradient.png` add, vidas
aleatorias desincronizadas + SegmentedColor pulsante, más 3 quads billboard 60×60 girando (rot 25,25 en property,
scale 2.5, timeOffset −2). Los "gates" (`gate_jump`) añaden: 300 partículas sobre malla `fx_gate_effect.obj` con
RotationalVelocity y (vórtice), 100 streaks cilíndricos con RotateToHeading, y capa `black_hole.png` blend normal.

### UI 3D (`abstract_ui_marked_red`) — marcador de objetivo
1 malla `fx_target_skoll.obj` (crosshair) add, Scale pulsante 1.5↔1.6 (cycle 1 s) + RotationalVelocity y (w=2:
gira 2 ciclos/seg). customParameters `{scalable:1, dispose:"scaleDown"}`. Los `glow_*` (`glow_slow_pulse_red`) son
1 quad `simple_gradient.png` + InitialColor para el tinte + Scale cíclica — mismo asset genérico recoloreado.

### Otros datos útiles
- `star_dust`: 1500 quads de 3×3 en volumen 2000×400×2000, deriva lenta, colores variados; se INSTANCIA EN MOSAICO
  cada 2000 u sobre todo el mapa (`§_-i7§`), elegido por atributo XML `@starfield` del mapa.
- `impact_shield`: media esfera `half_sphere.obj` + SegmentedColor fade 0.6 s, `end@0.6s` — el "escudo golpeado".
- `shockwave1`: `fx_shock_wave_medium.obj` (anillo plano) Scale 1→45 en 0.3 s. El anillo NO es billboard: vive en el
  plano del mapa.
- `muzzle_flash_red`: 3 capas de 0.15 s (flash + spikes + 10 chispas), bounds 10.
- `warp_ship`: coreografía de 11 s en un solo .awp usando startTime como partitura (build_up 0–9 s, clímax en t=10:
  shockwave + burst + luz `light1` + evento `explosion` + `end@4.291s`... con playSpeed ajustado por instancia).

---

## 3. Runtime: carga, pooling, attach, beams, lifecycle

### Carga y caché
`AssetsManager3D` (`§_-o2x§`, singleton) expone `§_-m3a§` (loader `§_-u1u§`). `§_-c2T§(url, lifetime, useCache)`
devuelve una **promesa** del template `§_-JU§`; cachea por URL con `lifetime` (0/1/2): `reset(nivel)` al cambiar de
mapa purga los de lifetime ≤ nivel. El parser corre async (frame-budget) y el resultado es UN template por URL.

### Instanciación = clone del template
Cada uso hace `template.clone()` (clona las meshes compartiendo geometría/material/animación — barato). El wrapper
`§_-r1f§`:
- avanza la animación manualmente cada frame: `animator.time += dt*1000*playbackSpeed` (no hay autoplay);
- aplica `scalable`/`maxScale`/`billboard` (lookAt cámara con UP=(0,−1,0) y +90° en X);
- escucha particleEvents (end→dispose, light*→luz dinámica);
- `disposeView()`: si `dispose=="scaleDown"` escala a 0 en 0.5 s antes de soltar.

**Pooling opcional** (`§_-321§\§_-eI§`): diccionario estático de vectores por clave `"Particle3D_"+path`;
`retrieve` reutiliza si hay ≥2 en el pool, `give` devuelve al morir. Se usa para efectos de alta rotación
(impactos, muzzle flashes). El pool entero se vacía al resetear el mapa.

### Attach a entidades
`§_-42y§` guarda una REFERENCIA VIVA a la posición del modelo (`§_-a2F§` mutable del ship) y cada frame hace
`view.moveTo(pos.x, pos.z, -pos.y)` (mapa 2D x,y → mundo x,z,−y) + toma la escala de la nave del trait de tamaño.
Se registra en la lista `_updatables` del mapView (update centralizado por frame).

### Beams entre dos naves (`§_-01b§\§_-w4§`) — LA respuesta al estiramiento
El beam es una malla de longitud fija (~110 u en Z) con UV scroll. Cada frame:
```
dist   = |target - source|
scaleZ = dist / 110 * rampa        // rampa 0→1 en rampTime seg (aparición progresiva)
scaleX = dist / 250 * rampa        // solo si "stretchX" (chain bolt: el rayo gordo escala también en ancho)
rotationY = atan2(dy, dx) + 90°    // orientación en el plano del mapa (juego top-down: no hay pitch)
posición = source (via §_-42y§)
```
Con tope opcional de longitud (beam_heal: 820). **Chain lightning** (`§_-U1q§`): un beam por salto
(attacker→v1→v2...) creado con 0.2 s de retardo entre saltos + un fx `chain_bolt_corners` en cada vértice;
autodispose a los 3 s.

### Lifecycle resumido
`§_-c2T§` (promesa) → clone/pool → addChild a la capa de efectos del mapa + registro en updatables →
updateObj(dt) cada frame → evento "end" o disposeView() → scaleDown opcional → pool o dispose real
(animator.stop + mesh.dispose).

### Límites por calidad (`Settings3D.effects`: low/medium/high/max, BindableSettings)
No hay un "max particles" global: el recorte es **por sitio de invocación**:
- **medium**: starfield ambiental, efecto de escudo, fx de estado en naves.
- **high**: LightsManager (todas las luces dinámicas de eventos `light*`), trails de motor, fx secundarios de naves.
- **max** (+`qualityExplosion`): sustitución de material por `ExplosionMaterial` en capas `shader_fireball*`.
En low prácticamente solo quedan los fx de gameplay imprescindibles (beams, explosión básica, impactos).

---

## 4. Trail de motor: ribbon CPU (no es .awp)

`§_-G1y§\§_-Pd§.as`: una mesh con `EngineTrailMaterial` compartido. Por cada motor de la nave (puntos del
descriptor XML) guarda un **ring buffer de 12 muestras** de posición mundial, muestreadas cada **30 ms**
(8 floats por muestra: pos xyz, alpha, −rotación de la nave, padding). El alpha decae linealmente
`1 − i/12` hacia la cola; el vertex shader (hasta `MAX_TRAILS_PER_DRAW_CALL` trails por draw, 2 registros por
muestra) extruye la cinta. Color por nave: atributo `engineTrailColor` del XML del ship (default 0x5AC258).
Un salto >100 u (teleport) rellena todo el buffer con la posición nueva (evita el "latigazo").

## 5. Cómo consiguen el look

1. **Additive en todo**: `blendMode:"add"` + `alphaBlending:true` es el default de facto; los overlaps SUMAN luz.
   No hay bloom de postproceso: el "glow" son texturas con gradiente radial suave (`simple_gradient.png`,
   `fire_orb.png`) sumadas.
2. **Texturas patrón + UV scroll**: beams = malla larga + textura repeat + ParticleUVNode (cycle 0.3–0.5 s).
   Formula 2 (seno) para vaivén.
3. **SegmentedColor como envolvente**: fade-in/pico/fade-out se hace SIEMPRE con el gradiente multi-punto de color
   (alpha y RGB a la vez), nunca con opacidad del material.
4. **Desincronización estadística**: startTime/duration aleatorios por partícula + timeOffset por capa → loops que
   nunca se ven repetidos.
5. **Anillos y shockwaves = mallas planas .obj escaladas**, ancladas al plano del mapa (rot 90,0,0), no billboards.
6. **Recoloreo por InitialColor** sobre assets grises genéricos (familia glow_*, star_dust_colors_*).
7. **Luces dinámicas puntuales** sincronizadas por particleEvents (`light1@0s` + customParameters).
8. **ExplosionMaterial** (solo max): atlas de 2 filas (color + máscara de fuego, offset v=0.5), mezcla
   `color + (mask − life)·fireIntensity(8)` con saturación → frente de combustión que avanza y alpha (1−t)² —
   un burn/dissolve clásico.

## 6. Efectos 2D / Starling

Starling está en el SWF pero solo lo usa el **cliente 2D legacy** (tilemap de fondo, `net\bigpoint\as3toolbox\
starling\mapfactory\`, view2D). **No existe ningún ParticleSystem de Starling**; el HUD del modo 3D es display list
Flash clásico. Conclusión: no hay nada 2D que replicar para los fx — todo el sistema de efectos del modo 3D es el
descrito arriba.

---

## 7. Mapa a Godot (GPUParticles3D / shaders)

| Técnica DarkOrbit | Equivalente Godot recomendado |
|---|---|
| Capa .awp (mesh de N quads + nodos GPU) | `GPUParticles3D` con `ParticleProcessMaterial` (o process shader custom); `amount=num`, `lifetime=duration`, `explosiveness` según startTime (0 = burst, spread aleatorio = `explosiveness 0` + `randomness 1`) |
| Efecto multi-capa | Un `Node3D` escena por fx con varios GPUParticles3D hijos; el .tscn ES el .awp |
| blend add + alphaBlending | `StandardMaterial3D`: `blend_mode=BLEND_MODE_ADD`, `shading_mode=UNSHADED`, `billboard` según capa |
| ParticleBillboardNode | `billboard_mode = BILLBOARD_PARTICLES` en el material |
| RotateToHeading | `ParticleProcessMaterial.align_y = true` (partícula alineada a velocidad) con quad orientado |
| SegmentedColor/SegmentedScale | `color_ramp` (GradientTexture1D) y `scale_curve` (CurveTexture) — mapeo 1:1 de los segmentPoints |
| Velocity esfera/cilindro | `emission_shape` SPHERE_SURFACE/RING + `direction`+`spread=180` + `initial_velocity_min/max` |
| Position esfera/caja | `emission_shape` (sphere/box) |
| Acceleration (gravedad) | `gravity` / `linear_accel` |
| RotationalVelocity / Orbit | `angular_velocity` / `orbit_velocity` (para eje arbitrario: process shader custom) |
| Oscillator | `turbulence` ligera o process shader con `sin(TIME)` |
| SpriteSheet / uvGrid por copia | `particles_anim_h_frames/v_frames` en StandardMaterial3D (+ `anim_offset` random para celda aleatoria) |
| UVNode (beam scroll) | **MeshInstance3D (no partículas)** con shader espacial: `uv.x += TIME/cycle` (o `sin`), `blend_add`, `cull_disabled`; textura repeat |
| Beam estirado entre naves | Nodo Beam: `look_at(target)` en el plano + `scale.z = dist/LARGO_MALLA`; rampa 0→1 con Tween. Igual que el original: malla de largo fijo + escala |
| ParticleFollowNode (trail humo) | GPUParticles3D con `local_coords = false` como hijo del que se mueve (las partículas quedan en mundo) |
| Trail de motor (ribbon 12×30 ms) | Ribbon propio: `ImmediateMesh`/MultiMesh con ring buffer CPU idéntico (12 muestras/30 ms/alpha 1−i/N), o `Trail` de GPUParticles3D |
| particleEvents | `AnimationPlayer` en la escena del fx (llama métodos/enciende `OmniLight3D` en t exactos) — más natural que timers |
| eventos light* | `OmniLight3D` con Tween de energía (duration/fading del customParameter) |
| customParameters scalable/maxScale | export vars en el script raíz del fx |
| dispose scaleDown | Tween de escala a 0 en 0.5 s antes de `queue_free()` |
| Pool + template/clone | Cachear el `PackedScene` (es el template) y opcionalmente pool de instancias para impactos/muzzle |
| Calidades | Igual: gates por sitio (starfield ≥medium, luces ≥high, shader burn solo max) — no hace falta budget global |
| ExplosionMaterial | Shader burn: atlas color+mask, `mix(color, fire, clamp((mask - life)*k)*intensity)`, alpha `(1-t)^2` |
| OneDCurve por índice | En process shader: usar `float(INDEX)/float(total)` como coordenada de una CurveTexture |

Reglas de oro para que se vea igual: 1 unidad DO ≈ escala actual del mundo MexOrbit (el beam mide 110 u; naves ~100 u);
todo additive y unshaded; el fade SIEMPRE por color ramp, no por transparencia de material; anillos en el plano del
suelo sin billboard; aleatorizar startTime y lifetime SIEMPRE (es lo que mata el aspecto "de cajón" de los loops).
