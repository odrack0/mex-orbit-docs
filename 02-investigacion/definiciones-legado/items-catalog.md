# Catálogo de ítems (imágenes CMS)

Este documento se genera a partir de la carpeta `MexOrbit.CMS/public/do_img/global/items`.

- **Categoría**: segmentos de ruta relativos a `items`, unidos por guiones bajos (p. ej. `equipment/weapon/laser` → `equipment_weapon_laser`). Los assets sueltos en la raíz de `items` suelen duplicar ítems ya cubiertos por subcarpetas; **no** se listan aquí (catálogo orientado a tienda / rutas canónicas).
- **Nombre de ítem**: nombre de archivo sin extensión, quitando sufijos de tamaño (`_100x100`, `_30x30`, …), `_top` e `_icon` cuando aplican; así todas las variantes de un mismo ítem se cuentan una sola vez.
- **Ámbito**: solo se indexó `do_img/global/items`. Otras carpetas bajo `do_img/global` (companies, hof, loadingscreen, pilotSheet, ranks, xml, etc.) no se incluyen aquí.

**Generado:** 2026-04-12  
**Archivos indexados:** 1716  
**Categorías distintas:** 26  

Omisiones respecto al árbol completo de `items/`: ver [items-catalog-omissions.md](items-catalog-omissions.md).

**Precios de referencia (créditos / uridium):** ver [Precios de referencia DarkOrbit retail](#precios-de-referencia-darkorbit-retail) al final del documento. No están codificados en este repositorio como tabla única de tienda; sirven como guía tomada de fuentes públicas del juego original.

## Resumen por categoría

| Categoría | Carpeta (relativa a `items`) | Ítems únicos |
|-----------|--------------------------------|--------------|
| `ammunition_firework` | ammunition\firework | 3 |
| `ammunition_laser` | ammunition\laser | 8 |
| `ammunition_mine` | ammunition\mine | 9 |
| `ammunition_rocket` | ammunition\rocket | 6 |
| `ammunition_rocketlauncher` | ammunition\rocketlauncher | 9 |
| `ammunition_specialammo` | ammunition\specialammo | 4 |
| `deal` | deal | 3 |
| `drone` | drone | 4 |
| `drone_designs` | drone\designs | 2 |
| `drone_formation` | drone\formation | 13 |
| `equipment_aiprotocol` | equipment\aiprotocol | 13 |
| `equipment_booster` | equipment\booster | 16 |
| `equipment_extra` | equipment\extra | 1 |
| `equipment_extra_cpu` | equipment\extra\cpu | 34 |
| `equipment_extra_repbot` | equipment\extra\repbot | 5 |
| `equipment_generator_shield` | equipment\generator\shield | 5 |
| `equipment_generator_speed` | equipment\generator\speed | 6 |
| `equipment_petgear` | equipment\petgear | 11 |
| `equipment_weapon_laser` | equipment\weapon\laser | 5 |
| `equipment_weapon_rocketlauncher` | equipment\weapon\rocketlauncher | 2 |
| `hangar` | hangar | 1 |
| `module` | module | 10 |
| `resource` | resource | 8 |
| `resource_blueprint` | resource\blueprint | 2 |
| `resource_ore` | resource\ore | 9 |
| `resource_wordpuzzle-letter` | resource\wordpuzzle-letter | 12 |

## Detalle por categoría

### `ammunition_firework`

- **Ruta:** `items/ammunition/firework`

Ítems:

- `fwx-l`
- `fwx-m`
- `fwx-s`

### `ammunition_laser`

- **Ruta:** `items/ammunition/laser`

Ítems:

- `cbo-100`
- `job-100`
- `lcb-10`
- `mcb-25`
- `mcb-50`
- `rsb-75`
- `sab-50`
- `ucb-100`

### `ammunition_mine`

- **Ruta:** `items/ammunition/mine`

Ítems:

- `acm-01`
- `ddm-01`
- `empm-01`
- `rb-02`
- `rb-e01`
- `rb-e02`
- `sabm-01`
- `sl-01`
- `slm-01`

### `ammunition_rocket`

- **Ruta:** `items/ammunition/rocket`

Ítems:

- `bdr-1211`
- `eco-10`
- `plt-2021`
- `plt-2026`
- `plt-3030`
- `r-310`

### `ammunition_rocketlauncher`

- **Ruta:** `items/ammunition/rocketlauncher`

Ítems:

- `bdr1212`
- `cbr`
- `eco-10`
- `hstrm-01`
- `sar-01`
- `sar-02`
- `shg-01`
- `shg-02`
- `ubr-100`

### `ammunition_specialammo`

- **Ruta:** `items/ammunition/specialammo`

Ítems:

- `dcr-250`
- `emp-01`
- `pld-8`
- `wiz-x`

### `deal`

- **Ruta:** `items/deal`

Ítems (las llaves `booty-key*` están solo en `resource`; no duplicar aquí):

- `extra-energy`
- `jumpvoucher`
- `logfile`

### `drone`

- **Ruta:** `items/drone`

Ítems (un registro canónico por familia; en disco hay variantes `*-0`…`*-5`):

- `apis`
- `flax`
- `iris`
- `zeus`

### `drone_designs`

- **Ruta:** `items/drone/designs`

Ítems (un registro canónico por diseño; en disco hay variantes `havoc-0`…`havoc-5`):

- `havoc`
- `hercules`

### `drone_formation`

- **Ruta:** `items/drone/formation`

Ítems:

- `f-01-tu`
- `f-02-ar`
- `f-03-la`
- `f-04-st`
- `f-05-pi`
- `f-06-da`
- `f-07-di`
- `f-08-ch`
- `f-09-mo`
- `f-10-cr`
- `f-11-he`
- `f-12-ba`
- `f-13-bt`

### `equipment_aiprotocol`

- **Ruta:** `items/equipment/aiprotocol`

Ítems:

- `ai-aim1`
- `ai-al1`
- `ai-cr1`
- `ai-e1`
- `ai-eco1`
- `ai-hp1`
- `ai-lm1`
- `ai-pm1`
- `ai-r1`
- `ai-s1`
- `ai-sm1`
- `cgm-02`
- `csr-02`

### `equipment_booster`

- **Ruta:** `items/equipment/booster`

Ítems:

- `dmg-b01`
- `dmg-b02`
- `hon-b01`
- `hon-b02`
- `hp-b01`
- `hp-b02`
- `rep-b01`
- `rep-b02`
- `res-b01`
- `res-b02`
- `shd-b01`
- `shd-b02`
- `sreg-b01`
- `sreg-b02`
- `xp-b01`
- `xp-b02`

### `equipment_extra`

- **Ruta:** `items/equipment/extra`

Ítems:

- `hmd-07`

### `equipment_extra_cpu`

- **Ruta:** `items/equipment/extra/cpu`

Ítems:

- `aim-01`
- `aim-02`
- `ajp-01`
- `alb-x`
- `arol-x`
- `cl04k-m`
- `cl04k-xl`
- `cl04k-xs`
- `dr-01`
- `dr-02`
- `fb-x`
- `g3x-amry-l`
- `g3x-amry-m`
- `g3x-amry-s`
- `g3x-crgo-x`
- `ish-01`
- `jp-01`
- `jp-02`
- `min-t01`
- `min-t02`
- `nc-agb-x`
- `nc-awb-x`
- `nc-awl-x`
- `nc-awr-x`
- `nc-rrb-x`
- `rb-x`
- `rd-x`
- `rllb-x`
- `rok-t01`
- `sle-01`
- `sle-02`
- `sle-03`
- `sle-04`
- `smb-01`

### `equipment_extra_repbot`

- **Ruta:** `items/equipment/extra/repbot`

Ítems:

- `rep-1`
- `rep-2`
- `rep-3`
- `rep-4`
- `rep-s`

### `equipment_generator_shield`

- **Ruta:** `items/equipment/generator/shield`

Ítems:

- `sg3n-a01`
- `sg3n-a02`
- `sg3n-a03`
- `sg3n-b01`
- `sg3n-b02`

### `equipment_generator_speed`

- **Ruta:** `items/equipment/generator/speed`

Ítems:

- `g3n-1010`
- `g3n-2010`
- `g3n-3210`
- `g3n-3310`
- `g3n-6900`
- `g3n-7900`

### `equipment_petgear`

- **Ruta:** `items/equipment/petgear`

Ítems:

- `cgm-02`
- `csr-02`
- `g-al1`
- `g-ar1`
- `g-el1`
- `g-ex1`
- `g-kk1`
- `g-pd1`
- `g-rep1`
- `g-rl1`
- `g-tra1`

### `equipment_weapon_laser`

- **Ruta:** `items/equipment/weapon/laser`

Ítems:

- `lf-1`
- `lf-2`
- `lf-3`
- `lf-4`
- `mp-1`

### `equipment_weapon_rocketlauncher`

- **Ruta:** `items/equipment/weapon/rocketlauncher`

Ítems:

- `hst-1`
- `hst-2`

### `hangar`

- **Ruta:** `items/hangar`

Ítems:

- `slot`

### `module`

- **Ruta:** `items/module`

Ítems:

- `defm-1`
- `dmgm-1`
- `honm-1`
- `hulm-1`
- `ltm-hr`
- `ltm-lr`
- `ltm-mr`
- `ram-la`
- `ram-ma`
- `repm-1`

### `resource`

- **Ruta:** `items/resource`

Ítems:

- `booty-key`
- `booty-key-blue`
- `booty-key-red`
- `ec`
- `lgf`
- `logfile`
- `lottery-euro-2012`
- `pet-fuel`

### `resource_blueprint`

- **Ruta:** `items/resource/blueprint`

Ítems:

- `apis-part`
- `zeus-part`

### `resource_ore`

- **Ruta:** `items/resource/ore`

Ítems:

- `duranium`
- `endurium`
- `palladium`
- `promerium`
- `prometid`
- `prometium`
- `seprom`
- `terbium`
- `xenomit`

### `resource_wordpuzzle-letter`

- **Ruta:** `items/resource/wordpuzzle-letter`

Ítems:

- `a`
- `b`
- `blank`
- `c`
- `d`
- `e`
- `h`
- `i`
- `k`
- `n`
- `o`
- `r`

## Precios de referencia DarkOrbit retail

Referencias de **DarkOrbit (BigPoint)** obtenidas de documentación comunitaria ([DarkOrbitWiki](https://darkorbitwiki.com/), [Fandom](https://darkorbit.fandom.com/), foros oficiales). **MexOrbit puede diferir** (economía propia, eventos, parches). **C** = créditos, **U** = uridium. Cuando se indica “/ unidad” es precio por disparo, mina o cohete según el ítem.

### Alcance y moneda

- En el juego original, la mayoría de objetos “élite” o temporales se pagan en **uridium**; barcos, generadores básicos y parte de la munición en **créditos**.
- **Premium / descuentos** (rebate, ofertas) alteran el uridium efectivo; [Buy now](https://darkorbit.fandom.com/wiki/Buy_now(in-game)) cita ejemplos de paquetes con premium.
- **Subasta** y **ensamblaje (Assembly)** permiten obtener muchos ítems sin precio fijo de tienda.

### `ammunition_firework`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `fwx-l` | — | — | Fuegos artificiales; suelen ser **evento** / promoción; sin precio de tienda estable en fuentes consultadas. |
| `fwx-m` | — | — | Ídem. |
| `fwx-s` | — | — | Ídem. |

### `ammunition_laser`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `lcb-10` | 10 / unidad | — | Batería básica; precio por disparo en tienda (wiki). |
| `mcb-25` | — | 0,5 / unidad | |
| `mcb-50` | — | 1 / unidad | |
| `sab-50` | — | 1 / unidad | Absorbe escudo. |
| `rsb-75` | — | 5 / unidad | Cadencia lenta, alto multiplicador. |
| `cbo-100` | — | 5 / unidad | A veces solo en **eventos** (wiki). |
| `job-100` | — | 9 / unidad | Daño distinto PvE/PvP (referencia wiki). |
| `ucb-100` | — | — | **No en tienda habitual**; GG, cajas, eventos. |

### `ammunition_mine`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `acm-01` | — | ~100 / unidad | ACM-1; área (wiki / Fandom). |
| `ddm-01` | — | — | Daño directo; ver tienda in-game / wiki minas. |
| `empm-01` | — | ~150–500 / unidad | EMP; distintas variantes en fuentes (EMP-M01 vs EMP-01). |
| `rb-02` | — | — | Mina estándar; consultar wiki actual. |
| `rb-e01`, `rb-e02` | — | — | Variantes; subasta frecuente. |
| `sabm-01` | — | — | |
| `sl-01` | — | — | Smart bomb relacionada con minas en mecánicas DO. |
| `slm-01` | — | — | |

### `ammunition_rocket`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `r-310` | 100 / unidad | — | Cohete básico (Fandom / wiki). |
| `eco-10` | variable | — | A veces créditos; ver tienda. |
| `plt-2021` | — | ~5 / unidad | Oferta “Buy now” con MCB-25 en mismos rangos UR (Fandom). |
| `plt-2026` | — | — | Similar a familia PLT; uridium en tienda. |
| `plt-3030` | — | 7 / unidad | |
| `bdr-1211` | — | — | Busbedarf / especial; eventos o tienda temporal. |

### `ammunition_rocketlauncher`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `cbr` | — | — | Munición lanzacohetes; ver [rocket launchers wiki](https://darkorbitwiki.com/equipment/rocket-launchers-and-ammunition/). |
| `eco-10` | — | — | |
| `hstrm-01` | — | — | Hellstorm; elite. |
| `sar-01`, `sar-02` | — | — | |
| `shg-01`, `shg-02` | — | — | |
| `ubr-100` | — | — | |
| `bdr1212` | — | — | |

### `ammunition_specialammo`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `dcr-250` | — | — | Deceleración; uridium/evento. |
| `emp-01` | — | — | EMP munición especial. |
| `pld-8` | — | — | |
| `wiz-x` | — | — | |

### `deal`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `extra-energy` | — | — | Paquetes / ofertas; no precio fijo único. |
| `jumpvoucher` | — | — | Necesario para saltos con AJP; tienda o eventos. |
| `logfile` | — | — | Objeto de misión / deal; no tienda estándar. |

### `drone`

Coste **por drone sucesivo** en tienda (no un solo precio por ítem gráfico); [DarkOrbitWiki — Drones](https://darkorbitwiki.com/drones/).

| Ítem | C | U | Notas |
|------|---|-----|------|
| `flax` | 100k → 12,8M (8.º drone) | — | Serie: 100k, 200k, 400k, 800k, 1,6M, 3,2M, 6,4M, 12,8M; **total ~25,5M** los 8. |
| `iris` | — | 15k → 200k (8.º) | Serie en U; total referencia **~647k U** los 8; también subasta en créditos. |
| `apis` | — | — | **Ensamblaje** ~45 partes × ~24k U/part ≈ **1,08M U** (wiki; varía). |
| `zeus` | — | — | **Ensamblaje** ~45 × ~33k ≈ **1,485M U** (wiki). |

### `drone_designs`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `havoc` | — | 100k / diseño (ref.) | Cajas Zeta, eventos, pagos (wiki). |
| `hercules` | — | — | Kappa / eventos / tienda pagos. |

### `drone_formation`

[DarkOrbitWiki — formaciones](https://darkorbitwiki.com/drones/) (orden típico F-01…F-13 = Turtle, Arrow, Lance, Star, Pincer, Double Arrow, Diamond, Chevron, Moth, Crab, Heart, Barrage, Bat).

| Ítem | C | U | Notas |
|------|---|-----|------|
| `f-01-tu` | 1.000.000 | — | Turtle |
| `f-02-ar` | 1.000.000 | — | Arrow |
| `f-03-la` | — | — | Lance: **Assembly** en wiki (no solo créditos fijos). |
| `f-04-st` | 75.000 | — | Star |
| `f-05-pi` | 100.000 | — | Pincer |
| `f-06-da` | 75.000 | — | Double Arrow |
| `f-07-di` | 100.000 | — | Diamond |
| `f-08-ch` | 75.000 | — | Chevron |
| `f-09-mo` | 100.000 | — | Moth |
| `f-10-cr` | 100.000 | — | Crab |
| `f-11-he` | 100.000 | — | Heart |
| `f-12-ba` | 100.000 | — | Barrage |
| `f-13-bt` | 125.000 | — | Bat |

### `equipment_aiprotocol`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `ai-*`, `cgm-02`, `csr-02` | — | — | Protocolos de IA / PET: **laboratorio, eventos, uridium** según versión DO; sin tabla única aquí. |

### `equipment_booster`

Boosters de tienda típicos **~10.000 U / 10 h** (versión no compartida); B02 compartidos también en rango similar ([extras/boosters wiki](https://darkorbitwiki.com/equipment/extras-boosters/)).

| Ítem | C | U | Notas |
|------|---|-----|------|
| `dmg-b01`, `dmg-b02` | — | ~10.000 (10 h) | |
| `hon-b01`, `hon-b02` | — | ~10.000 (10 h) | |
| `hp-b01`, `hp-b02` | — | ~10.000 (10 h) | |
| `rep-b01`, `rep-b02` | — | ~10.000 (10 h) | |
| `res-b01`, `res-b02` | — | ~10.000 (10 h) | |
| `shd-b01`, `shd-b02` | — | ~10.000 (10 h) | |
| `sreg-b01`, `sreg-b02` | — | ~10.000 (10 h) | |
| `xp-b01`, `xp-b02` | — | ~10.000 (10 h) | |

### `equipment_extra`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `hmd-07` | — | — | Extra; consultar tienda in-game. |

### `equipment_extra_cpu`

[Listado detallado](https://darkorbitwiki.com/equipment/cpu/). Ejemplos con precio en **créditos** en tienda (orden según wiki; verificar en cliente):

| Ítem | C | U | Notas |
|------|---|-----|------|
| `jp-01` | ~10.000 | — | Jump CPU 10 saltos |
| `jp-02` | ~25.000 | — | Jump CPU 20 saltos |
| `ajp-01` | ~600.000 | — | Advanced Jump |
| `sle-01` | ~10.000 | — | +2 slots extras |
| `sle-02` | ~25.000 | — | +4 slots |
| `sle-03` | ~25.000 | — | +6 slots |
| `sle-04` | ~50.000 | — | +10 slots |
| `rok-t01` | ~15.000 | — | Rocket turbo |
| `arol-x` / `alb-x` | ~25.000 | — | Auto rocket |
| `cl04k-xs` | ~500 | — | Cloak pequeño |
| `cl04k-m` | ~5.000 | — | |
| `cl04k-xl` | ~50.000 | — | |
| `fb-x` | ~5.000 | — | Combustible PET automático |
| `rd-x` | ~5.000 | — | Radar |
| `rb-x` | ~10.000 | — | Rocket CPU auto-compra |
| `aim-01` | ~150.000 | — | Targeting +25% |
| `aim-02` | ~200.000 | — | Targeting +50% |
| `ish-01` | ~150.000 | — | Insta-shield |
| `smb-01` | ~75.000 | — | Smart bomb |
| `min-t01` | ~15.000 | — | Turbo mina -25% CD |
| `min-t02` | ~25.000 | — | Turbo mina -50% CD |
| `rllb-x` | variable | — | Rocket launcher auto |
| `dr-01`, `dr-02` | ~150.000 / ~5.000.000 | — | Drone repair (orden de magnitud wiki) |
| `nc-agb-x`, `nc-awb-x`, `nc-awl-x`, `nc-awr-x`, `nc-rrb-x` | 15k–200k | — | Auto-boost / reparación; rango aproximado |
| `g3x-amry-l`, `g3x-amry-m`, `g3x-amry-s`, `g3x-crgo-x` | — | — | Generadores de almacén PET; tienda elite |

### `equipment_extra_repbot`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `rep-s` | 0 | — | Gratis en wiki |
| `rep-1` | ~10.000 | — | |
| `rep-2` | ~64.000 | — | |
| `rep-3` | variable | — | |
| `rep-4` | variable | — | |

### `equipment_generator_shield`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `sg3n-a01` | 8.000 | — | |
| `sg3n-a02` | 16.000 | — | |
| `sg3n-a03` | 256.000 | — | |
| `sg3n-b01` | 2.560.000 | — | Según wiki (serie SG3N-B). |
| `sg3n-b02` | — | 10.000 | |

### `equipment_generator_speed`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `g3n-1010` | 2.000 | — | +2 velocidad |
| `g3n-2010` | 4.000 | — | +3 |
| `g3n-3210` | 8.000 | — | +4 |
| `g3n-3310` | 16.000 | — | +5 |
| `g3n-6900` | 256.000 | — | +7 |
| `g3n-7900` | — | 2.000 | +10 |

### `equipment_petgear`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `g-*` | — | variable | Geares PET; tienda uridium / eventos. |
| `cgm-02`, `csr-02` | — | — | Solapan con protocolos; ver laboratorio PET. |

### `equipment_weapon_laser`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `lf-1` | 10.000 | — | |
| `lf-2` | 40.000 | — | |
| `lf-3` | 250.000 | — | |
| `lf-4` | — | ~10.000 | LF-4 clásico en uridium o **pagos** / eventos (cambió con años). |
| `mp-1` | 50.000 | — | Daño distinto alien/jugador |

### `equipment_weapon_rocketlauncher`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `hst-1` | 500.000 | — | Hellstorm 3 cohetes/volley |
| `hst-2` | — | 15.000 | Hellstorm 5 cohetes/volley |

### `hangar`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `slot` | variable | — | Slot adicional de hangar; precio según oferta DO. |

### `module`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `defm-1`, `dmgm-1`, `honm-1`, `hulm-1`, `repm-1` | — | — | Módulos estación; **Assembly** / eventos en DO moderno. |
| `ltm-hr`, `ltm-lr`, `ltm-mr` | — | — | |
| `ram-la`, `ram-ma` | — | — | |

### `resource`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `booty-key` | — | variable | Llaves caja pirata; uridium / ofertas. |
| `booty-key-blue`, `booty-key-red` | — | variable | |
| `ec` | — | — | Energía cupón; evento. |
| `lgf` | — | — | |
| `logfile` | — | — | |
| `lottery-euro-2012` | — | — | Legacy / colección. |
| `pet-fuel` | — | variable | Combustible PET por unidades. |

### `resource_blueprint`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `apis-part` | — | ~24.000 / parte | ~45 partes para Apis (wiki). |
| `zeus-part` | — | ~33.000 / parte | ~45 partes para Zeus (wiki). |

### `resource_ore`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `prometium`, `endurium`, `terbium`, `prometid`, `duranium`, `promerium`, `seprom`, `palladium`, `xenomit` | mercado | mercado | **Recursos**: precio por **subasta / trueque** o refinado; no hay precio fijo de tienda único. |

### `resource_wordpuzzle-letter`

| Ítem | C | U | Notas |
|------|---|-----|------|
| `a`…`blank` | — | — | Evento puzzle; sin precio de tienda. |

### Fuentes consultadas (enlace)

- [DarkOrbitWiki — láseres y munición](https://darkorbitwiki.com/equipment/lasers-and-ammunition/)
- [DarkOrbitWiki — generadores](https://darkorbitwiki.com/equipment/generators/)
- [DarkOrbitWiki — CPUs](https://darkorbitwiki.com/equipment/cpu/)
- [DarkOrbitWiki — cohetes](https://darkorbitwiki.com/equipment/rockets/)
- [DarkOrbitWiki — boosters](https://darkorbitwiki.com/equipment/extras-boosters/)
- [DarkOrbitWiki — drones y formaciones](https://darkorbitwiki.com/drones/)
- [Fandom — Buy now](https://darkorbit.fandom.com/wiki/Buy_now(in-game))
- [Fandom — R-310 / PLT-3030](https://darkorbit.fandom.com/wiki/R-310) (páginas individuales de munición)

## Carpetas reservadas sin archivos

Bajo `items/category` existe una jerarquía de carpetas (posible taxonomía o contenido pendiente) que **no contiene ningún archivo** en el momento de generar este documento. Los identificadores de categoría siguen la misma regla (ruta con `_`).

| Categoría (id) | Carpeta (relativa a `items`) |
|----------------|------------------------------|
| `category` | `category` |
| `category_ammunition` | `category\ammunition` |
| `category_ammunition_especialammo` | `category\ammunition\especialammo` |
| `category_ammunition_firework` | `category\ammunition\firework` |
| `category_ammunition_laser` | `category\ammunition\laser` |
| `category_ammunition_laser_cbo` | `category\ammunition\laser\cbo` |
| `category_ammunition_laser_job` | `category\ammunition\laser\job` |
| `category_ammunition_laser_lcb-10` | `category\ammunition\laser\lcb-10` |
| `category_ammunition_mine` | `category\ammunition\mine` |
| `category_ammunition_rocket` | `category\ammunition\rocket` |
| `category_ammunition_rocketlauncher` | `category\ammunition\rocketlauncher` |
| `category_deal` | `category\deal` |
| `category_drone` | `category\drone` |
| `category_drone_apis` | `category\drone\apis` |
| `category_drone_desings` | `category\drone\desings` |
| `category_drone_desings_havoc` | `category\drone\desings\havoc` |
| `category_drone_desings_hercules` | `category\drone\desings\hercules` |
| `category_drone_flax` | `category\drone\flax` |
| `category_drone_formation` | `category\drone\formation` |
| `category_drone_formation_f-01-tu` | `category\drone\formation\f-01-tu` |

Nota: en el árbol real aparece el nombre `desings` (typo) y `especialammo` (typo); las rutas con imágenes usan `drone\designs` y `ammunition\specialammo`.

Las carpetas bajo `items/category` que corresponden a categorías **omitidas** del catálogo (`booster`, `design`, `package`, `pet`, `ship`, etc.) no se listan aquí; ver [items-catalog-omissions.md](items-catalog-omissions.md).
