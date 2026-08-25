# Inicialización del cliente de mapa (Flash) y del Game Server

Este documento resume **qué ocurre cuando el jugador entra al juego** (cliente Flash / mapa), **desde qué capa se configuran** calidad 3D, ventanas, munición, CPUs por defecto y tutoriales, y **por qué** algo (por ejemplo refinamiento o munición “infinita”) puede no coincidir con el DarkOrbit retail.

---

## 1. Vista general de capas

| Capa | Rol |
|------|-----|
| **Navegador + SWF** | Carga el cliente Flash, recursos (`loadingScreen.php` / APIs legacy en `Legacy/`), idioma embebido, hints de pantalla de carga. **No** es el Game Server. |
| **Autenticación HTTP** (CMS / API) | Login, `sessionId`, datos de cuenta; el SWF suele obtener IP/puerto del **socket del mapa** y credenciales de sesión. |
| **TCP al Game Server** (`MexOrbit.GameServer`) | Tras conectar, el cliente envía el **login del juego**; el servidor instancia **`LoginRequestHandler`** y envía **comandos binarios** (Netty) que configuran UI y estado. |
| **MySQL `player_settings`** | JSON por columnas (`quality`, `display`, `window`, `gameplay`, `inGameSettings`, etc.). Si hay fila/columnas, **sobrescriben** los valores por defecto del código. |

La “verdad” de **posición de ventanas, calidad, auto-refinamiento, CPUs seleccionados**, etc. suele ser: **defaults en `SettingsManager.cs`** + **persistencia en `player_settings`** cuando el jugador guarda ajustes o el servidor inserta datos.

---

## 2. Secuencia en el Game Server tras el login

Archivo principal: **`MexOrbit.GameServer/Net/netty/handlers/LoginRequestHandler.cs`**.

1. **`QueryManager.GetPlayer(userId)`** (o reutiliza sesión): carga `player_accounts`, **`player_settings`** (todos los JSON), `player_equipment` parcial, `SetEquipment`, etc.
2. **`Execute()`**: mapa base si no hay posición, tick del jugador, guardado de información.
3. **`SendSettings(player)`** (antes de meter al jugador en el mapa):
   - `SetCurrentCooldowns()`
   - `SendUserKeyBindingsUpdateCommand()` — teclas.
   - **`SendUserSettingsCommand()`** — **calidad, pantalla, audio, ventanas (escala/barras), gameplay, filtros de misiones** (`UserSettingsCommand` con módulos `QualitySettingsModule`, `DisplaySettingsModule`, …).
   - `SendMenuBarsCommand()` — **qué iconos hay en barras** (usuario, nave, chat, mapa, log, pet, booster, entrenamiento si rank 21, etc.).
   - **`SendSlotBarCommand()`** — **slot bars** y **categorías** de ítems (láseres, cohetes, minas, CPUs, techs, formaciones, …).
   - **`SendHelpWindows()`** — lista de **IDs de tutoriales / ayudas** (`class_F2I`) que el cliente puede mostrar.
4. **`SendPlayer(player)`**:
   - **`GetShipInitializationCommand()`** — inicialización de nave en cliente.
   - Paquetes de drones, config (`0|S|CFG|`), booty keys, spaceball, **`SetSpeedCommand`**, objetos del mapa, boosters, pet, **`UpdateStatus`**, **`GetBeaconCommand()`**.
   - Muchas líneas están **comentadas** (Jump CPU, `CpuInitializationCommand`, vídeos, contactos, UBA temporada, etc.): **esas funciones no se envían** en el login actual.

**Conclusión:** La mayor parte de “qué ventanas y barras tiene el juego” y “calidad 3D” sale de **`SendSettings`** + datos en **`Player.Settings`** (BD o default).

---

## 3. Dónde se definen los defaults (sin BD)

Archivo: **`Game/Objects/Players/Managers/SettingsManager.cs`**, clases anidadas sobre `SettingsBase`:

### 3.1 Calidad y gráficos (`QualityBase`, `DisplayBase`)

- **`QualityBase`**: niveles por capas (`qualityAttack`, `qualityBackground`, `qualityShip`, `qualityEngine`, …). Valores por defecto en código (ej. `qualityBackground = 3`, `qualityCustomized = true`).
- **`DisplayBase`**: nombres de jugadores, recursos, cajas, chat, **flags de barra pro**, y sobre todo **ajustes 3D**:
  - `displaySetting3DqualityAntialias`, `displaySetting3DqualityEffects`, `displaySetting3DqualityLights`, `displaySetting3DqualityTextures`, `displaySetting3DsizeTextures`, `displaySetting3DtextureFiltering`, etc.

El cliente Flash interpreta estos módulos al recibir **`UserSettingsCommand`**.

### 3.2 Ventanas y menús (`WindowBase`)

- **`scale`**, **`barState`**, posiciones de **game feature bar**, **generic bar**, **category bar**, **slot bars** (estándar, premium, pro).
- Diccionario **`windows`**: posición/tamaño de ventanas por clave (`"user"`, `"ship"`, `"chat"`, `"minimap"`, `"spacemap"`, `"settings"`, `"help"`, …).

**`SendMenuBarsCommand`** solo añade entradas al menú **si** la clave existe en `windowSettings.windows` **y** según condiciones (pet, spaceball, boosters, rank 21 → training grounds). Lo que **no** se añade aquí **no** tendrá botón en esa barra (aunque el cliente pueda abrir otras pantallas por otro comando).

### 3.3 Gameplay (`GameplayBase`)

Incluye flags como:

- **`autoRefinement`** — por defecto **`false`** en código.
- **`autochangeAmmo`**, **`autoBoost`**, **`quickSlotStopAttack`**, etc.

Se envían en **`GameplaySettingsModule`** dentro de **`SendUserSettingsCommand`**.

### 3.4 CPUs y munición seleccionada (`InGameSettingsBase`)

Por defecto en código:

- **`selectedLaser`**: `LCB_10`
- **`selectedRocket`**: `R_310`
- **`selectedRocketLauncher`**: `HSTRM_01`
- **`selectedFormation`**: formación por defecto del `DroneManager`
- **`selectedCpus`**: lista con **`equipment_extra_cpu_arol-x`** (auto cohetes) y **`equipment_extra_cpu_rllb-x`** (auto hellstorm)

`SettingsManager.SetCurrentItems()` aplica eso a **`Player.Storage`** (AutoRocket, AutoRocketLauncher, etc.).

---

## 4. Munición “infinita”

En **`SendSlotBarCommand`** las categorías (láser, cohetes, …) usan **`GetItemStatus(...)`**, que construye un **`ClientUISlotBarCategoryItemStatusModule`** con **`counterValue = 0`** y **`maxCounterValue = 0`** (ver `GetItemStatus` en `SettingsManager.cs`).

En el protocolo DarkOrbit / cliente Flash, **contador 0 / máximo 0** suele interpretarse como **sin límite mostrado** o **ilimitado** en servidores privados que no descuentan munición en inventario.

**No** es un flag “infinite” explícito en un nombre de variable: es **consecuencia del estado del contador enviado** + **falta de lógica que descuente** munición al disparar (si el servidor no lo implementa, nunca baja).

---

## 5. Ventana de refinamiento y Skylab

- En **`GameplaySettingsModule`** existe el concepto de **auto-refinamiento** (`autoRefinement`), pero la **UI de refinamiento** en el cliente depende de:
  - recursos/recetas del cliente Flash,
  - y comandos del servidor que **abran** o **habiliten** esa capa.

- **`UIOpenRequest`** define **`ACTION_REFINEMENT = "refinement"`**, pero **`UIOpenRequestHandler`** solo maneja **`ACTION_LOGOUT`** y un caso parcial de **`ACTION_SHIP_WARP`**. **No hay rama `refinement`**: el servidor **no** reacciona a “abrir refinamiento” desde ese handler.

- Los **IDs de ayuda** en **`SendHelpWindows`** (`class_F2I`) incluyen **Skylab** (`SKYLAB = 1`), **ITEM_UPGRADE**, **ORE_TRANSFER**, etc., pero **no hay una constante dedicada “REFINEMENT”** en `class_F2I.cs` comparable a otras features: el refinamiento suele ir **acoplado al cliente Skylab/ore** en el SWF.

**Por qué no ves la ventana de refinamiento:** suele ser **combinación** de (a) cliente/recursos incompletos, (b) servidor sin comandos que activen esa UI, (c) menú no expuesto en `SendMenuBarsCommand` (solo entradas listadas allí + `WindowBase.windows`).

---

## 6. CPUs en barra y categoría “CPU”

En **`SettingsManager`**, **`CpusCategory`** es una lista **muy acotada** en el código actual (solo **3 loot IDs** visibles en la categoría, el resto comentado). Los CPUs “por defecto” en **`selectedCpus`** siguen siendo **AROL-X** y **RLLB-X** para auto-ataques, pero **la barra de selección** solo muestra lo que **`CpusCategory`** y el inventario lógico permitan.

Si quieres más CPUs en la UI, hay que **ampliar `CpusCategory`** (y posiblemente lógica de equipado) en el mismo archivo.

---

## 7. Carga del Flash en el navegador (legacy)

En el repo, **`Legacy/DarkOrbit-CMS-Wolf_Fr/flashAPI/loadingScreen.php`** responde acciones como `getResourceAsRaw`, `getAllHints`, `setClientBrowserConfig`: eso alimenta **pantalla de carga e idioma**, no sustituye al **login del Game Server**.

El flujo típico es: **HTTP** para sesión y recursos → **socket** al **Game Server** → **`LoginRequestHandler`** como arriba.

---

## 8. Tabla rápida “¿dónde lo cambio?”

| Quiero… | Dónde mirar primero |
|--------|----------------------|
| Calidad 3D / efectos | `DisplayBase` + columna `display` en `player_settings` |
| Posición de ventanas / barras | `WindowBase` + columna `window` |
| Auto-refinamiento | `GameplayBase.autoRefinement` + columna `gameplay` |
| CPUs por defecto al entrar | `InGameSettingsBase.selectedCpus` |
| Iconos del menú superior izquierdo/derecho | `SendMenuBarsCommand` (condiciones + `WindowBase.windows`) |
| Quitar comentarios (Jump CPU, etc.) | `LoginRequestHandler.SendPlayer` |
| Munición finita con contador real | Implementar descuento al atar + enviar contadores reales en `GetItemStatus` / módulos de inventario (trabajo mayor) |

---

## 9. Referencias de código

| Tema | Archivo |
|------|---------|
| Login y orden SendSettings / SendPlayer | `Net/netty/handlers/LoginRequestHandler.cs` |
| Defaults de settings y envío de UI | `Game/Objects/Players/Managers/SettingsManager.cs` |
| Carga BD de `player_settings` | `Managers/QueryManager.cs` (`GetPlayer`) |
| Ayuda/tutorial IDs | `SendHelpWindows`, `Net/netty/commands/class_F2I.cs` |
| Apertura UI genérica (refinamiento sin implementar) | `Net/netty/handlers/UIOpenRequestHandler.cs`, `Net/netty/requests/UIOpenRequest.cs` |

---

## 10. Enlaces a otros documentos

- Equipamiento y stats: [equipment-stats-calculation.md](./equipment-stats-calculation.md)  
- Sincronización inventario: [equipment-and-game-server-sync.md](./equipment-and-game-server-sync.md)  
- Daño por disparo: [combat-damage-randomization.md](./combat-damage-randomization.md)
