# Modelo de datos: catálogo de ítems, posesión y equipamiento

**Definición canónica** para MexOrbit: catálogo (`server_*`), instancias (`player_item`), stats, equipamiento en ranuras (`player_equipment_slot`) y **saldos acumulables** (`player_resource_balance` → `server_item`). Los tipos de recurso **no** se duplican en una tabla aparte: usan el **mismo catálogo** que [definitions/items-catalog.md](../../definitions/items-catalog.md) (categorías `resource`, `resource_ore`, `resource_blueprint`, `resource_wordpuzzle-letter`, etc.). **No hay columnas de compatibilidad con el inventario Flash** en el esquema objetivo: el equipamiento referencia **solo** `player_item_id`.

Las migraciones SQL versionadas viven en `MexOrbit.Server/Scripts/YYYY.MM.DD.N/` con `rollout.sql` y `rollback.sql`, según [Scripts/README.md](../../../MexOrbit.Server/Scripts/README.md).

**Referencias:** [definitions/items-catalog.md](../../definitions/items-catalog.md), [plans/inventory-equipment-modernization.md](../plans/inventory-equipment-modernization.md), [player-equipment-legacy-migration.md](./player-equipment-legacy-migration.md) (migración desde `player_equipment`, validación y `DROP COLUMN`).

**Premisa:** no hay entorno productivo que exija convivencia larga con el modelo antiguo. Los datos viejos (`player_equipment.items`, `config*_*`, índices de loot) se consumen **únicamente en scripts de migración** que poblan `player_item` y `player_equipment_slot`; la aplicación en runtime **no** implementa fallback ni columnas legacy en las tablas nuevas.

**Convención temporal:** `created_at` NOT NULL; `updated_at` al modificar (alineado con `BaseEntity` en `Infrastructure.Sql/Entities/BaseEntity.cs`).

---

## 1. Alcance

| Área | Tablas |
|------|--------|
| Categorías y definición de ítems | `server_item_category`, `server_item`, `server_item_stat_type`, `server_item_stat` |
| Posesión por jugador | `player_item`; overrides `player_item_stat` (si aplica negocio) |
| Equipo en nave (configs 1 y 2) | `player_equipment_slot` (FK a `player_item`) |
| Saldos no instancia | `player_resource_balance` (FK a `server_item` del catálogo de recursos) |

**Reglas de producto:**

- Una fila en `player_item` = **una unidad** (sin `quantity` agregada).
- Cada fila en `player_equipment_slot` apunta a **una** instancia `player_item` equipada en esa ranura (`player_item_id` NOT NULL).
- **Cantidades acumulables** (moneda, **munición**, materiales, xenomita, seprom, etc.) viven en **`player_resource_balance`** por `server_item_id`; no como cientos de filas `player_item` ni como entradas en `player_equipment_slot`.
- Los **`server_item`** con `is_wallet_balance = 1` son los que usan esa tabla de saldo; el resto se representan como instancias `player_item` cuando aplica (equipo).

---

## 2. Principios de diseño

- **Categorías:** `server_item_category` con `code` estable (p. ej. `equipment_weapon_laser`, `ammunition_laser`; el subconjunto sembrado sigue [items-catalog.md](../../definitions/items-catalog.md)).
- **Ítems:** `item_key` + `loot_id` único global para API, Game Server y cliente.
- **Stats:** `server_item_stat_type` (`code`, `value_kind`) + `server_item_stat` con `DECIMAL` fijo.
- **Slots:** una tabla `player_equipment_slot` con `slot_kind` acotado; referencia directa a instancias mediante `player_item_id`.
- **Saldos:** moneda, munición y recursos acumulables comparten **`player_resource_balance`**; el catálogo (`server_item` + `is_wallet_balance`) distingue qué ítems van por saldo frente a instancias de equipo.

---

## 3. Catálogo y posesión

### 3.1 `server_item_category`

| Columna | Tipo | Notas |
|---------|------|-------|
| `id` | `INT UNSIGNED` PK AUTO_INCREMENT | |
| `code` | `VARCHAR(128)` NOT NULL UNIQUE | Slug estable. |
| `parent_id` | `INT UNSIGNED` NULL FK → `server_item_category(id)` | Opcional. |
| `display_name` | `NVARCHAR(255)` NULL | |
| `sort_order` | `INT NOT NULL DEFAULT 0` | |
| `created_at` | `DATETIME(6)` NOT NULL | |
| `updated_at` | `DATETIME(6)` NULL | |

### 3.2 `server_item_stat_type`

| Columna | Tipo | Notas |
|---------|------|-------|
| `id` | `SMALLINT UNSIGNED` PK AUTO_INCREMENT | |
| `code` | `VARCHAR(64)` NOT NULL UNIQUE | P. ej. `WEAPON_DAMAGE_FLAT`. |
| `value_kind` | `VARCHAR(16)` NOT NULL | `flat`, `percent`, `multiplier`. |
| `display_name` | `NVARCHAR(255)` NULL | |
| `created_at` | `DATETIME(6)` NOT NULL | |
| `updated_at` | `DATETIME(6)` NULL | |

### 3.3 `server_item`

| Columna | Tipo | Notas |
|---------|------|-------|
| `id` | `INT UNSIGNED` PK AUTO_INCREMENT | |
| `category_id` | `INT UNSIGNED` NOT NULL FK → `server_item_category(id)` | |
| `item_key` | `VARCHAR(128)` NOT NULL | P. ej. `lf-3`, `havoc`. |
| `loot_id` | `VARCHAR(191)` NOT NULL UNIQUE | Identificador global; seed según plan de inventario. **191** = límite práctico para índice único con `utf8mb4` en InnoDB (767 bytes; 191×4=764). |
| `friendly_name` | `NVARCHAR(255)` NULL | |
| `description` | `TEXT` NULL | |
| `max_owned` | `INT UNSIGNED` NULL | NULL = sin tope en BD. |
| `is_wallet_balance` | `TINYINT(1)` NOT NULL DEFAULT 0 | `1` = la cantidad poseída se guarda en **`player_resource_balance`** (moneda de juego, **munición**, materiales, xenomita, seprom, prometium, ofertas **`deal`**, boosters **`equipment_booster`**, etc.). `0` = el ítem se representa con **filas `player_item`** (equipo equipable: láseres, generadores, CPUs, diseños de dron, …). **No** hay columna `stackable`: todo lo que antes se consideraba “apilable” como número único pasa por saldo cuando `is_wallet_balance = 1`. |
| `created_at` | `DATETIME(6)` NOT NULL | |
| `updated_at` | `DATETIME(6)` NULL | |

**Restricción:** UNIQUE (`category_id`, `item_key`).

**Seed:** categorías e ítems alineados a [items-catalog.md](../../definitions/items-catalog.md). Marcar `is_wallet_balance = 1` en munición (`ammunition_*`), recursos (`resource`, `resource_ore`, …), **`deal`**, **`equipment_booster`** y monedas según diseño. El resto de categorías **`equipment_*`** (láser, generador, CPU, etc.) suelen llevar `0`. Monedas u otros códigos no listados en assets se modelan como `server_item` con `loot_id` estable.

### 3.4 `server_item_stat`

| Columna | Tipo | Notas |
|---------|------|-------|
| `id` | `BIGINT UNSIGNED` PK AUTO_INCREMENT | |
| `item_id` | `INT UNSIGNED` NOT NULL FK → `server_item(id)` ON DELETE CASCADE | |
| `stat_type_id` | `SMALLINT UNSIGNED` NOT NULL FK → `server_item_stat_type(id)` | |
| `value` | `DECIMAL(18,6)` NOT NULL | |
| `created_at` | `DATETIME(6)` NOT NULL | |
| `updated_at` | `DATETIME(6)` NULL | |

**Restricción:** UNIQUE (`item_id`, `stat_type_id`).

### 3.5 `player_item`

| Columna | Tipo | Notas |
|---------|------|-------|
| `id` | `BIGINT UNSIGNED` PK AUTO_INCREMENT | |
| `user_id` | `INT NOT NULL` | FK a cuenta; misma semántica que `player_equipment.userId`. |
| `item_id` | `INT UNSIGNED` NOT NULL FK → `server_item(id)` | |
| `level` | `SMALLINT UNSIGNED` NOT NULL | |
| `origin` | `VARCHAR(64)` NULL | |
| `bound` | `TINYINT(1)` NULL | |
| `created_at` | `DATETIME(6)` NOT NULL | |
| `updated_at` | `DATETIME(6)` NULL | |

**Índices:** `user_id`; `item_id`; (`user_id`, `item_id`).

### 3.6 `player_item_stat`

| Columna | Tipo | Notas |
|---------|------|-------|
| `id` | `BIGINT UNSIGNED` PK AUTO_INCREMENT | |
| `player_item_id` | `BIGINT UNSIGNED` NOT NULL FK → `player_item(id)` ON DELETE CASCADE | |
| `stat_type_id` | `SMALLINT UNSIGNED` NOT NULL FK → `server_item_stat_type(id)` | |
| `value` | `DECIMAL(18,6)` NOT NULL | |
| `created_at` | `DATETIME(6)` NOT NULL | |
| `updated_at` | `DATETIME(6)` NULL | |

**Restricción:** UNIQUE (`player_item_id`, `stat_type_id`).

---

## 4. Equipamiento: `player_equipment_slot`

Cada fila = **una** instancia `player_item` colocada en una ranura de la config de nave (`ship_config_id` 1 o 2). No existe columna de índice legacy ni referencia al formato Flash en esta tabla.

### 4.1 Qué absorbía el modelo antiguo (solo entrada para scripts de migración)

Antes, `config1_lasers`, `config1_drones`, etc., serializaban listas de enteros; `items` llevaba contadores y otros datos. Eso **no** se persiste en el esquema nuevo. Los scripts de migración leen esas fuentes **una vez**, materializan `player_item` y luego insertan en `player_equipment_slot`.

| Origen antiguo (migración) | Destino |
|----------------------------|---------|
| Contadores / posesión agregada | `player_item` (+ `server_item`) |
| Listas en `config*_*` | `player_equipment_slot` con `player_item_id` |
| Flags, skill tree, naves | Tablas satélite o campos acordados fuera de este documento |

### 4.2 Definición de `player_equipment_slot`

| Columna | Tipo | Descripción |
|---------|------|-------------|
| `id` | `BIGINT UNSIGNED` PK AUTO_INCREMENT | |
| `user_id` | `INT NOT NULL` | Jugador. |
| `ship_config_id` | `TINYINT UNSIGNED` NOT NULL | `1` o `2`. |
| `slot_kind` | `VARCHAR(32)` NOT NULL | `laser`, `generator`, `heavy_gun`, `extra`, `drone_item`, `drone_design`. |
| `sort_order` | `SMALLINT UNSIGNED` NOT NULL | Orden dentro del grupo. |
| `drone_slot_index` | `TINYINT UNSIGNED` NOT NULL | En DDL MySQL: **`255`** = ranura de nave (no dron); **`0`–`9`** = bahía de dron (evita ambigüedad de `NULL` en índices `UNIQUE`). |
| `player_item_id` | `BIGINT UNSIGNED` NOT NULL FK → `player_item(id)` | Instancia equipada. `ON DELETE RESTRICT` o comportamiento definido en aplicación (desequipar antes de borrar). |
| `created_at` | `DATETIME(6)` NOT NULL | |
| `updated_at` | `DATETIME(6)` NULL | |

**Unicidad:**

- `slot_kind` ∈ {`laser`, `generator`, `heavy_gun`, `extra`} y `drone_slot_index` IS NULL: UNIQUE (`user_id`, `ship_config_id`, `slot_kind`, `sort_order`).
- `slot_kind` ∈ {`drone_item`, `drone_design`}: UNIQUE (`user_id`, `ship_config_id`, `drone_slot_index`, `slot_kind`, `sort_order`).

**Índices:** (`user_id`, `ship_config_id`); (`player_item_id`).

### 4.3 `userId` en `player_equipment`

La PK `userId` debe coincidir con el id de cuenta del jugador (sin autoincremento incorrecto). Alinear con EF (`ValueGeneratedNever`).

### 4.4 Drones: bahías y reglas de producto

- Hasta **10** bahías: `drone_slot_index` `0`–`9`.
- Equipo por dron: filas `drone_item` con `sort_order` 0, 1, …; diseño: `drone_design` (p. ej. `sort_order` 0).
- **Tipo** de dron por bahía (Flax / Iris / Apis / Zeus) no está en `slot_kind`; puede documentarse en una tabla `player_drone_bay` o equivalente en la misma épica.
- Reglas (8+1+1, exclusión gen/CPU/láser en un hueco) se validan en servidor según categoría de `server_item`.

---

## 5. Recursos (saldos no instancia)

Monedas, **munición**, materiales y cualquier cantidad que **no** sea una instancia de equipo en ranura se modelan con **`player_resource_balance`** apuntando al **`server_item`** correspondiente (mismo catálogo CMS: `ammunition_*`, `resource`, `resource_ore`, etc.). Incluye lo que en otros diseños sería “inventario apilable”.

**No existe `server_resource_kind`.** El “código” del recurso es el **`server_item`** (p. ej. `item_key` `seprom`, `loot_id` estable como `resource_ore_seprom` o el acordado en seed), evitando duplicar `URIDIUM` / `SEPROM` en otra tabla.

Contexto de producto: [plans/inventory-equipment-modernization.md](../plans/inventory-equipment-modernization.md) (sección 3.3), con tipos de recurso unificados en `server_item`.

### 5.1 `player_resource_balance`

Saldo por jugador y por **ítem de catálogo** que representa un recurso acumulable. Una fila = par **(jugador, `server_item`)** con cantidad actual.

| Columna | Tipo | Notas |
|---------|------|-------|
| `id` | `BIGINT UNSIGNED` PK AUTO_INCREMENT | Surrogate; alternativa: PK compuesta (`user_id`, `server_item_id`). |
| `user_id` | `INT NOT NULL` | FK a cuenta / jugador; misma semántica que `player_equipment.userId`. |
| `server_item_id` | `INT UNSIGNED` NOT NULL FK → `server_item(id)` | Solo ítems con **`is_wallet_balance = 1`** (moneda, munición, recursos acumulables). |
| `amount` | `DECIMAL(20,6)` NOT NULL DEFAULT 0 | Cantidad actual; si solo enteros, `BIGINT UNSIGNED` es válido. |
| `created_at` | `DATETIME(6)` NOT NULL | |
| `updated_at` | `DATETIME(6)` NOT NULL | |

**Restricción:** UNIQUE (`user_id`, `server_item_id`).

**Índices:** `user_id`; `server_item_id`.

**Reglas:**

- Equipo equipable (láser, gen, diseño de dron, …) → **`player_item`** + `player_equipment_slot`; **no** filas de saldo por unidad.
- Moneda, **munición**, xenomita, seprom, etc. → **`player_resource_balance`**; el `server_item` debe tener **`is_wallet_balance = 1`**.
- Validar en aplicación que solo se inserte/actualice saldo para `server_item` con `is_wallet_balance = 1`.

---

## 6. `player_equipment` después de migrar

Sin producción que lo impida, el **rollout** puede, tras poblar las tablas nuevas y validar:

- Eliminar columnas **`items`**, **`config1_*`**, **`config2_*`** (u otras solo usadas por el inventario Flash), **o** truncar/vaciar y dejar la fila mínima por jugador si otras columnas (`skill_points`, `modules`, …) siguen en uso.

Esa decisión concreta va en el `README.md` del script de versión. El **rollback** restaura columnas y tipos según el `rollback.sql` del mismo carpeta.

La aplicación y el Game Server **solo** leen/escriben `player_item`, `player_equipment_slot`, `player_resource_balance`, etc.; **no** hay lógica de fallback a JSON en runtime.

---

## 7. Scripts de migración (contenido esperado)

**Implementación de referencia:** `MexOrbit.Server/Scripts/2026.04.12.1/` (`rollout.sql`, `seed_catalog_data.sql` generado desde `items-catalog.md`, `generate-catalog-seed.mjs`, `rollback.sql`, `README.md`).

Cada versión en `MexOrbit.Server/Scripts/YYYY.MM.DD.N/` debe incluir:

| Artefacto | Contenido |
|-----------|-----------|
| `rollout.sql` | `CREATE TABLE` del esquema anterior (si aún no existe), `ALTER`/`CREATE` de tablas nuevas, **seed** de `server_item_category` / `server_item` (incl. recursos según [items-catalog.md](../../definitions/items-catalog.md)), migración desde `player_equipment` y cuentas, **inserción** en `player_item`, `player_equipment_slot` y `player_resource_balance`, y opcionalmente `DROP COLUMN` o tabla sustituida. |
| `rollback.sql` | Revierte cambios de esquema de forma segura (recrear columnas, tipos, índices) según lo documentado en el README de la versión. |
| `README.md` | Qué hace el script, orden de ejecución, dependencias, advertencias (backup de BD). |

**Orden lógico sugerido dentro del rollout:**

1. Crear tablas `server_*`, `player_item`, `player_item_stat`, `player_equipment_slot`, `player_resource_balance`.
2. Poblar catálogo (`server_item`, stats) desde definiciones acordadas ([items-catalog.md](../../definitions/items-catalog.md) + reglas de juego).
3. Por cada jugador con `player_equipment` antiguo: parsear JSON / columnas; crear filas `player_item` necesarias; insertar `player_equipment_slot` con `player_item_id` resuelto.
4. Opcional: `ALTER TABLE player_equipment DROP COLUMN ...` para columnas sustituidas.
5. Actualizar código (EF, API, Game Server) para usar solo el modelo nuevo **antes** o **en el mismo despliegue** que el script.

No se añaden columnas `legacy_*` a las tablas nuevas: la única “lógica legacy” vive en el **script de migración** (lectura puntual de datos viejos).

---

## 8. Columnas `created_at` y `updated_at`

| Tabla | `created_at` | `updated_at` |
|-------|--------------|--------------|
| `server_item_category` | Obligatorio | Al actualizar |
| `server_item_stat_type` | Obligatorio | Al actualizar |
| `server_item` | Obligatorio | Al actualizar |
| `server_item_stat` | Obligatorio | Al actualizar |
| `player_item` | Obligatorio | Al actualizar |
| `player_item_stat` | Obligatorio | Al actualizar |
| `player_equipment_slot` | Obligatorio | Al actualizar |
| `player_resource_balance` | Obligatorio | Al actualizar |

---

## 9. Nomenclatura (referencia)

| Origen / documento antiguo | Tabla en este modelo |
|-----------------------------|----------------------|
| `item_catalog` | `server_item` |
| `stat_kind` | `server_item_stat_type` |
| `item_catalog_stat` | `server_item_stat` |
| `resource_kind` (diseño antiguo) | Saldos → `player_resource_balance.server_item_id` → `server_item` del catálogo (`resource`, `resource_ore`, …) |
| Columnas JSON `config*_*` | `player_equipment_slot` |
| JSON `items` (posesión agregada) | `player_item` + otras tablas según dominio |

---

*Documento canónico del modelo; los ejecutables son los scripts en `MexOrbit.Server/Scripts/`.*
