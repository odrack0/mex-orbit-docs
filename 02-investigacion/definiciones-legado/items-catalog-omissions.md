# Omisiones del catálogo de ítems (CMS)

Este documento complementa [items-catalog.md](items-catalog.md). Lista qué carpetas y categorías del árbol `do_img/global/items` **no** forman parte del catálogo sembrado en BD, y el motivo.

**Estado:** provisional. Lo omitido puede volver a incorporarse cuando tienda / servidor lo requieran.

**Incluidas de nuevo en el catálogo:** `equipment_booster`, `drone_designs`, `equipment_petgear` (están en el seed y en el detalle del catálogo).

---

## 1. Categorías de primer nivel excluidas

| Código omitido | Carpeta en `items/` | Notas |
|----------------|---------------------|--------|
| `booster` | `booster` | Carpeta genérica de boosters en raíz; no confundir con **`equipment_booster`** (`equipment/booster`), que **sí** está en el catálogo. |
| `design` | `design` | Diseños genéricos en raíz; fuera del alcance actual. |
| `items_root` | (raíz de `items`) | Duplicaba ítems de subcarpetas. |
| `package` | `package` | Paquetes / bundles. |
| `pet` | `pet` | Mascotas; no confundir con **`equipment_petgear`**, que **sí** está en el catálogo. |

---

## 2. Todo el árbol bajo `ship/`

No hay ninguna categoría `ship_*` en [items-catalog.md](items-catalog.md). Quedan fuera **todas** las rutas bajo `items/ship/`, incluidas:

| Código omitido | Carpeta en `items/` |
|----------------|---------------------|
| `ship` | `ship` (raíz) |
| `ship_a-elite_design` | `ship/a-elite/design` |
| `ship_aegis_design` | `ship/aegis/design` |
| `ship_bigboy_design` | `ship/bigboy/design` |
| `ship_citadel_design` | `ship/citadel/design` |
| `ship_cyborg_design` | `ship/cyborg/design` |
| `ship_diminisher_design` | `ship/diminisher/design` |
| `ship_g-champion_design` | `ship/g-champion/design` |
| `ship_g-surgeon_design` | `ship/g-surgeon/design` |
| `ship_goliath_design` | `ship/goliath/design` |
| `ship_hammerclaw_design` | `ship/hammerclaw/design` |
| `ship_model` | `ship/model` |
| `ship_sentinel_design` | `ship/sentinel/design` |
| `ship_spearhead_design` | `ship/spearhead/design` |
| `ship_spectrum_design` | `ship/spectrum/design` |
| `ship_vengeance_design` | `ship/vengeance/design` |

---

## 3. Ítems en categorías que sí se listan

| Categoría | Ítem omitido | Motivo |
|-----------|----------------|--------|
| `deal` | `pet` | Enlace visual a la carpeta `pet/` omitida. |

---

## 4. Apéndice `items/category` (carpetas vacías)

En la tabla “Carpetas reservadas sin archivos” de [items-catalog.md](items-catalog.md) **no** se documentan taxonomías vacías que solo duplican carpetas omitidas (p. ej. `category\booster` frente a `booster/`, `category\desing`, `category\package`, `category\pet`, rutas bajo `ship\`, etc.). Las filas `category\drone\desings\…` **sí** aparecen allí porque corresponden a **`drone_designs`** en el catálogo principal.

---

## 5. Regenerar el seed

```text
node MexOrbit.Server/Scripts/2026.04.12.1/generate-catalog-seed.mjs
```
