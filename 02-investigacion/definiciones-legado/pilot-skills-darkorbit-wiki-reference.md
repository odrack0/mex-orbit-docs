# Pilot skills (Dark Orbit) — referencia extraída del wiki

Este documento es la **fuente de reglas de diseño** para el árbol de habilidades de piloto en MexOrbit (opción B: paridad amplia con Dark Orbit). El contenido se obtuvo de la página pública [PILOT SKILLS – DarkOrbitWiki](https://darkorbitwiki.com/pilot-skills/) (consulta en 2026). Los nombres y cifras siguen al wiki; donde el texto original tiene errores obvios, se indica en **Notas y erratas del wiki**.

**Copyright / atribución:** el juego Dark Orbit y sus assets son propiedad de Bigpoint GmbH. Esta recopilación es documentación interna de reglas; no sustituye la documentación oficial del juego.

---

## Convenciones

| Término | Significado |
|--------|-------------|
| **PP** | Pilot points (puntos de piloto). Cada nivel de habilidad suele costar **1 PP** además de créditos/seprom. |
| **Log disks** | Discos de registro necesarios para **desbloquear** un nuevo punto de piloto (ver tabla más abajo). |
| **Seprom** | Recurso de juego requerido en niveles avanzados junto a créditos. |

**Formato numérico:** el wiki usa separador de miles estilo europeo (p. ej. `10.000` = diez mil créditos). En tablas de implementación se puede mapear a enteros `10000`.

---

## 1. Orden sugerido de inversión (“WHERE INVEST THE POINTS”)

Ruta de referencia publicada en el wiki (una posible build; al final hay **rama alternativa**):

| Paso | Habilidad | Objetivo de niveles |
|------|-----------|----------------------|
| I | Ship Hull I | 2/2 |
| II | Engineering | 1/5 |
| III | Shield Engineering | 5/5 |
| IV | Evasive Maneuvers I | 2/2 |
| V | Ship Hull II | 3/3 |
| VI | Shield Mechanics | 5/5 |
| VII | Evasive Maneuvers II | 3/3 |
| VIII | Electro-Optics | 5/5 |
| IX | Rocket Fusion | 5/5 |
| X | Bounty Hunter I | 2/2 |
| XI | Bounty Hunter II | 3/3 |
| XII | Cruelty I | 2/2 |
| XIII | Alien Hunter | 5/5 |
| XIV | Heat-Seeking Misilles | 5/5 |
| XV | **Alternativa:** Cruelty II **2/3** *o* Logistics **2/5** *o* Engineering **3/5** |

---

## 2. Catálogo de habilidades (niveles, efectos, costes, prerequisitos)

Cada subsección incluye: niveles, efecto por nivel, coste (1 PP por nivel salvo indicación), y requisitos para **poder invertir** en esa habilidad.

### Ship Hull I

| Nivel | Efecto |
|-------|--------|
| 1 | Aumenta el HP máximo en **5.000** |
| 2 | Aumenta el HP máximo en **10.000** |

**Coste por nivel:** 1 PP, créditos 10.000 / 20.000.  
**Prerequisitos:** ninguno (–).

---

### Ship Hull II

| Nivel | Efecto |
|-------|--------|
| 1 | Aumenta el HP máximo en **15.000** |
| 2 | Aumenta el HP máximo en **25.000** |
| 3 | Aumenta el HP máximo en **50.000** |

**Coste por nivel:** 1 PP + créditos (100.000 / 200.000 / 300.000) + seprom (100 / 200 / 300).  
**Prerequisitos:**

- 2 PP invertidos en **Ship Hull I**
- 2 PP en **alguna** de: Bounty Hunter I, Luck I, Evasive Maneuvers I

**Efecto visual (wiki):** efecto visual (no hace falta maxear); muestra el aumento de HP en la nave.

---

### Engineering

| Nivel | Efecto |
|-------|--------|
| 1 | Los repair bots reparan **5%** más HP por segundo |
| 2 | **10%** más HP por segundo |
| 3 | **15%** más HP por segundo |
| 4 | **20%** más HP por segundo |
| 5 | **30%** más HP por segundo |

**Coste por nivel:** 1 PP + créditos 10.000 / 20.000 / 30.000 / 40.000 / 50.000.  
**Prerequisitos:** 2 PP en **alguna** de: Detonation I, Tactics, Ship Hull I.

**Efecto visual (wiki):** al maxear, el robot de reparación aparece azul.

---

### Shield Engineering

| Nivel | Efecto |
|-------|--------|
| 1 | +**4%** fuerza de escudo |
| 2 | +**8%** |
| 3 | +**12%** |
| 4 | +**18%** |
| 5 | +**25%** |

**Coste por nivel:** 1 PP + créditos 10.000 … 50.000 (misma progresión que otras skills de 5 niveles “baratas”).  
**Prerequisitos:** 2 PP en **alguna** de: Explosives, Logistics, Engineering.

**Efecto visual (wiki):** al maxear, efecto de escudo blanco.

---

### Shield Mechanics

| Nivel | Efecto |
|-------|--------|
| 1 | El escudo aguanta **2%** más daño |
| 2 | **4%** |
| 3 | **6%** |
| 4 | **8%** |
| 5 | **12%** |

**Coste por nivel:** 1 PP + créditos (1.000.000 … 5.000.000) + seprom (1.000 … 5.000).  
**Prerequisitos:** 3 PP en **Ship Hull II**.

**Efecto visual (wiki):** al maxear, efecto de escudo blanco (texto duplicado en el wiki).

---

### Evasive Maneuvers I

| Nivel | Efecto |
|-------|--------|
| 1 | Reduce la probabilidad de que te acierten **2%** |
| 2 | **4%** |

**Coste por nivel:** 1 PP + créditos (100.000 / 200.000) + seprom (100 / 200).  
**Prerequisitos:** 2 PP en **alguna** de: Heat-Seeking Misilles, Shield Engineering.

---

### Evasive Maneuvers II

| Nivel | Efecto |
|-------|--------|
| 1 | Reduce la probabilidad de que te acierten **6%** |
| 2 | **8%** |
| 3 | **12%** |

**Coste por nivel:** 1 PP + créditos (1.000.000 / 2.000.000 / 3.000.000) + seprom (1.000 / 2.000 / 3.000).  
**Prerequisitos:**

- 3 PP en **Shield Mechanics**
- 2 PP en **Evasive Maneuvers I**

**Efecto visual (wiki):** los disparos que fallan por esta habilidad se muestran en azul en lugar de morado.

---

### Tactics

| Nivel | Efecto |
|-------|--------|
| 1 | +**2%** EXP por alien eliminado |
| 2 | +**4%** |
| 3 | +**6%** |
| 4 | +**8%** |
| 5 | +**12%** |

**Coste por nivel:** 1 PP + créditos 10.000 … 50.000.  
**Prerequisitos:** ninguno (–).

---

### Logistics

| Nivel | Efecto |
|-------|--------|
| 1 | +**4%** capacidad de bodega (cargo) |
| 2 | +**8%** |
| 3 | +**12%** |
| 4 | +**18%** |
| 5 | +**25%** |

**Coste por nivel:** 1 PP + créditos 10.000 … 50.000.  
**Prerequisitos:** 1 PP en **alguna** de: Detonation I, Tactics, Ship Hull I.

---

### Luck I

| Nivel | Efecto |
|-------|--------|
| 1 | +**2%** uridium en cajas bonus |
| 2 | +**4%** |

**Coste por nivel:** 1 PP + créditos (100.000 / 200.000) + seprom (100 / 200).  
**Prerequisitos:** 2 PP en **Logistics**.

---

### Luck II

| Nivel | Efecto |
|-------|--------|
| 1 | +**6%** uridium en cajas bonus |
| 2 | +**8%** |
| 3 | +**12%** |

**Coste por nivel:** 1 PP + créditos (1.000.000 / 2.000.000 / 3.000.000) + seprom (1.000 / 2.000 / 3.000).  
**Prerequisitos:**

- 2 PP en **Luck I**
- 3 PP en **Bounty Hunter II**
- 3 PP en **Evasive Maneuvers II**

---

### Cruelty I

| Nivel | Efecto |
|-------|--------|
| 1 | +**4%** puntos de honor |
| 2 | +**8%** puntos de honor |

**Coste por nivel:** 1 PP + créditos (100.000 / 200.000) + seprom (100 / 200).  
**Prerequisitos:** 2 PP en **alguna** de: Bounty Hunter I, Luck I, Evasive Maneuvers I.

**Nota:** en el wiki la fila del nivel 2 repite “Level 1”; debe interpretarse como **nivel 2** al **8%**.

---

### Cruelty II

| Nivel | Efecto |
|-------|--------|
| 1 | +**12%** honor |
| 2 | +**18%** honor |
| 3 | +**25%** honor |

**Coste por nivel:** 1 PP + créditos (1.000.000 / 2.000.000 / 3.000.000) + seprom (1.000 / 2.000 / 3.000).  
**Prerequisitos:**

- 2 PP en **Cruelty I**
- 3 PP en **alguna** de: Electro-Optics, Tractor Beam II

---

### Greed

| Nivel | Efecto |
|-------|--------|
| 1 | +**4%** créditos por bajas de aliens |
| 2 | +**8%** |
| 3 | +**12%** |
| 4 | +**18%** |
| 5 | +**25%** |

**Coste por nivel:** 1 PP + créditos (1.000.000 … 5.000.000) + seprom (1.000 … 5.000).  
**Prerequisitos:** 3 PP en **alguna** de: Alien Hunter, Tractor Beam I.

---

### Tractor Beam I

| Nivel | Efecto |
|-------|--------|
| 1 | +**1%** botín de cajas de cargo |
| 2 | +**2%** |
| 3 | +**3%** |
| 4 | +**4%** |
| 5 | +**6%** |

**Coste por nivel:** 1 PP + créditos (100.000 … 500.000) + seprom (100 … 500).  
**Prerequisitos:** 2 PP en **alguna** de: Rocket Fusion, Cruelty I, Ship Hull II.

---

### Tractor Beam II

| Nivel | Efecto |
|-------|--------|
| 1 | +**2%** botín de cajas bonus |
| 2 | +**6%** |
| 3 | +**10%** |
| 4 | +**15%** |
| 5 | +**20%** |

**Coste por nivel:** 1 PP + créditos (1.000.000 … 5.000.000) + seprom (1.000 … 5.000).  
**Prerequisitos:** 5 PP en **alguna** de: Detonation II, Greed, Shield Mechanics.

---

### Detonation I

| Nivel | Efecto |
|-------|--------|
| 1 | Minas hacen **7%** más daño |
| 2 | **14%** más daño |

**Coste por nivel:** 1 PP + créditos 10.000 / 20.000.  
**Prerequisitos:** ninguno (–).

---

### Detonation II

| Nivel | Efecto |
|-------|--------|
| 1 | Minas hacen **21%** más daño |
| 2 | **28%** más daño |
| 3 | **50%** más daño |

**Coste por nivel:** 1 PP + créditos (1.000.000 / 2.000.000 / 3.000.000) + seprom (1.000 / 2.000 / 3.000).  
**Prerequisitos:**

- 2 PP en **Detonation I**
- 3 PP en **alguna** de: Alien Hunter, Tractor Beam I

**Efecto visual (wiki):** al maxear, efecto visual.

---

### Explosives

| Nivel | Efecto |
|-------|--------|
| 1 | +**4%** radio de explosión de minas |
| 2 | +**8%** |
| 3 | +**12%** |
| 4 | +**18%** |
| 5 | +**25%** |

**Coste por nivel:** 1 PP + créditos 10.000 … 50.000.  
**Prerequisitos:** 1 PP en **alguna** de: Detonation I, Tactics, Ship Hull I.

**Efecto visual (wiki):** al maxear, efecto visual.

---

### Heat-Seeking Misilles

**Nombre wiki:** “Misilles” (error ortográfico de *Missiles*).  

| Nivel | Efecto |
|-------|--------|
| 1 | +**1%** probabilidad de impacto de cohetes |
| 2 | +**2%** |
| 3 | +**4%** |
| 4 | +**6%** |
| 5 | +**10%** |

**Coste por nivel:** 1 PP + créditos 10.000 … 50.000.  
**Prerequisitos:** 1 PP en **alguna** de: Explosives, Logistics, Engineering.

**Efecto visual (wiki):** al maxear, efecto visual.

---

### Bounty Hunter I

| Nivel | Efecto |
|-------|--------|
| 1 | Láseres +**2%** daño en PvP |
| 2 | +**4%** daño en PvP |

**Coste por nivel:** 1 PP + créditos (100.000 / 200.000) + seprom (100 / 200).  
**Prerequisitos:** 2 PP en **alguna** de: Heat-Seeking Misilles, Shield Engineering.

---

### Bounty Hunter II

| Nivel | Efecto |
|-------|--------|
| 1 | Láseres +**6%** daño en PvP |
| 2 | +**8%** daño en PvP |
| 3 | +**12%** daño en PvP |

**Coste por nivel:** 1 PP + créditos (1.000.000 / 2.000.000 / 3.000.000) + seprom (1.000 / 2.000 / 3.000).  
**Prerequisitos:**

- 2 PP en **Bounty Hunter I**
- 3 PP en **alguna** de: Electro-Optics, Tractor Beam II

**Efecto visual (wiki):** al maxear, efecto visual.

---

### Rocket Fusion

| Nivel | Efecto |
|-------|--------|
| 1 | Cohetes +**2%** daño |
| 2 | +**4%** |
| 3 | +**6%** |
| 4 | +**8%** |
| 5 | +**15%** |

**Coste por nivel:** 1 PP + créditos (100.000 … 500.000) + seprom (100 … 500).  
**Prerequisitos:** 2 PP en **alguna** de: Bounty Hunter I, Luck I, Ship Hull II.

**Efecto visual (wiki):** al maxear, efecto visual.

---

### Alien Hunter

| Nivel | Efecto |
|-------|--------|
| 1 | Láseres +**2%** daño vs aliens |
| 2 | +**4%** |
| 3 | +**6%** |
| 4 | +**8%** |
| 5 | +**12%** |

**Coste por nivel:** 1 PP + créditos (100.000 … 500.000) + seprom (100 … 500).  
**Prerequisitos:** 2 PP en **cada una** de: Rocket Fusion **y** Cruelty I (requisito “en ambas”, no OR).

---

### Electro-Optics

| Nivel | Efecto |
|-------|--------|
| 1 | +**5%** probabilidad de acierto de láseres |
| 2 | +**10%** |
| 3 | +**15%** |
| 4 | +**20%** |
| 5 | +**25%** |

**Coste por nivel:** 1 PP + créditos (1.000.000 … 5.000.000) + seprom (1.000 … 5.000).  
**Prerequisitos:** 5 PP en **alguna** de: Detonation II, Greed, Shield Mechanics.

**Efecto visual (wiki):** disparos que fallarían sin la habilidad muestran números azules en lugar de rojos.

---

## 3. Discos de registro (log disks) por punto de piloto

Para desbloquear el **punto de piloto número N** (1…50), el wiki indica discos requeridos y coste en **uridium**.

| Punto # | Log disks | Uridium (wiki) | Uridium (valor numérico) |
|---------|-----------|----------------|--------------------------|
| 1 | 30 | 9.000 | 9000 |
| 2 | 33 | 9.900 | 9900 |
| 3 | 36 | 10.800 | 10800 |
| 4 | 40 | 12.000 | 12000 |
| 5 | 44 | 13.200 | 13200 |
| 6 | 48 | 14.400 | 14400 |
| 7 | 53 | 15.900 | 15900 |
| 8 | 58 | 17.400 | 17400 |
| 9 | 64 | 19.200 | 19200 |
| 10 | 71 | 21.300 | 21300 |
| 11 | 78 | 23.400 | 23400 |
| 12 | 86 | 25.800 | 25800 |
| 13 | 94 | 28.200 | 28200 |
| 14 | 104 | 31.200 | 31200 |
| 15 | 114 | 34.200 | 34200 |
| 16 | 125 | 37.500 | 37500 |
| 17 | 138 | 41.400 | 41400 |
| 18 | 152 | 45.600 | 45600 |
| 19 | 167 | 50.100 | 50100 |
| 20 | 183 | 54.900 | 54900 |
| 21 | 202 | 60.600 | 60600 |
| 22 | 222 | 66.600 | 66600 |
| 23 | 244 | 73.200 | 73200 |
| 24 | 269 | 80.700 | 80700 |
| 25 | 295 | 88.500 | 88500 |
| 26 | 325 | 97.500 | 97500 |
| 27 | 358 | 107.400 | 107400 |
| 28 | 393 | 117.900 | 117900 |
| 29 | 433 | 129.900 | 129900 |
| 30 | 476 | 142.800 | 142800 |
| 31 | 523 | 156.900 | 156900 |
| 32 | 576 | 172.800 | 172800 |
| 33 | 633 | 189.900 | 189900 |
| 34 | 697 | 209.100 | 209100 |
| 35 | 766 | 229.800 | 229800 |
| 36 | 843 | 252.900 | 252900 |
| 37 | 927 | 278.100 | 278100 |
| 38 | 1020 | 306.000 | 306000 |
| 39 | 1122 | 336.600 | 336600 |
| 40 | 1234 | 370.200 | 370200 |
| 41 | 1358 | 407.400 | 407400 |
| 42 | 1494 | 448.200 | 448200 |
| 43 | 1643 | 492.900 | 492900 |
| 44 | 1807 | 542.100 | 542100 |
| 45 | 1988 | 596.400 | 596400 |
| 46 | 2187 | 656.100 | 656100 |
| 47 | 2405 | 721.500 | 721500 |
| 48 | 2646 | 793.800 | 793800 |
| 49 | 2911 | 873.300 | 873300 |
| 50 | 3202 | 960.600 | 960600 |

**Totales (wiki):** **34.917** log disks y **10.475.100** uridium para desbloquear los 50 puntos (según la tabla del wiki).

---

## 4. Coste de reset del árbol de habilidades

Cada vez que reseteas, el wiki lista **coste en uridium** por número de reset acumulado:

| Reset # | Coste (uridium, wiki) | Valor numérico |
|---------|------------------------|----------------|
| 1 | 1.000 | 1000 |
| 2 | 2.000 | 2000 |
| 3 | 4.000 | 4000 |
| 4 | 8.000 | 8000 |
| 5 | 16.000 | 16000 |
| 6 | 32.000 | 32000 |
| 7 | 64.000 | 64000 |
| 8 | 128.000 | 128000 |
| 9 | 256.000 | 256000 |
| 10 | 512.000 | 512000 |
| 11 | 1.024.000 | 1024000 |
| 12 | 2.048.000 | 2048000 |
| 13 | 4.096.000 | 4096000 |
| 14 | 8.192.000 | 8192000 |
| 15 | 16.384.000 | 16384000 |
| 16 | 32.768.000 | 32768000 |
| 17 | 65.536.000 | 65536000 |
| 18 | 131.072.000 | 131072000 |
| 19 | 262.144.000 | 262144000 |
| 20 | 524.288.000 | 524288000 |
| … | … | (continúa duplicando) |

**Regla:** a partir del primer valor, cada reset multiplica el coste por **2** (serie geométrica):  
`coste(reset_n) = 1000 × 2^(n-1)` en uridium, para n ≥ 1, según la tabla del wiki.

---

## 5. Notas y erratas detectadas en el wiki

- **Cruelty I:** segundo nivel aparece como “Level 1” otra vez; corresponde al **nivel 2** al 8% honor.
- **Cruelty I prereq:** “Lucky I” → en el juego suele ser **Luck I**.
- **Heat-Seeking Misilles:** ortografía; **Missiles** en inglés.
- **Shield Engineering / Mechanics:** “strenght” → **strength**; “affect” → **effect**.
- **Luck II:** texto “3 PP Evasive Maneuvers II” sin “Required in”; en el wiki se entiende como **3 PP en Evasive Maneuvers II** (coherente con otras filas).

---

## 6. Uso en MexOrbit

- Este `.md` es la **base de datos de reglas** para implementar validación de prerequisitos, costes, límites de nivel y reset en API/CMS/GameServer.
- Los IDs internos (strings estables) y el grafo de dependencias deberían derivarse de este catálogo en un paso siguiente (p. ej. YAML/JSON generado o tablas SQL).
- Las reglas de **combate y efectos visuales** son responsabilidad del cliente/servidor de juego; el CMS puede limitarse a datos y restricciones hasta que el GameServer adopte el árbol completo.
