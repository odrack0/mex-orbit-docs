# Plan: migración de datos legacy en `player_equipment` y retirada de columnas

Este documento define cómo **poblar** el modelo canónico en [sql-schema.md](./sql-schema.md) a partir de `player_equipment`, y cuándo **eliminar** columnas que ya no usen API / Game Server.

**Referencias:** [sql-schema.md](./sql-schema.md) §4–7, [inventory-equipment-modernization.md](../plans/inventory-equipment-modernization.md), catálogo [items-catalog.md](../../definitions/items-catalog.md).

---

## 0. Premisa (entorno actual)

**No hay carga productiva que exija conservar la configuración equipada.** Por tanto:

- **No se migran** las columnas **`config1_*`** ni **`config2_*`** (láseres, generadores, drones por bahía, heavy guns, extras). Es aceptable **perder** qué ítems estaban colocados en cada ranura.
- El trabajo útil de esta fase es **generar filas en `player_item`** a partir del JSON **`items`** ([`EquipmentItems`](../../../MexOrbit.Server/Infrastructure/MexOrbit.Infrastructure.Sql/ValueObjects/EquipmentItems.cs)): contadores y flags reflejan **posesión** (cuántas unidades tiene el jugador), no el loadout.
- **`player_equipment_slot`** no se rellena desde legacy en este paso (no hay datos de config que traducir); los jugadores volverán a equipar desde el cliente o flujos nuevos cuando existan.

Si más adelante hubiera requisito de PROD con config a preservar, habría que reabrir el diseño (mapeo índice L → `server_item` + slots).

---

## 1. Contexto

### 1.1 Tabla legacy actual

Orden de columnas típico:  
`userId`, `config1_lasers`, `config1_generators`, `config1_drones`, `config1_heavy_guns`, `config1_extras`, `config2_lasers`, `config2_generators`, `config2_drones`, `config2_heavy_guns`, `config2_extras`, `items`, `skill_points`, `modules`, `boosters`.

| Columna MySQL | Uso en *esta* migración |
|----------------|-------------------------|
| `config1_*`, `config2_*` | **Ignorar** (no migrar). |
| **`items`** | **Única fuente** para generar `player_item` (y reglas asociadas). |
| `skill_points`, `modules`, **`boosters`** | Sin cambio; [§1.2](#12-fuera-de-alcance-en-esta-fase-se-quedan-en-la-tabla). |

### 1.2 Fuera de alcance en esta fase (se quedan en la tabla)

**Decisión:** las columnas **`skill_points`**, **`modules`** y **`boosters`** **no** se migran al modelo nuevo en esta épica; **permanecen** en `player_equipment` (lectura/escritura según el código actual). El subobjeto **`skillTree`** dentro del JSON **`items`** se trata en la [Fase 6](#fase-6--skill-tree-módulos-y-boosters-futura).

**No** hay traslado de **`boosters`** a `player_resource_balance` de momento; se revisará cuando se defina el producto y el mapeo JSON → `server_item`.

Los `DROP COLUMN` **no** incluyen `skill_points`, `modules` ni `boosters` salvo plan futuro explícito.

### 1.3 Modelo destino (resumen)

| Necesidad | Tabla |
|-----------|--------|
| Catálogo | `server_item`, … |
| Posesión por contador legacy | **`player_item`** — N filas por N unidades (`item_id` → `server_item`) |
| Equipo en nave (configs 1 y 2) | `player_equipment_slot` — **vacío** tras migración solo desde `items`; se llena en runtime |
| Boosters (columna legacy) | **No** migrar a `player_resource_balance` en esta fase (ver §1.2) |

---

## 2. `items` (JSON) → `player_item`

### 2.1 Regla general

Por cada campo numérico de tipo **conteo** (`lf4Count`, `havocCount`, …): insertar **esa cantidad** de filas en `player_item` con el `item_id` que corresponda al **mismo** ítem en catálogo (`server_item` sembrado desde [items-catalog.md](../../definitions/items-catalog.md)).

Por **flags booleanos** (`apis`, `zeus`, `pet`): si es `true`, insertar **una** fila (o las que defina negocio) para el `server_item` del dron / ítem asociado; si es `false`, ninguna.

**No** migrar desde JSON a `player_item`: `ships`, `designs`, `petModules`, `skillTree` (quedan en JSON o en columnas satélite hasta otra fase).

### 2.2 Ejemplo real (dos cuentas)

Valores de ejemplo leídos de BD (solo `items` relevante para la migración de inventario):

**Usuario `1`:**

```json
{
  "lf4Count": 40,
  "havocCount": 10,
  "herculesCount": 10,
  "apis": true,
  "zeus": true,
  "pet": false,
  "petModules": [],
  "ships": [],
  "designs": {},
  "skillTree": { "logdisks": 0, "researchPoints": 0, "resetCount": 0 },
  "hst01Count": 2,
  "hst02Count": 2,
  "clkXlCount": 1,
  "albXCount": 1,
  "rbXCount": 1,
  "jp02Count": 1,
  "arolXCount": 1,
  "dr01Count": 1,
  "repSCount": 1,
  "hm07Count": 1,
  "smb01Count": 1,
  "aim02Count": 1
}
```

**Usuario `2`:** mismo contenido lógico; el orden de claves en JSON puede variar (p. ej. `skillTree` al final); el parser debe ser **por nombre de propiedad**, no por orden.

### 2.3 Mapeo conceptual (campos → categoría / `item_key` en catálogo)

Definir en el script una tabla interna **nombre JSON → `loot_id` o `server_item.id`**. Ejemplos alineados al catálogo actual:

| Campo `EquipmentItems` | Interpretación | Ejemplo en catálogo |
|------------------------|----------------|---------------------|
| `lf4Count` | N láseres LF-4 | `equipment_weapon_laser` / `lf-4` |
| `havocCount` | N diseños Havoc | `drone_designs` / `havoc` |
| `herculesCount` | N diseños Hercules | `drone_designs` / `hercules` |
| `apis` | Posee dron Apis | `drone` / `apis` |
| `zeus` | Posee dron Zeus | `drone` / `zeus` |
| `hst01Count`, `hst02Count` | Lanzacohetes | `equipment_weapon_rocketlauncher` / `hst-1`, `hst-2` |
| `clkXlCount`, `albXCount`, … | CPUs / extras | `equipment_extra_cpu` / `cl04k-xl`, `alb-x`, … (normalizar nombres a **minúsculas** y guiones como en el catálogo) |

Si un campo del JSON **no** tiene `server_item` en el seed, el script debe **loguear** y **omitir** o abortar según decisión.

### 2.4 Otras columnas legacy (no config)

| Origen | Destino | Notas |
|--------|---------|--------|
| `boosters` | Ninguno en esta fase | Columna intacta; sin migración a `player_resource_balance` por ahora. |
| `modules` | Ninguno en esta fase | Permanece en columna. |
| `skill_points` | Ninguno en esta fase | Permanece en columna. |

---

## 3. Fases del proyecto

### Fase 0 — Prerrequisitos

1. Catálogo sembrado (`seed_catalog_data.sql`) y listado de **todos** los `loot_id` necesarios para los campos de `EquipmentItems` que vamos a migrar.
2. Tabla de correspondencia **propiedad JSON → `server_item_id`** (o script que resuelva por `loot_id`).
3. Backup de BD si la política del equipo lo exige (aunque no sea PROD).

### Fase 1 — Script de migración

1. `SELECT userId, items FROM player_equipment` (no leer `config*` ni **`boosters`** para este script).
2. Por cada jugador: deserializar `items` (misma semántica que EF).
3. Insertar filas `player_item` según §2 (nivel por defecto acordado, p. ej. `1` o `0`).
4. Idempotencia: **truncar** `player_item` de prueba o usar marca de migración; evitar duplicar si se reejecuta el script (decisión: `DELETE`/`TRUNCATE` por entorno de desarrollo o migración única).

**Entregable:** `MexOrbit.Server/Scripts/YYYY.MM.DD.N/` con script o herramienta + `README.md`.

### Fase 2 — Validación

1. Por usuario de prueba: suma de filas `player_item` por `item_id` vs contadores esperados desde el JSON `items` original.
2. **No** validar contra `config*` (no migradas).
3. Staging: inventario desde **`player_item`**; `skill_points`, `modules` y **`boosters`** siguen en `player_equipment` si el código aún los lee.

### Fase 3 — Corte de código

1. API / Game Server: leer posesión y equipamiento desde **`player_item`** / **`player_equipment_slot`** (slots vacíos o rellenados solo por acciones nuevas).
2. Dejar de usar `items` como fuente de verdad de contadores una vez migrado el inventario.
3. `skill_points`, `modules`, **`boosters`** y `skillTree` en `items`: según §1.2 hasta Fase 6.

### Fase 4 — `DROP COLUMN`

Cuando no haya binarios que dependan de ellas, se pueden eliminar **en particular** las columnas **`config*_*`** y, si ya no se usa el JSON de posesión, **`items`** (solo si **`skillTree`** y el resto de datos necesarios ya están resueltos o se acepta pérdida; coordinar con §1.2).

**Excluir** de borrado hasta plan futuro: `skill_points`, `modules`, **`boosters`** (y `items` mientras contenga datos no migrados).

### Fase 5 — Limpieza EF y documentación

Ajustar entidad `PlayerEquipment`, `DbContext` y flujos en [equipment-and-game-server-sync.md](../../flows/equipment-and-game-server-sync.md).

### Fase 6 — Skill tree, módulos y boosters (futura)

Migración y modelo destino para **`skill_points`**, **`modules`**, **`skillTree`** en `items`, y traslado de la columna **`boosters`** a **`player_resource_balance`** (u otro diseño): plan aparte.

---

## 4. Riesgos y mitigación

| Riesgo | Mitigación |
|--------|------------|
| Campo JSON sin `server_item` en catálogo | Inventario de mapeos + tests; log por jugador/campo. |
| Reejecutar script y duplicar `player_item` | Truncado en dev o flag de migración única. |
| `items` se dropea antes de mover `skillTree` | No dropear `items` hasta Fase 6 o copia de respaldo. |

---

## 5. Checklist de salida

- [ ] Tabla JSON → `server_item` completa para los campos de `EquipmentItems` que se migran.
- [ ] Script ejecutado en entorno de desarrollo/staging; totales `player_item` coherentes con `items`.
- [ ] Código sin depender de contadores dentro de `items` para inventario (post-corte); **`boosters`** sigue en `player_equipment` hasta la Fase 6.
- [ ] `DROP` de columnas `config*` (y opcionalmente `items`) documentado en `rollout`/`rollback` cuando corresponda.

---

*Plan vivo: sin migración de config ni de la columna `boosters`; foco en inventario desde `items` → `player_item`; skill tree, módulos y boosters en fase posterior.*
