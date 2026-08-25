# Plan: API REST de equipamiento (sin legacy Flash)

**Contexto:** no hay PROD; se elimina soporte a inventario Flash/Base64. Nuevo controlador **`EquipmentController`** (JSON limpio).

**Base de rutas (fija):** **`/api/v1/equipment`**.

**Referencias:** [sql-schema.md](../items%20&%20equipment/sql-schema.md) (`player_equipment_slot`, `player_item`, `server_item`), [player-equipment-legacy-migration.md](../items%20&%20equipment/player-equipment-legacy-migration.md).

---

## 1. Objetivos

1. **Eliminar** `FlashInventoryController` y rutas `flashAPI/inventory` (y `.php`).
2. **Eliminar** `EquipmentApiController` en su forma actual (init/move/drones/base64).
3. **Conservar la semántica** de cambio de nave y cambio de diseño que hoy exponen `POST api/equipment/ship/change` y `POST api/equipment/ship/design`, migrándolas al nuevo controlador con **respuestas JSON** (sin Base64).
4. Implementar lectura y mutación del equipamiento sobre **`player_item`** + **`player_equipment_slot`** (modelo canónico).
5. **Reutilizar** las mismas reglas de negocio que ya existen en `EquipmentService` para estado online / zona de equipar / cooldown / sincronización con Game Server.

---

## 2. Reglas de negocio a preservar (desde `EquipmentService`)

Extraer a un **mismo flujo reutilizable** (helper o servicio interno) invocado por los nuevos endpoints:

| Regla | Comportamiento actual (resumen) | Aplicar a |
|--------|----------------------------------|-----------|
| Zona de equipar | Si el jugador **está online**, solo puede mutar equipamiento si **`IsPlayerInEquipZoneAsync`** es true. Si **offline**, se permite. | Clear loadout, equip/unequip, cambios que toquen slots o inventario equipado. |
| Mensaje de error | *"Equipping isn't possible. You must be at a location with a hangar facility!"* | Misma UX en JSON con **`409 Conflict`** (ver §7). |
| Online + GS | Tras persistir cambios relevantes, si **`IsPlayerOnlineAsync`**, llamar **`UpdatePlayerStatusAsync`** (no fallar la operación si el GS falla; log warning). | Mutaciones de slots / clear. |
| Cambio de **diseño** de nave | Validar `lootId` / normalización por facción; **`targetShip.BaseShipId == currentShip.BaseShipId`**; online → zona + **`CanPlayerChangeShipAsync`** (cooldown 5 s); éxito → **`ChangePlayerShipAsync`** si online. | Endpoint `ship/design`. |
| Cambio de **clase** de nave | Comprobar posesión (`ownedShipIds` / lógica actual); no confundir con cambio de diseño; cooldown si online. | Endpoint `ship/change`. |

**Nota:** La implementación nueva debe operar sobre **tablas nuevas** (`player_equipment_slot`), no sobre JSON de `player_equipment`. El servicio actual sigue escribiendo legacy; este plan asume **nuevo servicio** (p. ej. `ShipEquipmentService` / métodos en `IEquipmentService` v2) que reemplaza gradualmente las rutas.

---

## 3. Modelo de datos (recordatorio para rutas)

- **`ship_config_id`:** `1` o `2` (presets de nave).
- **`slot_kind`:** `laser`, `generator`, `heavy_gun`, `extra`, `drone_item`, `drone_design` (texto acotado en aplicación).
- **`sort_order`:** orden dentro del grupo de `slot_kind` en esa bahía/config.
- **`drone_slot_index`:** `255` = ranura de **nave**; `0`–`9` = bahía de **dron**.
- **`player_item_id`:** instancia poseída; el **nivel** está en `player_item.level`.
- **`server_item.is_wallet_balance`:** `0` = ítem representado por filas **`player_item`** (equipo, instancias equipables). `1` = cantidad en **`player_resource_balance`** (no debe aparecer en el listado de §4.1).

Los GET con datos deben devolver **resolución de catálogo** (`server_item.loot_id`, `item_key`, categoría) cuando aplique.

---

## 4. Convenciones de respuesta HTTP

- **Éxito:** el **código HTTP** es la señal principal (`200`, `201`, `204`). **No** se envuelve el resultado en `{ "success": true }`.
  - **GET** con datos: `200` + cuerpo JSON (recurso o colección).
  - **Mutaciones sin cuerpo de retorno necesario:** `204 No Content` (p. ej. clear, equip/unequip, cambio de nave/diseño si el cliente no necesita payload).
  - **Mutaciones que devuelven estado actualizado:** `200` + DTO útil (p. ej. `shipId` tras cambio de nave), **sin** campo `success`.
- **Error:** cuerpo `{ "error": { "code": "...", "message": "..." } }` (código de negocio estable además del HTTP).

---

## 5. Diseño REST propuesto

- Prefijo **`/api/v1/equipment`** para todos los endpoints de esta área.
- **`Content-Type: application/json`** donde haya cuerpo.

### 5.1 Ítems poseídos (solo instancias equipables: `is_wallet_balance = 0`)

| | |
|---|---|
| **Método / ruta** | `GET /api/v1/equipment/player-items` |
| **Descripción** | Lista todas las filas **`player_item`** del jugador cuyo **`server_item`** tiene **`is_wallet_balance = 0`**. Excluye moneda, munición, recursos por saldo y cualquier cosa que viva en `player_resource_balance`. |
| **Respuesta (idea)** | `{ "items": [ { "id", "level", "serverItem": { "lootId", "itemKey", "categoryCode", ... } } ] }` |
| **Uso** | Pantalla de inventario / selección de ítem para equipar en un slot. |

Implementación: `JOIN player_item → server_item` con **`WHERE server_item.is_wallet_balance = 0`** (y `user_id` del token).

---

### 5.2 Obtener equipamiento actual (configs + ítems + nivel)

| | |
|---|---|
| **Método / ruta** | `GET /api/v1/equipment/ship-loadout` |
| **Descripción** | Configuraciones de nave con filas de `player_equipment_slot` resueltas a `player_item` + `server_item` + nivel. |
| **Query opcional** | `configurationId` — si se envía (`1` o `2`), devolver **solo** esa configuración (menos payload). Si se omite, devolver **ambas**. |
| **Respuesta (idea)** | `{ "shipConfigurations": [ { "id": 1, "slots": [ ... ] }, ... ] }` |

---

### 5.3 Limpiar equipamiento

| | |
|---|---|
| **Método / ruta** | `DELETE /api/v1/equipment/ship-loadout` |
| **Query** | `configurationId` opcional (`1`, `2` o omitido = **ambas**). |
| **Descripción** | Elimina filas de `player_equipment_slot` correspondientes (solo desequipa; **no** borrar `player_item` salvo decisión de producto). |
| **Respuesta** | **`204 No Content`** en éxito. |

Validación: misma regla **offline / online + zona de equipar** + `UpdatePlayerStatusAsync` si online.

---

### 5.4 Equipar / desequipar por slot

Identificación única de un slot = (`shipConfigurationId`, `slotKind`, `sortOrder`, `droneSlotIndex`).

| | |
|---|---|
| **Equipar / sustituir** | `PUT /api/v1/equipment/ship-configurations/{shipConfigurationId}/slot-assignments` |
| **Cuerpo** | `{ "slotKind": "laser", "sortOrder": 0, "droneSlotIndex": 255, "playerItemId": 12345 }` |
| **Desequipar** | Mismo `PUT` con `"playerItemId": null` **o** `DELETE` con query `slotKind`, `sortOrder`, `droneSlotIndex`. |
| **Respuesta** | **`204 No Content`** o `200` con el slot actualizado si el cliente lo necesita (sin wrapper `success`). |

**Validación:** posesión del `player_item`, compatibilidad con `slot_kind`, bahía de dron; online + zona de equipar; `UpdatePlayerStatusAsync` tras éxito.

---

### 5.5 Cambiar de nave (clase)

| | |
|---|---|
| **Método / ruta** | `POST /api/v1/equipment/ship/change` |
| **Cuerpo** | `{ "lootId": "ship_goliath" }` (alinear con `Ships.LootId`). |
| **Descripción** | Cambio de **clase** de nave, reglas de posesión y cooldown como hoy. |
| **Respuesta** | **`204 No Content`**, o `200` con un DTO mínimo si hace falta al cliente (p. ej. `{ "shipId": 42 }`) — **sin** `{ "success": true }`. |

---

### 5.6 Cambiar diseño de nave

| | |
|---|---|
| **Método / ruta** | `POST /api/v1/equipment/ship/design` |
| **Cuerpo** | `{ "lootId": "ship_aegis-eic" }` |
| **Descripción** | Misma semántica que `ChangeShipModelAsync`: mismo `baseShipId`, normalización de facción, zona + cooldown. |
| **Respuesta** | **`204 No Content`**, o `200` con datos mínimos de nave si aplica — **sin** `{ "success": true }`. |

---

## 6. Tabla resumen de endpoints

| # | Acción | Método | Ruta |
|---|--------|--------|------|
| 1 | Listar ítems instancia del jugador (`is_wallet_balance = 0`) | `GET` | `/api/v1/equipment/player-items` |
| 2 | Equipamiento actual (query opcional `configurationId`) | `GET` | `/api/v1/equipment/ship-loadout` |
| 3 | Vaciar equipamiento | `DELETE` | `/api/v1/equipment/ship-loadout` |
| 4 | Asignar o quitar ítem en un slot | `PUT` | `/api/v1/equipment/ship-configurations/{shipConfigurationId}/slot-assignments` |
| 5 | Quitar ítem (alternativa) | `DELETE` | `/api/v1/equipment/ship-configurations/{shipConfigurationId}/slot-assignments` + query |
| 6 | Cambiar clase de nave | `POST` | `/api/v1/equipment/ship/change` |
| 7 | Cambiar diseño de nave | `POST` | `/api/v1/equipment/ship/design` |

Paridad conceptual con lo existente bajo `api/equipment/ship/change` y `api/equipment/ship/design`.

---

## 7. Fases de implementación

1. **Contratos:** DTOs de request/response y interfaz de servicio (sin Base64).
2. **Persistencia:** repositorios o `DbContext` sobre `PlayerEquipmentSlot`, `PlayerItem`, `ServerItem` (joins como en consultas SQL del plan de datos).
3. **Reglas:** factor común `EnsureCanModifyEquipmentAsync(playerId)` → online / zona / mensajes.
4. **Game Server:** llamadas `IGameServerClient` alineadas a `UpdatePlayerStatusAsync`, `ChangePlayerShipAsync`, `CanPlayerChangeShipAsync`.
5. **Controller:** `EquipmentController` con `[Authorize]`, rutas §6.
6. **Eliminación:** borrar `FlashInventoryController`, `EquipmentApiController`; limpiar `Program.cs` / OpenAPI; actualizar cliente Vue.
7. **Deprecación código muerto:** métodos de `IEquipmentService` solo-Flash cuando ningún caller los use (o dividir interfaz legacy vs nueva).

---

## 8. Códigos HTTP sugeridos

| Situación | Código |
|-----------|--------|
| No autenticado | `401` |
| Slot / ítem no encontrado o no poseído | `404` |
| **Online y fuera de zona de equipar** | **`409 Conflict`** |
| Cooldown cambio de nave | `429` (preferido) o `409` si se alinea con otros conflictos de estado |
| Validación de body | `400` |

Los errores llevan **`error.code`** y **`error.message`** en el cuerpo.

---

## 9. Referencias de código actual

- Controladores a retirar: `MexOrbit.Api.External/Controllers/Flash/FlashInventoryController.cs`, `EquipmentApiController.cs`.
- Lógica a extraer / replicar: `MexOrbit.DomainService/EquipmentService.cs` (`MoveItemsAsync`, `ChangeShipModelAsync`, `ChangeShipAsync`, `ClearConfigAsync`, validaciones online/zona).
- Cliente GS: `IGameServerClient` (`IsPlayerOnlineAsync`, `IsPlayerInEquipZoneAsync`, `CanPlayerChangeShipAsync`, `UpdatePlayerStatusAsync`, `ChangePlayerShipAsync`).

---

*Documento de planificación; los nombres finales de DTOs pueden ajustarse a la convención global del repo antes de implementar.*
