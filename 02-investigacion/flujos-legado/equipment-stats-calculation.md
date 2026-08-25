# Cálculo de estadísticas desde el equipamiento (Game Server)

Este documento describe **cómo se obtienen daño, escudo, vida (HP), velocidad** y bonos relacionados cuando el jugador tiene ítems equipados. La lógica vive en el **servidor de juego** (`MexOrbit.GameServer`), no en el CMS ni en la API de inventario: esas capas solo persisten `player_equipment`; el **Game Server** relee la fila en MySQL y recalcula stats en memoria.

## Dónde está el código

| Qué | Archivo | Notas |
|-----|---------|--------|
| **Base numérica por equipamiento** (por config 1 y 2) | `MexOrbit.GameServer/Managers/QueryManager.cs` → `SetEquipment` | Lee `player_equipment`, suma según rangos de **índice L** de ítems. |
| **Estructura en memoria** | `Game/Objects/Players/Managers/SettingsManager.cs` → `ConfigsBase`, `EquipmentBase` | `Config1Hitpoints`, `Config1Damage`, `Config1Shield`, `Config1Speed` (y config 2). |
| **Modificadores en combate / mapa** | `Game/Objects/Player.cs` | Formaciones, boosters, skills, nave (`Ship`), almacenamiento temporal (`Storage`), etc. |

**Importante:** En `SetEquipment` **no** se procesan slots de **heavy guns (hellstorm)** ni **extras**; solo **láseres de nave**, **generadores de nave** y **drones** (ítems y diseños en el JSON de drones).

---

## 1. Flujo general

1. Al cargar el jugador (`GetPlayer`), se llama a `SetEquipment(player)`.
2. Se hace `SELECT * FROM player_equipment WHERE userId = ...`.
3. Para **configuración 1** y **configuración 2** (`i = 1..2`), se leen arrays JSON:
   - `config{i}_lasers`
   - `config{i}_generators`
   - `config{i}_drones` (estructura con `items` y `designs` por drone)
4. Se acumulan valores en arrays locales `hitpoints`, `damage`, `shield`, `speed` (dos entradas, una por config).
5. Al final se construye `ConfigsBase(hitpoints[0], damage[0], shield[0], speed[0], hitpoints[1], damage[1], shield[1], speed[1])` y se asigna a `player.Equipment`.

Los **índices numéricos** (`itemId`, `design`) son los **valores L** del modelo legacy (mismo criterio que en el inventario Flash / tablas de ítems).

---

## 2. Constantes de “tipo de ítem” (nave)

En `SetEquipment` están definidos (valores fijos en código):

| Constante | Valor | Uso |
|-----------|-------|-----|
| `lf3Damage` | 150 | Daño por láser LF-3 |
| `lf4Damage` | 200 | Daño por láser LF-4 |
| `bo2Shield` | 15_000 | Escudo por generador tipo BO2 |
| `g3nSpeed` | 10 | Velocidad por generador tipo G3N |

### 2.1 Láseres de la nave (`config{i}_lasers`)

Por cada ID en el array:

- **`0 ≤ itemId < 40`** → suma **`lf3Damage`** (150) al daño de esa config.
- **`itemId ≥ 140`** → suma **`lf4Damage`** (200).
- Otros rangos → no suman en este bloque.

### 2.2 Generadores de la nave (`config{i}_generators`)

Por cada ID:

- **`40 ≤ itemId < 100`** → suma **`bo2Shield`** (15_000) al escudo de esa config.
- **`100 ≤ itemId < 120`** → suma **`g3nSpeed`** (10) a la velocidad de esa config.

---

## 3. Vida (hitpoints) base por config

Antes de sumar drones u otros bonos por config:

- Se inicializa:  
  **`hitpoints[i] = player.Ship.BaseHitpoints + 60000`** (mismo para config 1 y 2 en el código actual).

Es decir: **HP base de la nave en BD** más un **offset fijo (+60000)** legado. El equipamiento de láseres/generadores **no** suma HP en el bucle principal; el HP extra por equipamiento entra sobre todo por la regla de **drones Hercules** (ver §4.3).

---

## 4. Drones (`config{i}_drones`)

Cada drone es un objeto con:

- **`designs`**: IDs de diseños (para contar Havoc / Hercules). **Regla de juego:** cada dron tiene **como mucho un** diseño equipado (no se apilan varios Havoc/Hercules en el mismo slot).
- **`items`**: IDs de ítems equipados en el drone.

### 4.1 Daño y escudo por ítem en el drone

Durante el bucle por drones, conviene llevar acumulados (por config):

- **`droneLaserDamage`**: daño aportado **solo** por láseres en slots de dron (excluye láseres de la nave).
- **`droneShieldTotal`**: escudo aportado **solo** por generadores en slots de dron (tras bonos por dron Hercules del §4.1; el bono de set del §4.3 va después).

Para cada drone se define:

- **`droneShield = bo2Shield + 2000`** → **17_000** (base de escudo por slot de generador en drone, antes de bonos Hercules).

Por cada ítem en `drone["items"]`:

- **`0 ≤ item < 40`** → daño **`lf3Damage + 15`** (165); se suma a **`damage[i]`** y a **`droneLaserDamage`**.
- **`item ≥ 140`** → daño **`lf4Damage + 20`** (220); igual.
- **`40 ≤ item < 100`** → escudo base **`droneShield`**. Si ese dron tiene diseño **Hercules**, el aporte de ese generador se incrementa en **`+1%`** (solo sobre esa acumulación de escudo en ese slot). El resultado se suma a **`shield[i]`** y a **`droneShieldTotal`**.

### 4.2 Contadores de diseños (Havoc / Hercules)

Por cada `design` en `drone["designs"]` (un diseño relevante por dron):

- **`120 ≤ design < 130`** → incrementa **`havocCount`** (dron con Havoc).
- **`130 ≤ design < 140`** → incrementa **`herculesCount`** (dron con Hercules).

### 4.3 Bonos de set después de procesar todos los drones (por config)

**Havoc (todo o nada)**  
- Si **`havocCount == drones.Count`** y hay al menos un dron (todos llevan Havoc):  
  **`damage[i] += 10%`** de **`droneLaserDamage`** — es decir, **solo** sobre el daño de láseres equipados en drones, **no** sobre láseres de la nave.  
- Si **cualquier** dron **no** tiene Havoc: **no** se aplica bono Havoc (**0%**).

**Hercules (por dron ya aplicado en §4.1; set completo aquí)**  
- Si **`herculesCount == drones.Count`** (todos los drones del array llevan Hercules):  
  - **`hitpoints[i] += 20%`** del HP acumulado hasta ese momento (base nave + offset, etc.).  
  - **`shield[i] += 5%`** de **`droneShieldTotal`** — bono **adicional** a todo el escudo aportado por generadores en drones (después del **+1%** por dron Hercules del §4.1).

Si no están **todos** los drones con Hercules, solo aplican los **+1%** por escudo en los drones que sí llevan Hercules; **no** hay **+20%** HP ni **+5%** global de escudos de drones.

**Nota:** Havoc y Hercules en set completo son **mutuamente excluyentes** en la práctica (un dron no puede llevar ambos diseños). Una mezcla (p. ej. parte Havoc, parte Hercules) implica **0%** bono Havoc y **no** activa el set Hercules.

---

## 5. Ajuste final de velocidad (ambas configs)

Tras calcular `speed[0]` y `speed[1]` (base de nave + G3N de generadores, etc.):

- **`speed[0] += 20%`** de `speed[0]`
- **`speed[1] += 20%`** de `speed[1]`

Es un **multiplicador fijo +20%** sobre la velocidad ya sumada para cada configuración.

---

## 6. De `ConfigsBase` a lo que ve el combate: `Player.cs`

Los valores guardados en `Equipment.Configs` son la **base por configuración activa** (`CurrentConfig` 1 o 2). Las propiedades públicas aplican **otra capa** de reglas:

### 6.1 Daño (`Damage`)

- Parte de **`Config1Damage` / `Config2Damage`**.
- **`+60%`** (comentario en código: *seprom*).
- Porcentaje del **booster** de daño.
- Ajustes por **formación de drones** seleccionada.
- **`Ship.GetLaserDamageBoost(...)`** (bonos por modelo de nave y facción).
- **`Storage.DamageBoost`** (efectos temporales / almacenamiento en sesión).

Ese valor es la **base** del láser; **en cada disparo** el servidor aplica variación aleatoria y posible fallo — ver [combat-damage-randomization.md](./combat-damage-randomization.md).

### 6.2 Escudo máximo (`MaxShieldPoints`)

- Parte de **`Config1Shield` / `Config2Shield`**.
- **`+40%`** fijo.
- Boosters de escudo, skill **Shield Engineering**, formaciones.
- **`Ship.GetShieldPointsBoost(...)`** (bonos por modelo de nave).

### 6.3 Vida máxima (`MaxHitPoints`)

- Parte de **`Config1Hitpoints` / `Config2Hitpoints`**.
- Boosters de MAXHP, formaciones.
- **`Ship.GetHitPointsBoost(...)`** (ej. ciertos modelos +10% / +20%).

### 6.4 Velocidad (`Speed`)

- Parte de **`Config1Speed` / `Config2Speed`**.
- Formaciones, penalizaciones DCR/SLM/R-IC3, bonus Lightning, **`Storage.SpeedBoost`**, etc.

### 6.5 Absorción y penetración de escudo

- **`ShieldAbsorption`** y **`ShieldPenetration`** dependen sobre todo de **formación**, no de los números de `SetEquipment` directamente.

---

## 7. Resumen mental

1. **MySQL** (`player_equipment`) → JSON de configs con **IDs L** en láseres, generadores y drones.
2. **`SetEquipment`** traduce esos IDs a números con **tablas de rangos fijas** + **constantes** (150/200/15000/10, etc.) y bonos **Havoc/Hercules** y **+20% velocidad**.
3. **`Player`** aplica **multiplicadores y sumas** de gameplay (seprom 60%, escudo +40%, formación, nave, boosters, skills).

Para cambiar el balance **solo del equipamiento “en bruto”**, el sitio principal es **`QueryManager.SetEquipment`**. Para el balance **en combate**, hay que revisar también **`Player.cs`** y **`Ship.cs`**.

---

## 8. Referencias cruzadas

- Sincronización BD ↔ Game Server: [equipment-and-game-server-sync.md](./equipment-and-game-server-sync.md).
- Variación por disparo (RNG, “miss”, munición): [combat-damage-randomization.md](./combat-damage-randomization.md).
