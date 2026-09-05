# ASTRION LOW-POLY MODELING STANDARD

**Norma de producción de geometría para los assets 3D de Astrion.**
Versión 1.0 — 4 de septiembre de 2026.

Este documento **no** resume DarkOrbit. Convierte lo aprendido midiendo 448
mallas de su cliente ([DARKORBIT_ASSET_ANALYSIS.md](DARKORBIT_ASSET_ANALYSIS.md))
en reglas para producir assets **originales** de Astrion. De DarkOrbit se toma
la ingeniería; nada de su dirección artística, sus siluetas ni sus diseños.

> **Ley rectora**
> **La geometría define la identidad. La textura vende la complejidad. Godot
> pone la luz.** Un detalle que no cambia la silueta ni produce volumen visible
> desde la cámara real no es geometría: es textura.

**A quién obliga.** A todo asset que entre en `mex-orbit-art/source/3d-models/`
y de ahí al cliente: naves, NPCs, drones, props, estaciones, portales, cajas.
El validador de `tools/asset-audit/` comprueba las partes automatizables.

**Con qué se contrasta.** Los números de referencia de DarkOrbit aparecen entre
paréntesis como `(DO: …)` para que se vea de dónde sale cada decisión y dónde
nos separamos a propósito.

---

## 1. La cámara manda

Todo lo demás se deriva de aquí. Astrion usa la cámara oblicua del cliente 3D
(ver `darkorbit-3d/camara-proyeccion.md`): perspectiva **FOV vertical 30°**,
elevación **45°** (bajando a ~25° con el zoom máximo), azimut **25°**,
distancia base **1740 unidades / zoom**, zoom 1–3.

De ahí sale la única constante que hay que tener en la cabeza al modelar:

```
px por unidad de mundo (zoom 1, 1080p)  = 1,158
px por unidad de mundo (zoom 1, 1440p)  = 1,544
px por unidad de mundo (zoom 3, 1440p)  = 4,63
```

**Regla del pulgar**: multiplica el tamaño de tu modelo en unidades por **1,5**
y tienes su altura en píxeles a 1440p con el zoom normal. Una nave de 200
unidades se ve a 300 px. Un dron de 15 unidades, a 23 px.

> **Antes de modelar, escribe en el brief cuántos píxeles va a ocupar el asset.**
> Todo el resto de este documento depende de ese número.

### 1.1 Convención de unidades

DarkOrbit no tuvo ninguna y lo pagó: sus mallas van de 1,1 a 50.425 unidades de
diagonal y lo compensan con un `scale` por instancia (DO: 22 de 309 assets
fuera del rango 10–2.000). Imposible validar, imposible razonar en unidades de
mundo.

Astrion sí la tiene, y ya está en el cliente: **`entity_node.gd` escala la
malla para que su extensión mayor valga el `screen_size` del JSON de la
especie** (`data/npcs/*.json`, `data/ships/*.json`; por defecto 141). El GLB
fuente viene normalizado; el tamaño real lo pone ese número.

| Regla | Valor |
|---|---|
| **`screen_size` va en el brief, antes de modelar** | es de donde salen los píxeles y, con ellos, todo lo demás de este documento |
| `screen_size` de un asset jugable | **20–400 unidades** (hoy: 124–248) |
| Eje de avance | **−Z** (morro hacia −Z), **+Y** arriba |
| Origen | en el **centro de masa visual**, apoyado en el plano de juego (Y = 0) |
| Plano de espejo | **X = 0** exacto |

> **Regla de proceso.** Hoy el `screen_size` se decide al final, cuando el
> modelo ya existe. Tiene que decidirse al principio: es el número del que
> dependen el umbral de 3 px, los segmentos de cada cilindro y el presupuesto
> entero.

El validador toma `--screen-size=<n>` y reproduce exactamente el escalado del
cliente; rechaza el asset fuera de 20–400 unidades o con el origen a más del
5 % de la diagonal del centro.

---

## 2. Geometry

### 2.1 Silueta

1. **La silueta es el asset.** Lo primero que se bloquea, lo último que se
   toca. Si dos assets de Astrion no se distinguen por su recorte negro a 100
   px, uno de los dos está mal diseñado — y ningún presupuesto de polígonos lo
   arregla.
2. **Presupuesto mínimo de curvatura de silueta: un cambio de dirección cada
   ~8 px de contorno.** A 300 px de nave eso son ~30–40 vértices repartidos por
   todo el perímetro visible. Por debajo se ve poligonal; por encima no se nota.
3. **Los rasgos que hacen reconocible la nave (alas, garras, cañones, aletas)
   se modelan a mano con pocos polígonos.** Nunca se dejan al decimador: la
   prueba de simplificación de DarkOrbit muestra que **lo primero que un
   decimador destruye son exactamente esos elementos finos**, mucho antes de
   que la métrica de silueta se entere (DO: Goliath al 25 % pierde los cañones
   con solo 6 % de píxeles de silueta cambiados).
4. **Aplana.** Con cámara a 45° el volumen vertical no se lee. Objetivo de
   `alto / mayor(ancho, largo)`:

   | Tipo | aplanamiento objetivo | (DO medido) |
   |---|---|---|
   | Nave de jugador, nave NPC | **0,35 – 0,55** | 0,44 |
   | PET, dron | 0,45 – 0,65 | 0,48 / 0,58 |
   | NPC grande / criatura | 0,50 – 0,75 | 0,53 |
   | Estructura, estación | 0,7 – 1,1 | 0,84 |
   | Portal, anillo | 0,9 – 1,1 | 1,00 |

5. **No borres la cara inferior.** Cuesta poco, evita agujeros al girar la
   cámara con el zoom, y el ahorro real está en aplanar, no en recortar
   (DO llegó al mismo sitio: 60 % de triángulos ocultos y aun así el fondo
   modelado).

### 2.2 Volúmenes y piezas independientes

6. **Construye por piezas, no por malla continua.** Un asset de Astrion es un
   volumen principal más un puñado de piezas. Es más rápido de hacer, más fácil
   de variar y más barato que retopologizar uniones.
7. **Presupuesto de piezas** — el coste de polígonos correlaciona con el número
   de piezas más que con nada más (DO: r = +0,70 contra r = +0,20 del tamaño
   físico). Por eso se acota:

   | Tipo | piezas visibles | (DO mediana) |
   |---|---:|---:|
   | Prop | 1 – 4 | — |
   | Dron | 1 – 6 | 10 |
   | NPC normal | 4 – 12 | 36 |
   | NPC complejo / élite | 8 – 20 | 36 |
   | Boss / uber | 12 – 30 | 50 |
   | Nave de jugador | 8 – 20 | 46 |
   | Estación / estructura | 15 – 40 | 130 |

   **Somos deliberadamente más restrictivos que DarkOrbit.** Sus 36 piezas de
   mediana (y hasta 483) son el síntoma de un kit-bash sin control; con mallas
   4–8× más densas ese número tiene que bajar, no subir.
8. **El volumen principal debe llevar ≥40 % de los triángulos.** Si la pieza
   mayor tiene menos, el asset es confeti (DO: 17,8 % de mediana — eso **no**
   se copia).
9. **Las piezas se intersecan, no se fusionan.** Nada de booleanas, nada de
   soldar a mano uniones que la cámara nunca va a inspeccionar. Es la práctica
   de DarkOrbit y es correcta (DO: 284/309 assets con piezas solapadas).
10. **Toda pieza que se interseca debe penetrar al menos un 15 % de su grosor**
    en el cuerpo receptor. Piezas que solo se besan crean costuras de
    z-fighting y grietas de luz.

### 2.3 Grosor

11. **Nada de planos sin grosor en geometría de asset.** Ni alas, ni placas, ni
    aletas. Una carta plana se ve como papel en cuanto la cámara se mueve, y
    Astrion tiene rotación de nave y zoom. **Grosor mínimo: 0,8 % de la
    dimensión mayor de la pieza, y nunca menos de 0,3 unidades.**
    (DO llegó a la misma conclusión: solo el **0,7 %** de sus 10.872 piezas son
    cartas de grosor cero.)
12. **La única excepción son los FX**, que sí son cartas (§7).
13. **Los shells abiertos se permiten donde la abertura es físicamente
    invisible** (el interior de un tubo que va tapado, el reverso de una placa
    pegada al casco). No se permiten en el volumen principal.

### 2.4 Paneles, juntas y microdetalle

14. **Un panel nunca es geometría.** Ni una junta, ni un remache, ni una
    rejilla, ni una escotilla, ni una ranura, ni un número, ni un desgaste, ni
    un impacto. Todo eso vive en Base Color + Normal (§5).
15. **Umbral de existencia**: si un detalle mide **menos de 3 px** en pantalla
    al zoom normal, no puede ser geometría. A 1,5 px/unidad eso son **2
    unidades de mundo**. Nada más pequeño que eso se modela.
16. **Umbral de bisel**: los biseles de arista no se modelan hasta que la
    arista mide más de ~60 px en pantalla (40 unidades). Por debajo, el normal
    map y el specular hacen el trabajo.
17. **Un saliente merece geometría si cumple las dos**: cambia el contorno del
    asset **y** su sombra propia se lee a tamaño de juego. Si solo cumple una,
    es textura.

### 2.5 Simetría

18. **Modela la mitad y espeja en X = 0.** Las piezas espejadas deben quedar
    idénticas al milímetro; el validador mide la fracción de vértices con
    contraparte espejada y exige **≥0,95** en naves y drones.
19. **La asimetría se pinta, no se modela.** Marcas de escuadrón, numeración,
    daño de un lado, manchas: van en la textura sobre UVs espejadas
    (DO: factor de solape UV 1,30 de mediana — los dos lados comparten texel y
    la diferencia es pintada).
20. **Excepción**: criaturas y NPCs orgánicos pueden ser asimétricos de origen.
    Ahí la exigencia baja a ≥0,60 y se documenta en el brief.

### 2.6 Anclajes funcionales

21. **Motores, bocas de arma y luces son anclajes, no geometría.** Se exportan
    como `Empty` de Blender con nombre normativo, no como cajas
    (DO usaba cajas de 8 triángulos: 388 `laserpoint_*`, 114 `engine_*`, 71
    `light_position`; nosotros gastamos **0** triángulos en lo mismo).

    | Nombre | Qué cuelga |
    |---|---|
    | `muzzle_<n>` | origen del disparo |
    | `engine_<n>` | estela y llama del motor |
    | `light_<n>` | luz de posición / parpadeo |
    | `hit_<n>` | punto de impacto para el FX de daño |

22. **La tobera del motor sí se modela como hueco** (un cilindro corto hundido
    con la boca oscura), pero **el fuego y la estela son partículas**. La
    geometría solo tiene que dar el sitio donde nace el efecto.
23. El pipeline de anclajes ya existente (`marcar-anclajes.py`,
    `find-anchors.py`) es la fuente de verdad de nombres y ejes.

### 2.7 Topología

24. **Quads donde se pueda, triángulos donde convenga. Ningún n-gon.**
    Objetivo: **≥60 % de los triángulos formando pares coplanares**
    (DO: 47,6 %). Los n-gons se rechazan: rompen el import de Godot y el
    horneado de normales.
25. **No hay obligación de loops limpios.** Esto son game assets de 300 px, no
    modelos de subdivisión. Un loop que termina en un triángulo está bien si la
    superficie es plana.
26. **Aristas duras a partir de 35°**, marcadas explícitamente; el resto suave.
    Se exportan **normales explícitas siempre** (DO no lo hizo en el 59 % de sus
    assets y dejó el suavizado al motor: no lo repetimos).
27. **Prohibido**: caras interiores, caras duplicadas, triángulos degenerados,
    aristas no-manifold, vértices sueltos. Tolerancia cero — el validador los
    cuenta y son error, no aviso.
28. **Geometría enterrada permitida solo si una animación la revela**, y se
    documenta en el brief. El umbral automático es **≤5 % de triángulos nunca
    visibles desde ninguna dirección** (DO: 2,8 % de mediana, pero con casos de
    52 % y 88 % que son justamente los peores assets de su corpus).

---

## 3. Curved surfaces

La medición de 10.872 piezas de DarkOrbit da una escalera muy clara: mediana de
**6 segmentos**, p90 de 12, y solo los anillos protagonistas suben a 16–24.
Astrion trabaja a mayor resolución y con más zoom, así que **subimos un
peldaño, no dos**.

### 3.1 Cilindros, tubos, cañones

| Diámetro en pantalla (zoom 1, 1440p) | segmentos Astrion | (DO) |
|---|---:|---:|
| < 6 px (antena, cable, mástil) | **4** | 4 |
| 6 – 15 px (cañón, tubo fino) | **6** | 6 |
| 15 – 40 px (tobera, tubo grueso) | **8 – 10** | 8 |
| 40 – 90 px (cuerpo cilíndrico) | **12 – 16** | 12 |
| > 90 px (anillo protagonista, portal) | **20 – 24** | 16–20 |

29. **Un cilindro nunca lleva tapa a menos que la tapa se vea.** Cuando se ve,
    la tapa es un abanico desde el centro, no una rejilla.
30. **El cilindro se corta en el punto de intersección, no antes.** Un cañón
    que atraviesa un ala termina dentro del ala, no en su superficie.

### 3.2 Esferas y domos

31. **Nunca uses la esfera por defecto.** La UV-sphere de Blender (32×16 = 960
    tris) y la icosfera de subdivisión 3 (1.280 tris) son ambas caras de más
    para casi todo. La escalera de Astrion:

    | Uso | segmentos × anillos | tris | (DO) |
    |---|---|---:|---:|
    | Bolita, remache grande, cabeza de sensor | 8 × 5 | **64** | 48 |
    | Domo pequeño, núcleo | 12 × 7 | **144** | — |
    | Cuerpo esférico de NPC pequeño | 16 × 9 | **256** | 224 |
    | NPC esférico normal | 20 × 11 | **400** | 368 |
    | NPC esférico protagonista / boss | 24 × 13 | **576** | 546 |
    | Planeta, skybox, cuerpo de fondo | 32 × 17 | **1.024** | **768** |

32. **Un domo es media esfera con el mismo número de segmentos**, no una esfera
    recortada con un booleano.
33. **Para NPCs esféricos hay dos gramáticas válidas, y hay que elegir una:**
    - **Torneada**: cascarón + anillos concéntricos + núcleo, todo con orden
      rotacional exacto. Barata, mecánica, se lee como máquina.
    - **Deformada de una pieza**: una esfera esculpida y triangulada, cerrada,
      sin piezas sueltas. Se lee como criatura.
    Mezclar las dos produce el peor resultado de ambas.
34. **La esfera deformada es la única forma en la que se permite renunciar al
    kit-bash** — y a cambio se le exige malla cerrada, 0 aristas de borde y 0
    triángulos interiores.

### 3.3 Anillos

35. **Un anillo es un toro de sección cuadrada o hexagonal**, no un toro
    redondo. Sección de 4 lados hasta 60 px de grosor de aro; 6 por encima.
36. **Segmentos del anillo**: los de la tabla §3.1 según el diámetro en
    pantalla. Un portal de 400 px de diámetro usa **24**, no 64.
37. **Los dientes, bloques y greebles del anillo se reparten por instancia
    rotada**, no se modelan uno a uno. 16–24 copias de una caja de 12
    triángulos son 200–300 triángulos y leen como maquinaria compleja.

### 3.4 Motores redondos

38. **Un motor es**: un cilindro exterior de 8–10 segmentos, un anillo interior
    hundido de los mismos segmentos, y una cara de fondo oscura. **3 anillos de
    vértices, ~60–100 triángulos.** El brillo, las lamas y el calor son
    Emission + Normal.
39. **Nunca modeles álabes, rejillas ni turbinas dentro de la tobera.** A 20 px
    de boca no existen.

---

## 4. Triangle budgets

Astrion corre en Godot en 2026, no en Flash en 2010. Las mediciones propias del
proyecto (ver `mex-orbit-art/README.md`) dicen que **los triángulos no son el
cuello de botella en la iGPU de referencia**: 30 entidades de 150.000 tris dan
los mismos fps que 30 de 56.000. El coste real es **VRAM de texturas y tiempo
de carga**.

Por lo tanto estos presupuestos existen por **legibilidad, disciplina de
producción y coherencia visual**, no por fps. Un asset que se pasa no rompe el
juego: rompe la coherencia y esconde un error de diseño.

### 4.1 Rangos

| Tipo | **Astrion** | píxeles típicos (1440p, zoom 1) | DO medido |
|---|---:|---:|---:|
| Prop simple (mineral, caja, escombro) | **200 – 800** | 30 – 120 | 250 – 690 |
| Prop ambiental grande (asteroide, roca) | **800 – 2.500** | 200 – 900 | 692 |
| Dron | **250 – 700** | 30 – 60 | **250** |
| PET | **2.000 – 4.000** | 80 – 150 | 1.262 |
| NPC normal | **2.000 – 4.000** | 90 – 160 | **1.626** |
| NPC complejo | **4.000 – 7.000** | 130 – 220 | 1.982 |
| Élite | **6.000 – 10.000** | 150 – 260 | 1.982 |
| Boss | **9.000 – 16.000** | 250 – 450 | 2.549 † |
| Uber / hero NPC | **14.000 – 25.000** | 400 – 700 | 3.972 † |
| **Nave de jugador** | **10.000 – 20.000** | 250 – 400 | **1.831** |
| Estación / estructura | **8.000 – 20.000** | 400 – 1.800 | 1.980 |
| Portal | **4.000 – 9.000** | 350 – 550 | 1.866 |

† DarkOrbit no tiene categoría de boss ni de uber en su XML; esas dos filas
citan `sibelon` y `sibelon-emperor`, sus NPCs más caros. El resto de la columna
sí es la mediana medida del rol.

**Dónde subimos y dónde no.** Para naves, NPCs y estructuras los rangos son
**5–8× los de DarkOrbit**: es lo que justifica pasar de 1080p a 1440p, de un
zoom fijo a un zoom ×3 y de Stage3D a PBR en Godot. Para **drones y props
seguimos prácticamente en sus números** (250–700 contra 250), porque su tamaño
en pantalla no ha cambiado: un dron sigue midiendo 40 px y ahí no hay nada que
gastar. La filosofía no cambia; el techo sube solo donde el jugador mira.

### 4.2 El techo de densidad en pantalla

Los rangos de la tabla son el control principal — **igual que en DarkOrbit, el
presupuesto es por rol, no por cobertura de pantalla**. Eso se midió: sobre 30
assets con escala válida, la pendiente de `log(densidad)` contra
`log(área en pantalla)` es **−0,85** (r = −0,86). Una pendiente de −1 sería
"el mismo número de triángulos pase lo que pase": DarkOrbit está casi ahí.
Y hace bien — en una cámara cenital lo que cubre media pantalla suele ser
fondo (estación, portal, asteroide), y darle presupuesto proporcional al área
sería regalárselo al elemento menos importante.

Lo que sí hace falta es un **techo que cace el caso patológico**: el asset que
es diminuto en pantalla y pesa como un boss.

```
densidad = triángulos / (miles de px² que el asset cubre a zoom 1, 1440p)

se aplica solo a assets de más de 1.000 triángulos
   aviso   por encima de   800 tris / 1.000 px²
   error   por encima de  2.000 tris / 1.000 px²
```

**Por debajo de 1.000 triángulos el techo no aplica**, porque manda el suelo de
forma: por pocos píxeles que ocupe, una silueta reconocible necesita sus 250 o
400 triángulos. Un dron de 40 px sale con densidad alta y está bien.

Referencia medida en DarkOrbit a 1440p y zoom 1: naves y NPCs entre **70 y 390
tris/1.000 px²** (Goliath 82, streuner 150, nostromo 177, phoenix 313);
estructuras y fondo entre 1 y 30. Nuestro techo de 800 es deliberadamente
generoso respecto a eso: pagamos PBR, 1440p y zoom ×3.

`validate_asset.py` calcula la densidad rasterizando la máscara real del asset
desde la cámara de juego, no estimándola de la caja.

### 4.3 Y la regla que manda sobre todo

**Usa la menor cantidad de geometría que conserve la silueta y los volúmenes
que importan en gameplay.** No se añaden caras porque quede presupuesto. Un NPC
de 2.100 triángulos que lee perfecto es mejor que uno de 4.000 que lee igual.

### 4.4 Presupuesto de textura y VRAM

Aquí sí hay un coste medido y hay que respetarlo (12 MB por asset a 1024 con
todos los canales).

| Tipo | Base Color | Normal | ORM (packed) | Emission |
|---|---|---|---|---|
| Prop, dron | 512 | 512 | 512 | 256 o ninguna |
| NPC normal | 1024 | 1024 | 512 | 512 |
| NPC complejo, élite | 1024 | 1024 | 1024 | 512 |
| Boss, uber, nave de jugador | 2048 | 2048 | 1024 | 1024 |
| Estación, portal | 2048 | 1024 | 1024 | 1024 |

40. **Roughness, Metallic y AO van empaquetados en un solo ORM** (R = AO,
    G = Roughness, B = Metallic). Tres canales, una textura.
41. **Emission solo si el asset emite.** Si la máscara de emisión es negra en
    más del 95 %, va a 256 o se borra y se resuelve con el color base del
    material (DO: el glow del Goliath es negro casi entero con **un** emisor).
42. **Densidad de texel objetivo: 200–600 texels por triángulo.** Por debajo de
    100 estás pagando geometría que la textura no puede vestir; por encima de
    1.500 estás pagando textura que nadie ve (DO: 110 de mediana — nosotros
    subimos porque nuestras texturas son PBR y nuestros zooms mayores).

---

## 5. Texture vs Geometry

La tabla de decisión. **Ante la duda, baja una fila.**

| Elemento | Geometry | Normal | Base Color | Roughness | Metallic | Emission | Efecto Godot |
|---|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| Silueta exterior, contorno | ✅ | | | | | | |
| Alas, garras, aletas, quillas | ✅ | | ✅ | | | | |
| Cañones, tubos, mástiles | ✅ | | ✅ | | | | |
| Volumen del casco | ✅ | ✅ | ✅ | | | | |
| Carcasa de motor / tobera | ✅ | ✅ | ✅ | | | ✅ | |
| Placas grandes que sobresalen (> 8 px) | ✅ | ✅ | ✅ | | | | |
| Escotillas, compuertas grandes | | ✅ | ✅ | ✅ | | | |
| **Líneas de panel, juntas** | ❌ | ✅ | ✅ | ✅ | | | |
| **Remaches, tornillos, pernos** | ❌ | ✅ | ✅ | | | | |
| **Rejillas, lamas, ranuras de ventilación** | ❌ | ✅ | ✅ | ✅ | | | |
| **Cables, mangueras finas (< 3 px)** | ❌ | ✅ | ✅ | | | | |
| **Micro-greebles, cajitas, antenas pequeñas** | ❌ | ✅ | ✅ | | | | |
| Biseles de arista (< 60 px de arista) | ❌ | ✅ | | ✅ | | | |
| Curvatura suave de una placa plana | ❌ | ✅ | | | | | |
| Ventanas, cabina, cristal | ❌ | ✅ | ✅ | ✅ | | ✅ | |
| Luces de posición, testigos | ❌ | | ✅ | | | ✅ | Glow |
| Franjas y venas energéticas | ❌ | ✅ | ✅ | | | ✅ | Glow |
| Llama y estela del motor | ❌ | | | | | | Partículas + anclaje |
| Disparos, rayos, escudos | ❌ | | | | | | Carta alpha + shader |
| Daño, quemaduras, arañazos | ❌ | ✅ | ✅ | ✅ | | | |
| Suciedad, óxido, desgaste | ❌ | | ✅ | ✅ | ✅ | | |
| Metal contra pintura contra goma | ❌ | | ✅ | ✅ | ✅ | | |
| Oclusión de recovecos | ❌ | | | | | | AO en ORM |
| Numeración, insignias, calcas | ❌ | ✅ | ✅ | | | | |
| Diferencia visual izquierda/derecha | ❌ | | ✅ | | | | UV espejadas |
| Halo, bruma, resplandor exterior | ❌ | | | | | | Glow + partículas |
| Sombra en el suelo | ❌ | | | | | | Decal / blob |

### 5.1 Las tres preguntas antes de añadir un polígono

1. **¿Cambia el contorno del asset visto desde 45°?** Si no → textura.
2. **¿Mide más de 3 px en pantalla al zoom normal?** Si no → textura.
3. **¿Su sombra propia se distingue a tamaño de juego?** Si no → normal map.

Tres noes seguidos y el detalle no existe. Un sí y solo uno: normal map con
Roughness de apoyo.

---

## 6. Top-down readability

El asset va a vivir entre 20 y 400 píxeles. Todo lo que sigue es consecuencia
de eso.

43. **Diseña el recorte antes que el modelo.** Un asset de Astrion tiene que
    ser identificable como silueta negra maciza a **80 px**. Prueba obligatoria
    antes de aprobar cualquier modelo.
44. **Tres masas, no treinta.** Un lector a 150 px distingue tres o cuatro
    volúmenes. Reparte así: **una masa dominante (50–65 % del área
    proyectada), una secundaria (20–30 %), y acentos**. Si tienes seis masas
    del mismo tamaño, el asset lee como una mancha.
45. **Contraste de valor, no de detalle.** Lo que separa un asset de otro a 100
    px es el reparto de claros y oscuros, no el número de paneles. El Base
    Color debe tener **al menos una zona oscura grande y una clara grande**.
46. **Ningún detalle de textura por debajo de 4 px de pantalla.** A escalas de
    juego, un panel de 2 px se convierte en ruido gris al mipmapear. Es peor
    que no ponerlo.
47. **El color identifica; la geometría categoriza.** Todas las naves de una
    familia comparten lenguaje de forma; la variante se distingue por color y
    emisión. Es lo que permite reutilizar malla entre tiers.
48. **La emisión es el ancla de lectura.** Dos o tres acentos emisivos, en
    posiciones consistentes por familia, con **menos del 5 % del área**. La
    emisión es lo único que sobrevive intacto al escalado a 30 px.
49. **Orienta el detalle hacia arriba.** La cámara ve la cara superior y el
    canto. Lo que va en el vientre existe (§2.1.5) pero no recibe presupuesto
    de textura ni de forma.
50. **Prueba de zoom 3.** El zoom baja la elevación a ~25° y triplica el
    tamaño: el canto lateral del asset pasa a ser protagonista. Revisa siempre
    a 45°/zoom 1 **y** a 25°/zoom 3.
51. **Prueba de multitud.** Doce copias del asset a 100 px en pantalla, con
    fondo de mapa. Si no se distinguen entre sí ni del fondo, el problema no se
    arregla con polígonos.

---

## 7. FX: la excepción de las cartas

52. Los efectos **sí** son cartas planas con alpha, y cuanto más simples mejor.
    (DO: 7 de sus 126 FX son literalmente **2 triángulos**; 53 contienen
    cartas.)

| FX | Geometría Astrion |
|---|---|
| Rayo, láser, haz | **1 quad** (2 tris) estirado, textura animada |
| Estela de motor | tira de quads (8–16 tris) o partículas |
| Onda de choque, anillo de impacto | anillo plano de 16–24 segmentos |
| Escudo | esfera de 20×11 con shader Fresnel |
| Explosión | partículas + 2–3 cartas orientadas a cámara |
| Portal energético | 2–3 cartas concéntricas + anillo |
| Marcador de selección | 1 quad con decal |

53. **Ninguna carta de FX lleva más de 24 triángulos.** Si necesitas más, es un
    shader, no una malla.

---

## 8. AI modeling rules

Instrucciones directas para un agente que controle Blender. Redactadas para ser
seguidas literalmente, sin criterio artístico.

### Antes de modelar

```
R01. Lee el brief y extrae: tipo de asset, presupuesto de triángulos,
     tamaño objetivo en UNIDADES y su equivalente en PÍXELES (unidades × 1,5).
     Si el brief no trae el tamaño en píxeles, calcúlalo y escríbelo en el
     nombre del blend. No modeles sin ese número.

R02. Define PX_MIN = 3 px. Convierte a unidades: UMIN = 3 / 1,5 = 2,0
     unidades. Ningún rasgo geométrico del asset podrá ser menor que UMIN en
     ninguna de sus tres dimensiones.

R03. Bloquea primero la silueta en vista cenital con 6-12 vértices. No pases a
     volumen hasta que ese contorno sea identificable.
```

### Al construir

```
R04. Si un detalle no altera la silueta vista a 45° de elevación, NO lo
     modeles: anótalo en la lista de textura del asset. Prefiere siempre
     información de textura o normal antes que geometría adicional.

R05. Todo elemento con dimensión menor que UMIN se elimina y se anota como
     detalle de Base Color + Normal. Sin excepciones, sin "por si acaso".

R06. Modela SOLO la mitad X ≥ 0 y aplica un modificador Mirror sobre X = 0.
     Aplica el mirror antes de exportar. Excepción: criaturas orgánicas
     declaradas asimétricas en el brief.

R07. Aplana el asset al ratio del §2.1.4 según su tipo. Comprueba
     dimensión_Y / max(dimensión_X, dimensión_Z) antes de continuar.

R08. Construye por piezas separadas que se INTERSECAN. No uses booleanas, no
     sueldes piezas distintas, no hagas retopo de uniones. Cada pieza debe
     penetrar >= 15 % de su grosor en el cuerpo receptor.

R09. Respeta el presupuesto de piezas del §2.2.7. Si superas el máximo, funde
     piezas pequeñas vecinas en una sola forma mayor o conviértelas en textura.
     No las repartas en más objetos.

R10. La pieza mayor debe contener >= 40 % de los triángulos del asset. Si no,
     el volumen principal está poco desarrollado y las piezas sobran.

R11. Ningún elemento con grosor cero. Extruye toda placa a >= 0,8 % de su
     dimensión mayor, mínimo 0,3 unidades.

R12. Cilindros: elige los segmentos con la tabla del §3.1 según el DIÁMETRO EN
     PÍXELES, no según el diámetro en unidades. Nunca uses el valor por defecto
     de 32. Sin tapas salvo que la tapa se vea; si se ve, abanico desde el
     centro.

R13. Esferas y domos: usa exclusivamente los peldaños del §3.2.31. Nunca la
     UV-sphere ni la icosfera por defecto.

R14. Anillos: toro de sección cuadrada (4) o hexagonal (6), con los segmentos
     del §3.1. Los dientes y greebles del anillo son instancias rotadas de UNA
     pieza, no piezas modeladas por separado.

R15. Motores: cilindro exterior + anillo interior hundido + fondo oscuro. 3
     anillos de vértices, 60-100 triángulos. Nada dentro de la tobera.

R16. Los puntos funcionales son Empties, NO geometría:
     muzzle_<n>, engine_<n>, light_<n>, hit_<n>.
     Orientados con -Z hacia fuera.

R17. Marca como hard edge todo ángulo diedro > 35°; el resto suave. Exporta
     normales personalizadas explícitas SIEMPRE.

R18. No dejes n-gons. Triangula solo al exportar, nunca durante el modelado.
```

### Antes de exportar

```
R19. Ejecuta la comprobación y corrige hasta que salga limpia:
       - 0 caras interiores
       - 0 caras duplicadas
       - 0 triángulos degenerados
       - 0 aristas no-manifold
       - 0 vértices sueltos
       - 0 n-gons
       - 0 piezas con grosor cero (excepto FX)

R20. Comprueba la escala con el `screen_size` del brief: escala el modelo para
     que su extension mayor valga esas unidades y comprueba que la diagonal
     resultante cae entre 20 y 400. Origen en el centro de masa visual,
     apoyado en Y = 0. Morro hacia -Z.

R21. Comprueba el presupuesto: triángulos dentro del rango del §4.1. Si el
     asset pasa de 1.000 triángulos, comprueba además la densidad en pantalla
     (§4.2): por encima de 800 tris/1.000 px² reduce; por encima de 2.000 es
     error y el asset está mal dimensionado para el sitio que ocupa.

R22. Comprueba la simetría: fracción de vértices espejados en X >= 0,95
     (>= 0,60 en orgánicos declarados).

R23. Comprueba lo oculto: <= 5 % de triángulos invisibles desde todas las
     direcciones. Si te pasas, tienes geometría enterrada: bórrala, salvo que
     el brief declare una animación que la revele.

R24. Empaqueta las UVs con >= 75 % de ocupación. Espeja las islas simétricas
     para que compartan texel. Da MÁS área de UV a las zonas que la cámara ve
     (cara superior, cantos) y menos al vientre: la densidad de texel NO tiene
     que ser uniforme.

R25. Renderiza la prueba de lectura antes de dar el asset por bueno:
       - silueta negra a 80 px
       - render a 45°/zoom 1 al tamaño real en píxeles
       - render a 25°/zoom 3
     Si el asset no se identifica en la primera, vuelve a R03.
```

### Prohibiciones absolutas

```
R26. NUNCA subdividas una superficie plana "para tener más resolución".
R27. NUNCA uses un modificador Subdivision Surface en un asset de juego.
R28. NUNCA generes greebles procedurales sobre el casco.
R29. NUNCA modeles líneas de panel, remaches, rejillas ni ranuras.
R30. NUNCA dejes el resultado de un decimador como asset final sin repasar a
     mano los elementos finos (cañones, aletas, antenas): son lo primero que el
     decimador destruye.
R31. NUNCA uses una malla de importación generativa (Meshy/Tripo/similar) sin
     pasar por la cadena de normalización del proyecto. Es fuente de referencia
     de forma, no asset.
R32. NUNCA añadas triángulos porque "sobra presupuesto".
```

---

## 9. Anti-patterns

Lo que **no** hacemos, con especial atención a lo que las herramientas
generativas producen por defecto.

### 9.1 De las herramientas generativas

| Anti-patrón | Por qué es malo | Qué hacer |
|---|---|---|
| **Malla de 150.000 triángulos "y luego decimamos"** | El decimador destruye antes los rasgos identificables que la superficie plana. El resultado es un asset borroso con el mismo presupuesto. | Diseñar al presupuesto final desde el bloqueo. |
| **Micro-greebles por todo el casco** | A 150 px son ruido gris; al mipmapear desaparecen y solo queda el coste. | Tres masas grandes + textura. |
| **Superficies planas densamente trianguladas** | Cientos de triángulos coplanares que no aportan silueta ni sombreado. | Una cara. El validador cuenta grupos coplanares. |
| **Topología uniforme tipo remesh** | Reparte el presupuesto por igual entre la punta del ala (importa) y el vientre (no importa). | Densidad donde cambia la forma. |
| **Ruido de superficie confundido con detalle** | Un remesh adaptativo mete micro-ondulación que se lee como suciedad de malla. | Superficie limpia + normal map. |
| **Primitiva imperfecta donde debería haber una perfecta** | Un "cilindro" generado con 47 lados irregulares no se lee como cilindro y no se puede animar ni instanciar. | Reemplazar por la primitiva exacta de la escalera del §3. |
| **Todo fundido en una sola malla** | Impide instanciar, variar, animar por piezas y aplicar materiales distintos. | Piezas separadas que se intersecan. |
| **Detalles por debajo del umbral de píxel** | Coste sin retorno; en el peor caso, aliasing. | Aplicar R02/R05 sin piedad. |

### 9.2 De construcción

| Anti-patrón | Por qué es malo |
|---|---|
| **Shells abiertos en el volumen principal** | Se ve el interior al girar la cámara con el zoom; el sombreado se rompe en el borde. |
| **Placas de grosor cero** | Se ven como papel en cuanto la cámara se mueve; el normal map no las salva. |
| **Agujeros y grietas entre piezas** | Piezas que se besan sin penetrar dejan una línea de fondo visible. Penetrar ≥15 %. |
| **Caras interiores y geometría enterrada** | Coste puro. Solo se permite si una animación la revela y está en el brief. |
| **Placas flotantes complejas** | Un panel despegado del casco con 200 triángulos que se lee como una mancha. Si flota, que sea simple y grande. |
| **Booleanas para unir piezas** | Genera n-gons, triángulos degenerados y topología imposible de retocar. |
| **Escala de autoría incoherente** | El pecado de DarkOrbit: mallas de 1 a 50.000 unidades compensadas con `scale` por instancia. Rompe validación, física y razonamiento espacial. |
| **N-gons** | Rompen el import de Godot y el horneado de normales. |
| **Normales delegadas al motor** | Pérdida de control artístico y suavizado impredecible entre versiones de Godot. |
| **Asimetría modelada** | Duplica el trabajo, duplica los vértices y duplica el área de UV, para una diferencia que nadie ve a 150 px. |
| **Un asset con 100 piezas** | El coste de polígonos correlaciona con el número de piezas (r = 0,70). Cien piezas es un error de método, no de presupuesto. |
| **Emisión generosa** | Una máscara de glow que cubre el 30 % del asset lo convierte en una mancha de bloom. Menos del 5 %. |
| **Densidad de texel uniforme** | Da los mismos texels al vientre que a la cara superior. Deliberadamente no uniforme. |

---

## 10. Cómo se comprueba

Las herramientas están en
[`mex-orbit-art/tools/asset-audit/`](../../../mex-orbit-art/tools/asset-audit/)
y funcionan igual sobre un GLB de Astrion que sobre las mallas de referencia.

```bash
# validacion completa contra este estandar (screen_size del JSON de la especie)
py tools/asset-audit/validate_asset.py assets/npcs/aci-01.glb --type=boss --screen-size=150

# renders de diagnostico (solid / wire / solid+wire / densidad, cenital y 3/4)
blender -b -P tools/asset-audit/render_diagnostics.py -- <dir_obj> <dir_salida>

# mapa de silueta: que triangulo paga contorno, cual paga superficie, cual nada
py tools/asset-audit/render_silhouette_map.py <salida> modelo.obj

# prueba de simplificacion a 75/50/25 % con render al tamano real de juego
blender -b -P tools/asset-audit/decimation_test.py -- <obj> <salida> <escalas.json> <nombre>
```

El detalle de cada script y de cada métrica está en el
[README de asset-audit](../../../mex-orbit-art/tools/asset-audit/README.md).

### Umbrales del validador

| Comprobación | Umbral | Severidad |
|---|---|---|
| Diagonal de la caja | 20 – 400 unidades | error |
| Triángulos | rango del §4.1 | aviso |
| Densidad en pantalla (solo > 1.000 tris) | aviso > 800, error > 2.000 tris/1.000 px² | aviso / error |
| Caras interiores, duplicadas, degeneradas, no-manifold, n-gons | 0 | error |
| Piezas de grosor cero (fuera de FX) | 0 | error |
| Triángulos nunca visibles | ≤ 5 % | error |
| Piezas visibles | rango del §2.2.7 | aviso |
| Cuota de la pieza mayor | ≥ 40 % | aviso |
| Simetría en X | ≥ 0,95 (0,60 orgánicos) | aviso |
| Ratio de quads | ≥ 0,60 | aviso |
| Ocupación de UV | ≥ 75 % | aviso |
| Normales explícitas exportadas | sí | error |
| Texels por triángulo | 200 – 600 | aviso |

---

## Apéndice: la tabla de un vistazo

Para pegar en la pared del que modela.

```
CÁMARA           45° elevación, 25° azimut, FOV 30, zoom 1-3
PÍXELES          unidades × 1,5 = px de alto a 1440p, zoom 1
UMBRAL           nada geométrico por debajo de 3 px (= 2 unidades)
ESCALA           screen_size EN EL BRIEF · 20-400 u · morro a -Z · Y arriba · origen centrado
APLANAMIENTO     nave 0,35-0,55 · NPC 0,50-0,75 · estructura 0,7-1,1
PIEZAS           nave 8-20 · NPC 4-12 · dron 1-6 · estación 15-40
PIEZA MAYOR      >= 40 % de los triángulos
GROSOR           >= 0,8 % de la dimensión mayor, mínimo 0,3 unidades
CILINDROS        4 / 6 / 8-10 / 12-16 / 20-24 segmentos según px de diámetro
ESFERAS          64 / 144 / 256 / 400 / 576 / 1024 triángulos
MOTOR            60-100 tris, 3 anillos, nada dentro
ANCLAJES         Empties: muzzle_ engine_ light_ hit_
NAVE JUGADOR     10.000-20.000 tris · 2048 base color
NPC NORMAL       2.000-4.000 tris · 1024 base color
DRON             250-700 tris · 512 base color
DENSIDAD         techo 800 tris/1.000 px² (error 2.000), solo si > 1.000 tris
EMISIÓN          < 5 % del área
PANELES          nunca geometría
```
