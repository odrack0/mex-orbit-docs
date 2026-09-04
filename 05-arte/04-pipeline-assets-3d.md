# Astrion --- Pipeline de Producción de Assets 3D para Godot

## Objetivo

Este documento define el pipeline estándar para producir assets 3D de
**Astrion**, un MMO espacial top-down 3D inspirado en la legibilidad
visual de juegos como DarkOrbit, pero utilizando un pipeline moderno con
generación asistida por IA, materiales PBR y Godot.

El objetivo es obtener **alta calidad visual en gameplay sin
desperdiciar geometría, memoria ni rendimiento**.

> **Principio central: la geometría define la identidad; la textura
> vende la complejidad.**

La estrategia NO consiste en generar modelos de millones de caras y
posteriormente destruirlos mediante decimation. El asset debe diseñarse
desde el principio para su presupuesto de triángulos final.

> **Complementado el 3-sep-2026** con lo que hace la cadena automática
> después del GLB (§17), lo que el mundo de Astrion le pide a una textura
> (§18) y las medidas reales que sostienen o matizan cada regla (§19).
> Fuente de verdad de los diales: `mex-orbit-art/README.md`,
> `mex-orbit-client/README.md` y el skill `mexorbit-asset-3d`.

------------------------------------------------------------------------

## 1. Filosofía del pipeline

Cada tipo de información tiene una responsabilidad distinta:

-   **Geometry Reference:** forma, silueta, proporciones y volúmenes
    físicos.
-   **Texture Reference:** riqueza visual y detalle superficial.
-   **Godot:** iluminación, emisión, bloom/glow y presentación final.

No mezclar estas responsabilidades.

### Regla de decisión

Antes de añadir un detalle al modelo, preguntar:

> **¿Este detalle necesita cambiar la silueta o producir volumen visible
> desde la cámara real del juego?**

-   Si **sí**, puede justificar geometría.
-   Si **no**, normalmente debe resolverse mediante Base Color, Normal,
    Roughness, Metallic, AO o Emission.

------------------------------------------------------------------------

## 2. Concepto visual

Primero se diseña libremente la identidad del asset:

-   silueta;
-   proporciones;
-   lenguaje visual;
-   placas principales;
-   zonas energéticas;
-   materiales;
-   colores;
-   personalidad del NPC/nave;
-   nivel de desgaste deseado.

Este concept puede ser visualmente rico. **No debe utilizarse
necesariamente como entrada directa del generador 3D.**

Su función es definir qué queremos obtener al final.

> El contrato de prompts para el concept (luces como parches planos de un
> solo color saturado, casco gris medio, luz pareja) vive en
> `mex-orbit-art/prompts/README.md`. Lo que ahí se pide es exactamente lo
> que la cadena automática sabe extraer (§17.1 y §18.4).

------------------------------------------------------------------------

## 3. Definir el presupuesto geométrico ANTES de generar

El presupuesto se decide según la importancia del asset y su tamaño real
en pantalla.

  Tipo de asset                    Target inicial aproximado
  ------------------------------ ---------------------------
  Prop simple                                300--1,000 tris
  NPC simple                               1,000--2,000 tris
  NPC complejo                             2,000--5,000 tris
  Elite                                    3,000--6,000 tris
  Boss                                    5,000--10,000 tris
  Uber / Hero NPC                         8,000--15,000 tris
  Nave de jugador / Hero asset            8,000--20,000 tris

Estos valores son **targets iniciales, no cuotas obligatorias**.

> Utilizar la menor cantidad de geometría que conserve correctamente la
> silueta y los volúmenes importantes en gameplay.

No agregar caras simplemente porque exista presupuesto disponible.

### Lo que dice la medida (3-sep-2026)

Con los LOD automáticos del import de Godot, **los triángulos ya no son
el coste** en la iGPU de referencia: 30 bichos de 150 000 tris dan los
mismos fps que 30 de 56 000 (81,9 contra 81,2), y 15 de 58 000 los mismos
que 15 de 9 900. Lo que cuesta es la **VRAM de texturas** (12 MB por
asset a 1024) y la carga. Ojo: en un juego cenital **no hay LOD
cerca/lejos**, todos los visibles están a la misma distancia y comparten
nivel, así que los triángulos en pantalla son reales.

Consecuencia: la tabla de arriba se cumple por **legibilidad y
disciplina**, no por fps. Un asset de 13 686 tris (ACI-01 de Tripo) es
la referencia de «bien hecho». El validador avisa a 120 000, que es la
alarma contra un alto sin procesar, no un objetivo.

------------------------------------------------------------------------

## 4. Crear la Geometry Reference

A partir del concept se genera una imagen específicamente preparada para
el modelador 3D por IA.

### Características

-   Asset único y centrado.
-   Fondo gris neutro y limpio.
-   Material clay/gris neutro.
-   Iluminación suave y uniforme.
-   Vista elevada 3/4 cercana a top-down.
-   Aproximadamente 20--30° fuera de la vertical cuando convenga mostrar
    profundidad.
-   Silueta extremadamente clara.
-   Simetría clara cuando corresponda.
-   Grandes masas continuas.
-   Pocas piezas independientes.
-   Biseles solamente donde aporten forma importante.
-   Grosor de piezas claramente interpretable.

### Eliminar de la Geometry Reference

-   óxido;
-   scratches;
-   suciedad;
-   decals;
-   pintura decorativa;
-   micro-panelado;
-   tornillos pequeños;
-   pequeñas ranuras;
-   greebles;
-   cables insignificantes;
-   partículas;
-   glow;
-   bloom;
-   microemisivas;
-   detalles que desaparecerían desde la cámara de gameplay.

### Emisivas

Si una zona energética afecta claramente la estructura, puede
representarse como una forma grande y simple. Evitar docenas de pequeños
puntos o líneas.

> **Si un detalle no merece geometría, no debe aparecer como detalle
> geométrico en esta referencia.**

### Orientación y proa (lo que la cadena espera)

-   El juego pone la **proa en −Z** del GLB y la cámara mira el modelo
    desde arriba y desde el sur. La referencia debe dejar claro **qué es
    proa**: un ojo, una cabina, una punta. Sin eso, la orientación se
    decide a ciegas y cuesta dos retratos del bestiario por intento.
-   Si el bicho es **casi esférico** (caja con menos de un 10 % entre
    ejes), la cadena no puede deducir el «alto» y hay que fijar la
    orientación a mano (`TUMBAR=0` + `orientation` en el JSON).
-   Nada más fino que **1/60 del ancho del modelo**: a 150 px en pantalla
    son 2–3 px, y por debajo desaparece o parpadea.

------------------------------------------------------------------------

## 5. Generación del modelo en Tripo

La **Geometry Reference** es la entrada principal para generar el mesh.

Flujo:

``` text
Geometry Reference
        ↓
      Tripo
        ↓
Target de geometría
        ↓
      Mesh
```

No intentar resolver todavía el acabado visual definitivo.

### Objetivo

Conseguir directamente una malla cercana al presupuesto de producción,
en lugar de:

``` text
Modelo 2–3M caras
      ↓
Decimation extrema
      ↓
Pérdida de silueta / deformaciones / agujeros
```

La topología perfecta de modelado tradicional no es el objetivo
principal para assets rígidos sin deformación. Lo importante es que la
malla sea estable, visualmente correcta y eficiente en Godot.

### Lo que Tripo entrega hoy (medido con el ACI-01 v4)

-   Una sola malla, triangulada, UV limpias, caja de 1 unidad con el
    pivote en la base. La cadena la centra y la escala (§17.2).
-   **Solo color base** (8192 en JPG). Sin metallic-roughness, sin mapa de
    normales, sin AO. Si Tripo ofrece exportar esos mapas, **pedirlos**:
    con luz fija y modelo girando, el normal map sí se lee (§18.6).
-   Nombres generados con el prompt y un hash; la cadena los renombra.
-   El archivo va a `mex-orbit-art/source/3d-models/pulido/<bicho>.glb`.
    No se versiona (como `crudo/`); el master que sale de la cadena sí.

------------------------------------------------------------------------

## 6. Gate de aprobación geométrica

**No texturizar un mesh que todavía tiene problemas estructurales.**

Revisar:

-   silueta;
-   proporciones;
-   simetría;
-   grosor;
-   agujeros;
-   geometría flotante accidental;
-   superficies derretidas;
-   piezas fusionadas incorrectamente;
-   normales problemáticas;
-   deformaciones visibles;
-   lectura desde arriba;
-   lectura al tamaño real del gameplay.

Si el resultado falla significativamente, primero modificar/simplificar
la Geometry Reference y regenerar.

No intentar ocultar problemas geométricos mediante texturas.

> Herramienta: `validar-modelo.py` (§17.4) mide piezas, triángulos, caja,
> pivote y esqueleto en segundos y sin Blender. Y el render a 45° en gris
> desde el alto (`inspeccionar.py`) enseña la silueta sin que la textura
> distraiga. Las mallas parásitas (un cubo suelto de 12 tris del export)
> se descartan solas.

------------------------------------------------------------------------

## 7. Crear la Texture Reference

Una vez aprobada la geometría, crear una segunda imagen que conserve
**la misma identidad, proporciones y preferiblemente el mismo ángulo**.

Aquí sí se permite introducir toda la riqueza visual.

### Ejemplos de información para textura

-   titanio;
-   acero;
-   gunmetal;
-   brushed metal;
-   diferencias de roughness;
-   edge wear;
-   scratches;
-   chipped paint;
-   suciedad localizada;
-   óxido ligero;
-   variaciones de color;
-   decals;
-   marcas superficiales;
-   falso panelado;
-   detalles mecánicos fingidos;
-   emisivas.

> **Geometry Reference = forma. Texture Reference = riqueza.**

### Diseño para top-down

El detalle debe sobrevivir cuando el asset ocupa pocos píxeles.

Priorizar:

-   grandes bloques claros/oscuros;
-   contraste entre placas;
-   bordes legibles;
-   zonas energéticas grandes;
-   patrones medianos;
-   variaciones de material visibles.

Evitar depender excesivamente de microdetalle fotográfico.

> Lo que el mundo de Astrion le pide concretamente a esa textura (luz,
> cámara, material, emisión) está en §18. Resumen: casco gris medio-oscuro
> y frío, contraste por placas y por rugosidad, nada de luz cocida
> direccional, y las luces como parches planos de UN color saturado.

------------------------------------------------------------------------

## 8. Materiales PBR

Idealmente el asset final utiliza:

``` text
Base Color / Albedo
Normal
Metallic
Roughness
Ambient Occlusion
Emission Mask
```

El Normal Map puede aportar relieve superficial sin convertir cada
detalle en geometría real.

### Importante

No confundir detalle de normal map con detalle que cambia la silueta.

Una placa elevada grande puede necesitar geometría. Un scratch, una
pequeña ranura o un patrón superficial normalmente no.

### Cómo los aplica el cliente

-   **Con mapa metallic-roughness:** el cliente multiplica la rugosidad
    por 0,9 y el metal por 0,6 (`lighting.json`). Un metal PBR refleja lo
    que tiene alrededor, y aquí alrededor hay espacio negro: por eso el
    metal al 100 % sale negro y se le baja.
-   **Sin mapa:** rugosidad fija 0,35 (brillo cerrado, no mate, el gloss
    50 del original) y metal 0. El modelo se lee como pintura satinada.
-   **Rim** 0,3 siempre: separa la silueta del negro del fondo.
-   **La máscara de emisión NO se pinta aparte**: la cadena la extrae del
    color base por dominancia de canal (§17.1). Un mapa de emisión
    separado, si viene, se ignora.

------------------------------------------------------------------------

## 9. Emisivas

Las emisivas son parte importante de la identidad visual de Astrion,
pero deben mantenerse controladas.

### Preferir

-   franjas grandes;
-   núcleos grandes;
-   formas simples;
-   cyan saturado y limpio;
-   distribución intencional;
-   pocos elementos claramente visibles.

### Evitar

-   cientos de puntos luminosos;
-   partículas horneadas;
-   bloom dentro del Base Color;
-   manchas cyan accidentales;
-   líneas extremadamente finas;
-   microemisivas que desaparecen al reducir el asset.

La textura debe identificar **qué emite luz**. Godot debe controlar
**cómo se ve esa emisión**.

### El contrato medible

-   La cadena decide qué emite con una **máscara por canal**: para el
    cian, `min(g, b) − r`; para un primario, `canal − max(otros dos)`.
    Lo que pasa el umbral (`UMBRAL`, 0,25 de serie) emite; lo demás no.
-   Antes de aceptar una textura se mide: **cobertura ≥ 3 % con p99 de la
    máscara > 0,5**. Por debajo, las luces no se separan del casco y el
    pulso queda invisible (le pasó al Drony v1 con luces «pintadas»).
-   El albedo se **apaga** donde emite (`APAGAR`): el color encendido
    tiene que venir solo de la emisión, o el sol lo ilumina igual con el
    pulso al mínimo que al máximo (medido: 0,63 contra 0,68).
-   Consecuencia para la textura: **ningún otro elemento del casco puede
    tener el tono de las luces**. Un casco azul-gris con luces cian
    enciende el casco entero (la estación: 92 % de la textura dominaba en
    azul). Casco gris neutro, luces de un solo tono saturado.
-   Colores admitidos: primarios `r/g/b`, secundarios `c/m/y`, luces
    pálidas por luminancia `l`, y dos tonos a la vez (`c+m`, máximo de
    las dos máscaras). La **ganancia** del normalizador es la palanca de
    intensidad de color (el Skarnox va a 2,0); no se toca el pulso para
    compensar una textura floja.
-   **Luces manchadas** (el generador pinta sombra, polvo o metal dentro
    del parche): no se retexturiza. `LIMPIAR=N` limpia la máscara
    (cierre + apertura morfológica, N px a 1024) y `PLANO=1` hace la
    emisión de **un solo color uniforme**. La textura dice dónde hay
    energía; el motor dice cómo se ve. Receta para un pulido de Tripo:
    `TUMBAR=0 UMBRAL=0.25 LIMPIAR=3 PLANO=1`.

------------------------------------------------------------------------

## 10. Exportación

Formato preferido:

``` text
GLB / glTF 2.0
```

Mantener nombres consistentes y una estructura predecible de assets.

Ejemplo:

``` text
npc_streuner_01.glb
boss_streuner_01.glb
ship_player_01.glb
```

> En los repos el nombre es el **código de especie** (`aci-01.glb`,
> `vexor.glb`), el mismo que lleva `npc_catalog` en la base y
> `data/npcs/<code>.json` en el cliente. Un nombre, tres sitios.

------------------------------------------------------------------------

## 11. Importación y presentación en Godot

Godot es responsable de la presentación final.

Revisar/configurar según el asset:

-   LODs;
-   mipmaps;
-   compresión de texturas;
-   materiales PBR;
-   Metallic/Roughness;
-   Normal Maps;
-   Emission;
-   glow/bloom del entorno;
-   sombras;
-   culling;
-   distancia de cámara;
-   iluminación real del mapa.

No hornear en la textura efectos que deberían pertenecer al engine,
especialmente bloom/glow exagerado.

### Lo que ya está fijado en el proyecto

-   `generate_lods` y `create_shadow_meshes` activos en cada GLB. No hay
    LOD manuales ni `ship_lod1.glb`.
-   Godot **extrae las texturas** del GLB a `assets/npcs/<code>_<papel>.png`
    al importar: por eso los nombres limpios importan.
-   Reimportar tras copiar un GLB (`godot --headless --path . --import`):
    sin eso Godot sirve el recurso viejo de la caché.
-   El material y la luz del mundo viven en **un solo dial**
    (`lighting.json`, `AssetDefs`). Nunca se monta un `Environment` ni una
    luz a mano para un asset.
-   La emisión late con el **pulso** por especie (`pulse` en el JSON:
    modo `wave`, mínimo, máximo, `sharpness`), el reloj lo pone
    `wings.cycle`, y el glow del entorno (`threshold` 0,9, `bloom` 0,25)
    hace el resto.

------------------------------------------------------------------------

## 12. Validación en gameplay

**El visor 3D no es la prueba final.**

El asset debe evaluarse dentro de Godot utilizando:

-   cámara real;
-   zoom real;
-   iluminación real;
-   fondo espacial real;
-   escala real;
-   movimiento real;
-   cantidad aproximada de NPCs esperada.

Preguntas importantes:

1.  ¿La silueta se entiende inmediatamente?
2.  ¿El NPC se distingue de otros NPCs?
3.  ¿Las emisivas siguen siendo legibles?
4.  ¿Los materiales mantienen contraste?
5.  ¿El detalle importante sobrevive a la distancia?
6.  ¿Existe una diferencia perceptible entre esta versión y otra con más
    tris?

Si aumentar de 5k a 20k tris no produce una mejora visible durante
gameplay, utilizar la versión de 5k.

> Herramientas: el **bestiario** (`dev-run.ps1 -Bestiario`) saca un
> retrato por especie con la cámara, la luz y el fondo reales, y comprueba
> movimiento y cambio de calidad; el **Taller de assets** (F8 en el
> cliente de desarrollo) edita el JSON de la especie en vivo y simula
> reposo, patrulla, ataque, bajo ataque y huida; y `tests/bench_3d`
> mide fps con N instancias del modelo.

------------------------------------------------------------------------

## 13. LODs

El modelo base tampoco debe considerarse el único nivel necesario.

Ejemplo conceptual para un asset hero:

``` text
LOD0     12,000 tris
LOD1      6,000 tris
LOD2      2,500 tris
LOD3        800 tris
```

Los valores concretos deben ajustarse según tamaño en pantalla y
profiling.

No perseguir una reducción porcentual arbitraria si destruye la silueta.

> En Astrion los genera Godot al importar y los elige por tamaño en
> pantalla. Con cámara cenital todos los NPC visibles comparten nivel; el
> LOD trabaja sobre todo con el **zoom**. No fabricar niveles a mano.

------------------------------------------------------------------------

## 14. Anti-pipeline: lo que NO debemos volver a hacer como proceso estándar

Evitar:

``` text
Concept extremadamente detallado
        ↓
Meshy / generador
        ↓
2–3 millones de caras
        ↓
Decimation a 5k
        ↓
Deformaciones
        ↓
Bake contra high-poly
        ↓
Intentar recuperar lo perdido
        ↓
Texturizar
```

Este enfoque puede servir en workflows tradicionales cuidadosamente
controlados, pero no debe ser el pipeline automático principal de
Astrion si la reducción destruye la geometría generada por IA.

### Matiz medido

La ruta larga desde Meshy (alto de 2–3 M → `decimar-y-vestir.py` a 100 k
→ normales horneadas del alto) **no destruye la geometría**: medido en el
ACI-03, el Decimate desde el alto conserva juntas, filos y púas que el
remesh de Meshy derrite con los mismos triángulos. Lo que destruía era el
**remesh de Meshy usado como malla** (4–15 k quads). Esa ruta sigue
disponible y documentada para cuando la fuente sea Meshy; pero es más
larga, pesa diez veces más y depende de dos exportaciones. **La ruta
estándar es la corta con Tripo (§15).**

------------------------------------------------------------------------

## 15. Pipeline estándar definitivo

``` text
             CONCEPT
                │
                ├─────────────────────────┐
                │                         │
                ▼                         │
       GEOMETRY REFERENCE                 │
  simple / limpia / game-ready            │
                │                         │
                ▼                         │
              TRIPO                       │
                │                         │
                ▼                         │
       TARGET POLY BUDGET                 │
                │                         │
                ▼                         │
        GEOMETRY REVIEW                   │
                │                         │
          ¿Aprobada? ── No ──→ simplificar referencia
                │
               Sí
                │
                ▼
        TEXTURE REFERENCE
    rica / realista / PBR-friendly
                │
                ▼
           TEXTURIZADO
                │
                ▼
   BaseColor / Normal / M-R / AO
        + Emission Mask
                │
                ▼
              GLB  ──→  source/3d-models/pulido/<code>.glb
                │
                ▼
      CADENA AUTOMÁTICA (§17)
   medir canal → normalize → [rig] → validar
                │
                ▼
     CLIENTE: assets/npcs/<code>.glb + data/npcs/<code>.json
     BACKEND: migración npc_catalog + map_npc_spawn
                │
                ▼
     VALIDACIÓN EN GAMEPLAY (bestiario, Taller)
                │
                ▼
            PROFILING (bench_3d)
                │
                ▼
     COMMITS (cliente, arte + README, skill)
```

------------------------------------------------------------------------

## 16. Reglas para Claude al ayudar con este pipeline

Cuando se le pida a Claude automatizar, revisar o modificar un asset de
Astrion, debe respetar estas prioridades:

1.  **No aumentar complejidad sin una razón visual comprobable.**
2.  **No asumir que más polígonos equivalen a más calidad.**
3.  Preservar primero silueta y grandes volúmenes.
4.  No convertir microdetalle superficial en geometría.
5.  No aplicar decimation extrema automáticamente.
6.  Antes de modificar geometría, identificar el problema concreto que
    se intenta resolver.
7.  No utilizar el high-poly de millones de caras como requisito del
    pipeline.
8.  Priorizar soluciones reproducibles y automatizables; el usuario no
    depende de modelado manual avanzado en Blender.
9.  Evaluar los assets según su apariencia en la cámara top-down real de
    Godot.
10. Antes de optimizar, medir/profilar.
11. Si una versión de menor complejidad es visualmente equivalente en
    gameplay, elegirla.
12. Mantener separadas geometría, detalle de textura y efectos del
    engine.
13. **Medir el canal de emisión antes de normalizar**, y decidir canal y
    ganancia por la distribución (p50/p99), no por la cobertura sola.
14. **No «arreglar» un asset para callar al validador**: si rechaza, o el
    asset está mal o el contrato del validador está desfasado, y se
    corrige el que toque.
15. **Todo dial calibrable se documenta en el README de su repo en el
    mismo commit**, y la orientación se decide con dos retratos del
    bestiario, no con renders de Blender.
16. Los scripts de Blender corren headless: fallar, no avisar. Un aviso al
    lado de un «OK» se lee como OK.

------------------------------------------------------------------------

## 17. Lo que hace la cadena automática después del GLB

Todo lo de esta sección lo corre Claude (o cualquiera) con comandos;
nada exige tocar Blender a mano. Rutas relativas a
`C:\Source\MexOrbit\mex-orbit-v1\`.

### 17.0 Dónde va cada archivo

| Archivo | Ruta | ¿Git? |
|---|---|---|
| Pulido de Tripo (malla + textura) | `mex-orbit-art/source/3d-models/pulido/<code>.glb` | no |
| Alto de Meshy (solo si la fuente es Meshy) | `mex-orbit-art/source/3d-models/crudo/alto/<code>.glb` | no |
| Remesh texturizado de Meshy (ídem) | `mex-orbit-art/source/3d-models/crudo/<code>.glb` | no |
| **Master normalizado** (lo que sale de la cadena) | `mex-orbit-art/source/3d-models/<code>.glb` | **sí** |
| Asset del juego | `mex-orbit-client/assets/npcs/<code>.glb` (+ texturas extraídas) | sí |
| Diales de la especie | `mex-orbit-client/data/npcs/<code>.json` | sí |

La ranura del master la escribe la cadena; no dejar ahí una fuente.

### 17.1 Medir el canal de emisión

Un minuto de numpy sobre el color base: cobertura de la máscara del color
de las luces (`>0,2`), p99, y albedo medio. Criterio: **≥ 3 % con p99 >
0,5**. Si no llega, la textura no separa las luces del casco y hay que
retexturizar; no se compensa con diales aguas abajo.

### 17.2 Normalizar → master

``` bash
UMBRAL=0.25 [TUMBAR=0] [APAGAR=1] blender --background --factory-startup \
    --python tools/normalize-model.py -- \
    source/3d-models/pulido/<code>.glb source/3d-models/<code>.glb 1024 <canal> <ganancia> 0.0005
```

Hace, en orden: une piezas, aplica transformaciones a la malla, **tumba**
(el eje fino pasa a ser el alto, salvo `TUMBAR=0`: esferas y props de
pie), centra el pivote, escala a 1,9 de lado, **suelda** costuras
(0,0005), baja texturas a **1024**, renombra imágenes por papel
(`base_color`, `metallic_roughness`, `normal`) y el material generado a
`mat`, extrae la **emisiva** por canal con la ganancia, la **limpia**
(`LIMPIAR`), la **aplana** a un color (`PLANO`) y apaga el albedo donde
emite. Sale un GLB con Principled + emisión, con tangentes.

Si la fuente es Meshy, antes van `decimar-y-vestir.py` (alto + remesh →
malla de 100 k vestida) y `hornear-normales.py` (relieve del alto). Ver
README de arte, «La cadena desde el alto».

### 17.3 Rig (solo si el bicho se articula)

``` bash
blender --background --factory-startup --python tools/riguear-modelo.py -- \
    source/3d-models/<code>.glb <cliente>/assets/npcs/<code>.glb BISAGRA BANDA COLA_DESDE COLA_SEG CUERNO_DESDE CUERNO_BANDA
```

Alas (bisagra por perfil de la malla, no copiada de otro bicho), cola por
segmentos, cuernos/pinzas (`GIRO_Z`, `CUERNO_X_MIN`), o anillo **radial**
de tentáculos (`RADIAL=N`, ángulos medidos, rampa de altura para los
colgantes). Un bicho sin partes móviles se salta esto: el master se copia
tal cual. Las naves llevan además `marcar-anclajes.py` (toberas y cañones
medidos sobre el render, no sobre la malla).

### 17.4 Validar

``` bash
py -3 mex-orbit-testing/assets/validar-modelo.py <cliente>/assets/npcs/<code>.glb
```

Python puro, segundos. Piezas, triángulos (tope 120 000), texturas (tope
1024), caja plana o casi esférica, pivote, emisión declarada, desbalance
de luz cocida (aviso > 0,22), esqueleto y marcadores. Todo `RECHAZAR`
manda; se corre después de **cada** paso, no al final.

### 17.5 Enchufar al cliente

1.  Copiar el GLB a `assets/npcs/<code>.glb` y reimportar
    (`godot --headless --path . --import`).
2.  `data/npcs/<code>.json`: `code`, `screen_size` (124–248 px según el
    rango), `click_radius`, `turn` (`steps` 0 = giro continuo,
    `deg_per_sec`), `pulse` (`wave`, mínimo ~0,05 para apagar casi del
    todo, máximo ~3 para disparar el glow, `sharpness`), `wings.cycle` (el
    reloj del pulso aunque no haya alas), `orientation` (`yaw/pitch/roll`
    en grados: proa a −Z; el ojo hacia la cámara es +Z), `model`, y un
    `_comentario` por bloque explicando el porqué. Opcionales: `arms`,
    `horns_deg`, `lava`, `luz`, `heading_frozen`.
3.  Backend: migración fechada en `mex-orbit-data-base/migrations/` con
    `npc_catalog` (código, vida, escudo, daño, velocidad, botín) y
    `map_npc_spawn` (cuántos y en qué mapa), aplicada con
    `tools/migrate.ps1`, y **reiniciar el game server** (carga el catálogo
    al arrancar). Se hace desde PowerShell (`-Detener` y
    `-SoloServicios`), nunca desde una tubería de Bash.
4.  Autotest: añadir el código al `bestiary` de `data/config/autotest.json`.

### 17.6 Ver, calibrar, medir

-   **Bestiario** (`tools/dev-run.ps1 -Bestiario`, cuenta TestBot): un
    retrato por especie en `C:/Tools/autotest-<code>.png`, movimiento y
    cambio de calidad en caliente. Para la **orientación**, dos retratos
    con yaw/pitch distintos y el dummy congelado (`heading_frozen`) son el
    árbitro; los renders de seis caras de Blender fallan en los polos.
-   **Taller de assets** (F8, solo en build de desarrollo): edita el JSON
    en vivo, guarda, y simula reposo / patrulla / ataque / bajo ataque /
    huida con un dummy real.
-   **Banco** (`godot --path . res://tests/bench_3d.tscn -- --model=... --n=30`)
    cuando se cambie de presupuesto: fps media y 1 % peor.
-   El **autotest completo** (`-Autotest`) es el gate antes del commit;
    al iterar basta el bestiario.

### 17.7 Cerrar

Commits en español, uno por repo, por lista explícita de archivos (nunca
`git add -A`): cliente (GLB, texturas extraídas, JSON), arte (master, fila
del **catálogo** en el README con canal/ganancia/tris/diales/orientación),
base de datos (migración), y skill si cambió el procedimiento. Relanzar
el cliente con `tools/dev-run.ps1`.

------------------------------------------------------------------------

## 18. Qué textura favorece al mundo de Astrion

Todo esto sale de cómo está montado el mundo hoy (`lighting.json`,
`camera.json`, `entity.json`), no de gusto. Si cambian esos diales,
cambia esta sección.

### 18.1 El mundo, en números

| Cosa | Valor | Lo que implica para la textura |
|---|---|---|
| Luz del mundo | **una** direccional blanca, energía 1, especular 0,7, desde tilt 100 / pan 35 (arriba, algo a la izquierda y del lado del espectador) | El modelo **gira** y la luz no: nada de sombras ni luz cocida direccional en el albedo. El validador avisa si el gradiente global pasa de 0,22. |
| Ambiente | rosado `FFA5AE` a 0,2, plano (sin cielo para el difuso) | Las sombras propias caen a un rosa oscuro, no a negro. Un casco muy oscuro pierde forma; uno muy claro se lava. |
| Cielo de reflexión | gradiente oscuro (azul frío arriba, cálido en el horizonte, negro abajo), energía 0,6 | Un **metal puro se lee como cromo oscuro**, no como espejo brillante. El metal al 100 % sin variación sale negro. |
| Material con mapas | rugosidad × 0,9, metal × 0,6, rim 0,3 | El mapa de rugosidad es lo que más se nota bajo una luz única. |
| Material sin mapas | rugosidad 0,35, metal 0, rim 0,3 | Satinado, «pintura»: el detalle tiene que ir en el color base y la geometría. |
| Cámara | FOV 30, **45°** de elevación a zoom 1 y **25°** a zoom 3, mirando desde el sur | Se ve la **mitad superior** y la cara sur. El vientre no existe. Al acercar aparecen los laterales. |
| Tamaño en pantalla | 124–248 px por bicho (`screen_size`) | Un detalle menor de 1/60 del ancho (2–3 px) desaparece. |
| Textura en el juego | **1024** por mapa, venga de donde venga | Lo que en 4096 es un rasguño, en 1024 es ruido. 2048 de fuente basta. |
| Emisión | máscara por canal del color base, albedo apagado, pulso 0,05–3,0, glow a partir de 0,9 | Las luces son parches **planos** de un solo tono; el brillo lo pone el motor. |
| Luz del héroe | punto azul `2E7DFF` a 0,6 sobre la nave propia | La nave propia recibe un tinte azul: su casco no debe depender de un tono azul para leerse. |
| Fondo del 1-1 | planeta y nebulosa **cálidos** (naranja, marrón), estrellas | Un casco **frío** (gris neutro a gris azulado) contrasta; un casco naranja o marrón se funde con el planeta. |

### 18.2 Color base

-   **Albedo medio entre 0,25 y 0,40**, dominado por gris neutro o gris
    frío. Los que funcionan hoy: ACI-03 (0,29), ACI-01 Tripo (0,34). Por
    debajo de 0,2 el bicho es un recorte negro con borde; por encima de
    0,45 se lava con el sol blanco y el rim, y las luces pierden
    contraste.
-   **Contraste por placas**, no por textura: placas claras contra placas
    oscuras, en bloques del tamaño de 1/8 del modelo o más. Es lo único
    que sobrevive a 150 px.
-   **Nada del tono de las luces en el casco** (§9). Acentos secundarios,
    si los hay, en otro tono que la máscara ignore (naranja de aviso con
    luces cian, por ejemplo).
-   Desgaste con criterio: óxido y suciedad **localizados y de baja
    frecuencia** (manchas grandes, no salpicaduras). En 1024 las
    salpicaduras finas se vuelven ruido gris.
-   **Sin luz cocida direccional ni AO fuerte**: el modelo gira 360° bajo
    una luz fija. Un AO suave y neutro en las juntas sí ayuda.

### 18.3 Rugosidad y metal

-   Bajo una única luz blanca con especular 0,7, **el mapa de rugosidad es
    el que vende el material**: placas satinadas contra franjas mate se
    leen a cualquier tamaño; los microrasguños no.
-   Rango útil de rugosidad en la fuente: **0,4–0,7**. El cliente lo
    multiplica por 0,9. Por debajo de 0,3 es espejo negro; por encima de
    0,8 es tiza.
-   Metal: **0,3–0,7** y con variación. Un casco todo a 1,0 refleja el
    cielo oscuro y se apaga. Pintado sobre metal (metal bajo en la
    pintura, alto en los bordes desgastados) es lo que mejor se lee.
-   Si el generador no da mapas, no inventarlos con ruido: el cliente ya
    pone un satinado correcto. Es mejor sin mapa que con un mapa plano.

### 18.4 Emisivas

-   Un **solo tono saturado por especie**, como parche **plano** y sin
    degradado ni halo (el halo lo pinta el glow). Cian de referencia:
    `min(g, b) − r` > 0,5 en el píxel encendido.
-   **Cobertura entre 3 % y 8 %** de la textura. Menos no se separa; más
    convierte al bicho en una lámpara y satura el glow.
-   Grosor mínimo de una franja: **2 % del ancho del modelo** (3 px a
    150 px). Núcleos y ojos: 8 % o más.
-   Donde emite, el albedo lo apaga la cadena: no hace falta pintarlo
    oscuro, pero sí evitar que ese parche lleve textura encima (rasguños
    o suciedad dentro de una luz salen como manchas parpadeando).
-   Las luces **arriba y al frente**: la cámara ve la mitad superior y la
    cara sur. Una franja en el vientre no existe.
-   Emisión que **viaja** (lava, energía por vetas): existe el dial `lava`
    en el JSON; la textura solo tiene que marcar el recorrido.

### 18.5 Orientación y lo que se ve

-   **Proa a −Z** del GLB; la cámara mira desde +Z. Un «ojo hacia la
    cámara» es un ojo en +Z. Si la fuente no lo deja claro, se corrige con
    `orientation` en el JSON, pero es mejor que la Geometry Reference lo
    resuelva.
-   La cara superior es el **póster** del bicho: ahí va la identidad
    (placas, luces, ojo). Los laterales aparecen al acercar el zoom (25°):
    tienen que tener continuidad de color, no detalle.
-   Simetría bilateral facilita todo: tumbado automático, rig por
    bisagra, y lectura de la proa.

### 18.6 Normal map

-   Con luz fija y modelo girando, un normal map **sí se lee** (ACI-03 con
    normales del alto: juntas y biseles vivos a 150 px). Si el generador
    lo entrega, se usa; la cadena lo respeta y le exporta tangentes.
-   Si no lo entrega (Tripo hoy), las **juntas van en la geometría**, como
    hace Tripo, y el resto en el color base. No hornearlo desde un alto
    que no existe.
-   Lo que un normal map no hace: silueta. Una placa que sobresale es
    geometría.

### 18.7 Resolución y peso

-   1024 en el juego = **12 MB por especie** (color, MR, normal, emisiva).
    Ese es el coste real de un bicho, no sus triángulos.
-   Fuente a 2048 basta; 4096 u 8192 solo mejoran el reescalado un poco y
    pesan diez veces más en `pulido/`.
-   JPG en la fuente es aceptable para el color base; para normal y MR,
    PNG (los artefactos de JPG en un normal map salen como cuadrícula).

------------------------------------------------------------------------

## 19. Medidas que sostienen este documento (3-sep-2026)

| Medida | Resultado | Dónde |
|---|---|---|
| 15 bichos a 58 106 tris vs 9 884 | 102 vs 103,5 fps | README arte, «presupuesto con LOD» |
| 30 bichos a 150 000 / 100 000 / 56 265 tris | 81,9 / 81,9 / 81,2 fps | README arte, «cadena desde el alto» |
| Decimate del alto vs remesh de Meshy, mismos tris | Decimate conserva juntas y púas; remesh derretido y con agujero | `escalera-aci-03.png` |
| Textura traspasada del remesh vs alto texturizado | traspaso limpio; alto texturizado con franjas dentadas | `traspaso-aci-03.png` |
| Tripo 13 686 tris vs Meshy 20 846 + normales | juntas más limpias, sin MR ni normales, albedo 0,34 vs 0,29 | `tripo-vs-actual-aci-01.png` |
| Metal Meshy 0,6–0,9 sin cielo | negro | `metallic_scale` 0,6 + cielo de reflexión |
| Pulso sobre albedo saturado | invisible (0,63 vs 0,68) | `APAGAR` |
| Estación con casco azul-gris y luces cian | 92 % de la textura «emitía» | canal `c+m`, casco neutro |
| Esfera 0,962 × 0,996 × 1,000 con tumbado automático | ojo al oeste, escondido | `TUMBAR=0` |

------------------------------------------------------------------------

## Regla maestra

> **No modelar lo que el jugador no puede percibir.**
>
> Usar geometría para silueta y volumen, textura para riqueza
> superficial y Godot para iluminación y efectos.

El objetivo no es producir el modelo más detallado posible. El objetivo
es producir **el asset más barato que entregue exactamente la calidad
visual que Astrion necesita en gameplay**.
