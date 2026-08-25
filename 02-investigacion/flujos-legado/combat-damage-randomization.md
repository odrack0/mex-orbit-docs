# Variación de daño por disparo (no lineal / no constante)

El daño mostrado en stats (`Player.Damage`, equipamiento, etc.) es una **base**. En cada disparo de **láser** (y en otros usos de la misma función) el servidor aplica **dos mecanismos** encadenados:

1. **Variación por tirada discreta** (porcentaje fijo sobre la base de ese disparo).
2. **Posible anulación del daño** (“miss”) según una probabilidad configurable.

Todo está centralizado en **`AttackManager.RandomizeDamage`** (`MexOrbit.GameServer/Game/Objects/Players/Managers/AttackManager.cs`).

---

## 1. Función `RandomizeDamage(int baseDmg, double missProbability = 1.00)`

### 1.1 Paso A — multiplicador aleatorio (8 casos equiprobables)

Se elige un entero **`Randoms.random.Next(0, 8)`** (valores **0 a 7**, cada uno con probabilidad **1/8**).

| Caso | Multiplicador aplicado a `baseDmg` |
|------|-------------------------------------|
| 0 | × **1.10** |
| 1 | × **0.98** |
| 2 | × **1.02** |
| 3 | × **1.05** |
| 4 | × **0.92** |
| 5 | × **0.99** |
| 6 | × **0.97** |
| 7 (default) | × **1.01** |

El resultado se convierte a **`int`** (truncamiento implícito al cast).

**Interpretación:** el daño **no** sigue una curva continua (no es una Gaussiana); es una **mezcla de 8 valores discretos** alrededor de la base. En la práctica oscila aproximadamente entre **~92%** y **~110%** de `baseDmg` en un solo disparo, según la tabla.

### 1.2 Paso B — posible fallo (daño a cero)

Después del multiplicador:

```csharp
if (missProbability > Randoms.random.NextDouble())
    value = 0;
```

`NextDouble()` devuelve un valor en **[0.0, 1.0)**.

- Si **`missProbability`** es **0.1**: el daño pasa a **0** cuando `0.1 > random`, es decir cuando `random ∈ [0, 0.1)` → aproximadamente **10%** de disparos sin daño.
- Si **`missProbability`** es **0.5**: aproximadamente **50%** de disparos sin daño.
- Si **`missProbability`** es **1.0** (valor por defecto del parámetro si no se pasa el segundo argumento): **`1.0 > random`** es casi siempre cierto → el daño queda en **0** casi siempre. Los llamadores que quieren daño deben pasar explícitamente una probabilidad menor que 1.

---

## 2. Láser (`LaserAttack`)

Tras comprobar objetivo y cooldown:

```csharp
var damage = RandomizeDamage(
    (GetDamageMultiplier() * Player.Damage),
    (Player.Storage.underPLD8 ? 0.5 : 0.1));
```

1. **`baseDmg`** = `GetDamageMultiplier() * Player.Damage`  
   - El multiplicador depende de la **munición de láser** seleccionada (LCB=1, MCB-25=2, …, RSB=5, etc.; ver `GetDamageMultiplier()` en el mismo archivo).

2. **`missProbability`** = **0.5** si el jugador que dispara tiene `Storage.underPLD8`, si no **0.1** (~10% de disparos “fallidos” a daño 0).

3. Después de `RandomizeDamage`, se aplican reducciones por **Spectrum** (atacante / defensor) antes de `Damage(...)`.

**Orden conceptual:** primero stats (`Player.Damage` ya incluye seprom, formación, nave, etc.) → × munición → **variación + fallo** en `RandomizeDamage` → modificadores Spectrum → absorción escudo/HP en `Damage`.

---

## 3. Cohetes y otros usos de `RandomizeDamage`

| Contexto | `baseDmg` | Segundo parámetro (`missProbability`) |
|----------|-----------|--------------------------------------|
| Cohetes normales (ruta que llama `RandomizeDamage(Player.RocketDamage, ...)`) | `Player.RocketDamage` (skills/formación, etc.) | `Player.RocketMissProbability` (base **0.1**, reducible con skill *Heat-seeking Missiles*, **0** con *Precision Targeter*) |
| Lanzacohetes (suma por cada carga) | Daño fijo por tipo de launcher | `Player.RocketMissProbability` |
| SAB / absorbación (fragmento visto en el mismo archivo) | `2 * Player.Damage` | `underPLD8 ? 0.5 : 0.1` (misma lógica que láser) |
| ECI / `ChainImpulse` | `ChainImpulse.DAMAGE` | Por defecto **1.0** si solo se pasa un argumento → ver §1.2 |

Los **NPC** también usan `RandomizeDamage` con su propio daño base y probabilidad (p. ej. `Npc.cs` con PLD-8).

---

## 4. Relación con el documento de stats de equipamiento

- **[equipment-stats-calculation.md](./equipment-stats-calculation.md)** explica cómo se obtiene **`Player.Damage`** (y escudo, HP, velocidad) a partir del equipamiento y de `Player.cs`.
- **Este documento** cubre lo que ocurre **en cada tick de disparo**: la base ya calculada se **escala** con la tabla de 8 multiplicadores y puede **anularse** según `missProbability`.

No es un modelo “físico” lineal: es **determinista por tirada** con RNG discreto + posible miss.

---

## 5. Referencia de código

- `AttackManager.RandomizeDamage` — multiplicadores y comprobación `missProbability`.
- `AttackManager.LaserAttack` — `GetDamageMultiplier()`, `Player.Damage`, PLD-8 / 0.1.
- `AttackManager.RocketAttack` / `LaunchRocketLauncher` — cohetes y lanzacohetes.
- `Player.RocketMissProbability` — base 0.1 y reducciones.
