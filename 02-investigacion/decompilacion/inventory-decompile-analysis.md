# Análisis del SWF de inventario decompilado

**Ruta analizada:** `Decompile/inventory/` (principalmente `inventory decompiled/scripts/net/bigpoint/darkorbit/equipment/…` y `net/bigpoint/dataInterchange/`).

**Herramienta típica de decompilación:** JPEXS / FFDec (ActionScript 3).

Este documento recopila **constantes, acciones HTTP, claves JSON, flujos PureMVC y puentes JS** útiles para alinear el **backend** (`flashAPI` / `EquipmentService`) con el cliente Flash.

---

## 1. Alcance del módulo

| Área | Descripción |
|------|-------------|
| **Patrón** | **PureMVC**: `Proxy` (datos), `Mediator` (vistas), `Command`, notificaciones por string. |
| **Transporte** | Solo **HTTP POST** a un único endpoint PHP (configurable por host). **No** hay socket en este SWF para inventario. |
| **Paquete raíz lógico** | `net.bigpoint.darkorbit.equipment` |
| **Otros assets** | `Decompile/inventory/graphics/ui/scrollComponents decompiled/scripts/` — clips de UI (botones, tabs, scroll); misma base visual que el módulo principal. |

---

## 2. Endpoint HTTP y formato de petición

**Clase:** `net.bigpoint.dataInterchange.DataInterchange`

| Constante / código | Valor |
|--------------------|--------|
| URL | `Settings.dynamicHost + "flashAPI/inventory.php"` |
| Método | `POST` (`HTTPConnector`) |
| Campos POST | `action` = nombre de la acción (string). `params` = **JSON serializado** del objeto de parámetros (`serializedParameters`). |

**Respuesta esperada (después de deserializar JSON):**

- `isError` — `1` = error (ver `error.message`).
- `data` — payload (`data.ret` en init contiene el hangar/inventario).

Lógica en `InterchangeBroker.replyReceived`: si `isError == 1` → `hasErrors`; si hay error, mensaje en `error.message`.

**Equivale** al contrato que ya documentáis en la API .NET (envelope Flash con `isError` / `error.message`).

---

## 3. Inicialización (`init`)

| Paso | Detalle |
|------|---------|
| Entrada | `ConnectionProxy.beginConnection()` crea `DataInterchange` y llama `fetchConfig(1)`. |
| Acción | `executeAction("init", { nr: 1 }, …)` → `action = "init"`, parámetros `{ nr: 1 }`. |
| Callback | `CONFIG_RECEIVED` → si la acción es `"init"` o `"dummyInit"`, `buildStructure(response.data)` y notificación `BOOTSTRAP_FINISHED` (`ApplicationNotificationNames.BOOTSTRAP_FINISHED` = `"ASSETS_LOADED"`). |
| Datos | `param1.ret` = estructura hangar; `param1.map` = mapa de ítems para `FilterManager.initNameMaps`. |

---

## 4. Acciones de servidor (string `action`)

Estas son las cadenas que el cliente envía en **`request.action`**. Deben coincidir con lo que el PHP / controlador Flash acepta.

| `action` (valor) | Uso principal |
|------------------|----------------|
| `init` | Carga inicial (ver §3). |
| `move` | Mover ítems (equip/desequip/reordenar según `Transporter`). |
| `droneEquip` | Equipar varios drones (batch). |
| `sell` / `sellDrone` / `sellShip` / … | Venta (según transporter). |
| `repair` / `repairDrone` / `repairModule` | Reparación. |
| `destroy` | `ActionIdentifiers.REMOVE` — destruir ítem. |
| `split` | Partir stack. |
| `renamePet` | Renombrar pet. |
| `quickBuy` | Compra rápida. |
| `unlockSlot` | Desbloquear slot. |
| `clearConfig` | Limpiar configuración de equipo. |
| `getHangar` | Datos de hangar/slots (`GET_HANGARSLOT_DATA`). |
| `setAbCpuAmmo` | `ActionIdentifiers.CPU_MODE` — modo CPU munición. |
| `changeShipModel` | Cambio de diseño de nave. |

**Especiales en `DataInterchange`:**

- `clearShipEquipmentConfiguration` → internamente `executeAction(ActionIdentifiers.CLEAR_EQUIPMENT_CONFIG, { configID, wipe: 1 }, …)` → acción **`clearConfig`**.
- `designChangeRequest` → acción **`changeShipModel`** (ver `ActionIdentifiers.DESIGN_CHANGE`).
- `petNameChangeRequest` → **`renamePet`**.
- `cpuModeChange` → misma tubería `executeAction` con la acción pasada (p. ej. `setAbCpuAmmo`).

---

## 5. `ActionIdentifiers` (constantes completas)

Archivo: `…/events/ActionIdentifiers.as`

**Tabs / contexto**

- `SHIP` = `"ship"`, `PET` = `"pet"`, `DRONES` = `"drone"`, `DESIGN` = `"design"`.
- Slots / categorías: `LASERS`, `GENERATORS`, `HEAVY_GUNS`, `EXTRAS`, `GEARS`, `PROTOCOLS`, `RESOURCE`, `MODULE`.

**Acciones y claves de arrastre**

- `MOVE` = `"move"`, `DRONE_EQUIP` = `"droneEquip"`, `SELL`, `SELL_DRONE`, `SELL_SHIP`, `SELL_PET`, `REPAIR`, `CPU_MODE` = `"setAbCpuAmmo"`, `REPAIR_DRONE`, `REPAIR_MODULE`, `REMOVE` = `"destroy"`, `SPLIT`, `CHANGE_PET_NAME` = `"renamePet"`, `QUICK_BUY` = `"quickBuy"`, `BUY_SLOT` = `"unlockSlot"`, `CLEAR_EQUIPMENT_CONFIG` = `"clearConfig"`, `GET_HANGARSLOT_DATA` = `"getHangar"`.
- Internas UI: `MOVE_TO_INVENTORY`, `INVENTORY_REARRANGE`, `INVENTORY_ITEM_SPLIT`, `EQUIPMENT_SHIP_MOVE`, `EQUIPMENT_DRONE_MOVE`, `EQUIPMENT_NDRONE_MOVE`, `SWAPPED_ITEM_*`, `DRAGGED_*`, `PUT_DRAGGED_ITEM_BACK_FUNCTION`, etc.
- `DESIGN_CHANGE` = `"changeShipModel"`.
- `MOVE_ALL` = `1`, `MOVE_NONE` = `0`.

**CDN / resolución de iconos**

- `CDN_30` / `CDN_63` / `CDN_100` / `CDN_TOP` y `RES_30`, `RES_63`, `RES_100`, `RES_TOP`, `RES_ICON`.
- `LF4` = `"lf-4"` (filtro/categoría).

---

## 6. `ApplicationNotificationNames` (notificaciones PureMVC)

Archivo: `…/application/ApplicationNotificationNames.as`

Incluye (entre otras): `STARTUP`, `INIT_APPLICATION`, `BOOTSTRAP_FINISHED` (`ASSETS_LOADED`), `INIT_INVENTORY`, `CONFIG_SWITCHED`, `UPDATE_EQUIPMENT`, `QUERY_*` para movimientos, `TOGGLE_SUSPENDED_VIEW`, `SHOW_POP_UP` (`SHOW_TOOL_TIP`), `FILTER_INVENTORY_BY_CONTEXT`, `QUERY_SERVER_FOR_CPU_MODE_CHANGE`, `CHANGE_CONTEXT_BUTTON`, `EXTEND_EXTRA_SLOTS`, `HANGAR_SLOT_CHANGED`, `INIT_INVENTORY_FOR_SPACEMAP`, etc.

Sirven para **seguir el flujo en el código decompilado** (buscar `sendNotification(ApplicationNotificationNames.…)`).

---

## 7. `DataInterchange` — claves JSON usadas en el protocolo

Archivo: `…/dataInterchange/DataInterchange.as` (constantes estáticas).

| Constante | Significado (resumen) |
|-----------|------------------------|
| `ITEM_ID` / `I`, `ITEM_LOOT_ID` / `L`, `ITEM_QUANTITY` / `Q`, `ITEM_SLOT` / `S` | Identificación y cantidad de ítems. |
| `ITEM_SLOTSETS`, `ITEM_GROUPS`, `ITEM_HITPOINTS` / `HP`, `ITEM_SELLING` | Inventario y venta. |
| `SELECTED_DESIGN` / `SM`, `AVAILABLE_DESIGNS` / `M` | Diseños de nave. |
| `EQUIPPED_ITEMS` / `EQ` | Equipamiento en hangar. |
| Drones | `DRONE_DAMAGE`, `DRONE_EFFECT` / `EF`, `DRONE_DESIGN` / `DE`, `REPAIR_PRICE`, `DRONE_CURRENCY`, etc. |
| Pet | `PET_NAME` / `PN`, slots y costes (`COST_*_SLOT`). |
| Otros | `UNLOCKABLE_SLOTS` / `lockedSlots`, `MODULE_INSTALLED` / `IN`, `WORDPUZZLE_*` (evento puzzle). |

Estas claves deben alinearse con el JSON que genera el **servidor** (mismo modelo que `EquipmentService` / PHP legacy).

---

## 8. `ConnectionProxy` — hallazgos clave

Archivo: `…/model/ConnectionProxy.as`

| Símbolo | Valor / comportamiento |
|---------|-------------------------|
| `PROXY_NAME` | `"ConnectionProxy"` |
| `INVENTORY_EQUIPPED` | `0` |
| `MAX_CONFIGS` | `2` |
| `autoBuyAmmo` | **`"equipment_extra_cpu_alb-x"`** |
| `autoBuyRocket` | **`"equipment_extra_cpu_rb-x"`** |
| `EXTENDER_CPUS` | Mapa: `sle-01`…`sle-04` → slots extra = `Settings.defaultNumberOfExtrasSlots + 2/4/6/10` (`fillCPUExtenders`). |

**Flujos que llaman al servidor:**

- `itemMoveRequest`, `itemRepairRequest`, `itemSellRequest`, `rearrangeInventoryRequest`, `splitInventoryItemRequest`, `buyExtraPetSlotRequest`, `shipSellRequest`.
- `getHangarSlotDataRequest` → `serverRequest("getHangar", null)`.
- Cambio de diseño: `designChangeRequest` → `changeShipModel`.
- Respuesta genérica: `handleServerActionReply` revisa `sessionInvalid`, `update`, `ret`, acciones `SELL_DRONE`, `REPAIR_*`, `BUY_SLOT`, etc.

---

## 9. `Settings` (config inyectada desde HTML/JS)

Archivo: `…/model/Settings.as`

| Campo | Uso |
|-------|-----|
| `dynamicHost`, `cdnHost` | Host del juego y CDN. |
| `userID`, `username`, `password`, `session` implícitos vía params | Contexto de sesión. |
| `WINDOW_WIDTH` / `WINDOW_HEIGHT` | **770 × 395** (tamaño lógico del panel). |
| `defaultNumberOfExtrasSlots` | Base para slots extra y tabla `EXTENDER_CPUS`. |
| `USER_HAS_DRONES`, `USER_HAS_PET` | Feature flags de UI. |
| `DRONE_REPAIR_COST`, `PET_RENAME_COST`, arrays de precios de slots pet | Economía UI. |
| `language` | Por defecto `"dev"`. |
| `INVENTORY_NAME` / `ACCORDION_NAME` | Identificadores de UI. |

---

## 10. UI: tabs y popups

**`TabButtonComponent`**

- `ID_SHIP` = `0`, `ID_DRONES` = `1`, `ID_PET` = `2`.

**`PopUpDefiner`** (`…/transporter/PopUpDefiner.as`)

Tipos: `SELL_POPUP`, `SELL_SHIP_POPUP`, `SPLIT_STACK_POPUP`, `BUYABLE_SLOT_POPUP`, `REPAIR_ITEM_POPUP`, `REPAIR_MODULE_POPUP`, `RENAME_PET`, `QUICK_BUY_ITEM`, `AMMO_CPU_CHOICE`, `ERROR_POPUP`, `CLEAR_SHIP_CONFIG_CONFIRMATION_POPUP`.

**`PopUp.as`**

- `BIG_POP_UP` / `SMALL_POP_UP`, `TOTAL_PULSES` = `6` (animación).
- Casos especiales para CPUs `ConnectionProxy.autoBuyAmmo` y `autoBuyRocket` en flujos de compra automática.

---

## 11. Eventos de vista `MoveItemEvent`

Archivo: `…/events/MoveItemEvent.as`

Tipos: `ITEM_REARRANGED_WITHIN_INVENTORY`, `ITEMS_MOVED_TO_INVENTORY`, `ITEMS_MOVED_TO_SHIP_EQUIPMENT`, `ITEMS_MOVED_TO_DRONE_EQUIPMENT`, `ITEMS_MOVED_TO_N_DRONE_EQUIPMENT`, `ITEMS_MOVED_TO_PET_EQUIPMENT`, `SELL_ITEM`, `REMOVE_ITEM`, `STACK_MOVED_TO_EMPTY_SLOT`, `QUICK_BUY_ITEM`, `SELL_SHIP_OR_PET` (constante = `"SELL_PET"`).

---

## 12. `ExternalInterfaceManager` (puente JavaScript)

Funciones JS esperadas en la página (si `ExternalInterface.available`):

- `referToURL`, `redirectToExternalHome`, `redirectToItemUpgradeSystem`, `allowBrowserScroll`, `openShopCategory`, `updateMoneyDisplay`, `eval`.
- Deep links de ejemplo: `indexInternal.es?action=internalDock&tpl=internalDockPetGear` y `…internalDockDrones`.

Sirve para **redirecciones** a tienda / home cuando el SWF no puede completar la acción solo con HTTP.

---

## 13. `ConfigManager` (singleton)

- Operaciones locales: `ADD` / `REMOVE` sobre configs de equipo tras respuesta del servidor.
- Construye estructura por `ship`, `pet`, slotsets (`lasers`, `generators`, etc.) alineada con `ActionIdentifiers`.

---

## 14. Carpeta `graphics/ui/scrollComponents decompiled`

Decenas de símbolos gráficos (`inventory.as`, `config.as`, `sellConfirm.as`, `quickBuy.as`, …): **no contienen lógica de red**, solo **MovieClips** y componentes visuales para el mismo flujo de inventario.

---

## 15. Relación con MexOrbit (.NET)

| Cliente Flash (este análisis) | Backend MexOrbit |
|------------------------------|------------------|
| `POST flashAPI/inventory.php` `action` + `params` | `FlashInventoryController` / rutas equivalentes bajo `flashAPI/` + `EquipmentService` para REST moderno. |
| Misma semántica `move`, `clearConfig`, `changeShipModel`, `droneEquip`, etc. | Implementado o mapeado en `EquipmentService` y DTOs `Flash*`. |
| `isError` / `data` / `error.message` | Mismo envelope Base64/JSON en respuestas emuladas. |

Para **validar** una acción nueva: buscar el string `action` en el PHP o en el controlador Flash y comparar con la tabla del **§4**.

---

## 16. Cómo seguir ampliando este documento

1. Buscar en el AS3 todas las cadenas `"executeAction("` y `serverRequest(` por si hay acciones extra en otras clases.
2. Comparar con `FlashInventoryController.cs` / PHP `inventory.php` del repo **línea a línea** para parámetros exactos (`from`/`to`, `configId`, etc.).
3. Mantener **versión del SWF** anotada en la cabecera de este doc cuando se reemplace el binario (los strings `action` pueden variar entre parches).

---

*Generado a partir del árbol decompilado bajo `Decompile/inventory/` (abril 2026).*
