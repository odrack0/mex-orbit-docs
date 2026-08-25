# Plan: modernización de inventario, catálogo de ítems y equipamiento (MexOrbit “normal”)

Este documento define un **plan de implementación por fases** para sustituir el modelo actual (orientado al inventario Flash legacy y columnas JSON densas) por un diseño **más escalable**, con **API REST coherente**, **tablas relacionales en MySQL** y **lectura alineada en el Game Server**, manteniendo el flujo de sincronización documentado en [equipment-and-game-server-sync.md](../../flows/equipment-and-game-server-sync.md) y el contexto de login en [game-client-initialization.md](../../flows/game-client-initialization.md).

---

## 1. Estado actual (resumen del análisis)

### 1.1 Base de datos y dominio (EF Core)

- **`player_equipment`** concentra casi todo: listas de IDs por configuración (`config1_lasers`, `config1_generators`, `config1_drones`, …), objeto **`EquipmentItems`** serializado en columna **`items`** (conteos por tipo: `Lf4Count`, `HavocCount`, flags de naves/diseños, etc.), JSON de **`modules`**, **`boosters`**, **`skill_points`**, etc.
- **`PlayerEquipment`** (`Infrastructure.Sql/Entities/PlayerEquipment.cs`) y **`EquipmentItems`** (`ValueObjects/EquipmentItems.cs`) mezclan **semántica de negocio** (conteos por “tipo de loot”) con **presentación hangar** en `EquipmentService`.
- No existe hoy un **catálogo relacional** de ítems: los “tipos” se infieren por **rangos numéricos de IDs** en el Game Server (p. ej. LF-3 vs LF-4) y por lógica duplicada entre API y servidor.

### 1.2 API

- **`EquipmentApiController`** expone rutas REST (`GET /api/equipment/init`, `POST /api/equipment/move`, …) pero en la práctica **delega en `EquipmentService`** pensado para compatibilidad Flash: muchas respuestas siguen siendo **Base64 de JSON** que el controlador **decodifica** para Vue (`DecodeBase64Response`).
- **`EquipmentService`** es **muy grande** y contiene construcción de arrays/objetos alineados al cliente legacy (`BuildInitResponse`, `BuildItemsArray`, movimientos tipo Flash `MoveItemsAsync` con `FlashInventoryMoveParams`, etc.).
- **`FlashInventoryController`** sigue existiendo para rutas legacy; el inventario “web” y el “Flash” comparten servicio, lo que **aumenta acoplamiento**.

### 1.3 Frontend CMS (Vue — `MexOrbit.CMS`)

- **Stack:** Vue 3, Pinia, Vite, Axios (`src/services/equipmentService.ts`). Autenticación con JWT en `Authorization` (token en `localStorage`); interceptor 401 redirige a `/login`.
- **Ruta del hangar:** `GET /hangar` (`HangarView.vue`) monta **`EquipmentPanel`** — sustituto web del cliente Flash `inventory.swf` (comentario en vista).
- **Contrato con backend:** el CMS **no** consume aún un inventario REST “moderno”: todo pasa por las mismas rutas **`/api/equipment/...`** que el servidor expone para compatibilidad Flash. Métodos usados: `GET /equipment/init`, `POST /equipment/move`, `POST /equipment/drones/equip`, `POST /equipment/ship/change`, `POST /equipment/ship/design`, `POST /equipment/config/clear` (base URL configurable con `VITE_API_BASE_URL`, por defecto `http://localhost:5007/api`).
- **Tipos y estado:** `src/types/equipment.ts` modela el envelope **`EquipmentFlashResponse`** (`isError`, `data.ret`, `money`, `data.map` con `types` / `lootIds`) y la estructura tipo cliente legacy: `hangars`, `items` (`I`, `L`, `LV`, …), `itemInfo`, configs **1** y **2**, `playerShips`. El store **`useEquipmentStore`** (`src/stores/equipment.ts`) **normaliza** esos datos a vistas procesadas (nave, drones, inventario vs equipado, slots) y traduce al español mensajes de error frecuentes devueltos en inglés por la API.
- **Iconos:** `src/utils/itemImageResolver.ts` carga el manifiesto estático **`/do_img/global/xml/resource_items.xml`** (assets servidos por el propio CMS) para resolver rutas de imágenes por `loot_id`.
- **Alcance UI:** pestañas previstas en tipos (`EquipmentTab`: nave, drones, pet, inventario); la adaptación al nuevo modelo y la **visualización de `level`** por instancia corresponden a la **§6 Fase 4** (sustituir envelope Flash, rutas §4, servicio/store/componentes bajo `src/components/equipment/`).

### 1.4 Game Server (`MexOrbit.GameServer`)

- **`QueryManager.GetPlayer` / `SetEquipment`** leen **`player_equipment`** con SQL en texto (`SELECT *`), deserializan JSON con **Newtonsoft** y calculan **daño, escudo, velocidad, HP** iterando IDs en rangos (`itemId >= 0 && itemId < 40`, etc.).
- La **fuente de verdad** en runtime sigue siendo MySQL; la sincronización tras cambios desde el CMS es **guardar BD → `UpdateStatus` por socket → `SetEquipment`** (correcto conceptualmente; el problema es el **formato de datos** difícil de evolucionar y duplicado).

### 1.5 Problemas identificados

| Área | Problema |
|------|----------|
| Escalabilidad | Añadir un ítem nuevo o cambiar reglas suele tocar **varias capas** y rangos de IDs. |
| Datos | **Instancias** de ítems no están modeladas (solo listas de enteros y conteos agregados). |
| API | REST “de fachada”: respuestas aún moldeadas al **contrato Flash/Base64**. |
| GS vs API | **Dos lecturas** del mismo concepto (EF + SQL crudo) con riesgo de **desalineación**. |
| Tests | Difícil probar reglas de equipo sin arrancar BD y sin un modelo de dominio claro. |

---

## 2. Objetivos del diseño objetivo

1. **Catálogo de ítems** (`item_catalog`): metadatos estables (`loot_id`, categoría, flags de comportamiento). **Stats desacoplados** en tablas de atributos tipados (ver §3.2) para poder añadir bonos, escalados o nuevos tipos de stat sin rediseñar columnas fijas.
2. **Instancia de ítem** (`player_item`): **una fila = una unidad** (equipable: láser, dron, CPU, nave, etc.). **Sin `quantity`** en la fila: varias unidades del mismo tipo = **varias filas** (`COUNT` ≤ `max_owned`). Incluye **`level`**. El catálogo define el **tipo**; la posesión de ítems es el conjunto de filas en `player_item`.
3. **Recursos de billetera** (`player_resource_balance`): todo lo que **solo se suma** y **no** es un ítem de instancia (uridium, créditos, seprom, xenomita, premium, honor como saldo, etc.) — ver §3.3. No duplicar el mismo concepto en `player_item`.
4. **Equipamiento del jugador** (`player_equipment_loadout`): relación **jugador + contexto (config, slot, drone)** → **instancia** de `player_item`.
5. **API REST** explícita: recursos con verbos HTTP estándar, **JSON sin Base64**, códigos HTTP y errores tipados. **Sin prefijo de versión en la URL** (`/api/v2/...`, `/api/v1/...`): aún no hay producción; cuando exista necesidad de compatibilidad con clientes antiguos, se puede introducir versionado explícito.
6. **Game Server**: lee **catálogo + stats + loadout + recursos** con las mismas tablas que la API; el motor de combate **resuelve** stats mediante códigos de atributo estables, no rangos mágicos de `loot_id` dispersos en código.

### Premisa (sin producción actual)

No hay entorno productivo que exija **ventanas de mantenimiento** ni **doble escritura** prolongada. La estrategia puede ser **agresiva**:

- Un **único script de migración** que **lee** (solo lectura) los datos actuales de **`player_equipment`** y JSON relacionados, **puebla** las tablas nuevas (`item_catalog`, `stat_kind` / `item_catalog_stat`, `server_item` (recursos del catálogo) / `player_resource_balance`, `player_item`, loadout). La tabla **`player_equipment` no se modifica en esquema**: se conserva tal cual y **solo se deja de usar** en API y Game Server (ver §3.6).
- Hacer **backup de la BD** antes de ejecutar el script (disciplina mínima aunque solo sea dev).

---

## 3. Modelo de datos propuesto (MySQL)

> Nombres orientativos; ajustar a convención del repo (`snake_case` en BD si ya es el estándar).

### 3.1 `item_catalog` (catálogo — sin stats “duros” en la fila)

- `id` (PK)
- `loot_id` (int, único) — ID de juego / cliente
- `category` (enum o tabla de categorías: `equipable`, `ammo`, `ship`, `resource`, … — **los recursos puros pueden tener categoría propia** y no usar `player_item`; ver §3.3)
- `stackable`, **`max_owned`**, `name_key`, etc.
- **`max_owned` — límite de posesión por jugador:** define el **tope** de unidades de ese tipo: **`COUNT(*)` por `catalog_item_id` en `player_item`** ≤ `max_owned`. El **Game Server** (y cualquier otorgamiento de ítems) debe validar antes de insertar otra fila. Valores **orientativos** de balance (ajustar en seed/CMS):

| Tipo (ejemplo) | `max_owned` (ejemplo) |
|------------------|------------------------|
| Iris (`drone_iris`) | 8 |
| Zeus (`drone_zeus`) | 1 |
| Apis (`drone_apis`) | 1 |
| Havoc | 10 |
| Hercules | 10 |
| LF-3 | 35 |
| LF-4 | 35 |
| Escudo (p. ej. SG3N-BO2 / BO-2) | 15 |
| Generador (p. ej. G3N-7900) | 15 |
| *(otros tipos)* | según diseño |

- **`stackable`:** flag opcional de **UI / presentación** (agrupar en el cliente varias instancias del mismo tipo); **persistencia:** siempre **N filas** en **`player_item`**, nunca una tabla intermedia de cantidades.
- **No** meter columnas fijas tipo `damage`, `shield_bonus` por ítem: eso vive en §3.2 para escalar sin migraciones DDL cada vez que añadáis un tipo de bono.

### 3.2 Abstracción de stats del catálogo (escalable)

Objetivo: nuevos tipos de stat (p. ej. resistencia, penetración, bonos por rango) = **filas nuevas** o **códigos nuevos**, no columnas nuevas en `item_catalog`.

**Opción recomendada — diccionario de tipos + valores por ítem:**

| Tabla | Rol |
|-------|-----|
| `stat_kind` | Catálogo de **qué** se mide: `code` estable (`WEAPON_DAMAGE_FLAT`, `SHIELD_FLAT`, `SPEED_FLAT`, `SHIELD_PCT`, …), `value_type` (flat \| percent \| multiplier), orden para UI, opcional `stacking_rule` (sum_max, etc.). |
| `item_catalog_stat` | FK `item_catalog_id`, FK `stat_kind_id`, **`value` DECIMAL** (p. ej. `DECIMAL(18,6)` o `DECIMAL(12,4)` según precisión de balanceo), opcional `condition_json` muy acotado (p. ej. solo contra NPC) si hace falta **sin** convertir todo el combate en JSON. **No** usar BIGINT para el valor del stat salvo excepción documentada. |

- El **Game Server** y la **API** resuelven stats con `JOIN` por `loot_id`/`item_catalog.id` + `stat_kind.code` en código C# (switch por código o motor de reglas).
- **Alternativa** para casos raros: columna `extra_stats_json` en `item_catalog` solo como **escape hatch**; la regla del proyecto puede ser “todo stat de combate balanceable va a `item_catalog_stat`”.

Ventajas: balanceo desde CMS/BD, tests por `stat_kind`, menos duplicación con `QueryManager` actual (rangos de IDs).

### 3.3 Recursos acumulables (no son “instancias” de ítem)

Separado de `player_item`: materiales y monedas que **solo tienen cantidad** (xenomita, seprom, prometium, uridium, créditos, honor si se modela como saldo, etc.).

| Tabla | Rol |
|-------|-----|
| `server_item` (categorías `resource`, `resource_ore`, …) | Mismo catálogo que assets/CMS; `item_key` / `loot_id` para xenomita, seprom, uridium, etc. Ver [sql-schema.md](../items & equipment/sql-schema.md). |
| `player_resource_balance` | `user_id`, `server_item_id` FK → `server_item(id)`, `amount` (DECIMAL o BIGINT), `updated_at`. UNIQUE (`user_id`, `server_item_id`). |

- **Regla de producto:** si solo se **suma/resta** como saldo (moneda/material sin instancia de ítem de equipo) → **`player_resource_balance`**. Los ítems de equipo con identidad → **`player_item`** (múltiples filas si hay varias unidades).
- **Migración:** mapear columnas/contadores legacy (p. ej. campos en `player_accounts` o JSON) a filas de `player_resource_balance`; el script de migración puede poblar `server_item` (recursos) desde el catálogo y luego `player_resource_balance` por jugador.

### 3.4 `player_item` (instancias discretas — **sin `quantity`**)

- `id` (PK), `user_id`, `catalog_item_id` (FK `item_catalog`)
- **`level`** — nivel de la **instancia** (p. ej. `SMALLINT UNSIGNED` o `TINYINT UNSIGNED`, `NOT NULL`, default según diseño, p. ej. `0` o `1`). Corresponde al progreso por unidad (equivalente semántico al **`LV`** del cliente legacy en `InventoryItem`), no al nivel global del jugador. Subir de nivel / upgrade actualiza esta columna; el **Game Server** y el combate deben usar `level` junto con `item_catalog` / stats para resolver bonos.
- Metadatos de posesión: `acquired_at`, `source`, `bound`, etc.
- **Una fila = una unidad.** “Tener 12 LF-3” = **12 filas** en `player_item` con el mismo `catalog_item_id` (y `level` según corresponda).

**Regla de producto:** no hay flags/`EquipmentItems` como fuente de verdad (legado Flash). **Cada obtención** de una unidad discreta **inserta una fila** en `player_item`. El loadout referencia `player_item_id`, no pools sintéticos.

### 3.5 `player_equipment_loadout` (equipo equipado)

- Normalizado: `user_id`, `config_id`, `slot_kind`, `slot_index`, `drone_id` (nullable), `player_item_id` (nullable FK)
- Evita JSON para el orden de slots si queréis SQL claro en el GS.

### 3.6 Tabla `player_equipment` (conservar sin cambios, dejar de usar)

- **DDL:** no se altera la definición de **`player_equipment`** (columnas, tipos, índices permanecen como están hoy).
- **Fuente de verdad:** la aplicación (API + Game Server) pasa a leer y escribir **solo** el nuevo modelo (`player_item`, loadout, `player_resource_balance`, stats de catálogo, etc.). **`player_equipment` queda obsoleta para runtime** — no se borra para permitir auditoría, rollback manual o comparación histórica si hace falta.
- **Escrituras:** el código nuevo **no** debe actualizar `player_equipment`. Opcional: trigger de solo lectura, feature flag, o documentar “no escribir” en revisión de código; migración futura a borrar la tabla sería **otra** decisión explícita.
- **`player_settings`** y login pueden seguir como están en paralelo al alcance de inventario.

---

## 4. API REST (diseño orientativo)

| Recurso | Métodos | Descripción |
|---------|---------|-------------|
| `/api/inventory` | `GET` | Instancias (`player_item`); el cliente puede **agrupar** por tipo usando `stackable` del catálogo (solo presentación). |
| `/api/inventory/items/{id}` | `GET` | Detalle de una instancia en `player_item`. |
| `/api/equipment/loadout` | `GET` | Vista del equipo actual (configs 1/2). |
| `/api/equipment/loadout` | `PUT` / `PATCH` | Sustituir slots (transacción; validaciones hangar online igual que hoy vía `IGameServerClient`). |
| `/api/catalog/items` | `GET` (admin o público limitado) | Catálogo para CMS. |
| `/api/catalog/items/{id}/stats` | `GET` | Stats resueltas por ítem (lectura desde `item_catalog_stat` + `stat_kind`). |
| `/api/stacks` | `GET` | Saldos **`player_resource_balance`** (xenomita, seprom, uridium, créditos, etc.). **No** usar `/api/resources` ni `/api/wallet`. |

- **Ingresos de saldo / créditos:** **sin** endpoint REST por ahora (ni `/api/resources/{code}/credit` ni equivalente bajo `/api/stacks`); el ajuste de balances sigue en **Game Server** u otros procesos internos hasta que se defina un `POST` de transacciones.
- **Ítems de instancia:** altas/bajas de filas **`player_item`** las realiza el **Game Server** (o pipeline interno autorizado), no rutas REST públicas en esta fase salvo las que defináis para el hangar.
- **DTOs** explícitos (records o clases) en `MexOrbit.Domain` o `Application`; **sin** Base64.
- **Errores**: `ProblemDetails` o `{ "code", "message", "details" }` estable.
- **Autorización**: mismo JWT que hoy; políticas si en el futuro hay rol admin en catálogo.

Objetivo: **reemplazar** `EquipmentApiController` / `EquipmentService` centrados en Base64 por controladores y servicios que hablen **solo** con el nuevo modelo (el código legacy puede borrarse en la misma épica una vez el CMS apunte a las nuevas rutas).

---

## 5. Game Server: cambios necesarios

1. **Lectura de equipo**: sustituir la lógica basada en rangos sobre JSON por `JOIN` `player_item` + `item_catalog` + `player_equipment_loadout` (o vista `v_player_equipment_effective`). El equipo equipado apunta a **instancias** concretas; “cuántos Havoc tengo” = `COUNT(*)` en `player_item` para ese tipo — no un campo `HavocCount`.
2. **Resolución de stats**: para cada ítem equipado, cargar **`item_catalog_stat`** + **`stat_kind`** y acumular según `stat_kind.code` (daño, escudo, velocidad, …), aplicando reglas que dependan del **`level`** de la fila **`player_item`** cuando el diseño lo exija. Un módulo C# compartido (o librería referenciada por API + GS) puede traducir códigos → contribución numérica al `ConfigsBase` / combate.
3. **Recursos del jugador**: leer **`player_resource_balance`** donde el gameplay lo requiera (crafting, costes en mapa, etc.). **No** mezclar saldos con ítems de instancia (`player_item`) sin reglas claras.
4. **Contrato con API**: tras persistir inventario/equipo, seguir **`UpdateStatus`** como hoy; los recursos pueden tener su propio canal de invalidación o el mismo tick según diseño.

---

## 6. Fases de implementación (enfoque agresivo, sin producción)

### Fase 1 — Esquema nuevo + script de migración único

- DDL: `item_catalog`, `stat_kind`, `item_catalog_stat` (**`value` DECIMAL**), `server_item` (incl. recursos), `player_resource_balance` (FK `server_item_id`), `player_item` (**sin `quantity`**; **con `level`** por instancia), `player_equipment_loadout` (nombres finales según convención del repo).
- **Script de migración de datos** que:
  1. **Backup** antes de ejecutar.
  2. Lee **`player_equipment`** y JSON relacionados (`EquipmentItems`, columnas de config).
  3. Puebla **`item_catalog`** (loot_id deduplicados).
  4. Seed o backfill de **`stat_kind`** + **`item_catalog_stat`** (mapear al menos daño/escudo/velocidad de los ítems actuales; el resto puede ir en oleadas posteriores si el script se alarga).
  5. Puebla **`server_item`** (recursos) y **`player_resource_balance`** a partir de contadores legacy — **definir mapa** xenomita/seprom/… → filas `server_item` / `loot_id` del catálogo.
  6. Crea **`player_item`** y **loadout**: a partir de los datos legacy, **materializar** — p. ej. `Lf4Count = 12` → **12 filas** en `player_item` con **`level`** mapeado desde el **`LV`** de cada ítem en JSON donde exista. Tras la migración, **nuevas** unidades solo entran por flujos de obtención, no por contadores agregados en `EquipmentItems`.
  7. **Cleanup de código** legacy (EF/servicios que escribían en `player_equipment`) cuando API+GS validen — **sin** `DROP TABLE player_equipment` salvo decisión posterior.

No hay **dual-write**: el código nuevo trabaja **solo** contra el esquema nuevo; el script es el puente **una sola vez** desde el esquema viejo.

### Fase 2 — API y EF solo sobre el nuevo modelo

- Entidades EF para `item_catalog`, `stat_kind`, `item_catalog_stat`, `server_item`, `player_resource_balance`, `player_item`, loadout.
- **Eliminar o sustituir** el uso de `PlayerEquipment` / columnas JSON en la ruta del inventario: servicios REST directos, sin Base64, según §4.
- Dejar **estable** el contrato JSON (DTOs, Swagger) que consumirán el **Game Server** (Fase 3) y el **CMS** (Fase 4).

### Fase 3 — Game Server

- Refactor de `QueryManager.SetEquipment` / `GetPlayer` para leer **solo** tablas nuevas (o vista).
- Validar stats en juego (daño, escudo, velocidad) frente a lo esperado; ajustar catálogo o fórmulas; respetar **`player_item.level`** en cálculos.
- Flujo **`UpdateStatus`** sin cambios conceptuales.

### Fase 4 — CMS: hangar / equipamiento (Vue)

Objetivo: adaptar **`MexOrbit.CMS`** al nuevo backend (§1.3) cuando la **API (Fase 2)** y el **GS (Fase 3)** ya lean el modelo relacional (así el hangar y el mapa no divergen).

- **Cliente HTTP:** sustituir el consumo del envelope **`EquipmentFlashResponse`** y rutas `/equipment/*` por los recursos del §4 (`GET /api/inventory`, `GET /api/equipment/loadout`, `PUT`/`PATCH` de loadout, etc.) y tipos TypeScript nuevos (sin Base64).
- **Store Pinia (`equipment.ts`):** mapear respuestas a estado de UI (inventario, configs, slots, drones, naves); eliminar dependencia de `itemInfo`/`L` indexados como en Flash salvo capa de compatibilidad temporal muy acotada.
- **Componentes** (`EquipmentPanel`, `InventoryGrid`, `ItemCard`, `ItemDetail`, `ShipTab`, `DronesTab`, …): enlazar slots y listas a **`player_item_id`** y metadatos del catálogo; refrescar tras cambios de equipo según respuestas HTTP estándar.
- **Nivel de ítem (`level`):** la UI debe mostrar el **nivel de cada instancia** según la columna **`player_item.level`** (§3.4), no solo un nivel inferido del modelo legacy. Incluir badge o texto en tarjetas de ítem, detalle, filas de inventario y, donde aplique, en slots equipados (nave/drones). Si el diseño visual ya mostraba algo análogo desde `LV`, sustituir la fuente de datos por el campo persistido en BD.
- **Saldos:** `GET /api/stacks` expone **`player_resource_balance`**; **inventario** = instancias `player_item` (el CMS puede agrupar por tipo si `stackable`).

### Fase 5 — Limpieza de código muerto

- Retirar rutas/servicios que dependían de **`player_equipment`** como persistencia activa; la **entidad/tabla puede seguir en el esquema EF** mapeada por compatibilidad o eliminarse del modelo **solo** cuando no quede código que la referencie (la tabla física en MySQL sigue existiendo).
- Retirar en el CMS restos del contrato Flash (`EquipmentFlashResponse`, parsers de `ret`/`map`) cuando la **Fase 4** esté cerrada.
- Documentar el modelo final en [equipment-and-game-server-sync.md](../../flows/equipment-and-game-server-sync.md) y aclarar que **`player_equipment` está deprecada** pero intacta en BD.

---

## 7. Riesgos y mitigaciones

| Riesgo | Mitigación |
|--------|------------|
| Paridad stats GS ≠ cliente | Comparar stats **antes** de migración (dump o query) vs **después** en los mismos `userId` de prueba. |
| Script mal generado | Probar en copia de BD; script idempotente o transacción donde MySQL lo permita; rollback documentado. |
| Alcance infinito | Primera entrega: inventario + equipamiento + **stats abstraídos** + **recursos** mínimos (códigos acordados); skill tree / boosters complejos pueden quedar fuera del primer script o en JSON temporal. |
| Desalineación API ↔ CMS | Tras Fase 2 y GS (Fase 3), validar contratos con el CMS (Fase 4); pruebas manuales o E2E en `/hangar` con niveles visibles. |
| Pérdida de datos | Backup antes del script; la tabla **`player_equipment` no se toca en DDL** — el riesgo es solo coherencia del **nuevo** modelo respecto a lo leído en migración. |

---

## 8. Entregables (checklist)

- [ ] Script DDL (`item_catalog`, `stat_kind`, `item_catalog_stat` con **`value` DECIMAL**, `server_item` recursos, `player_resource_balance` → `server_item_id`, `player_item` **sin `quantity`** y **con `level`**, loadout) + **migración única** con mapa de recursos legacy → catálogo `server_item`.
- [ ] Backup / restore documentado en una línea en el README del script.
- [ ] EF + DTOs + controladores REST + Swagger (sin Base64 en la ruta principal).
- [ ] **CMS hangar (Fase 4):** `equipmentService` + store + componentes Vue alineados a la API nueva; **mostrar `level`** (`player_item.level`) en inventario, detalle y slots.
- [ ] `QueryManager` / GS leyendo solo tablas nuevas.
- [ ] Eliminación de **código** que escriba en `player_equipment`; tabla **`player_equipment`** sin cambios en BD (deprecada para uso).
- [ ] Actualización de `docs/flows/equipment-and-game-server-sync.md` con el modelo definitivo.
- [ ] **Seed SQL del catálogo** alineado con la tabla del §9 (o script generado a partir de ella).
- [ ] Flujos de **obtención** que insertan filas en **`player_item`** (y ajustan **`player_resource_balance`** cuando aplique), sin contadores agregados en `EquipmentItems`.

---

## 9. Seed del `item_catalog`: ítems que el código maneja hoy

Esta sección **plasma** los **tipos** de ítem que el código legacy conoce (`loot_id`, stats, rangos GS), para que el seed de BD (`item_catalog` + `stat_kind` / `item_catalog_stat` con **`value` DECIMAL**) sea **reproducible**. **No** implica inventario inicial fijo: el catálogo lista **definiciones**; la posesión de ítems es **`player_item`** (N filas por N unidades); lo que no es ítem de instancia va a **`player_resource_balance`** (§3.3), más la migración desde legacy (§6 Fase 1).

**Fuentes:** `MexOrbit.DomainService/EquipmentService.cs` (`HellstormItemTypes`, `ExtraItemTypes`, `BuildItemsArray`, `BuildItemInfoArray`, `BuildDronesArray`, `BuildMapData`), `Infrastructure.Sql/ValueObjects/EquipmentItems.cs`, `GameServer/Managers/QueryManager.cs` (`SetEquipment`).

### 9.1 Identificadores `loot_id` (strings) ya usados en el cliente/API

Orden conceptual (no implica orden de filas en seed):

| `loot_id` | Nombre en UI | Rol |
|-----------|--------------|-----|
| `equipment_generator_shield_sg3n-b02` | SG3N-BO2 (BO-2) | Generador escudo |
| `equipment_generator_speed_g3n-7900` | G3N-7900 | Generador velocidad |
| `drone_iris` | Iris | Tipo dron (legacy: pool; modelo nuevo: una fila `player_item` por dron) |
| `drone_apis` | Apis | Tipo dron (legacy: flag `EquipmentItems.Apis`; **objetivo:** instancia(s) en `player_item` al desbloquear/comprar/conseguir) |
| `drone_zeus` | Zeus | Tipo dron (legacy: flag `EquipmentItems.Zeus`; **objetivo:** igual que Apis) |
| `drone_designs_havoc` | Havoc | Diseño de dron |
| `drone_designs_hercules` | Hercules | Diseño de dron |
| `equipment_weapon_laser_lf-3` | LF-3 | Láser |
| `equipment_weapon_laser_lf-4` | LF-4 | Láser |
| `equipment_weapon_hellstorm_hst-1` | HST-01 | Lanzacohetes Hellstorm |
| `equipment_weapon_hellstorm_hst-2` | HST-02 | Lanzacohetes Hellstorm |
| `equipment_extra_cpu_clk-xl` | CLK-XL | CPU extra |
| `equipment_extra_cpu_alb-x` | ALB-X | CPU extra |
| `equipment_extra_cpu_rb-x` | RB-X | CPU extra |
| `equipment_extra_cpu_jp-02` | JP-02 | CPU extra |
| `equipment_extra_cpu_arol-x` | AROL-X | CPU extra |
| `equipment_extra_cpu_dr-01` | DR-01 | CPU extra |
| `equipment_extra_cpu_rep-s` | REP-S | CPU extra |
| `equipment_extra_cpu_hm-07` | HM-07 | CPU extra |
| `equipment_extra_cpu_smb-01` | SMB-01 | CPU extra |
| `equipment_extra_cpu_aim-02` | AIM-02 | CPU extra |

**Naves (hardcode en `BuildMapData` / `BuildItemInfoArray`, L 9–17):**

| `loot_id` |
|-----------|
| `ship_aegis-mmo`, `ship_aegis-eic`, `ship_aegis-vru` |
| `ship_citadel-mmo`, `ship_citadel-eic`, `ship_citadel-vru` |
| `ship_spearhead-mmo`, `ship_spearhead-eic`, `ship_spearhead-vru` |

**Más naves:** todas las filas de **`Ships`** en BD (`ship.LootId`) se añaden al mismo orden que en `BuildItemInfoArray` / `BuildMapData`.

### 9.2 Legacy vs modelo objetivo (`player_item`)

**Comportamiento actual (Flash / `EquipmentService`):** el inventario inicial se **sintetiza** con pools fijos (p. ej. 40 LF-3, 60 BO-2, 20 G3N) y conteos por campo en **`EquipmentItems`** (`Lf4Count`, `HavocCount`, `HerculesCount`, `Hst01Count`, CPUs, etc.), más flags booleanos (`EquipmentItems.Apis`, `EquipmentItems.Zeus`) para incluir o no esos drones en el array de ítems. Eso sirve para alimentar el cliente legacy, **no** es el modelo de datos objetivo.

**Modelo objetivo:** todo ítem de equipo = filas en **`player_item`** (sin `quantity`; **con `level`** por instancia). Varios “del mismo tipo” = varias filas con el mismo `catalog_item_id`. “Cuántos Havoc” = `COUNT(*)` sobre `player_item` para ese tipo. El seed del **catálogo** declara **un** tipo LF-3 con stats **DECIMAL** en `item_catalog_stat`; **40 unidades** = **40 filas** en `player_item`.

**Tabla de referencia — tipos que el legacy agrupaba por contador (solo para migración y paridad):**

| Tipo (catálogo) | En legacy | En modelo nuevo |
|-----------------|-----------|-------------------|
| LF-3, BO-2, G3N-7900, LF-4, diseños Havoc/Hercules, Hellstorm, CPUs, etc. | Pools / `*Count` en `EquipmentItems` | Una fila `player_item` por unidad |
| Apis, Zeus | Flags booleanos | Instancia(s) cuando el jugador obtiene el dron |
| Iris | 8 copias en pool | Ocho instancias iniciales en migración **o** cero + obtención por gameplay — **definir regla de producto** en el script |

**PET:** `EquipmentItems.Pet`, `PetModules` — fuera del listado de `loot_id` anterior; decidir si el seed del catálogo incluye ítems PET en la misma épica o después.

**Skill tree:** `SkillTreeInfo` (logdisks, research points, reset) — no es catálogo de ítem equipable; puede seguir en tabla propia o JSON según el plan de habilidades.

### 9.3 Índice interno `L` (cliente Flash / itemInfo)

Valores usados en `BuildItemInfoArray` (referencia para mapear a `item_catalog.category` / metadata):

| L | Nombre | T (tipo num.) | C (categoría) |
|---|--------|----------------|---------------|
| 0 | SG3N-BO2 | 4 | generator |
| 1 | G3N-7900 | 3 | generator |
| 2 | Iris | 24 | drone |
| 3 | Apis | 24 | drone |
| 4 | Zeus | 24 | drone |
| 5 | Havoc | 16 | ship (diseño dron en naming legacy) |
| 6 | Hercules | 16 | ship |
| 7 | LF-3 | 0 | laser |
| 8 | LF-4 | 0 | laser |
| 9–17 | Aegis / Citadel / Spearhead | 22 | ship (hardcode) |
| … | Resto de naves | 22 | ship (desde BD) |
| offset+0,1 | HST-01, HST-02 | 1 | heavy_gun |
| offset+… | CLK-XL … AIM-02 | 7 | extra |

El **offset** `newItemLOffset` depende del número de naves en BD; el seed debe **fijar** el mismo criterio que el código o normalizar a **solo `loot_id`** sin depender de L. A medio plazo, el cliente y el GS deben identificar equipo por **`player_item_id`** / catálogo, no por `L` preasignado.

### 9.4 Game Server — rangos de ID en slots (solo combate, `QueryManager.SetEquipment`)

Estos rangos aplican a los **enteros** guardados en `config*_lasers`, `config*_generators`, `config*_drones` JSON (no confundir con el índice secuencial del `EquipmentService`):

| Slot / uso | Rango `itemId` | Efecto aproximado |
|------------|----------------|-------------------|
| Láser | 0–39 | Daño LF-3 (+150) |
| Láser | ≥ 140 | Daño LF-4 (+200) |
| Generador | 40–99 | Escudo BO-2 (+15000) |
| Generador | 100–119 | Velocidad G3N (+10) |
| Diseño dron | 120–129 | Cuenta Havoc |
| Diseño dron | 130–139 | Cuenta Hercules |
| Láser en dron | 0–39 / ≥140, gen 40–99 | Bonos adicionales sobre drones |

El seed del catálogo y las filas de **`item_catalog_stat`** deben permitir **reproducir** estos números sin hardcodear rangos en el GS a medio plazo; hasta entonces, documentar esta tabla como **paridad obligatoria** con el comportamiento actual.

### 9.5 Entrega del seed

- **Catálogo:** un script SQL o CSV con **una fila por tipo** (`loot_id` / `item_catalog`), más `item_catalog_stat` alineados a §9.4 donde aplique; incluir **`stackable`** y **`max_owned`** por tipo (límites de posesión — ver tabla de ejemplos en §3.1).
- **Instancias:** **no** forman parte del seed del catálogo; se crean por **migración** y por **servicios de obtención**. Opcional: `INSERT` de prueba en `player_item` en dev.
- **Opcional:** generar el seed del catálogo desde un **YAML/JSON** versionado en `docs/implementation/seeds/` que un tool convierta a SQL en CI.

---

## 10. Referencias de código actuales

| Componente | Ruta |
|------------|------|
| API equipo “REST” | `MexOrbit.Server/Api/MexOrbit.Api.External/Controllers/EquipmentApiController.cs` |
| Lógica inventario | `MexOrbit.Server/Domain/MexOrbit.DomainService/EquipmentService.cs` |
| Entidad equipo | `MexOrbit.Server/Infrastructure/MexOrbit.Infrastructure.Sql/Entities/PlayerEquipment.cs` |
| Valor `EquipmentItems` | `MexOrbit.Server/Infrastructure/MexOrbit.Infrastructure.Sql/ValueObjects/EquipmentItems.cs` |
| GS stats desde BD | `MexOrbit.Server/Service/MexOrbit.GameServer/Managers/QueryManager.cs` (`SetEquipment`) |
| Socket `UpdateStatus` | `MexOrbit.Server/Service/MexOrbit.GameServer/Net/SocketServer.cs` |
| Inventario / lootIds / pools | `MexOrbit.Server/Domain/MexOrbit.DomainService/EquipmentService.cs` |
| CMS hangar (vista) | `MexOrbit.CMS/src/views/HangarView.vue` |
| CMS panel equipo / inventario | `MexOrbit.CMS/src/components/equipment/EquipmentPanel.vue` y subcomponentes (`InventoryGrid`, `ShipTab`, `DronesTab`, …) |
| CMS cliente API equipo | `MexOrbit.CMS/src/services/equipmentService.ts` |
| CMS store Pinia | `MexOrbit.CMS/src/stores/equipment.ts` |
| CMS tipos envelope Flash | `MexOrbit.CMS/src/types/equipment.ts` |
| CMS imágenes ítems | `MexOrbit.CMS/src/utils/itemImageResolver.ts` |

---

*Documento de planificación. La migración introduce tablas nuevas y deja de usar **`player_equipment` en aplicación**, sin alterar la definición de esa tabla en MySQL.*

*No sustituye el script SQL concreto ni pruebas en tu copia de BD.*
