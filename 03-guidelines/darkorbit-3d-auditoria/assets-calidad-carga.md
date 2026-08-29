# Auditoría: pipeline de assets, calidad y carga vs. guidelines DarkOrbit 3D

**Ítems del checklist §14 cubiertos**: tiers de calidad, auto-quality, carga asíncrona/caché, resolución por categoría, glow por HP, presupuesto de luces, materiales/pooling, robustez, telemetría.

**Alcance real**: el encargo apunta a `C:\Source\MexOrbit\MexOrbit.GodotClient\` (el **prototipo**, réplica del cliente 2D Flash). Pero el pipeline de tres calidades de la skill `mexorbit-asset-3d` (alta = malla en SubViewport, media/baja = PNG horneados) **no vive ahí**: vive en el **cliente v1** (`C:\Source\MexOrbit\mex-orbit-v1\mex-orbit-client\`). El prototipo no tiene ni SubViewport, ni `load_threaded_request`, ni entity_node — su `tools/` son extractores de assets del SWF (no existen `normalize-model.py` ni `hornear-sprite.py`, que viven en `mex-orbit-art/tools/`). Se auditan **ambos**, marcando en cada ítem cuál es cuál. El veredicto de cabecera es del cliente v1, que es donde se juega el futuro.

Referencia: `mex-orbit-docs\03-guidelines\darkorbit-3d-guidelines.md` §7/§8/§12 y sus informes `pipeline-assets.md` y `calidad-rendimiento.md`.

---

## Ítem 1 — Tiers de calidad con booleanos derivados, señal `changed`, aplicables en vivo, gates por sitio

**Veredicto: ✅** (con un matiz menor)

| | DarkOrbit | v1 (`mex-orbit-client`) | Prototipo (`MexOrbit.GodotClient`) |
|---|---|---|---|
| Modelo | tier entero LOW/MED/HIGH/ULTRA + booleanos derivados `effects.medium/high/max` precalculados | niveles 0–2 **por subsistema** (7 claves: npc, shader, emissive, engine, collectable, background, explosion) | niveles 0–3 por subsistema (8 claves del QualitySettings del server) |
| Señal de cambio | señal `changed` bindable | **`signal cambiada(claves)`** — `game/quality.gd:15` | sin señal: `Quality.apply()` devuelve las claves cambiadas — `game/quality.gd:31-37` |
| Aplicación en vivo | materiales/luces/AA se re-aplican por señal | **sí**: `world.gd:566-586` (`_on_calidad_cambiada`) rehace entidades (`reconstruir()`), portales, cajas y fondo sin reiniciar; `entity_node.gd:610-632` reconstruye solo la parte visual | **sí**: `world.gd:2043-2046` reconstruye el fondo; el resto de gates se consulta al dibujar/crear |
| Gates por sitio | cada efecto compara su booleano localmente | **sí**: `Quality.nivel("engine") >= 1` en `entity_node.gd:282,503`, `nivel("emissive")` en 265/343, `nivel("shader")` en 678/695, `nivel("npc")` en 219/225 | **sí**: 18 sitios (`world.gd:1289,1750,1766,1895,1943…`, `ship_sprite_2d.gd:211,372`, `box_sprite_2d.gd:44`, `map_background.gd:57,65,122`, `projectile_2d.gd:244`, `entity.gd:1091`) |
| Persistencia | SharedObject | `user://quality.cfg` **por cuenta** — `quality.gd:58-66` | server (`player_settings.quality`, request 20038) + preferencias locales |

Lo que recorta cada tier del v1 está documentado en el propio `quality.gd:19-30` y en el README (sección «Calidad gráfica: niveles por subsistema»): baja = PNG fijo sin shaders/emisivas/llamas/explosiones; media = arte fijo + shaders + emisivas (los shaders se conservan a propósito: casi gratis y dan vida a los bichos sin video); alta = atlas animados/malla 3D + fondo completo. El corte caro (alta→media) libera ~58 MB de VRAM (comentario en `quality.gd:33`).

**Matiz (no bloqueante)**: no hay booleanos precalculados — cada gate hace `Dictionary.get` + comparación. En DO importaba porque los checks corrían por frame en AS3; aquí casi todos corren al construir/reconstruir. El único check por-tick es `Quality.level("engine")` en `ship_sprite_2d.gd:211` del prototipo (20 Hz por nave): irrelevante en coste. No hace falta corregir.

---

## Ítem 2 — Auto-quality por FPS con histéresis

**Veredicto: ❌ inexistente en ambos clientes**

| Constante DO (Profiler) | Valor DO | MexOrbit |
|---|---|---|
| Ventana de promedio | 20 s (muestreo 1/s) | — |
| Umbral de bajada | < 10 fps promedio → +1 recorte | — |
| Umbral de subida | > 60 fps → −1 recorte (histéresis) | — |
| Escalera | 5 recortes ordenados por costo/valor (preview portales → humo → map-assets animados → explosión low → sin explosiones/starfield) | — |
| Sin foco | se desengancha en DEACTIVATE (no mide FPS falsos) | — |
| Aviso al usuario | "low performance" tras N ventanas, máx 3 veces | — |

Grep exhaustivo (`Profiler`, `TIME_FPS`, `auto`, `reduction`) no encuentra nada en ninguno de los dos clientes. El único consumo de FPS es de display (ítem 9).

**Corrección concreta (v1)**: en el autoload `Quality`, un Timer de 1 s que acumule `Performance.get_monitor(Performance.TIME_FPS)` en ventana de 20 muestras; si promedio < umbral_bajo y `preset != "baja"`, `aplicar()` el preset inferior (o mejor: una escalera de claves individuales — la infraestructura de niveles por subsistema ya lo permite sin tocar consumidores); si promedio > umbral_alto durante una ventana, recuperar. **No medir sin foco**: pausar el timer en `NOTIFICATION_APPLICATION_FOCUS_OUT`. Ojo con la interacción con la persistencia por cuenta: la reducción automática NO debe pisar el preset guardado (en DO `autoQualityReduction` es una variable aparte del setting, y al desactivar auto-quality vuelve a 0 — replicar eso). Nota de los propios docs del repo (`pruebas/README.md:130`): `get_frames_per_second()` es media móvil; para los umbrales vale (DO también promedia), pero no usarla para diagnosticar picos.

---

## Ítem 3 — Carga asíncrona con placeholder + caché con lifetime + refcount con gracia

**Veredicto: ❌ carga síncrona bloqueante; sin lifetimes ni refcount** (mitigado porque los assets son locales)

| | DarkOrbit | MexOrbit (ambos clientes) |
|---|---|---|
| Descarga/parseo | red + AWD parseado con presupuesto **30 ms/frame**; subida ATF **asíncrona** (TEXTURE_READY) | `load()` **síncrono en el hilo principal**, en el momento del spawn. Cero usos de `ResourceLoader.load_threaded_request` en todo el árbol (grep) |
| Placeholder | plano con el sprite 2D si el AWD no llegó en **1000 ms** (2500 ms nave propia) | v1: el PNG fijo es *fallback de error* (`entity_node.gd:289-293`: si el GLB no carga → sprite), no placeholder temporal. Prototipo: `setup()` devuelve false si no hay assets y el llamador conserva su placeholder (réplica del replacementShips) — mismo carácter: fallback, no async |
| Caché | promise por resKey (una descarga en vuelo) | caché de Godot + dicts estáticos de referencia fuerte (`MapAssets._textures` — `map_assets.gd:308`, `ShipAssets._textures` — `ship_assets.gd:16`, `AssetDefs._cache` — `asset_defs.gd:149`) |
| Lifetimes | 0 transitorio / 1 mapa / 2 permanente; `reset(1)` al cambiar de mapa | **no existen**: todo lo cacheado es de facto permanente; nada se purga al cambiar de mapa |
| Refcount | con **gracia de 5 s** al llegar a 0 | no existe |

**Contexto que absuelve parcialmente**: DO streameaba 2854 ATF por HTTP; MexOrbit empaqueta todo en `res://`. Un `load()` local de un PNG importado es rápido, y la primera carga de cada especie ocurre una vez (la caché de Godot retiene mientras alguien referencie). Pero el camino 3D del v1 **sí paga hitch real**: `load(GLB)` + `instantiate()` + montar SubViewport+Environment por entidad en `setup()`, todo síncrono en el frame en que llega el spawn (`entity_node.gd:290-314`). Con varios spawns simultáneos al entrar a un mapa, eso es un tirón medible.

**Correcciones concretas** (en orden de valor):
1. `ResourceLoader.load_threaded_request` para el GLB en `_construir_malla_3d`, mostrando **el PNG fijo de media mientras llega** — MexOrbit tiene gratis lo que DO tuvo que inventar (el plano con sprite 2D): media y baja son *por diseño* horneados del mismo modelo, o sea el placeholder perfecto. Al completarse, swap a la textura del viewport.
2. La única presión de VRAM real (los atlas de video, ~58 MB) ya tiene válvula (cambiar a media); un `reset` de mapa como el de DO solo tendría sentido si los catálogos crecen mucho. Anotar como deuda, no como urgencia.
3. El refcount con gracia de 5 s solo importa si se implementa (2); entonces sí: no liberar el GLB de una especie hasta 5 s después de que muera su último ejemplar (los respawns de NPC son exactamente el thrashing que la gracia evita).

---

## Ítem 4 — Resolución por categoría × calidad × zoom; elección entre las 3 calidades

**Veredicto: ⚠️ la selección de calidad es limpia; falta la dimensión zoom y los topes por categoría**

**Cómo elige el v1 entre las 3 calidades** (`entity_node.gd:207-260`, `_construir_visual`):
```
Quality.nivel("npc") >= 2  y el JSON trae `modelo`  → malla 3D en SubViewport (alta)
Quality.nivel("npc") >= 2  sin modelo               → atlas de video animado (alta)
Quality.nivel("npc") <  2                           → PNG fijo (media y baja; media conserva
                                                      shaders/emisiva, baja los apaga por sus claves)
```
Reconstruible en caliente (`reconstruir()`); el fallback si el GLB no carga cae solo al camino 2D. Es fiel al espíritu del pipeline de la skill.

**Resolución del render 3D**: `lado = screen_size × MARGEN(1.15) × VIEWPORT_FACTOR(2)` (`entity_node.gd:83-87,295`). Como `screen_size` viene del JSON por asset, la resolución **sí escala por categoría implícitamente** (un bicho chico pide un viewport chico). Comparación con DO:

| Aspecto | DarkOrbit | v1 |
|---|---|---|
| Tabla categoría×tier | drone 64-128, ship 128-256, ship_big 256-512, building 256-1024; **un dron nunca >128 ni en ULTRA** | implícito vía `screen_size` por asset; sin tope declarado por categoría |
| Multiplicador de zoom | ×2 cuando zoom de cámara > 1.3, **en caliente** (señal → recarga de mapas) | ninguno: `VIEWPORT_FACTOR = 2` constante — se paga siempre el escalón alto |
| Zoom lejano | mipmapping solo en HIGH | mipmaps en el sprite estático (`entity_node.gd:255`, con la distinción atlas-sin-mipmaps bien razonada); el README documenta la medición (energía de alta frecuencia 15,54 → …) |
| normal/specular a mitad de resolución | sí (`size >> 1`) | n/a (el relieve 2D usa un solo mapa por asset) |

La decisión de DO era *pagar resolución solo cuando el zoom la enseña*; la del v1 es *pagar ×2 siempre porque el área es pequeña* (comentario en `entity_node.gd:80-83`: «a 2 aguanta el acercamiento sin costar cuatro veces, porque el área sigue siendo pequeña»). Es defendible y está medida en el banco — pero **no está medida con N viewports** (aviso literal de `pruebas/banco_3d`: sus cifras son N modelos en UN mundo; la técnica viewport-por-entidad «es más cara y sigue sin medirse»).

**Corrección concreta**: si al medir viewport-por-entidad con población real el fill rate duele, la palanca de DO es la buena: derivar `VIEWPORT_FACTOR` del zoom de cámara (1 en el rango normal, 2 cuando zoom > umbral), redimensionando `_vp.size` en el cambio de zoom (equivale al «sube un escalón en caliente» de DO). Y documentar en el README un tope de `screen_size×2` por categoría de bicho, para que el día que entre un jefe gigante nadie pida un destino de render de 30 MB (la lección de la estación ya está escrita en la skill).

---

## Ítem 5 — Glow emissivo por canal, factor animable por instancia, ligado a HP

**Veredicto: ⚠️ el mecanismo está completo; el eslabón HP no está cableado**

| Pieza DO | DO | v1 |
|---|---|---|
| Glow como 2ª textura sumada, no ambient | `GlowMapMethod`: `mul glow, factor; add output, glow` | emisión por canal horneada en el GLB (skill: máscara por color r/g/b/c/m/y, `GLOW_*`), `emission_texture` del material + `glow_enabled` del Environment (`entity_node.gd:343-347` — sin él la emisión se recorta a 1.0, medido: 0.291 vs 0.377) |
| Factor escalar **por instancia**, animable | `renderable.extra.glow` (default 1); UberAnimation3D lo tuena; **alpha del glow ~ %HP** (`§_-33D§`) | `mat.emission_energy_multiplier = e` por frame (`entity_node.gd:900-907`), materiales duplicados por entidad (`asset_defs.gd:345-365`); en 2D, `_emissive.self_modulate` (`entity_node.gd:854-858`); la lava (`lava_flujo.gdshader` como next_pass) late en fase |
| Ligado a HP | **sí** (glow alpha ~ %HP: la nave herida se apaga) | **no**: el pulso va al reloj de las alas (`_pulse_min/max` del JSON, fase áurea por entidad). `set_hp_pct` (`entity_node.gd:1167-1169`) solo redimensiona la barra. Grep de `hp` × emisión: cero cruces |

**¿La emisión por canal de la skill cubre esto?** Cubre el *mecanismo* al 100 % — el factor por instancia existe y ya se escribe cada frame, que era la parte cara. Lo que falta es una línea de política: multiplicar el rango del pulso por una función de `_hp_pct`.

**Corrección concreta** (`entity_node.gd::_process`, bloque de `_mats_3d`, y el bloque `_emissive` para el camino 2D): `e *= lerpf(0.25, 1.0, _hp_pct)` (o el mapeo que se calibre — DO usaba el alpha directamente proporcional). Dos avisos de calibración heredados de la skill: (1) es un dial sobre emisión de material, NO cruzar el valor desde el blend aditivo 2D sin medirlo (`medir_emision.tscn`); (2) media no puede reproducirlo (su halo va horneado) — decidir si media degrada solo la capa emisiva 2D o se acepta la divergencia, y documentarlo en el README en el mismo commit (convención del repo).

---

## Ítem 6 — Presupuesto de luces / distinción del héroe

**Veredicto: ✅ parcial — sol + luz del héroe portados con constantes exactas; el pool de 3 luces de efecto es ➖ (la arquitectura lo vuelve inaplicable)**

| Luz DO | Constantes DO | v1 |
|---|---|---|
| 1 DirectionalLight "sol" | 0xA3FFFF, diffuse 0.8 (defaults Settings3D), tilt 100 | `AssetDefs.sol_mundo()`: **0xA3FFFF, energía 0.8**, elevación −54° (= tilt 100), `shadow_enabled = false` (`asset_defs.gd:36-38,119-125`). Dial ÚNICO para juego+pruebas+horno (la lección de las ocho copias) |
| 1 PointLight del héroe | **0x2E7DFF, diffuse 0.6**, specular 1.5, fallOff 450 | `LUZ_HEROE_COLOR = Color("2e7dff")`, `LUZ_HEROE_ENERGIA = 0.6` (`asset_defs.gd:71-72`); OmniLight3D solo en el viewport del héroe, rango dimensionado del modelo (`entity_node.gd:367-377`). El fallOff 450 no porta a propósito (unidades de mundo vs de modelo) — decisión razonada y documentada |
| Pool de exactamente 3 PointLights de efectos | explosión 0xF7C0C0 fallOff 200, etc., tween de fade, solo HIGH | **no existe** — y no puede existir tal cual: cada entidad vive en un `own_world_3d` aislado, una luz de explosión no tiene mundo compartido que iluminar. El rol lo cumplen los FX 2D aditivos (llamas, impactos, beams con BLEND_ADD), que es la receta §9.5 de DO («el glow son texturas radiales sumadas») |
| Sombras | ninguna | ninguna (`shadow_enabled = false` en sol y héroe) ✅ |
| Sin foco por calidad | LOW sin luces, MEDIUM solo sol+1 | en baja/media no hay 3D, así que no hay luces: el gate es de más arriba (nivel npc) ✅ funcionalmente |

**¿La nave propia se distingue por luz?** Sí en alta (rim azul 0x2E7DFF sobre tu malla — «la distinción del héroe es luz, no UI», ley #12, con el aviso de arquitectura honesto en `asset_defs.gd:64-70`: no derrama sobre vecinos como en DO). En media/baja **no**: la distinción cae a color de nombre/mira (NTheme.CYAN, `entity_node.gd:639`, `_draw` 1256). Prototipo: sin luz alguna; héroe solo por UI. Si algún día importa en media: un tinte aditivo azul suave bajo el sprite del héroe sería el equivalente 2D barato.

---

## Ítem 7 — Materiales compartidos + pooling con warm-spare

**Veredicto: ❌ en las dos mitades, con atenuante documentado en la primera**

**Materiales**: DO comparte UN material entre todas las naves iguales (pool refcontado clave = shader+perfil+hash) y anima el glow con un factor *por renderable* sobre el material compartido. El v1 hace lo contrario **a propósito**: `AssetDefs.materiales_3d` duplica los materiales por instancia para poder pulsar `emission_energy_multiplier` por bicho (`asset_defs.gd:342-365`), y el comentario en `entity_node.gd:555-557` reconoce que «rompe el batching entre entidades» y que el banco lo midió con la copia puesta. Atenuante real: cada entidad rinde en su propio SubViewport, así que el batching entre entidades ya está roto por arquitectura — la copia de material añade memoria y cambios de estado, no draw calls cruzados.
**Corrección concreta**: si N sube, la vía Godot del «factor por renderable» de DO existe: `instance uniform float` en un ShaderMaterial compartido (per-instance uniforms), o `set_instance_shader_parameter`. Mantendría un material por especie con el pulso por instancia — exactamente el reparto de DO. A valorar solo con medición, porque convertir BaseMaterial3D→ShaderMaterial pierde comodidades.

**Pooling**: inexistente en ambos clientes (grep `pool`: cero). Comparación por consumidor:

| Consumidor | DO | MexOrbit |
|---|---|---|
| Salvas de láser / muzzle | pool genérico por clave con **warm-spare** (mantiene 1 tibio), wake/sleep | v1 `Projectile2D.fire`: `new` + `load()` por disparo (`projectile_2d.gd:18-33`); prototipo igual |
| Impactos | pool 2D precacheado por XML (`poolSize`) | `entity_node._sheet_anim` (`entity_node.gd:1226-1242`): construye **SpriteFrames + 8 AtlasTexture nuevos POR IMPACTO** (hasta 5 casco + 9 escudo simultáneos por entidad) |
| Humo | pool | prototipo: `SheetAnim2D.spawn` nuevo nodo cada 80 ms por nave a pleno empuje (`ship_sprite_2d.gd:220-231`) |
| Vaciado al cambiar de mapa | pools vaciados en reset | n/a (no hay pools); los nodos vivos sí se liberan (ítem 8) |

**Correcciones concretas**, en orden de rentabilidad: (1) cachear el `SpriteFrames` por hoja en un dict estático de `_sheet_anim` — es inmutable y hoy se reconstruye en cada impacto; una línea, elimina la asignación gorda del combate; (2) `load()` de la textura del beam fuera de `fire` (constante de clase); (3) pool por clave con warm-spare para proyectiles/impactos solo si el profiler lo pide — en Godot el coste de `new Sprite2D` es mucho menor que en Flash, así que medir antes (disciplina de la propia skill).

---

## Ítem 8 — Robustez: ¿la simulación sigue si el render falla? ¿el cambio de mapa purga?

**Veredicto: ✅** (con una nota sobre cachés)

- **Render que falla ≠ crash**: todos los caminos de asset ausente degradan con aviso, nunca tiran el cliente — GLB que no carga → sprite 2D (`entity_node.gd:289-293`), textura ausente del JSON → respaldo o null con `push_warning` (`entity_node.gd:745-753`), material de relieve sin mapa → null y sin shader (`asset_defs.gd:133-140`); en el prototipo, `MapAssets.load_texture` devuelve null y el llamador omite la capa (`map_assets.gd:312-313`), `setup()` false conserva el placeholder. La simulación es server-autoritativa (sombra + reconcile, `entity_node.gd:909-936,1149-1164`): nada del gameplay depende de que lo visual exista — la ley #1 de DO se cumple por construcción. El device-loss propiamente dicho lo gestiona Godot 4, no aplica réplica manual del patrón Stage3D.
- **Cambio de mapa**: v1 `_desmontar_mapa` (`world.gd:307-339`) libera entidades, cajas, portales, estación y fondo, y limpia el autopiloto (con el comentario del bug del portal viejo — el equivalente al anti-teleport de DO); solo se desmonta en salto real, no en reconexión (`world.gd:287-288`, correcto). Prototipo: `world.gd:786-792` libera `_entities_root` completo + limpia dicts en cada ship_init. 
- **Nota ⚠️**: las cachés estáticas de texturas del prototipo (`MapAssets._textures`, `ShipAssets._textures`) no se purgan nunca — todo mapa visitado queda en VRAM de por vida (lifetime «permanente» universal, sin el `reset(1)` de DO). En el prototipo, con su catálogo acotado, es tolerable; que no se herede al v1 si algún día cachea texturas por mapa.
- El autotest del v1 verifica el cambio de calidad **con el mundo poblado** (`world.gd:2259`: «si reconstruir rompiera algo, revienta aquí») — mejor red de seguridad que la que DO tenía.

---

## Ítem 9 — Telemetría / panel de rendimiento

**Veredicto: ✅ parcial**

| | DO | v1 | Prototipo |
|---|---|---|---|
| Panel | FPS/MS/MEM oculto (mrdoob-style) + triángulos/frame del collector | ventana de Ajustes muestra **FPS + memoria de textura** en vivo (`ui/settings_window.gd:126-131`: `Engine.get_frames_per_second()` + `Performance.RENDER_TEXTURE_MEM_USED`) — «los dos números que hacen falta para ELEGIR un preajuste»; el efecto de alta→media se ve en el propio ajuste | etiqueta FPS con atajo de teclado (acción 10 «Mostrar FPS», `world.gd:765-772, 2511-2512, 865-866`) |
| Frame time / picos | MS por frame | no — y sus propios docs lo saben (`pruebas/README.md:130`: la media móvil «esconde justo lo que interesa») | no |
| Telemetría al server | driver/profile/uso-2D/frameRate → decisiones de tiers con datos de población | no | no |

**Corrección concreta**: añadir frame time (`Performance.TIME_PROCESS` / `1000.0/fps` no vale — usar el delta máximo de la ventana) a la fila de la ventana de Ajustes; es una fila más en un panel que ya existe. La telemetría a servidor cobra sentido cuando exista auto-quality (ítem 2): son la misma decisión — los umbrales de la escalera se calibran con datos reales, que es como DO fijó sus tiers.

---

## Los 5 gaps más importantes

1. **Auto-quality por FPS: no existe** (ítem 2, ❌). Es la pieza entera del §12.2 de DO — ventana 20 s, histéresis 10/60, escalera de recortes, sin medir sin foco. La infraestructura del v1 (niveles por subsistema + `cambiada` + reconstrucción en caliente ya probada por autotest) deja el costo en solo la política. Cuidado con no pisar el preset persistido por cuenta.
2. **Carga síncrona del GLB al spawn, sin placeholder temporal** (ítem 3, ❌). `load()` + `instantiate()` + SubViewport en el frame del spawn. La corrección es barata y elegante: `load_threaded_request` + mostrar el PNG de media mientras llega — el pipeline de la skill ya fabrica el placeholder perfecto (mismo modelo horneado), que es exactamente el truco del plano-con-sprite de DO (§5.5).
3. **Cero pooling, y `_sheet_anim` reconstruye SpriteFrames+AtlasTextures en cada impacto** (ítem 7, ❌). Antes de montar el pool genérico de DO, la ganancia gorda es una caché estática del SpriteFrames por hoja (inmutable, hoy se rehace hasta 14 veces por entidad en combate).
4. **El glow no está ligado al HP** (ítem 5, ⚠️). El factor por instancia (`emission_energy_multiplier` escrito cada frame) ya existe — es el `renderable.extra.glow` de DO. Falta multiplicarlo por `_hp_pct`. Ojo a la homologación de media (su halo va horneado y no puede seguir al HP): decidir la degradación y documentarla en el README en el mismo commit.
5. **La técnica viewport-por-entidad sigue sin medirse con población real, y paga siempre el escalón alto** (ítems 4/7, ⚠️). `VIEWPORT_FACTOR=2` constante (DO subía resolución solo con zoom>1.3) + materiales duplicados por instancia (DO compartía con factor por renderable — el equivalente Godot son instance uniforms). El banco avisa en su propio README de que sus cifras no cubren esta técnica. Medir primero; las dos palancas de DO están identificadas para cuando el número mande.

## Tabla resumen

| # | Ítem | v1 | Prototipo |
|---|---|---|---|
| 1 | Tiers en vivo, gates por sitio | ✅ | ✅ |
| 2 | Auto-quality con histéresis | ❌ | ❌ |
| 3 | Carga async + placeholder + lifetimes | ❌ (síncrono; fallbacks sí) | ❌ (síncrono, catálogo local) |
| 4 | Resolución categoría×calidad×zoom | ⚠️ (calidad ✅, zoom ❌) | ➖ (sprites del SWF, resolución fija) |
| 5 | Glow por instancia ligado a HP | ⚠️ (mecanismo ✅, HP ❌) | ➖ (sin capa de glow) |
| 6 | Presupuesto de luces / héroe por luz | ✅ parcial (pool de efectos ➖ por arquitectura) | ➖ (2D sin luces; héroe por UI) |
| 7 | Materiales compartidos + pooling | ❌ (duplicado deliberado y medido; sin pools) | ❌ (sin pools) |
| 8 | Robustez render/purga de mapa | ✅ | ✅ (⚠️ cachés de textura nunca purgadas) |
| 9 | Telemetría/panel | ✅ parcial (FPS+VRAM en Ajustes) | ✅ parcial (FPS por atajo) |
