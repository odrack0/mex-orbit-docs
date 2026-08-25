# Plan: migrar el Game Server a `player_equipment_slot`

**Objetivo:** que **`MexOrbit.GameServer`** deje de derivar daño, escudo, velocidad y contribución de drones a partir de las columnas JSON **`config1_*` / `config2_*`** (y listas anidadas en **`config*_drones`**) en **`player_equipment`**, y pase a usar la verdad de datos ya persistida por la API en **`player_equipment_slot`** + **`player_item`** + **`server_item`** (+ **`server_item_stat`** cuando haga falta).

**Referencias:**

- Esquema y semántica de ranuras: [sql-schema.md](./sql-schema.md) (§4).
- Sincronización API ↔ GS (`UpdateStatus`, `SetEquipment`): [flows/equipment-and-game-server-sync.md](../../flows/equipment-and-game-server-sync.md).
- Fórmulas deseadas de stats: [flows/equipment-stats-calculation.md](../../flows/equipment-stats-calculation.md) (mantener o sustituir heurísticas actuales del GS de forma explícita).
- Reglas de negocio en API (ya aplicadas al escribir): `ShipEquipmentService.cs` (no el GS).

**Estado actual (resumen):**

- `QueryManager.SetEquipment` lee `SELECT * FROM player_equipment` y usa **rangos numéricos** sobre listas en `config{i}_lasers`, `config{i}_generators`, `config{i}_drones` (diseños Havoc/Hercules, módulos por bahía).
- Eso **no** refleja el modelo por instancia (`player_item_id`) ni el catálogo (`loot_id`, categorías, stats en BD).

**Estado objetivo:**

- Para cada **config activa** (1 y 2), cargar filas de **`player_equipment_slot`** con `user_id = player.Id`, `ship_config_id IN (1,2)`, y filtrar por `drone_slot_index` (p. ej. **255** = nave; **0–9** = bahía de dron según [sql-schema.md](./sql-schema.md) y entidad EF).
- Hacer **JOIN** (o consultas en dos pasos) a **`player_item`** → **`server_item`** (y opcionalmente **`server_item_stat`**) para obtener tipo de ítem y valores numéricos de daño/escudo/velocidad en lugar de constantes hardcodeadas (`lf3Damage`, `bo2Shield`, etc.).
- Mantener en `player_equipment` **solo** lo que el GS siga necesitando (p. ej. `boosters`, `skill_points`, `modules`, JSON `items` para pet/flags) hasta que esas piezas tengan tabla propia; **no** usar `config*_*` para stats de combate de nave/dron.

---

## 1. Principios de diseño

1. **Una sola fuente de stats para combate:** los mismos `server_item` / stats que ve el CMS deben alimentar `EquipmentBase` / `ConfigsBase` en memoria.
2. **Sin duplicar reglas de negocio de exclusión** (SLE, mutex CPU/REP, etc.) en el GS: la BD ya refleja solo combinaciones válidas; el GS **calcula números**, no revalida inventario.
3. **Config 1 y 2:** el código actual itera `i = 1..2`; debe seguir rellenando `damage[0..1]`, `shield[0..1]`, `speed[0..1]`, `hitpoints[0..1]` según el diseño de `player.Equipment`.
4. **Drones:** las filas `drone_item` / `drone_design` con `drone_slot_index` 0–9 y `drone_player_item_id` (casco) deben agruparse por bahía como hoy el JSON anidado; revisar `DroneManager` y cualquier lectura directa de `config*_drones` en el GS.
5. **Rendimiento:** una o pocas consultas parametrizadas por `user_id` (evitar N+1 por slot); índices en `player_equipment_slot` ya previstos en el esquema.

---

## 2. Inventario de puntos de código a tocar (Game Server)

| Área | Archivo / símbolo | Nota |
|------|-------------------|------|
| Stats principal nave | `Managers/QueryManager.cs` — `SetEquipment` | Sustitución principal. |
| Drones en memoria | `Game/Objects/Players/Managers/DroneManager.cs` (y usos de `config*_drones`) | Alinear con slots + casco por `drone_player_item_id`. |
| Socket / login | `Net/SocketServer.cs` — `UpdateStatus` | Tras migración, sigue llamando `SetEquipment`; sin cambio de contrato si `SetEquipment` lee slots. |
| Login | `Net/netty/handlers/LoginRequestHandler.cs` | Verificar que `GetPlayer` → `SetEquipment` sea suficiente. |

Buscar en el repo: `config1_`, `config2_`, `player_equipment` dentro de `MexOrbit.GameServer` para no dejar lecturas huérfanas.

---

## 3. Modelo de datos que el GS debe consultar

### 3.1 Tablas mínimas

- **`player_equipment_slot`:** `user_id`, `ship_config_id`, `slot_kind`, `sort_order`, `drone_slot_index`, `player_item_id`, `drone_player_item_id` (si aplica en EF).
- **`player_item`:** `id`, `item_id`, `user_id`, `level`.
- **`server_item`:** `id`, `loot_id`, `item_key`, `category_id`.
- **`server_item_category`:** `code` (p. ej. `equipment_weapon_laser`, `equipment_generator_shield`).
- Opcional: **`server_item_stat`** + **`server_item_stat_type`** para daño/escudo/velocidad por ítem en lugar de constantes en C#.

### 3.2 Mapeo conceptual (reemplazo de heurísticas)

| Antes (legacy en `SetEquipment`) | Después |
|----------------------------------|--------|
| Rango `itemId` 0–39 / ≥140 para láser | `slot_kind = laser` + categoría / `loot_id` / stat de daño |
| Rango 40–99 / 100–119 para generadores | `slot_kind = generator` + tipo escudo vs velocidad vía categoría o stats |
| JSON `drones[].designs` / `items` | Agrupar por `drone_slot_index` + `drone_design` / `drone_item` |

Documentar en un cuadro interno (o en [equipment-stats-calculation.md](../../flows/equipment-stats-calculation.md)) la **tabla de decisión** por `server_item_category.code` o por `loot_id` para no dispersar `if` por todo el GS.

---

## 4. Fases de implementación

### Fase 0 — Preparación

- [ ] Congelar lista de **stat types** (`server_item_stat_type`) necesarios para combate (daño láser, escudo, velocidad, bonos de diseño de dron, etc.).
- [ ] Verificar seed de catálogo: todo ítem equipable relevante tiene stats o reglas de fallback claras.
- [ ] Añadir pruebas de datos: jugador de prueba con loadout conocido en `player_equipment_slot` (puede reutilizar scripts de `MexOrbit.Server/Scripts/`).

### Fase 1 — Capa de acceso a datos en el Game Server

- [ ] Introducir clase (p. ej. `EquipmentSlotRepository` o métodos estáticos en `QueryManager`) que ejecute **una** consulta SQL parametrizada:
  - `SELECT` slots del jugador + `JOIN player_item` + `JOIN server_item` + `JOIN server_item_category`
  - Opcional: `LEFT JOIN server_item_stat` / tipos de stat.
- [ ] **No** usar concatenación de `player.Id` en SQL sin sanitizar; usar parámetros del cliente MySQL existente.

### Fase 2 — Nave: láseres y generadores (config 1 y 2)

- [ ] Para `slot_kind` `laser` / `generator` con `drone_slot_index = nave` (255), acumular stats en el array por `ship_config_id`.
- [ ] Eliminar dependencia de `config{i}_lasers` y `config{i}_generators` para **estas** contribuciones.
- [ ] Paridad: comparar números viejos vs nuevos en entorno de prueba para un mismo loadout (o aceptar diferencia si el catálogo es la nueva verdad).

### Fase 3 — Heavy guns y extras

- [ ] `heavy_gun` y `extra` según categoría (rockets, CPUs que afecten combate si el diseño lo exige).
- [ ] Si hoy no aportan a `ConfigsBase`, documentar “sin efecto en stats” o implementar según [equipment-stats-calculation.md](../../flows/equipment-stats-calculation.md).

### Fase 4 — Drones (bahías 0–9)

- [ ] Agrupar filas por `drone_slot_index` y `drone_player_item_id` (casco).
- [ ] Aplicar bonos de diseño (Havoc/Hercules) y módulos desde `server_item` + stats, equivalentes a los bloques `havocCount` / `herculesCount` / bucles actuales.
- [ ] Actualizar `DroneManager` para que no asuma el shape JSON de `config*_drones`.

### Fase 5 — Bonos globales (`speed` +20 %, etc.)

- [ ] El código actual aplica `Maths.GetPercentage(speed[0/1], 20)` a **toda** la config; decidir si es regla de nave, de ítem, o error y **dejar explícito** en código o en datos (por ejemplo solo si hay ítem X equipado).

### Fase 6 — Limpieza

- [ ] Dejar de leer `config1_*` / `config2_*` en `SetEquipment` para stats; si las columnas ya no existen en BD, eliminar deserialización.
- [ ] Mantener lectura de otras columnas de `player_equipment` que aún se usen (`boosters`, etc.) hasta su migración.
- [ ] Actualizar [equipment-and-game-server-sync.md](../../flows/equipment-and-game-server-sync.md) §1, §7 y §9 cuando el GS esté alineado.

---

## 5. Pruebas y validación

- **Unitarias (opcional):** funciones puras que, dado un conjunto de filas “fake” de slots + catálogo, produzcan los mismos enteros que el motor espera.
- **Integración:** conectar GS a BD de prueba; cargar jugador; verificar `player.Equipment` / paquetes tras `UpdateStatus`.
- **Regresión:** escenario jugador online en mapa → cambio de equipo por CMS → `UpdateStatus` → stats en cliente o logs coherentes.

---

## 6. Riesgos y mitigaciones

| Riesgo | Mitigación |
|--------|------------|
| Paridad numérica distinta al cliente antiguo | Acordar “catálogo manda”; documentar diferencias. |
| Consultas pesadas en cada `SetEquipment` | Una query agregada; caché por tick si hiciera falta (solo si medición lo exige). |
| Drones sin casco en fila (modelo API) | Seguir reglas de `ShipEquipmentService` (casco solo `player_item`); el GS debe reflejar el mismo modelo. |
| Columnas `player_equipment` eliminadas antes de tiempo | Orden: despliegue GS leyendo slots → luego script `DROP COLUMN` en `player_equipment` (alineado con [sql-schema.md](./sql-schema.md) §6). |

---

## 7. Orden sugerido de despliegue

1. Desplegar Game Server que **lee slots** pero **sigue** leyendo columnas antiguas si un flag de configuración está activo (opcional, solo si necesitáis rollback rápido).
2. Validar en staging.
3. Quitar flag / quitar lectura legacy.
4. Ejecutar migración SQL de columnas obsoletas si aplica.

---

## 8. Seguimiento

- Marcar este documento como **en progreso** y actualizar las casillas de las fases al cerrar.
- Cuando el GS solo use slots para stats de nave/dron, **cerrar** el ítem en el plan de producto “alinear GS con hangar REST”.

*Creado para el directorio `docs/implementation/items & equipment/`; revisar rutas de código si se reorganiza `MexOrbit.GameServer`.*
