# astrion-lowpoly — estudio y norma de geometría

Estudio técnico de septiembre de 2026: se midieron **448 mallas** del cliente
clásico de DarkOrbit para entender cómo un MMO cenital resolvió sus assets con
tan poca geometría, y se convirtió lo aprendido en la norma de producción de
Astrion.

DarkOrbit se usa **exclusivamente como referencia técnica**. Ningún diseño,
silueta, proporción ni identidad visual suya se traslada a Astrion.

| Documento | Qué es |
|---|---|
| **[ASTRION_LOW_POLY_MODELING_STANDARD.md](ASTRION_LOW_POLY_MODELING_STANDARD.md)** | **La norma.** Geometría, superficies curvas, presupuestos, tabla textura-contra-geometría, legibilidad cenital, reglas para el agente de Blender y anti-patrones. |
| [DARKORBIT_ASSET_ANALYSIS.md](DARKORBIT_ASSET_ANALYSIS.md) | El análisis del que sale todo: qué se midió, con qué método y qué salió. |
| [datos/](datos/) | Métricas crudas: 309 assets × 68 columnas, 10.872 piezas, 4.401 texturas. |
| [renders/](renders/) | Renders de diagnóstico de los casos más ilustrativos. |
| [`mex-orbit-art/tools/asset-audit/`](../../../mex-orbit-art/tools/asset-audit/) | Las herramientas. Funcionan igual sobre nuestros GLB. |

---

## Las 10 conclusiones que deberían cambiar nuestro pipeline

Ordenadas por impacto sobre lo que hacemos hoy.

### 1. El objetivo de 100 k triángulos nunca se derivó de la cámara

Midiendo los 15 assets que hoy tiene el cliente, escalados con su `screen_size`
real y rasterizados desde la cámara de juego a 1440p:

| asset | tris | px de diagonal | tris / 1.000 px² |
|---|---:|---:|---:|
| aci-03 | 99.992 | 343 | **3.196** |
| phoenix | 101.860 | 310 | **1.841** |
| aci-02 | 58.106 | 366 | **1.148** |
| aci-05 | 54.086 | 374 | **1.122** |
| aci-04 | 78.751 | 431 | 869 |
| mordax | 104.194 | 469 | 745 |
| gravit | 30.422 | 280 | 568 |
| vex | 10.434 | 285 | 379 |
| … | | | |
| **aci-01** | **13.686** | **396** | **119** |
| skarnox | 10.150 | 591 | 46 |

Mediana: **30.422 triángulos, 379 tris/1.000 px²**. Cuatro assets superan el
techo de aviso de 800 y `aci-03` está en **3.196** — ocho veces la nave más
densa de DarkOrbit (`phoenix`, 313).

Lo revelador es que **`aci-01`, el asset que el pipeline llama "bien hecho",
está en 119** — dentro del rango de DarkOrbit y ocho veces más eficiente que
`aci-03`, que ocupa *menos* pantalla. La diferencia no es de calidad visual: es
que a unos se les aplicó criterio y a otros se les aplicó el decimado a 100 k.

**El 100 k es un tope de seguridad del pipeline, no un presupuesto.** El
presupuesto se decide contra el número de píxeles que el asset va a ocupar
(§4 del estándar), y el validador ya lo calcula. Esto no es una urgencia de
rendimiento — nuestra propia medición dice que los triángulos no son el coste
en la iGPU de referencia — es una de coherencia y de control.

### 2. El número de piezas, no el tamaño, es lo que dispara el coste

Correlación con `log(tris)` sobre los 309 assets de DarkOrbit: piezas
**r = +0,70**, tamaño físico r = +0,20, materiales r = +0,10. Hay que poner
presupuesto de **piezas** en el brief, no solo de triángulos. Es la palanca
que de verdad controla el coste, y la que hoy no medimos.

### 3. Los cilindros llevan 6 lados, no 64

Mediana medida sobre 948 piezas torneadas de DarkOrbit: **6 segmentos**, p90
12. Un cañón del buque insignia tiene 6 lados y se lee perfecto a 160 px.

Nuestros modelos generativos traen otra cosa. En `aci-01`, el mejor de los
nuestros, hay **tres discos con 60, 64 y 66 posiciones angulares** (islas 3, 5
y 8) que suman **2.340 triángulos en tres arandelas** — el 17 % del asset. Y
otras 12 piezas redondas con geometría irregular donde debería haber una
primitiva exacta.

Sustituir toda primitiva redonda generada por la de la escalera del §3 del
estándar es, sin tocar la silueta, el recorte más limpio disponible.

### 4. Ninguna esfera pasa de 768 triángulos, y esa se reserva a planetas

La escalera completa: 48 · 224 · 288 · 368 · 416 · 546 · **768**. Hay un asset
llamado literalmente `planet-768-tris`. Nuestra escalera (64/144/256/400/576/
1024) ya es más generosa; lo que no puede pasar es que alguien meta la
UV-sphere por defecto de Blender.

### 5. Los paneles, juntas, remaches y rejillas nunca son geometría

El atlas de 512×512 del Goliath lleva pintados los paneles, las juntas, las
rejillas, el sombreado de oclusión, las calcas y la numeración. El normal map
lleva la curvatura y los biseles que la malla plana no tiene. **110 texels por
triángulo**: por cada triángulo que gastaron, pintaron cien píxeles. Nuestra
tabla textura-contra-geometría (§5 del estándar) tiene que ser vinculante en
el brief, no una recomendación.

### 6. Los motores y las bocas de arma no se modelan: son anclajes + partículas

388 `laserpoint_*`, 114 `engine_*` y 71 `light_position` en el corpus, todos
cajas vacías de 8 triángulos. La llama y la estela las pone el sistema de
partículas. Nosotros gastamos **0** triángulos en lo mismo usando Empties. Hay
que revisar si nuestros modelos traen toberas detalladas que no aportan nada.

### 7. Las naves son planas: alto = 0,44 × su huella

Con cámara a 45° el volumen vertical no se lee. DarkOrbit: naves 0,44, PET
0,48, NPC grande 0,53, estructuras 0,84, portales 1,00.

Nosotros estamos en **0,54 de mediana**, que está bien, con tres excepciones
que pagan volumen vertical que la cámara aplasta: **`aci-01` 1,00**, `mordax`
0,90 y `skarn` 0,79. Es la corrección más barata que podemos hacer y hay que
meterla en el brief de concept, no en el modelado: si la imagen de referencia
es un bicho alto, el modelo sale alto.

### 8. La mitad de la geometría de una nave no paga su sitio en la silueta

Prueba de simplificación a tamaño real de juego: el Goliath al **50 %** cambia
el 2,3 % de los píxeles de silueta y al **25 %** el 6 %; a 201 px son
prácticamente indistinguibles. **Pero** —y esto es lo importante— al 25 % los
cañones han desaparecido. **Nunca entregar el resultado de un decimador sin
repasar a mano los elementos finos**: son lo primero que destruye y lo último
que la métrica de silueta detecta.

### 9. No hay LOD de malla, y eso obliga a acertar el presupuesto base

Se buscó y no existe: ni clases de LOD en el cliente, ni mallas con sufijo de
LOD (0 de 322), ni atributo en el XML. La escalera de calidad de DarkOrbit es
de **texturas y efectos**, nunca de geometría. Coincide con nuestra propia
medición de que en un cenital todos los visibles están a la misma distancia y
comparten nivel de LOD. **La malla que entregas es la que se ve siempre.**

### 10. `screen_size` es el dato que falta en el brief, no en el JSON

DarkOrbit no tuvo convención de escala: sus mallas van de 1,1 a 50.425 unidades
de diagonal y lo compensan con un `scale` por instancia (22 de 309 fuera del
rango 10–2.000). Eso hace imposible validar automáticamente o razonar en
unidades de mundo.

Nosotros **sí** la tenemos — `entity_node.gd` escala la malla para que su
extensión mayor valga el `screen_size` del JSON de la especie — pero ese número
aparece **al final**, cuando el modelo ya está hecho. Y es el número del que
depende todo: los píxeles en pantalla, el umbral de 3 px, los segmentos de cada
cilindro y el presupuesto entero.

**`screen_size` tiene que estar en el brief antes de abrir Blender.** Es un
cambio de proceso de coste cero y es el que hace utilizable todo lo demás de
este documento.

---

### Cómo reproducir estos números

```bash
py mex-orbit-art/tools/asset-audit/validate_asset.py mex-orbit-client/assets/npcs/aci-01.glb --type=boss --screen-size=150
```

`--screen-size` es el del JSON de la especie (`data/npcs/*.json`): reproduce
exactamente lo que hace `entity_node.gd` al montar la malla, así que la
densidad que sale es la que el jugador ve.

---

## Bonus: lo que DarkOrbit hizo mal y no hay que copiar

- **Geometría enterrada sin animación que la justifique**: `moduleCircle`
  (8.464 tris, **51,9 % nunca visible**), `streuner-homebase` (21,7 %),
  `module01/03/04/05`. Sus assets más caros son los que peor amortizan.
- **Normales no almacenadas en el 59 % de los assets**: el motor las recalcula
  al cargar con el suavizado que le parezca.
- **36 piezas de mediana por asset, hasta 483**: kit-bash sin control.
- **Un mismo fichero con todos los estados** (`cubikon-full-stack001`).

## Método, en una línea

Se parsearon los 448 AWD (0 fallos), se midieron con numpy/scipy, se
rasterizaron desde la cámara documentada del cliente (FOV 30, elevación 45°,
azimut 25°, d = 1740) y se validó la aritmética de píxeles contra el
`visualSize` que declara el propio cliente: **r = 0,922 en log-log**. Lo que no
se pudo comprobar con los ficheros está marcado como no verificable en el
análisis (§10).
