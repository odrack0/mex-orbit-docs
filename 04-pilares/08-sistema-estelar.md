# El sistema estelar — mapas y portales

**Estado: análisis cerrado, sin implementar.** Fija la **estructura**: qué mapas hay, qué portales
tiene cada uno, en qué borde están y a dónde llevan. Sin fondos y sin NPCs.

**Alcance: 26 mapas.** `1-1`…`1-8`, `2-1`…`2-8`, `3-1`…`3-8`, `4-1`…`4-5`. **Los `5-x` quedan fuera**
por decisión del proyecto (2026-08-26).

## De dónde sale esto (y de dónde no)

**El servidor legado no sirve como fuente**, y no por estar equivocado sino por estar **vacío**: solo
10 de 28 mapas tienen portales en el dump, y los 18 que no los tienen incluyen **todos** los mapas base
de las tres facciones. Lo único que se conserva es la **numeración de ids** (`1-1`=1 … `3-8`=28,
`4-1`…`4-4`=13…16, `4-5`=29), por continuidad con los datos viejos.

**La fuente es el arte original del mapa estelar**, que el prototipo Godot tiene extraído en
`assets/ui/starmap/`: `advanced_page_0.png` (621×427) y `advanced_page_1.png` (710×432), más
`starmap.json` con la posición de cada miniatura. Ese arte **es** el grafo: rectángulos de 82×52
—196×123 para `4-4` y `4-5`— unidos por polilíneas, con la etiqueta `Jn` justo en el punto del borde
donde está cada portal.

**Se extrajo por análisis de imagen, no a ojo.** Se enmascaran los bordes de los rectángulos, queda la
red de conectores, y desde cada punto donde una línea toca un borde se recorre la línea hasta el otro
extremo. Se aceptan solo los **pares mutuos**: A es el extremo más cercano de B *y* B el de A. Leer
esto de una captura escalada habría dado un grafo con errores repartidos, y encima del grafo se
construye todo el sistema.

## Las dos validaciones

Ninguna se impuso: salen solas, y por eso valen.

**1 · El recuento de etiquetas.** La página 0 da 21 puertas de borde × 2 = **42 portales**, y las
etiquetas del original van de `J1` a `J42`. No sobra ni falta ninguna.

**2 · La simetría de facción.** Las tres facciones salen **idénticas** sin haberlo pedido:

| | `x-1` | `x-2` | `x-3` | `x-4` | `x-5` | `x-6` | `x-7` | `x-8` |
|---|---|---|---|---|---|---|---|---|
| portales | 1 | 3 | 3 | 4 | 4 | 2 | 2 | 2 |

Si la extracción tuviera un fallo, la simetría se rompería justo ahí. Los tres portales de `4-5` que
faltaban encajaron exactamente donde la simetría los pedía.

## Los tres portales que el arte NO puede mostrar

`4-1`, `4-2` y `4-3` tienen cada uno un portal a **`4-4` en el centro del mapa**, no en un borde. El
mapa estelar dibuja cruces de borde a borde, así que **es ciego a estos**: no es que falten en el arte,
es que no se pueden dibujar ahí.

Son los únicos datos del legado que se dan por buenos, porque son los únicos que el arte no contradice:

| Mapa | Posición del portal | Llega a `4-4` en |
|---|---|---|
| `4-1` | `10000, 6200` | `19200, 13400` |
| `4-2` | `10400, 6300` | `21900, 11900` |
| `4-3` | `10300, 6600` | `21900, 14500` |

El centro de un mapa de 20800×12800 es `10400, 6400`: los tres están ahí.

## `4-4` y `4-5` son mapas MÁS GRANDES

Dos indicios independientes:

- En el arte se dibujan a **196×123** cuando todos los demás son 82×52 — misma proporción, 2,4× el
  lado.
- El legado usa en `4-4` coordenadas de hasta **`28900, 20900`**, imposibles en un mapa de
  20800×12800.

No coinciden en el factor exacto, pero coinciden en el hecho. **Hay que decidir su tamaño real**, y de
esa decisión dependen sus coordenadas.

## Los errores del legado, uno a uno

Comparado mapa a mapa contra el arte. No es una impresión: es la lista.

### En los mapas bajos — el peor de todos

| Mapa | Qué le pasa |
|---|---|
| `4-1` | le falta la puerta a **`1-4`** |
| `4-2` | le falta la puerta a **`2-4`** |
| `4-3` | le falta la puerta a **`3-4`** |

**Al cluster `4-x` le faltan sus tres entradas.** En el legado no se puede llegar a `4-1`, `4-2` ni
`4-3` desde los mapas de facción: solo desde ellos mismos y desde `4-4`. Y a `4-4` se entra únicamente
por `1-6`/`2-6`/`3-6`, que son justo las puertas que el arte dice que **no existen**. Es decir: la
única forma de entrar a la zona `4-x` es por una puerta equivocada.

Los otros once mapas bajos (`1-1`…`1-4`, `2-1`…`2-4`, `3-1`…`3-4`) no tienen ningún dato.

### En los mapas altos

| Mapa | Le faltan | Sobran (no están en el arte) |
|---|---|---|
| `1-6` | `1-5` | `4-4` |
| `2-6` | `2-5` | `4-4` |
| `3-6` | `3-5`, `3-8` | `4-3` |
| `2-8` | `2-6`, `2-7` | **`2-1`** |
| `1-8` | `1-7` | — |
| `3-8` | `3-7` | — |
| `4-4` | `1-5`, `2-5`, `3-5` | `1-6`, `2-6`, `3-6` |

El `2-8 → 2-1` es especialmente revelador: une un mapa alto con uno bajo, algo que el grafo no hace en
ningún otro sitio, y `2-1` no tiene datos para devolver el viaje.

### Y tres fallos estructurales que no dependen del grafo

**1 · Cinco portales de una sola dirección** — se va y no se vuelve:

`4-4`→`3-6` · `2-6`→`2-8` · `2-8`→`2-1` · `3-6`→`4-3` · `3-8`→`3-6`

**2 · Cuatro puertas distintas aterrizan en el mismo punto.** En `4-4`, las vueltas hacia `4-2`, `1-6`,
`2-6` y `3-6` llegan **todas** a `10400, 6300`.

**3 · Dos colisiones más de llegada**: en `4-4` el punto `21900, 11900` recibe desde `4-2` y desde
`2-6`; en `4-3` el punto `10300, 6600` recibe desde `4-4` y desde `3-6`.

### Lo que esto nos deja como checklist

Los tres fallos estructurales son **comprobables por código** y no cuestan nada, así que nuestros datos
deberían pasarlos antes de sembrarse:

- toda puerta tiene sus **dos** lados;
- ninguna llegada **repite** coordenada dentro del mismo mapa;
- ningún portal apunta a un mapa **sin datos**.

## Página 0 — el sistema interior

21 puertas de borde + los 3 portales centrales a `4-4`.

| Mapa | # | Borde | Posición | Coordenada sugerida | Lleva a |
|---|---|---|---|---|---|
| **1-1** | 1 | derecho | final | `20176, 11796` | **1-2** |
| **1-2** | 1 | inferior | final | `18745, 12416` | **1-4** |
|  | 2 | superior | final | `18745, 384` | **1-3** |
|  | 3 | izquierdo | inicio | `624, 1254` | **1-1** |
| **1-3** | 1 | inferior | final | `17461, 12416` | **1-4** |
|  | 2 | superior | final | `17461, 384` | **2-3** |
|  | 3 | izquierdo | final | `624, 11043` | **1-2** |
| **1-4** | 1 | inferior | final | `17975, 12416` | **3-4** |
|  | 2 | superior | final | `17461, 384` | **1-3** |
|  | 3 | derecho | centro | `20176, 6776` | **4-1** |
|  | 4 | izquierdo | inicio | `624, 1756` | **1-2** |
| **2-1** | 1 | inferior | inicio | `2824, 12416` | **2-2** |
| **2-2** | 1 | derecho | inicio | `20176, 4015` | **2-1** |
|  | 2 | derecho | final | `20176, 9537` | **2-4** |
|  | 3 | izquierdo | final | `624, 10290` | **2-3** |
| **2-3** | 1 | superior | final | `18488, 384` | **2-2** |
|  | 2 | derecho | final | `20176, 10541` | **2-4** |
|  | 3 | izquierdo | final | `624, 11043` | **1-3** |
| **2-4** | 1 | inferior | inicio | `2311, 12416` | **3-3** |
|  | 2 | inferior | centro | `10271, 12416` | **4-2** |
|  | 3 | superior | inicio | `2311, 384` | **2-2** |
|  | 4 | derecho | inicio | `20176, 1254` | **2-3** |
| **3-1** | 1 | izquierdo | inicio | `624, 2007` | **3-2** |
| **3-2** | 1 | inferior | final | `19002, 12416` | **3-1** |
|  | 2 | derecho | inicio | `20176, 2760` | **3-3** |
|  | 3 | izquierdo | inicio | `624, 2760` | **3-4** |
| **3-3** | 1 | inferior | inicio | `2311, 12416` | **3-4** |
|  | 2 | inferior | final | `17975, 12416` | **3-2** |
|  | 3 | superior | inicio | `2054, 384` | **2-4** |
| **3-4** | 1 | inferior | final | `19772, 12416` | **3-2** |
|  | 2 | superior | centro | `10271, 384` | **4-3** |
|  | 3 | derecho | inicio | `20176, 2007` | **3-3** |
|  | 4 | izquierdo | inicio | `624, 2007` | **1-4** |
| **4-1** | 1 | inferior | final | `17975, 12416` | **4-3** |
|  | 2 | centro | — | `10004, 6195` | **4-4** ·  ⚠ centro |
|  | 3 | derecho | inicio | `20176, 1003` | **4-2** |
|  | 4 | izquierdo | centro | `624, 6776` | **1-4** |
| **4-2** | 1 | inferior | final | `18232, 12416` | **4-3** |
|  | 2 | superior | centro | `10271, 384` | **2-4** |
|  | 3 | centro | — | `10400, 6297` | **4-4** ·  ⚠ centro |
|  | 4 | izquierdo | final | `624, 10541` | **4-1** |
| **4-3** | 1 | superior | inicio | `1283, 384` | **4-2** |
|  | 2 | centro | — | `10296, 6604` | **4-4** ·  ⚠ centro |
|  | 3 | derecho | centro | `20176, 6274` | **3-4** |
|  | 4 | izquierdo | final | `624, 10039` | **4-1** |

## Página 1 — el sistema exterior

18 puertas, 36 portales.

| Mapa | # | Borde | Posición | Coordenada sugerida | Lleva a |
|---|---|---|---|---|---|
| **1-5** | 1 | inferior | inicio | `5392, 12416` | **1-7** |
|  | 2 | inferior | final | `14123, 12416` | **4-5** |
|  | 3 | superior | inicio | `5392, 384` | **1-6** |
|  | 4 | derecho | centro | `20176, 6525` | **4-4** |
| **1-6** | 1 | inferior | inicio | `4622, 12416` | **1-8** |
|  | 2 | inferior | final | `16434, 12416` | **1-5** |
| **1-7** | 1 | superior | inicio | `4622, 384` | **1-8** |
|  | 2 | superior | final | `17204, 384` | **1-5** |
| **1-8** | 1 | inferior | final | `17204, 12416` | **1-7** |
|  | 2 | superior | final | `17204, 384` | **1-6** |
| **2-5** | 1 | superior | inicio | `2311, 384` | **2-6** |
|  | 2 | derecho | inicio | `20176, 2007` | **2-7** |
|  | 3 | derecho | centro | `20176, 8282` | **4-5** |
|  | 4 | izquierdo | final | `624, 9035` | **4-4** |
| **2-6** | 1 | inferior | inicio | `2311, 12416` | **2-5** |
|  | 2 | derecho | inicio | `20176, 3513` | **2-8** |
| **2-7** | 1 | inferior | inicio | `2311, 12416` | **2-5** |
|  | 2 | superior | final | `18488, 384` | **2-8** |
| **2-8** | 1 | inferior | inicio | `2567, 12416` | **2-6** |
|  | 2 | inferior | final | `18488, 12416` | **2-7** |
| **3-5** | 1 | inferior | inicio | `3851, 12416` | **3-6** |
|  | 2 | superior | final | `14380, 384` | **4-5** |
|  | 3 | derecho | final | `20176, 10039` | **3-7** |
|  | 4 | izquierdo | inicio | `624, 3764` | **4-4** |
| **3-6** | 1 | derecho | final | `20176, 9286` | **3-8** |
|  | 2 | izquierdo | inicio | `624, 1756` | **3-5** |
| **3-7** | 1 | derecho | final | `20176, 10039` | **3-8** |
|  | 2 | izquierdo | final | `624, 10039` | **3-5** |
| **3-8** | 1 | superior | inicio | `3380, 384` | **3-7** |
|  | 2 | izquierdo | final | `624, 9286` | **3-6** |
| **4-4** | 1 | inferior | final | `18453, 12416` | **3-5** |
|  | 2 | derecho | inicio | `20176, 944` | **2-5** |
|  | 3 | izquierdo | centro | `624, 6190` | **1-5** |
| **4-5** | 1 | inferior | centro | `10453, 12416` | **3-5** |
|  | 2 | superior | centro | `10453, 384` | **2-5** |
|  | 3 | izquierdo | centro | `624, 6924` | **1-5** |

## Cómo leer las coordenadas

**Borde** y **posición** es lo que el arte dice de verdad. La **coordenada sugerida** es esa posición
relativa llevada a un mapa de 20800×12800: es **una propuesta, no un dato recuperado** — el arte no
guarda coordenadas de mundo. Ajustarlas es decisión de diseño; lo que no conviene cambiar sin motivo es
**el borde y el tercio**, porque eso es lo que hace que el sistema se sienta como el original.

Los portales van **en pares**: cada puerta son dos portales, uno en cada mapa, y en el original sus
etiquetas son números consecutivos (`J1`↔`J2`, `J27`↔`J28`…). Conviene conservar esa propiedad al
sembrar los datos — es la comprobación más barata de que no falta un lado.

## Decisiones pendientes

- **El tamaño de `4-4` y `4-5`.** Ver arriba; sin esto, sus coordenadas no se pueden fijar.
- **Los ids**: conservar la numeración del legado (1…29) o empezar limpio.
- **La posición de llegada** de cada portal. El arte no la da y el legado solo la tiene para las tres
  puertas centrales. Lo razonable es llegar **frente al portal de vuelta**, a una distancia fija, y que
  el dato se derive en vez de escribirse a mano — así no puede quedar descuadrado.
