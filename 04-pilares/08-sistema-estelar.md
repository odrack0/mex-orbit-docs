# El sistema estelar — mapas y portales

**Estado: análisis, sin implementar.** Este documento fija la **estructura**: qué mapas existen, qué
portales tiene cada uno, en qué borde están y a dónde llevan. Sin fondos y sin NPCs — eso viene
después.

## De dónde sale esto (y de dónde no)

**El servidor legado no sirve como fuente**, y no por estar equivocado sino por estar **vacío**:

| | |
|---|---|
| Mapas **con** portales en el dump | 10 — `4-1`…`4-4`, `1-6`, `1-8`, `2-6`, `2-8`, `3-6`, `3-8` |
| Mapas **sin ningún** portal | 18 — **todos** los de `1-1`…`1-4`, `2-1`…`2-4`, `3-1`…`3-4`, más `1-5`, `1-7`, `2-5`, `2-7`, `3-5`, `3-7` |
| Mapas que el mapa estelar muestra y el dump **no tiene** | `4-5`, `5-1`, `5-2`, `5-3` |

Lo único que se conserva del legado es la **numeración de ids** (`1-1`=1 … `3-8`=28, `4-1`…`4-4`=13…16,
`4-5`=29, `5-1`…`5-3`=91…93), por continuidad con los datos viejos.

**La fuente es el arte original del mapa estelar**, que el prototipo Godot tiene extraído en
`assets/ui/starmap/`: `advanced_page_0.png` (621×427) y `advanced_page_1.png` (710×432), más
`starmap.json` con la posición de cada miniatura. Ese arte **es** el grafo: rectángulos de 82×52 —196×123
para `4-4` y `4-5`— unidos por polilíneas, con la etiqueta `Jn` justo en el punto del borde donde está
cada portal.

**El grafo se extrajo por análisis de imagen, no a ojo.** Se enmascaran los bordes de los rectángulos,
queda la red de conectores, y desde cada punto donde una línea toca un borde se recorre la línea hasta
el otro extremo. Se aceptan solo los **pares mutuos**: A es el extremo más cercano de B *y* B el de A.
Leer esto de una captura escalada habría dado un grafo con errores repartidos, y encima del grafo se
construye todo el sistema estelar.

## Página 0 — mapas base (los 15 del sistema interior)

**Completa y validada por dos caminos independientes:**

1. **El recuento de etiquetas.** 21 puertas × 2 = **42 portales**, y las etiquetas del original van de
   `J1` a `J42`. No sobra ni falta ninguna.
2. **La simetría de facción.** Las tres facciones salen idénticas sin haberlo impuesto: `x-1` tiene
   **1** portal, `x-2` tiene **3**, `x-3` tiene **3**, `x-4` tiene **4**. Si la extracción tuviera un
   fallo, la simetría se rompería justo ahí.

| Mapa | Portal | Borde | Posición | Coordenada sugerida | Lleva a |
|---|---|---|---|---|---|
| **1-1** | 1 | derecho | final | `20176, 11801` | **1-2** |
| **1-2** | 1 | inferior | final | `18740, 12416` | **1-4** |
|  | 2 | superior | final | `18740, 384` | **1-3** |
|  | 3 | izquierdo | inicio | `624, 1254` | **1-1** |
| **1-3** | 1 | inferior | final | `17472, 12416` | **1-4** |
|  | 2 | superior | final | `17472, 384` | **2-3** |
|  | 3 | izquierdo | final | `624, 11046` | **1-2** |
| **1-4** | 1 | inferior | final | `17971, 12416` | **3-4** |
|  | 2 | superior | final | `17472, 384` | **1-3** |
|  | 3 | derecho | centro | `20176, 6771` | **4-1** |
|  | 4 | izquierdo | inicio | `624, 1753` | **1-2** |
| **2-1** | 1 | inferior | inicio | `2828, 12416` | **2-2** |
| **2-2** | 1 | derecho | inicio | `20176, 4019` | **2-1** |
|  | 2 | derecho | final | `20176, 9536` | **2-4** |
|  | 3 | izquierdo | final | `624, 10291` | **2-3** |
| **2-3** | 1 | superior | final | `18491, 384` | **2-2** |
|  | 2 | derecho | final | `20176, 10547` | **2-4** |
|  | 3 | izquierdo | final | `624, 11046` | **1-3** |
| **2-4** | 1 | inferior | inicio | `2308, 12416` | **3-3** |
|  | 2 | inferior | centro | `10275, 12416` | **4-2** |
|  | 3 | superior | inicio | `2308, 384` | **2-2** |
|  | 4 | derecho | inicio | `20176, 1254` | **2-3** |
| **3-1** | 1 | izquierdo | inicio | `624, 2009` | **3-2** |
| **3-2** | 1 | inferior | final | `19011, 12416` | **3-1** |
|  | 2 | derecho | inicio | `20176, 2764` | **3-3** |
|  | 3 | izquierdo | inicio | `624, 2764` | **3-4** |
| **3-3** | 1 | inferior | inicio | `2308, 12416` | **3-4** |
|  | 2 | inferior | final | `17971, 12416` | **3-2** |
|  | 3 | superior | inicio | `2059, 384` | **2-4** |
| **3-4** | 1 | inferior | final | `19780, 12416` | **3-2** |
|  | 2 | superior | centro | `10275, 384` | **4-3** |
|  | 3 | derecho | inicio | `20176, 2009` | **3-3** |
|  | 4 | izquierdo | inicio | `624, 2009` | **1-4** |
| **4-1** | 1 | inferior | final | `17971, 12416` | **4-3** |
|  | 2 | derecho | inicio | `20176, 998` | **4-2** |
|  | 3 | izquierdo | centro | `624, 6771` | **1-4** |
| **4-2** | 1 | inferior | final | `18241, 12416` | **4-3** |
|  | 2 | superior | centro | `10275, 384` | **2-4** |
|  | 3 | izquierdo | final | `624, 10547` | **4-1** |
| **4-3** | 1 | superior | inicio | `1289, 384` | **4-2** |
|  | 2 | derecho | centro | `20176, 6272` | **3-4** |
|  | 3 | izquierdo | final | `624, 10035` | **4-1** |

## Página 1 — mapas exteriores

**18 puertas confirmadas de ~23.** Mismo método, mismo criterio de pares mutuos.

| Mapa | Portal | Borde | Posición | Coordenada sugerida | Lleva a |
|---|---|---|---|---|---|
| **1-5** | 1 | inferior | inicio | `5387, 12416` | **1-7** |
|  | 2 | inferior | final | `14123, 12416` | **4-5** |
|  | 3 | superior | inicio | `5387, 384` | **1-6** |
|  | 4 | derecho | centro | `20176, 6528` | **4-4** |
| **1-6** | 1 | inferior | inicio | `4617, 12416` | **1-8** |
|  | 2 | inferior | final | `16432, 12416` | **1-5** |
| **1-7** | 1 | superior | inicio | `4617, 384` | **1-8** |
|  | 2 | superior | final | `17201, 384` | **1-5** |
| **1-8** | 1 | inferior | final | `17201, 12416` | **1-7** |
|  | 2 | superior | final | `17201, 384` | **1-6** |
| **2-5** | 1 | superior | inicio | `2308, 384` | **2-6** |
|  | 2 | derecho | inicio | `20176, 2009` | **2-7** |
|  | 3 | izquierdo | final | `624, 9036` | **4-4** |
| **2-6** | 1 | inferior | inicio | `2308, 12416` | **2-5** |
|  | 2 | derecho | inicio | `20176, 3520` | **2-8** |
| **2-7** | 1 | inferior | inicio | `2308, 12416` | **2-5** |
|  | 2 | superior | final | `18491, 384` | **2-8** |
| **2-8** | 1 | inferior | inicio | `2558, 12416` | **2-6** |
|  | 2 | inferior | final | `18491, 12416` | **2-7** |
| **3-5** | 1 | inferior | inicio | `3848, 12416` | **3-6** |
|  | 2 | derecho | final | `20176, 10035` | **3-7** |
|  | 3 | izquierdo | inicio | `624, 3763` | **4-4** |
| **3-6** | 1 | derecho | final | `20176, 9280` | **3-8** |
|  | 2 | izquierdo | inicio | `624, 1753` | **3-5** |
| **3-7** | 1 | derecho | final | `20176, 10035` | **3-8** |
|  | 2 | izquierdo | final | `624, 10035` | **3-5** |
| **3-8** | 1 | superior | inicio | `3390, 384` | **3-7** |
|  | 2 | izquierdo | final | `624, 9280` | **3-6** |
| **4-4** | 1 | inferior | final | `18449, 12416` | **3-5** |
|  | 2 | derecho | inicio | `20176, 947` | **2-5** |
|  | 3 | izquierdo | centro | `624, 6195` | **1-5** |
| **4-5** | 1 | izquierdo | centro | `624, 6924` | **1-5** |
| **5-1** | 1 | izquierdo | centro | `624, 6528` | **5-2** |
| **5-2** | 1 | derecho | centro | `20176, 5772` | **5-1** |
|  | 2 | izquierdo | centro | `624, 5772` | **5-3** |
| **5-3** | 1 | derecho | centro | `20176, 6528` | **5-2** |

### Lo que queda abierto

Nueve extremos de línea alrededor de **`4-4`** y **`4-5`** no cierran en pares mutuos: ahí las líneas
corren muy juntas y el trazado se contamina entre ellas. **No se han adivinado.** Son:

| Mapa | Borde | Punto en la página |
|---|---|---|
| `5-3` | izquierdo, abajo | (202, 85) |
| `5-1` | derecho, abajo | (469, 85) |
| `4-4` | superior | (349, 138) · (369, 138) · (388, 138) |
| `2-5` | derecho | (587, 164) |
| `3-5` | superior | (543, 269) |
| `4-5` | varios | (328, 297) · (425, 343) · (425, 382) · (328, 419) |
| `3-8` | derecho | (708, 363) — recortado por el borde de la página |

Por la forma del grafo, casi todos deben ser conexiones de `4-4` y `4-5` con `5-1`/`5-2`/`5-3` y con
los mapas de facción exteriores. Se resuelven mirando esa zona ampliada, uno a uno.

## Cómo leer las coordenadas

La columna **borde** y **posición** es lo que el arte dice de verdad: el portal está en tal borde, al
inicio / centro / final. La **coordenada sugerida** es esa posición relativa llevada a un mapa de
20800×12800, y es **una propuesta, no un dato recuperado** — el arte no guarda coordenadas de mundo.
Ajustarlas es decisión de diseño; lo que no se debe cambiar sin motivo es el borde y el tercio, porque
eso es lo que hace que el sistema se sienta como el original.

Los portales van **en pares**: cada puerta son dos portales, uno en cada mapa, y en el original sus
etiquetas son números consecutivos (`J1`↔`J2`, `J3`↔`J4`, `J27`↔`J28`…). Conviene conservar esa
propiedad al sembrar los datos: es la comprobación más barata de que no falta un lado.

## Decisiones pendientes

- **`5-3` queda fuera** por indicación expresa. `4-5`, `5-1` y `5-2` **sí entran**, aunque no existan
  en el legado.
- **Los ids**: conservar la numeración del legado (1…29, 91…93) o empezar limpio.
- **El tamaño de los mapas**: se asume 20800×12800 para todos, que es el del `1-1` actual. `4-4` y
  `4-5` se dibujan mucho más grandes en el mapa estelar (196×123 contra 82×52), lo que sugiere que en
  el original **son mapas mayores**. Si se respeta eso, sus coordenadas cambian.
