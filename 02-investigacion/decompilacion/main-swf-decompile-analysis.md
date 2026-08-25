# Análisis del SWF principal (main) decompilado

**Ruta:** `Decompile/main/` — contiene ~5 000 archivos AS3 (cliente de mapa principal).  
**Ofuscación:** alta; nombres de paquetes/clases tipo `§_-G4V§`, `§_-p1j§`, etc. Las constantes literales (strings, ints) son legibles.

---

## 1. FlashVars (parámetros del HTML al SWF)

**Archivo:** `net/bigpoint/darkorbit/settings/FlashVarsParser.as`

El SWF principal recibe su configuración del HTML que lo embebe. Estos parámetros se parsean al inicio:

| FlashVar | Destino | Tipo | Uso |
|----------|---------|------|-----|
| `host` / `dynamicHost` | `Settings.dynamicHost` | String → `"http://" + host + "/"` | Host base de recursos/API |
| `userID` | `§_-l3H§.userID` | int | ID del jugador |
| `factionID` | `§_-l3H§.factionID` | int | Facción (1=MMO, 2=EIC, 3=VRU) |
| `sessionID` | `§_-l3H§.sessionID` | String | Token de sesión |
| `mapID` | `Settings.mapID` | int | Mapa inicial |
| `basePath` | `Settings.basePath` | String | Ruta base del servidor |
| `cdn` | `Settings.staticHost` | String | CDN para assets estáticos |
| `lang` | `Settings.language` | String | Idioma (solo primera parte de locale) |
| `pid` / `projectID` | `Settings.instanceID` | int | ID de instancia del servidor |
| `localGS` / `localGs` | `Settings.defaultGameServer` | String | Host del Game Server (`"1"` → `"localhost"`) |
| `chatHost` | `Settings.chatHost` | String | Host del servidor de chat |
| `display2d` | `Settings.userType` | int | 1=3D, 2=2D, 0=no seleccionado |
| `autoStartEnabled` / `autostartEnabled` | `Settings.singleSessionAutoStartEnabled` | Boolean | Auto-inicio |
| `allowChat` | `Settings.createChat` | Boolean | Habilitar chat |
| `showAdvertisingHint` | `Settings.advertisingHintEnabled` | Boolean | Hints publicitarios |
| `useDeviceFonts` | `Settings.useEmbeddedFonts` | Boolean | `"1"` → device fonts |
| `browser` | `Settings.browser` | String | Nombre del navegador |
| `profileXmlHash` | `Settings.profileXMLHash` | String | Hash de XML de perfil |
| `gameXmlHash` | `Settings.gameXMLHash` | String | Hash de XML de juego |
| `resourcesXmlHash` | `Settings.resourceXMLHash` | String | Hash de XML de recursos |
| `languageXmlHash` | `Settings.languageXMLHash` | String | Hash de XML de idioma |
| `eventStreamContext` | `Settings.eventStreamContext` | String | Contexto de analytics |
| `boardLink` | `Settings.boardLink` | String | Link al foro |
| `helpLink` | `§_-g2R§.helpLink` | String | Link de ayuda |
| `loadingClaim` | Locale storage | String | Texto de carga (URI-decoded) |

---

## 2. Protocolo de red (socket binario)

### 2.1 Conexión

- **Socket:** `flash.net.Socket`, `Endian.BIG_ENDIAN`.
- Host/puerto vienen de **flashvars** (`Settings.defaultGameServer`, `Settings.chatHost`).
- Eventos: `CONNECTION_SUCCESS`, `CONNECTION_ERROR`, `CONNECTION_LOST`.

### 2.2 Formato de trama

1. Se leen **2 bytes**, se decodifican (codec XOR con clave dinámica `setSecretKey`).
2. El resultado es un `unsigned short` = **longitud del cuerpo**.
3. Se leen esos bytes, se decodifican.
4. Primer campo del cuerpo: `readShort()` = **command ID** (int16).
5. El resto lo consume `ICommand.read()` de la clase registrada para ese ID.

### 2.3 IDs de comando (582 entradas)

Cada paquete de red es una clase en `Decompile/main/main/scripts/§_-G4V§/` que implementa `ICommand` y declara `public static const ID:int = <número>`. En el decompilado hay **582** clases con ese patrón, con **582 IDs distintos** (ningún ID duplicado entre archivos). El **rango numérico** va de **39** a **32621** (no es contiguo: hay huecos).

**Listado completo (ID → archivo `.as` ofuscado):**

- [g4v-command-ids.md](g4v-command-ids.md) — tabla Markdown con las 582 filas.
- [g4v-command-ids.tsv](g4v-command-ids.tsv) — mismo dato en TSV (`id` + `file`) para scripts o hojas de cálculo.
- [g4v-server-crossref.md](g4v-server-crossref.md) — **cruce con MexOrbit.GameServer**: para cada ID, clase C# (`netty/commands`, `netty/requests` o `Handler.cs`) y dirección del paquete (~200 IDs con implementación en servidor; el resto solo en cliente).

**Algunos IDs ya citados en otra documentación (contexto):**

| ID | Archivo | Contexto (inferido del uso en el cliente) |
|----|---------|-------------------------------------------|
| 29864 | `§_-F2I§.as` | Sistema de hints / tutoriales |
| 18193 | `§_-G4w§.as` | Tipos de mapa (normal, galaxy gate, invisible…) |
| 26664 | `§_-G4F§.as` | Tipos de objetivo de misión |
| 23438 | `§_-r2Z§.as` | Eventos de mapa estacionales |
| 1038 | `§_-W2f§.as` | Tipos de efecto técnico de drones |

El **mapa semántico** (qué hace cada ID en gameplay) no está en el SWF con nombres legibles; conviene alinear estos números con los comandos del **Game Server** (.NET) o con capturas de tráfico.

En el mismo directorio hay **583** archivos `.as`; **582** contienen al menos un `public static const ID:int` (el que falta es `§_-ki§.as`, factoría / registro de comandos, sin ID de paquete propio).

---

## 3. Ventanas del juego (`GuiConstants`)

**Archivo:** `net/bigpoint/darkorbit/mvc/gui/GuiConstants.as`

Cada constante es el **string ID** que identifica una ventana en el sistema MVC. Si el servidor no la habilita o no envía datos para crearla, no aparece.

| Constante | String | Descripción |
|-----------|--------|-------------|
| `USER_WINDOW` | `"user"` | Panel de usuario |
| `SHIP_WINDOW` | `"ship"` | Info de nave |
| `MINIMAP_WINDOW` | `"minimap"` | Minimapa |
| `SPACEMAP_WINDOW` | `"spacemap"` | Mapa completo |
| `CHAT_WINDOW` | `"chat"` | Chat |
| `LOG_WINDOW` | `"log"` | Log de eventos |
| `GROUP_SYSTEM_WINDOW` | `"group"` | Sistema de grupo |
| `PET_WINDOW` | `"pet"` | Pet |
| `BOOSTER_WINDOW` | `"booster"` | Boosters activos |
| `SETTINGS_WINDOW` | `"settings"` | Ajustes |
| `SETTINGS_WINDOW_3D` | `"settings3d"` | Ajustes 3D |
| `HELP_WINDOW` | `"help"` | Ayuda |
| `LOGOUT_WINDOW` | `"logout"` | Desconexión |
| `SHIP_WARP_WINDOW` | `"ship_warp"` | Cambio de nave |
| `QUEST_WINDOW` | `"quests"` | Misiones |
| `TRADE_WINDOW` | `"ore_trade"` | Comercio de mineral |
| **`REFINEMENT_WINDOW`** | **`"refinement"`** | **Refinamiento** |
| `REFINEMENT_UPDATE_WINDOW` | `"refinement_update"` | Mejora refinamiento |
| `REFINEMENT_COUNT_WINDOW` | `"refinement_count"` | Conteo refinamiento |
| `REFINEMENT_REFINE_WINDOW` | `"refinement_refine"` | Acción refinar |
| `SPACEBALL_WINDOW` | `"spaceball"` | Evento Spaceball |
| `INVASION_WINDOW` | `"invasion"` | Evento invasión |
| `CTB_WINDOW` | `"ctb"` | Capture the Beacon |
| `TDM_STATUS_UI_WINDOW` | `"tdm"` | Team Deathmatch |
| `RANKED_HUNT_EVENT_WINDOW` | `"ranked_hunt"` | Caza clasificada |
| `HIGH_SCORE_GATE_WINDOW` | `"highscoregate"` | Puerta de puntuación |
| `SCOREEVENT_WINDOW` | `"scoreevent"` | Evento de puntos |
| `INFILTRATION_GAME_WINDOW` | `"infiltration"` | Infiltración |
| `WORD_PUZZLE_WINDOW` | `"word_puzzle"` | Puzzle de letras |
| `CURCUBITOR_COUNTDOWN_STATUS_WINDOW` | `"curcubitor"` | Curcubitor |
| `SECTOR_CONTROL_WINDOW` | `"sectorcontrol"` | Control de sector |
| `JACKPOT_STATUS_UI_WINDOW` | `"jackpot_status_ui"` | Arena Jackpot |
| `VIDEO_WINDOW` | `"video"` | Vídeo |
| `BANNER_ADD_WINDOW` | `"banner_add"` | Banner publicitario |
| `HELP_VIDEO_WINDOW` | `"help_video"` | Vídeo de ayuda |
| `HINT_SYSTEM_OVERVIEW_WINDOW` | `"hint_system_overview"` | Resumen de pistas |
| `CONTACTLIST_WINDOW` | `"contacts"` | Lista de contactos |
| `TRAINING_GROUNDS_WINDOW` | `"traininggrounds"` | Campo de entrenamiento |
| `TRAINING_GROUNDS_RESULTS_WINDOW` | `"traininggroundsResults"` | Resultados entrenamiento |
| `INFLUENCE_WINDOW` | `"influence"` | Influencia |
| `DOMINATION_WINDOW` | `"domination"` | Dominación |
| `TARGETED_OFFERS_WINDOW` | `"targetedOffers"` | Ofertas dirigidas |
| `PLAGUE_WINDOW` | `"spaceplague"` | Plaga espacial |
| `CONFIRMATION_POPUP` | `"confirmationPopup"` | Popup genérico |
| `RESET_PROMPT_WINDOW` | `"reset_prompt"` | Prompt de reset |
| `AUTOSTART_WARNING_WINDOW` | `"autostart_warning"` | Aviso autostart |
| `COMMAND_LINE_WINDOW` | `"cli"` | Línea de comandos |
| `CONNECTION_WINDOW` | `"connection"` | Ventana conexión |
| `CONNECTION_LOST_WINDOW` | `"lost_connection"` | Conexión perdida |
| `POPUP_WINDOW` | `"popup"` | Popup |

Ventanas Flex registradas aparte: `QuestGiver`, `BattleStationBuild`, `CompanyHierarchy`, `ArenaTournament`, `WordPuzzle`, `SectorControlLobby`, `TeamDeathMatch*`, `JackpotArenaMatchResult`, `SoundConfig`.

---

## 4. Settings del cliente

**Archivo:** `net/bigpoint/darkorbit/settings/Settings.as`

### 4.1 Tipo de renderizado

| Constante | Valor |
|-----------|-------|
| `USER_TYPE_2D` | `2` |
| `USER_TYPE_3D` | `1` |
| `USER_TYPE_NOT_SELECTED` | `0` |
| `USER_TYPE_NEW` | `-1` |

### 4.2 Calidad

| Constante | Valor |
|-----------|-------|
| `QUALITY_LOW` | `0` |
| `QUALITY_MEDIUM` | `1` |
| `QUALITY_GOOD` | `2` |
| `QUALITY_HIGH` | `3` |

Defaults: `qualityPresetting = 3` (HIGH), `qualityBackground/qualityPoizone = HIGH`, `qualityShip = LOW`.

### 4.3 Auto-reducción de calidad (`AQ_*`)

| Nivel | Efecto |
|-------|--------|
| `AQ_NONE` (0) | Sin reducción |
| `AQ_NO_PORTAL_TARGET_PREVIEW_LIMIT` (1) | Sin preview de portales |
| `AQ_NO_ENGINE_SMOKE_LIMIT` (2) | Sin humo de motor |
| `AQ_NO_ANIMATED_MAPASSETS_LIMIT` (3) | Sin assets animados |
| `AQ_LOW_EXPLOSION_DETAIL_LIMIT` (4) | Explosiones de baja calidad |
| `AQ_NO_STARFIELD_MOVEMENT_LIMIT` / `AQ_NO_EXPLOSION_LIMIT` (5) | Sin estrellas/explosiones |
| `AQ_MAX_REDUCTION` | `5` |

### 4.4 Gameplay y display

| Variable | Default | Descripción |
|----------|---------|-------------|
| `autochangeAmmo` | `true` | Cambio automático de munición |
| `autoBoost` | `false` | Auto-activar boosters |
| `autoRefinement` | `false` | Auto-refinamiento |
| `quickSlotStopAttack` | `true` | Detener ataque al cambiar slot |
| `doubleclickAttackEnabled` | `true` | Doble clic para atacar |
| `autoBuyBootyKeys` | `false` | Compra automática de llaves |
| `showLowHpWarnings` | `true` | Aviso de HP bajo |
| `showNotOwnedItems` | `false` | Mostrar ítems no propios |
| `autoStartEnabled` | `false` | Auto-inicio |
| `displayHitpointBubbles` | `true` | Burbujas de HP |
| `displayDrones` | `true` | Mostrar drones |
| `displayPlayerNames` | `true` | Nombres de jugadores |
| `displayBonusBoxes` | `true` | Cajas de bonus |
| `displayChat` | `true` | Chat visible |
| `displayWindowsBackground` | `true` | Fondo de ventanas |
| `displayBattlerayNotifications` | `false` | Notificaciones de rayo |
| `preloadUserShips` | `false` | Precargar naves |

### 4.5 Audio

`musicVolume = 50`, `soundVolume = 50`, `voiceVolume = 50`, `playCombatMusic = true`.

### 4.6 Variables de conexión y entorno

`defaultGameServer`, `chatHost`, `basePath`, `staticHost`, `dynamicHost` (`"/"`), `language`, `currency` (`"EUR"`), `instanceID`, `mapID`, `nextMapID`, `lastMapID`.

### 4.7 Calidad 3D detallada

Settings persistidas en `SharedObject` local (clave `"darkorbit"`):

| Variable | Default | Descripción |
|----------|---------|-------------|
| `displaySetting3DqualityAntialias` | `3` | Nivel de antialiasing 3D |
| `displaySetting3DqualityLights` | `3` | Calidad de luces 3D |
| `displaySetting3DqualityTextures` | `3` | Calidad de texturas 3D |
| `displaySetting3DsizeTextures` | `3` | Tamaño de texturas 3D |
| `displaySetting3DqualityEffects` | `3` | Calidad de efectos 3D |
| `displaySetting3DtextureFiltering` | `3` | Filtrado de texturas 3D |
| `FORCE_2D` | `false` | Forzar modo 2D |
| `gpuSupport` | `true` | Soporte GPU detectado |

Texturas por entidad (`Settings3D`): drone=64px, ship=128px, ship_big=256px, edificios/planetas=512-1024px.

---

## 5. Hints / tutoriales (`§_-F2I§`)

IDs de tipo `short` que el servidor envía para indicar qué tutoriales ha "completado" el jugador. **Coincide** con `class_F2I` en el Game Server.

| Constante | Valor | Feature |
|-----------|-------|---------|
| `SHIP_REPAIR` | 0 | Reparación de nave |
| `SKYLAB` | 1 | Skylab |
| `PVP_WARNING` | 2 | Aviso PvP |
| `THE_SHOP` | 3 | Tienda |
| `CHANGING_SHIPS` | 4 | Cambio de naves |
| `INSTALLING_NEW_EQUIPMENT` | 5 | Instalar equipo |
| `JUMP_GATES` | 6 | Puertas de salto |
| `PREPARE_BATTLE` | 7 | Preparar batalla |
| `GET_MORE_AMMO` | 8 | Más munición |
| `BOOST_YOUR_EQUIP` | 9 | Impulsar equipo |
| `SELL_RESOURCE` | 10 | Vender recurso |
| `WEALTHY_FAMOUS` | 11 | Rico y famoso |
| `WELCOME` | 12 | Bienvenida |
| `HOW_TO_FLY` | 13 | Cómo volar |
| `REQUEST_MISSION` | 14 | Solicitar misión |
| `POLICY_CHANGES` | 16 | Cambios de política |
| `EQUIP_YOUR_ROCKETS` | 17 | Equipar cohetes |
| `UNKOWN_DANGERS` | 18 | Peligros desconocidos |
| `ATTACK` | 19 | Ataque |
| `JUMP_DEVICE` | 20 | Dispositivo de salto |
| `FULL_CARGO` | 21 | Carga llena |
| `SECOND_CONFIGURATION` | 22 | Segunda config |
| `ITEM_UPGRADE` | 23 | Mejora de ítem |
| `GALAXY_GATE` | 24 | Galaxy gate |
| `SKILL_TREE` | 25 | Árbol de habilidades |
| `TECH_FACTORY` | 26 | Tech factory |
| `CLAN_BATTLE_STATION` | 27 | Estación de batalla |
| `PALLADIUM` | 28 | Paladio |
| `AUCTION_HOUSE` | 29 | Casa de subastas |
| `ROCKET_LAUNCHER` | 30 | Lanzacohetes |
| `EXTRA_CPU` | 31 | CPU extra |
| `SHIP_DESIGN` | 32 | Diseño de nave |
| `LOOKING_FOR_GROUPS` | 33 | Buscar grupo |
| `CONTACT_LIST` | 34 | Lista de contactos |
| `ORE_TRANSFER` | 35 | Transferencia mineral |
| `TRAINING_GROUNDS` | 36 | Entrenamiento |

---

## 6. Tipos de objetivo de misión (`§_-G4F§`)

| Constante | Valor | Descripción |
|-----------|-------|-------------|
| `TIMER` | 1 | Temporizador |
| `HASTE` | 2 | Rapidez |
| `KILL_NPC` | 6 | Matar NPC |
| `DAMAGE` | 7 | Hacer daño |
| `TAKE_DAMAGE` | 9 | Recibir daño |
| `TRAVEL` | 13 | Viajar |
| `EMPTY` | 18 | Vacío |
| `AMMUNITION` | 20 | Munición |
| `SAVE_AMMUNITION` | 21 | Ahorrar munición |
| `SPEND_AMMUNITION` | 22 | Gastar munición |
| `DAMAGE_NPCS` | 29 | Daño a NPCs |
| `DAMAGE_PLAYERS` | 30 | Daño a jugadores |
| `RESTRICT_AMMUNITION_KILL_NPC` | 56 | Restricción munición vs NPC |
| `RESTRICT_AMMUNITION_KILL_PLAYER` | 57 | Restricción munición vs jugador |
| `VISIT_QUEST_GIVER` | 59 | Visitar NPC de misión |
| `IN_CLAN` | 62 | Estar en clan |
| `FINISH_STARTER_GATE` | 64 | Terminar starter gate |
| `REFINE_ORE` | 65 | **Refinar mineral** |
| `USE_ORE_UPDATE` | 68 | Usar mejora de mineral |
| `VISIT_JUMP_GATE_TO_MAP_TYPE` | 69 | Visitar puerta a tipo mapa |
| `ACTIVATE_MAP_ASSET_TYPE` | 70 | Activar asset de mapa |
| `UPDATE_SKYLAB_TO_LEVEL` | 72 | **Subir Skylab a nivel** |
| `FINISH_GALAXY_GATE` | 75 | **Terminar Galaxy Gate** |
| `GAIN_INFLUENCE` | 76 | Ganar influencia |

---

## 7. Tipos de efecto técnico de drones (`§_-W2f§`)

| Constante | Valor | Descripción |
|-----------|-------|-------------|
| `ENERGY_LEECH_ARRAY` | 0 | Energy leech |
| `ENERGY_CHAIN_IMPULSE` | 1 | Chain impulse |
| `ROCKET_PROBABILITY_MAXIMIZER` | 2 | Precision targeter |
| `SHIELD_BACKUP` | 3 | Backup shields |
| `SPEED_LEECH` | 4 | Speed leech |
| `CLINGING_IMPULSE_DRONE` | 6 | Clinging impulse |

---

## 8. Eventos de mapa estacionales (`§_-r2Z§`, EventNotifications)

| Constante | Valor |
|-----------|-------|
| `FROSTED_GATES` | 0 |
| `CHRISTMAS_TREES` | 1 |
| `DEMOLISHED_HQ` | 8 |

Notificaciones: `ACTIVATE_EVENT`, `ADD_EVENT`, `EXCHANGE_MAP`, `MOOD_ASSETS`, `APRILS_FOOLS`, `EVENT_DOMINATION_FACTION`.

---

## 9. Tabla de loot IDs y datos de ítems (`§_-p1j§`)

**Archivo:** `net/bigpoint/darkorbit/§_-p1j§.as` — Dictionary con entradas para **cada** ítem del juego.

### 8.1 Munición láser
`ammunition_laser_lcb-10`, `mcb-25`, `mcb-50`, `sab-50`, `ucb-100`, `cbo-100`, `rb-214`, `pib-100`

### 8.2 Cohetes
`ammunition_rocket_r-310`, `plt-2021`, `plt-2026`, `plt-3030`; especiales: `dcr-250`, `wiz-x`, `emp-01`, `pld-8`, `r-ic3`

### 8.3 Minas
`ammunition_mine_acm-01`, `empm-01`, `sabm-01`, `ddm-01`, `slm-01`, `smb-01`

### 8.4 Lanzacohetes
`ammunition_rocketlauncher_hstrm-01`, `ubr-100`, `eco-10`, `sar-01`, `sar-02`, `cbr`

### 8.5 Láseres de equipo
`equipment_weapon_laser_lf-2`, `lf-3`, `lf-4`

### 8.6 Generadores
`equipment_generator_shield_sg3n-a02`, `b01`, `b02`; `equipment_generator_speed_g3n-3310`, `6900`, `7900`

### 8.7 CPUs extra
`equipment_extra_cpu_ajp-01`, `cl04k-m`, `cl04k-xl`, `cl04k-xs`, `dr-01`, `dr-02`, `fb-x`, `jp-01`, `jp-02`, `min-t01`, `min-t02`, `nc-rrb`, `nc-rrb-x`, `nc-awr`, `nc-awl`, `nc-agb`, `nc-awb`, `rb-x`, `rd-x`, `rllb-x`, `sle-01`…`sle-04`, `alb-x`, `arol-x`, `aim-01`, `aim-02`, `anti-z1`, `anti-z1-xl`, `hmd-07`, reparadores `rep-1`…`rep-s`

### 8.8 Naves y diseños
`ship_goliath`, `ship_vengeance`, `ship_aegis`, `ship_citadel`, `ship_spearhead`, `ship_liberator` + diseños `_design_*` (enforcer, bastion, solace, venom, kick, referee, saturn, centaur, razer, lightning, revenge, avenger, elite, etc.)

### 8.9 Drones y diseños
`drone_apis`, `drone_flax`, `drone_zeus`, `drone_designs_havoc`, `drone_designs_hercules`

### 8.10 Recursos y moneda
`resource_ore_*`, `resource_key_*`, `resource_blueprint_*`, `currency_credits`, `currency_uridium`, `currency_experience`, `currency_honour`

### 8.11 Módulos de estación de batalla
`module_defm-1`, `dmgm-1`, `honm-1`, `hulm-1`, `ltm-hr`, `ltm-lr`, `ltm-mr`, `ram-la`, `ram-ma`, `repm-1`, `xpm-1`

### 8.12 Extensores de slots (CPUs)

`ConnectionProxy` en el SWF de inventario define:

| CPU | Slots extras |
|-----|-------------|
| `sle-01` | `default + 2` |
| `sle-02` | `default + 4` |
| `sle-03` | `default + 6` |
| `sle-04` | `default + 10` |

---

## 10. Notificaciones MVC del GUI (`GuiNotifications`)

**Archivo:** `net/bigpoint/darkorbit/mvc/gui/GuiNotifications.as`

Estas son las notificaciones PureMVC que el cliente usa internamente para abrir/cerrar/gestionar ventanas:

| Constante | Valor (prefijo `"GuiNotification"` +) | Uso |
|-----------|---------------------------------------|-----|
| `INIT` | `Init` | Inicialización del GUI |
| `ERROR` | `Error` | Error genérico |
| `CREATE_WINDOW` | `createWindow` | Crear ventana por tipo |
| `TOGGLE_WINDOW_BY_ID` | `toggleWindowById` | Abrir/cerrar ventana |
| `CLOSE_WINDOW_BY_ID` | `closeWindowByID` | Cerrar ventana |
| `MINIMIZE_WINDOW_BY_ID` | `minimizeWindowById` | Minimizar ventana |
| `DELETE_WINDOW_BY_ID` | `deleteWindowByID` | Eliminar ventana |
| `CLOSE_WINDOW_BY_TYPE` | `CloseWindowByID` | Cerrar por tipo |
| `TOGGLE_ALL_WINDOWS` | `manageShowHideAllWindows` | Mostrar/ocultar todas |
| `OPEN_QUEST_WINDOW` | `OpenQuestWindow` | Abrir misiones |
| `CLOSE_QUEST_WINDOW` | `CloseQuestWindow` | Cerrar misiones |
| `OPEN_BATTLE_STATION_WINDOW` | `OpenBattleStationBuildWindow` | Estación de batalla |
| `OPEN_COMPANY_HIERARCHY_WINDOW` | `OpenCompanyHierarchyWindow` | Jerarquía de compañía |
| `CREATE_POP_UP` | `CreatePopUp` | Crear popup |
| `WRITE_TO_LOG` | `writeToLog` | Escribir al log |
| `WRITE_NOTIFICATION` | `writeNotification` | Notificación |
| `SHOW_CONNECTION_WINDOW` | `showConnectionWindow` | Conexión |
| `SAVE_BARS_SETTINGS` | `SaveWindowsSettings` | Guardar configuración barras |
| `UPDATE_WINDOWS_POSITIONS` | `updateWindowsPositions` | Actualizar posiciones |
| `SEND_WINDOW_UPDATE_REQUEST` | `saveWindowPositionAndSize` | Guardar posición/tamaño |
| `MANAGE_MENUS_CONFIG_MODE` | `manageMenusConfigMode` | Modo configuración menús |
| `CREATE_VIDEO_WINDOW` | `createVideoWindow` | Crear ventana de vídeo |
| `CLOSE_KILLSCREEN` | `CLOSE_KILLSCREEN` | Cerrar pantalla de muerte |
| `UISYSTEM_SETUP` | `UISYSTEM_SETUP` | Configuración del UI system |
| `PARSE_WINDOWS_XML_DATA` | `parseWindowsXmlData` | Parsear XML de ventanas |
| `CLOSE_ALL_FLEX_WINDOWS` | `CloseAllFlexWindows` | Cerrar todas las Flex |

---

## 11. Módulos de estación de batalla (`BattleStationModule`)

| Tipo | Constante | Trait set |
|------|-----------|-----------|
| Honor booster | `HONOR_BOOSTER = 9` | `module_hon` |
| Damage booster | `DAMAGE_BOOSTER = 10` | `module_dama` |
| Experience booster | `EXPERIENCE_BOOSTER = 11` | `module_xp` |
| + láseres, cohetes, reparación, etc. | — | — |

---

## 12. Puentes JavaScript (`ExternalInterface`)

Funciones JS esperadas por el SWF en la página host:

**Archivo:** `net/bigpoint/darkorbit/net/§_-T4Z§.as`

### Funciones llamadas al JS (ExternalInterface.call)

| Constante | String | Uso |
|-----------|--------|-----|
| `§_-j38§` | `"InternalMapRevolution.referToURL"` | Navegar a URL interna/externa |
| `§_-K1S§` | `"showHangar"` | Abrir hangar web |
| `§_-U2k§` | `"openPayment"` | Pago |
| `§_-o1j§` | `"internalDock"` | Dock web |
| `§_-O1C§` | `"internalDockEquipment"` | Dock equipo web |
| `§_-a1S§` | `"referToExternalURLInNewWindow"` | URL externa nueva ventana |
| `§_-k2M§` | `"openPaymentFromExternal"` | Pago externo |
| `§_-r2p§` | `"InternalMapRevolution.closeChildWindow"` | Cerrar ventana hija |
| `§_-zX§` | `"bpCloseWindow"` | Cerrar ventana BP |
| `§_-D11§` | `"document.location.reload"` | Recargar página |
| `§_-b4c§` | `"do_redirect"` | Redirigir |
| `§_-l2b§` | `"dataLayer.push"` | Analytics (GTM) |
| `§_-I1l§` | `"InternalMapRevolution.detectPepper"` | Detectar PepperFlash |
| `§_-23y§` | `"clientResolutionChanged"` | Cambio de resolución |
| `ACHIEVEMENT` | `"ACHIEVEMENT"` | Logro desbloqueado |
| `SPECIALOFFER` | `"SPECIALOFFER"` | Oferta especial |

### Eventos notificados al JS

| Constante | String | Cuándo |
|-----------|--------|--------|
| `§_-t3t§` | `"clientError"` | Error del cliente |
| `§_-M4I§` | `"clientEvent"` | Evento genérico |
| `§_-L3J§` | `"showPetFuel"` | Combustible de pet |
| `§_-4O§` | `"onTechExpired"` | Tech expirada |
| `§_-JF§` | `"tradePossible"` | Comercio posible |
| `§_-w2J§` | `"tradeNotPossible"` | Comercio no posible |
| `§_-P1l§` | `"questCompleteFinished"` | Misión completada |
| `§_-zg§` | `"jumpPossible"` | Salto posible |
| `§_-C4U§` | `"jumpNotPossible"` | Salto no posible |
| `§_-Y4L§` | `"userLowHP"` | HP bajo |
| `§_-83h§` | `"userLevelUp"` | Subida de nivel |
| `§_-f2a§` | `"cargoFull"` | Carga llena |
| `§_-I1f§` | `"laserAmmoEmpty"` | Munición láser vacía |
| `§_-K2N§` | `"rocketsEmpty"` | Cohetes vacíos |

### Acciones internas (usadas con `referToURL`)

| Acción | Destino web |
|--------|-------------|
| `"internalDock"` | Dock principal |
| `"internalDockEquipment"` | Equipamiento |
| `"internalDockShips"` | Naves |
| `"internalDockLaser"` | Láseres |
| `"internalSkylab"` | Skylab |
| `"internalGalaxyGates"` | Galaxy gates |
| `"internalNanoTechFactory"` | Tech factory |
| `"internalAuction"` | Casa de subastas |
| `"internalMessaging"` | Mensajería |
| `"internalItemUpgradeSystem"` | Mejora de ítems |
| `"internalPilotSheet"` | Hoja de piloto |
| `"internalPayment"` | Pagos |
| `"internalNewClan"` | Crear clan |

---

## 13. Refinamiento — por qué no se ve

El cliente **tiene** toda la UI de refinamiento (`REFINEMENT_WINDOW`, `refinement_update`, `refinement_count`, `refinement_refine`), con condiciones de misión (`REFINE_ORE = 65`, `USE_ORE_UPDATE = 68`) y setting `autoRefinement`.

**Lo que falta en el servidor:**
1. El **handler** de `UIOpenRequest` no procesa `ACTION_REFINEMENT`.
2. No se envía el **comando** que crea/habilita la ventana de refinamiento al login.
3. No hay lógica de recetas/costes ni persistencia de niveles de Skylab/refinería.

Para habilitarlo: implementar el comando binario que el cliente espera + handler en el Game Server + datos de recetas.

---

## 14. Features que existen en el cliente pero **no** en el servidor actual

| Feature | Client support | Server status |
|---------|---------------|---------------|
| Refinamiento | Completo (4 ventanas) | No implementado |
| Skylab | Hints + misiones | Solo hint |
| Galaxy Gates | Ventana + misiones | Parcial |
| Comercio de mineral | Ventana `ore_trade` | No verificado |
| Eventos (spaceball, invasión, CTB, TDM) | Ventanas completas | Spaceball parcial |
| Sector Control | Ventana | No implementado |
| Jackpot Arena | Ventana + notificaciones | No implementado |
| Training Grounds | Ventana | Solo rank 21 |
| Infiltración / Word Puzzle / Curcubitor | Ventanas | No implementado |
| Battle Station | Ventana Flex + módulos | Parcial |
| Contactos | Ventana | Comentado en login |
| Ofertas dirigidas / Banner | Ventanas | No implementado |

---

## 15. Preloader

**Ruta:** `Decompile/main/preloader/` — 82 archivos AS3.

El preloader es un SWF secundario embebido que se ejecuta antes que el cliente principal. Está **muy ofuscado** y contiene:

- **Clase principal:** `net.bigpoint.darkorbit.preloader.§-_-_--__§` — extiende `Sprite`.
- **Build number:** `"3311"` (constante embebida en el contexto derecho-clic).
- **FlashVars procesados:** `gameclientPath`, `cdn`, `host`, `loadingClaim`.
- **Flujo:**
  1. Se agrega al stage, configura `StageAlign.TOP_LEFT`, `StageScaleMode.NO_SCALE`.
  2. `Security.allowDomain("*")` — permite comunicación cross-domain.
  3. Construye la URL base: `"http://" + host + "/"` (o usa `cdn` si existe).
  4. Detecta si es versión "acp" (admin panel) y ajusta el build string con `" [cli]"`.
  5. Crea el handler global de errores (`GlobalErrorHandlerBpEventStream`).
  6. Muestra pantalla de carga con el `loadingClaim` (texto publicitario).
  7. Carga el SWF principal con callbacks de progreso y finalización.
  8. Al completar: remueve la pantalla de carga y llama `startGame()` en el módulo principal.
- **Analytics:** Emite `"flash_preloader_construct"` y `"flash_preloader_loadingScreenReady"` al event stream.
- **Librerías incluidas:** GreenSock (tweens), `FileCollection` (carga de assets por lotes con finishers para SWF/Image/JSON/XML/String), módulos Flex (ResourceManager, ModuleManager), librería de señales (Slot/Signal pattern similar a as3-signals).
- **Sin constantes de gameplay** — toda la lógica de juego está en el SWF `main`.

---

## 16. Archivos de referencia principal

| Tema | Archivo |
|------|---------|
| Ventanas UI | `net/bigpoint/darkorbit/mvc/gui/GuiConstants.as` |
| Settings cliente (2D/3D, gameplay) | `net/bigpoint/darkorbit/settings/Settings.as` |
| Settings 3D | `§_-J0§/Settings3D.as` |
| Hints / tutoriales | `§_-G4V§/§_-F2I§.as` |
| Objetivos de misión | `§_-G4V§/§_-G4F§.as` |
| Tabla de loot IDs | `net/bigpoint/darkorbit/§_-p1j§.as` |
| Efectos tech drones | `§_-G4V§/§_-W2f§.as` |
| Módulos estación | `net/bigpoint/darkorbit/mvc/battleStation/BattleStationModule.as` |
| Eventos mapa | `§_-G4V§/§_-r2Z§.as` |
| Menú funciones | `net/bigpoint/darkorbit/mvc/gui/featuresMenu/FeaturesMenuConstants.as` |
| Puentes JS | `net/bigpoint/darkorbit/net/§_-T4Z§.as` |
| Acciones web | `net/bigpoint/darkorbit/mvc/common/web/WebConstants.as` |
| Notificaciones MVC | `net/bigpoint/darkorbit/mvc/gui/GuiNotifications.as` |
| FlashVars parser | `net/bigpoint/darkorbit/settings/FlashVarsParser.as` |
| Socket/codec | `§_-n1i§/§_-v2v§.as`, `§_-n1i§/§_-e3I§.as` |

---

*Generado a partir del árbol decompilado `Decompile/main/` (abril 2026).*
