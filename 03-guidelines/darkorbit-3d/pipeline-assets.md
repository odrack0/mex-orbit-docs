# Pipeline de assets 3D del cliente DarkOrbit (spacemap) — ingeniería inversa

Fuentes: `D:\MexOrbit\Decompiled\spacemap\main\scripts\` (AS3 decompilado con JPEXS) y assets reales en
`C:\Source\MexOrbit\MexOrbit.CMS\public\spacemap\3d\`. Todas las rutas de evidencia son exactas (el carácter `§` es parte del nombre).

## 0. Mapa de clases ofuscadas → nombre deducido

| Ofuscado | Nombre deducido | Evidencia |
|---|---|---|
| `net\bigpoint\darkorbit\map\view3D\display3D\§_-o2x§` | **AssetsManager3D** (singleton) | ancla dada; contiene loaders + LightsManager |
| `§_-o2x§.§_-Y4h§=2 / §_-V4X§=1 / §_-7X§=0` | **LIFETIME_PERMANENT / LIFETIME_MAP / LIFETIME_TRANSIENT** | ver §5 (no son "prioridades de red", son vidas de caché) |
| `§_-o2x§.§_-J34§` | **LightsManager** (nombre real conservado) | getter en `§_-o2x§.as` L142-145 devuelve `LightsManager`. Su `update()` reconstruye la lista del light picker — **NO es cola de subida a GPU** |
| `§_-n3U§\GeometryLoader` / `TextureLoader` | nombres reales conservados | archivos homónimos |
| `§_-n3U§\§_-u1u§` | **ParticlesLoader** | método `§_-c2T§(url, lifetime)` carga `*.zip` |
| `§_-n3U§\§_-T2s§` | **LoaderBase** (caché + refcount) | dict `§_-p3s§` + `§_-N1§.§_-I4y§` |
| `§_-N1§\§_-I4y§` / `§_-f7§` | **UsageMonitor / UsageCounter** (refcount con periodo de gracia) | `§_-f7§.as`: `setTimeout(unused.dispatch, seconds*1000)` |
| `§_-n3U§\§_-Z39§` | **SingleFileLoader3D** (puente al AssetLoader de Away3D) | enruta `.atf`→TextureLoader, `.awd`→GeometryLoader |
| `§_-n3U§\§_-X4L§` | **AssetLoaderContext3D** (resolver de dependencias) | reescribe `.obj→.awd`, `.png→.atf` |
| `net.bigpoint.as3toolbox.filecollection.§_-o15§` | **FileCollection** (manifiesto XML → URL) | `resources_3d.xml` |
| `§_-x2v§\§_-n3T§` / `§_-72M§` | **FileVO / LocationVO** | `§_-jr§()` compone la URL |
| `§_-X2m§\§_-5B§` | **ATFTexture** (con upload asíncrono) | `uploadCompressedTextureFromByteArray(..., async)` |
| `§_-X2m§\§_-R4v§` | **ATFData** (parser de cabecera ATF) | lee magic "ATF", formato, tamaño, mips |
| `§_-X2m§\§_-64u§` | **ATFCubeTexture** | `createCubeTexture` + upload comprimido |
| `§_-X2m§\§_-y3T§` | **BitmapTexture** | fallback desde BitmapData |
| `§_-X2m§\§_-912§` | **BitmapCubeTexture** | AWD99Parser compone cubemap de 6 bitmaps |
| `§_-X2m§\§_-u2h§` / `§_-t2b§` / `§_-l1u§` | **Texture2DBase / TextureProxyBase / CubeTextureBase** | jerarquía Away3D renombrada |
| `§_-F1w§\§_-t1a§` | **CompositeAWDParser** | elige `AWD99Parser` o `AWD2Parser` por magic |
| `§_-F1w§\AWD99Parser` | nombre real; **variante AWD propietaria v99.1** | `supportsData`: bytes[3]==99 && bytes[4]==1 |
| `§_-F1w§\§_-74y§` | **ParserBase** (parseo incremental con presupuesto de 30 ms/frame) | `§_-528§(data, frameLimit=30)`, `getTimer()-start < limit` |
| `§_-jh§\§_-X34§` | **MaterialManager** (material compartido + refcount) | pool estático `§_-iO§` con `counter` |
| `§_-jh§\§_-LC§` | **MaterialDefinition** (VO desde XML del asset) | `§_-bL§(xml)` lee @geometry/@diffuse/@gloss... |
| `§_-jh§\§_-717§` | **PrefabView3D** | string de log literal `"- PrefabView3D.dispose"` |
| `§_-J0§\Settings3D` / `§_-w2X§` | nombre real / **TextureQualityProfile** | tamaños por categoría |
| `§_-S4H§\§_-15§` | **TextureMaterial** (Away3D) | texture/normalMap/specularMap/gloss/alpha |
| `§_-w2S§\§_-12c§` | **ShipMaterial** (TextureMaterial + glow + alphaMask) | añade 2 effect-methods |
| `§_-62N§\§_-82P§` | **GlowMapMethod** (emisivo aditivo) | AGAL: `mul glow, factor` + `add output, glow` |
| `§_-62N§\§_-gc§` | **AlphaMaskMethod** | análogo |
| `UberAssetsLib` (sin ofuscar) | catálogo de descriptores XML por entidad | `lib_ships`, `lib_drones`, ... |
| `§_-k4J§\§_-R4F§` / `§_-83r§\§_-T1c§` | **Mesh / ObjectContainer3D** | clone(), geometry, material, animator |

---

## 1. Flujo completo de carga de un modelo

### 1.1 Manifiesto (arranque del modo 3D)
`§_-o2x§.§_-gy§()` (`display3D\§_-o2x§.as` L70-96):
1. Descarga `xml/resources_3d.xml?__cv=` + `Settings.resourceXMLHash` desde `Settings.staticHost + Settings.basePath` → **FileCollection de geometría+texturas**.
2. Al terminar, descarga `xml/resources_3d_particles.xml?__cv=...` → **FileCollection de fx** (con flag `idIncludesExtension=true`, por eso los fx se piden como `"emp.zip"`, `"thruster.zip"` y las texturas/mallas como id pelado: `"aegis"`, `"aegis-eic_diffuse_512"`).

El manifiesto real (CMS `public\spacemap\xml\resources_3d.xml`, 559 KB) declara:
```xml
<location path="3d/meshes/" id="3d_meshes"/>
<location path="3d/textures/" id="3d_textures"/>
<location path="3d/fx/" id="3d_fx"/>
<file version="1" type="awd" name="aegis" location="3d_meshes" id="aegis" hash="b162e8ef7bab..."/>
<file version="1" type="atf" name="aegis-eic_diffuse_512" location="3d_textures" id="aegis-eic_diffuse_512" hash="..."/>
```
2.854 entradas `atf` + 245 `awd`; el de partículas mezcla `zip` y `atf`.

### 1.2 Patrón de URL exacto y cache-busting
`§_-x2v§\§_-n3T§.§_-jr§()` L41-49:
```
URL = staticHost + basePath + location.path + name + "." + type [ + "?__cv=" + hash ]
```
Doble cache-busting: el manifiesto va versionado con `resourceXMLHash` global y **cada archivo lleva su hash MD5 individual** (`@hash` del XML) como query `__cv`. Cambia el hash → revienta la caché HTTP solo de ese archivo.

### 1.3 De nombre lógico a escena
- Quien quiere un modelo llama `§_-o2x§.instance.§_-w21§.§_-O11§(resKey, lifetime)` (GeometryLoader). Devuelve un **promise** (`§_-Y0§.§_-P4u§`, señal con `addOnce`).
- `GeometryLoader.§_-O11§` (L89-99): si no está en caché (`§_-p3s§[resKey]`) crea un `SingleGeometryLoader` que pide los **bytes** a la FileCollection y los pasa a un AssetLoader de Away3D (`§_-L4j§.§_-u2i§`) con `loadData(bytes)`.
- Los `.awd` **NO van en zip**: se descargan directos (los zip son solo para fx de partículas).
- Parser: `GeometryLoader` registra estáticamente `AssetLibrary.enableParser(§_-t1a§)` y `OBJParser`. `§_-t1a§` (CompositeAWDParser) decide por magic: `AWD99Parser.supportsData` (bytes 3-4 == 99,1) o `AWD2Parser`. **Los .awd del CMS son AWD 2.1 estándar** (ver §6); AWD99 es un formato propietario posterior con soporte de texturas ATF embebidas, sombras, luces, partículas y primitivas.
- El resultado es un `ObjectContainer3D` compartido en caché. `PrefabView3D` (§_-jh§\§_-717§) recibe el container por el promise y **clona cada Mesh hijo** (`§_-m2V§` L234-247: `param1.clone()`); la geometría y el animator-set se comparten entre clones, el material se asigna desde el MaterialManager. Al terminar arranca la animación `"idle"` con offset aleatorio `Math.random()*10000` ms para desincronizar instancias (L275, L218).

### 1.4 Descriptores de entidades (qué geometría/textura usa cada nave)
`UberAssetsLib` embebe 7 XML (`lib_ships`, `lib_drones`, `lib_portals`, `lib_collectables`, `lib_battlestation`, `lib_maps`, `lib_assets`) y puede recargarlos de `../../config/data/lib_*.xml?v=<timestamp>`. Ejemplo real (binaryData `187_...LIB_SHIPS_XML.bin`):
```xml
<asset id="ship_phoenix">
  <display2D srcKey="ship1" resKey="mc"/>
  <display3D tex_settings="ship_small" geometry="phoenix" texture="phoenix" scale="0.7" visualSize="0.6"/>
</asset>
<asset id="ship_aegis_design_aegis-elite">
  <display3D visualSize="1.4">
    <mesh geometry="aegis" tex_settings="ship" texture="aegis" scale="0.8"/>
    <mesh id="pod" geometry="aegis-ship-pod" tex_settings="ship" texture="aegis" scale="0.8"/>
  </display3D>
</asset>
```
Atributos vistos: `geometry, texture, diffuse, normal, specular, glow, alphaMask` (valor `"none"` desactiva el mapa), `shader` (`uber_ship`, ...), `tex_settings` (perfil de calidad), `scale`, `visualSize`, `rotationY="random(360)"`, `specularity`, `specularityLow`, `gloss`, `glossLow`, `alphaBlending`, `blendMode`, `materialBothSides`, `light="false"`, `engineTrailColor`. `§_-n3Z§\§_-ig§.as` L80 conecta todo: `new PrefabView3D(§_-LC§.§_-bL§(descriptor), Settings3D.§_-V15§(@shader), Settings3D.§_-A3F§(@tex_settings))`.

---

## 2. Texturas

### 2.1 Elección de _128/_256/_512 — por perfil de calidad + zoom (no por distancia)
`§_-jh§\§_-X34§.§_-X1g§` L267-274 — fórmula literal:
```actionscript
nombreArchivo = base + "_" + tipo + "_" + Math.max(128, Math.min(perfil.size * multiplicadorZoom, 512))
```
- `perfil.size` viene de `Settings3D` (`§_-J0§\Settings3D.as` L69-78, L135-165). Perfiles por `tex_settings` y ajuste de usuario `displaySetting3DsizeTextures`:

| Perfil | LOW | MEDIUM | HIGH |
|---|---|---|---|
| ship_very_small / drone | 64 | 64 | 128 |
| ship_small | 64 | 64 | 128 |
| ship | 128 | 128 | 256 |
| ship_big | 256 | 256 | 512 |
| building_small / planet_small | 128 | 256 | 512 |
| building | 256 | 512 | 1024 |
| planet | 256 | 512 | 1024 |
| building_big | (1024 por defecto) | | |

- `multiplicadorZoom` (`§_-E1r§`, default 1): `§_-Oj§\§_-r16§.as` L40-47 lo pone a **2 cuando el zoom de cámara > 1.3** — al acercar la cámara todas las naves del holder suben un escalón de resolución (128→256). Se re-resuelve en caliente (señal `changed` → recarga de mapas).
- Clamps: mínimo 128 (no existen archivos _64), máximo 512 por defecto (por eso las variantes de archivo son _128/_256/_512; existen algunas _1024 para building/planet que usan cap distinto vía perfil).
- **Normal y specular se piden a MITAD de resolución**: `size >> 1` (L241, L248). Diffuse, glow y alpha a resolución completa (glow con cap explícito 512, L255).
- Toggles por calidad de textura (`displaySetting3DqualityTextures`, L123-133): normal y specular solo en HIGH; glow desde MEDIUM. `displaySetting3DtextureFiltering`: smoothing desde MEDIUM, mipmapping solo HIGH.

### 2.2 Formato ATF interno
`§_-X2m§\§_-R4v§.as`: magic `"ATF"`; si byte[6]==0xFF la cabecera es larga (pos 12). Byte de formato: bit7 = cubemap; bits 0-6: 0/1=BGRA, 2/3=`COMPRESSED` (DXT1), 4/5=`"compressedAlpha"` (DXT5). Luego log2(ancho), log2(alto), numTextures (niveles de mip).

Archivos reales (hexdump):
- `aegis-eic_diffuse_512.atf`: `41 54 46 00 00 00 FF 02 00 02 15 37 | 02 09 09 0A` → **DXT1 (COMPRESSED), 512×512, 10 mips (cadena completa)**.
- `aegis-elite-mmo_glow_128.atf`: `... | 02 07 07 08` → DXT1, 128×128, 8 mips.
Los mipmaps van **precalculados dentro del ATF** (numTextures>1 activa el flag `§_-b3C§` de mipmapping en la textura).

### 2.3 Combinación diffuse + glow en el material
- Material de naves: `§_-w2S§\§_-12c§` (**ShipMaterial**) `extends §_-15§` (TextureMaterial) con `repeat=true` y dos effect-methods añadidos: `§_-gc§` (alphaMask) y `§_-82P§` (glowMap).
- `§_-82P§` (**GlowMapMethod**, `§_-62N§\§_-82P§.as` L81-99) — el glow NO es ambient ni canal de TextureMaterial: es un método de fragmento que muestrea la textura glow, la multiplica por un **factor por-renderable** (`renderable.extra.glow`, default 1 — `§_-717§.§_-W2M§ = {"glow":1}`) y la **suma aditivamente al color final**:
```
tex glow ; mul glow.xyz, glow, factor.xxxx ; add output.xyz, output, glow
```
  Es independiente de la iluminación → emisivo puro, animable (UberAnimation3D tuena "glow" entre minValue/maxValue con duración ~6 s; `§_-U3Z§\§_-33D§.as` liga alpha del glow al % de HP).
- Asignación de mapas (`§_-X34§.§_-x3c§` L298-325): `texture=diffuse`, `normalMap`, `specularMap`, `glowMap`, `alphaMask`, `smooth`, `mipmap`.
- **Gloss/specular de naves**: defaults `specular=1`, `gloss=50` (`§_-LC§.as` L12-18), sobreescribibles por XML (`specularity`, `gloss`, y variantes `*Low` que se usan cuando el mapa specular está desactivado por calidad: L307-308 de `§_-X34§`).
- Detalle: `alphaPremultiplied = !(diffuse is ATFTexture)` (L312) — los ATF no van premultiplicados; los bitmaps sí.
- Shaders alternativos registrados por nombre (`Settings3D.§_-o2y§` L94-100): `basic`→TextureMaterial, `ship`→ShipMaterial (default), `organic_ship`, `uber_ship`→UberShipMaterial, `uber_organic_ship`, `sector_control_beacon`, `animated_cloud`.

### 2.4 Subida a GPU
`§_-X2m§\§_-5B§.§_-W3A§(stage3DProxy)` L46-67:
- Crea la Texture del Context3D si no existe o si el contexto cambió (soporta context-loss: cachea por índice de stage3D y compara contexto).
- `uploadCompressedTextureFromByteArray(atfData, 0, async=true)` → **subida asíncrona nativa de Flash**; hasta que llega `Event.TEXTURE_READY` el getter devuelve `null` y el material simplemente no muestrea (evita bloquear el frame). Señal `ready` propia.

---

## 3. Caché, refcounting y memoria

### 3.1 Estructura
Ambos loaders (`GeometryLoader`, `TextureLoader`) extienden `§_-T2s§`:
- **Caché por resKey** (`§_-p3s§: Dictionary`): un único `SingleXLoader` por clave; llamadas repetidas devuelven el mismo promise. `Settings3D.§_-mV§` (BindableBoolean debug) fuerza recarga.
- **Refcount con periodo de gracia de 5 segundos**: `super(callbackUnused, 5)`. `§_-N1§\§_-f7§`: `retain()` (`§_-h2d§`) incrementa y cancela el timer; `release()` (`§_-B10§`) al llegar a 0 arma `setTimeout(unused, 5*1000)`. Si nadie lo retoma en 5 s, dispara el callback.
- El callback (`GeometryLoader.§_-F45§` L118-133 / `TextureLoader.§_-fr§`) **solo desecha si `lifetime <= 0`** (transitorio). Assets de mapa (1) o precargados (2) sobreviven aunque su refcount llegue a 0.
- Quién retiene: `PrefabView3D.set geometry` retiene el container (`§_-Z1s§`/`§_-O2k§`); `§_-X34§` retiene cada textura al asignarla (`§_-a3i§`/`§_-H1w§`, L107-145).

### 3.2 reset() vs dispose()
- `§_-o2x§.reset()` (L162-168) — **cambio de mapa** (llamado desde `view3D\§_-i3S§.as` L91): dispara señal `§_-zH§` (los views se auto-destruyen) y llama `reset(1)` en los tres loaders → desecha todo con `lifetime <= 1` (transitorios + de mapa). Lo precargado (2) permanece.
- `§_-o2x§.dispose()` — apagado del 3D completo: `reset()` + anula loaders + `LightsManager.dispose()`.
- Disposal profundo de geometría: `GeometryLoader.§_-u3s§` L27-70 recorre el container y hace `geometry.dispose()`, `material.dispose()`, `animator.dispose()` (o `disposeAsset()` para animators de vértices) recursivo.

### 3.3 Materiales compartidos (pooling)
`§_-X34§` (MaterialManager) es un **pool estático refcontado**: clave = `getQualifiedClassName(shaderClass) + perfilCalidad.uid + materialDef.hash` (hash MD5 de todos los parámetros, `§_-LC§.hash`). Todas las naves iguales con misma calidad **comparten instancias de material** (contador `counter`; al llegar a 0 → `dispose()` y borra la entrada, L57-80). Dentro de un manager hay N variantes de material indexadas por firma (animationSet + alphaBlending + methods dinámicos) porque Away3D compila un programa por combinación (`§_-717§.§_-S36§` L288-314).

### 3.4 Clonado de meshes
`PrefabView3D.§_-m2V§`: `mesh.clone()` de Away3D comparte la `Geometry` (vertex/index buffers en GPU una sola vez) y clona solo el transform/animator-state. No hay pool de instancias de Mesh: se clona por entidad y se desecha con ella; lo pesado (geometría, texturas, materiales) sí está compartido y refcontado.

---

## 4. §_-J34§.update() — corrección importante

`§_-o2x§.§_-J34§` **es el LightsManager, no una cola de subidas a GPU**. Su `update(force)` (L88-114) reconstruye la lista de luces del `StaticLightPicker` principal solo cuando hay flag dirty (visibilidad de luces cambiada) — nada de throttling de uploads.

Composición de luces (`LightsManager.as`):
- 1 `DirectionalLight` "sol" + 1 `PointLight` "hero" (sigue a la nave propia) + **pool de exactamente 3 PointLights de efectos** (explosiones: `§_-Qz§` con TweenMax y timeout de vida ~0.06 s por defecto).
- Según `displaySetting3DqualityLights`: LOW = ninguna luz visible (materiales sin iluminar), MEDIUM = sol + 1 luz de efecto, HIGH = sol + hero + las 3 (L161-188).
- Dos pickers: el principal (naves) y uno para partículas que solo lleva el sol y solo si `effects == high` (L209-212).
- Valores por defecto del sol (`Settings3D.§_-s37§` L47): color 0xA3FFFF‑ish (10747903), diffuse 0.8, specular 1.1, ambientColor 16745820, ambient 0.5; el XML del mapa puede sobreescribirlos (`§_-V16§.as` L227).

**Los mecanismos anti-stutter reales son dos:**
1. **Parseo incremental**: `§_-F1w§\§_-74y§` (ParserBase de Away3D) parsea AWD con presupuesto de **30 ms por frame** (`§_-528§(data, 30)`, L113; chequeo `getTimer() - inicio < límite` L286). Un AWD grande se reparte entre frames.
2. **Subida asíncrona de texturas**: `uploadCompressedTextureFromByteArray(..., async=true)` + `TEXTURE_READY` (§2.4). El driver copia en segundo plano; el material devuelve `null` y no pinta esa textura hasta que está lista.
No hay límite explícito de "N uploads por frame": la combinación async-upload + parseo con budget + caché por resKey es todo el mecanismo.

---

## 5. "Prioridades" de carga = lifetimes de caché

`§_-o2x§`: `§_-Y4h§=2`, `§_-V4X§=1`, `§_-7X§=0`. El parámetro `param2:uint` de `§_-O11§`/`loadTexture`/`§_-c2T§` se guarda como `lifetime = max(lifetime, param2)` (solo puede subir). No reordena la red (las descargas salen en orden de petición); decide **cuánto vive en caché**:

| Valor | Uso | Se desecha |
|---|---|---|
| 0 TRANSIENT | geometría y texturas de naves (PrefabView3D/MaterialManager) | 5 s después de que su refcount llegue a 0 |
| 1 MAP | fx de partículas (`§_-01b§\*`, `§_-a3o§\*`), assets del mapa (`§_-P2o§\§_-413§`) | en `reset()` al cambiar de mapa |
| 2 PERMANENT | lista de precarga XML + skybox | solo en `dispose()` del 3D |

**Precarga** (`view3D\§_-i3S§.as` §_-32J§ L243-269): tras crear la vista 3D lee `patterns.preloadLists.pack.(@id=="3D").children()` (XML de configuración de recursos del servidor) y por cada hijo `<geometry>`, `<texture>`, `<particles>` llama al loader correspondiente con lifetime 2; cuando el contador llega a 0 dispara `_ready` (el juego espera a la precarga). El skybox se carga lazy en el primer render con lifetime 2 (`§_-i22§\DOSkybox.as` L41-45: `"skybox_stars"`, `"skybox_mask"`, `"skybox_geometry"`, malla escalada ×10000, shader propio animado).

Todo lo demás es **lazy**: la nave se carga al aparecer la entidad; mientras llega el AWD la entidad existe sin mesh (los descriptores tienen `_placeholder-ship`, `_placeholder-building`, etc. en `resources_3d.xml` para asignar modelos genéricos vía datos).

---

## 6. Assets reales (CMS `public\spacemap\3d\`)

- **Conteo**: `meshes` 322 archivos (9.9 MB), `textures` 4.401 (211.9 MB), `fx` 663 (4.3 MB).
- **Tamaño típico de .awd de nave**: 10-60 KB (aegis 27.8 KB, aegis-veteran 29 KB, anihilator 56 KB, annihilator 35.7 KB, módulos láser 7-10 KB). Son mallas muy ligeras (~1-3k triángulos a ojo de tamaño).
- **Cabeceras .awd** (hexdump):
  - `aegis.awd`: `41 57 44 02 01 E0 00 01 72 6C 00 00 78 DA` → magic `AWD`, **versión 2.1**, flags 0xE000, **compression=1 (zlib** — sigue `78 DA`), body length LE, cuerpo deflate.
  - `al-laser-module.awd`: igual pero flags 0x0000.
  - El cliente también soporta la variante propietaria **AWD "99.1"** (AWD99Parser: compresión 0=ninguna, 1=zlib, 2=**lzma**; puede embeber ATF/ATF-cube y bitmaps), pero los archivos de este snapshot son AWD2 estándar → **AWD2Parser de Away3D 4.x los abre tal cual**.
- **Cabeceras .atf**: ver §2.2 — DXT1, cadena de mips completa.
- **fx**: cada `.zip` contiene un único `.awp` = **JSON** con el grafo de nodos de partículas de Away3D (`ParticleTimeNode`, `ParticleBillboardNode`, `ParticleVelocityNode`, ... vía `*SubParser`) + material `{"id":"TextureMaterialSubParser","data":{"blendMode":"add","url":"simple_gradient.png",...}}`. La URL `.png` se reescribe a `.atf` (§_-X4L§) y se resuelve contra la colección de partículas (los `.atf` sueltos de `fx\` como `absorbtion_beam_pattern.atf`). Los materiales con mesh llamado `shader_fireball` se sustituyen por `ExplosionMaterial`; los nombrados `light_*` o listados en `lightPickerTargets` reciben el light picker de partículas (`§_-u1u§.§_-i45§` L90-106).
- **Convención de nombres**:
  - Mallas: `<nave>.awd` (`aegis`, `goliath`, `vengeance`), variantes `-veteran`, pods (`aegis-ship-pod`), módulos de láser `al-laser-module`/`am-laser-module`, props de mapa `asset-<facción mmo|eic|vru>-<objeto>[-letra]`, `building-*`, `cbs-*` (battlestation), `drone*` (29), `pet*` (26), jefes (`streuner*` 18), gates (`eventgate*`, `galaxygate*`, `jumpgate*`), `_placeholder-*` (10), `low-*` (LOD bajo de algo), `skybox_geometry`.
  - Texturas: `<base>_<tipo>_<tamaño>.atf` con tipo ∈ {diffuse (1.685), glow (1.078), specular (772), normal (742), alpha (59)} y tamaño ∈ {128, 256, 512, 1024 (72)}. Bases con facción/skin: `aegis-eic`, `aegis-mmo-elite`, `aegis-elite_design_a-elite-lava`, frames animados `eventgate-invasion_diffuse_frame_1_512`. Sueltas: `_ao_`, `_bump_`, `_clean_` (variantes de autoría no usadas por el código de materiales estándar), `skybox_*`.

---

## 7. Fallbacks y robustez

1. **Error de carga de textura** (`TextureLoader.SingleTextureLoader.handleLoadError` L193-199): dispatch `null`; el material queda con ese canal a null (Away3D pinta con el default). Silenciado explícitamente para claves `alpha_*`.
2. **Textura no-ATF**: si la FileCollection devuelve un finisher de imagen (png/jpg/jxr — registrados en `§_-o15§.§_-01d§` y `§_-o2x§.§_-gy§` L76 añade `jxr`), el loader crea **BitmapTexture** (`§_-y3T§`) en vez de ATFTexture (L180-186). Es la vía sin compresión: decide el **manifiesto** (tipo de archivo), no una detección de capacidades del hardware — hoy el manifiesto es 100 % atf.
3. **Fallo del upload síncrono/creación GPU**: try/catch alrededor de `§_-W3A§` (L138-147); si lanza, se despacha ready igualmente (textura null).
4. **Context loss**: ATFTexture cachea contexto por índice y re-crea+re-sube al detectar contexto distinto (L51-55); `§_-i3S§` escucha recreación del Context3D y recarga el mapa (L271-281).
5. **Error de geometría**: promise con `null`; el PrefabView queda vacío. El fallback de contenido es **por datos**: mallas `_placeholder-*` asignables desde los descriptores.
6. **Recarga ordenada**: `AssetNotifications.ADD_UNLOAD_FINISHER` — tras parsear el AWD los bytes crudos se liberan del finisher de la FileCollection (SingleGeometryLoader L221-223), no se duplican en memoria.
7. **Antialias por ajuste** (`§_-i3S§.§_-QK§` L234-241): MEDIUM=2, HIGH=8, ULTRA=16, LOW=0. Starling comparte contexto con perfil "baseline".
8. Ajustes 3D persistidos en `SharedObject "darkorbit.settings3D"` (`§_-o2x§` L119-122).

---

## 8. Recetas aplicables al cliente Godot (traducción directa)

1. **Manifiesto con hash por archivo** (`resources_3d.xml` → JSON): id lógico → ruta + hash de contenido; cache-bust por archivo, no global.
2. **Un loader-caché por tipo con promise por resKey**: pedir dos veces = mismo objeto en vuelo; nunca dos descargas del mismo asset.
3. **Refcount + gracia de 5 s antes de liberar**: evita thrashing cuando una nave sale y vuelve a entrar en pantalla.
4. **Tres lifetimes** (transient / mapa / permanente) y un `reset(1)` al cambiar de mapa; la precarga global sobrevive.
5. **Resolución de textura por perfil de categoría × ajuste de usuario × zoom** con clamp [128, 512]; normal/specular a mitad de resolución; toggles por calidad (normal+specular solo en high, glow desde medium).
6. **Materiales compartidos y refcontados** con clave = shader + perfil + hash de parámetros; los meshes se clonan compartiendo geometría.
7. **Glow = segunda textura aditiva** con factor escalar animable por instancia (equivale a `emission_texture` + `emission_energy` en Godot), ligable a HP.
8. **Parseo con presupuesto por frame** (30 ms) y carga/subida asíncrona (en Godot: `ResourceLoader.load_threaded_request` / import en hilo).
9. **Idle animation con offset aleatorio** por instancia para desincronizar flotas.
10. **Luces con presupuesto duro**: 1 direccional + 1 luz del héroe + pool de 3 luces de efecto con vida corta tuneada; lista de luces reconstruida solo on-dirty.
