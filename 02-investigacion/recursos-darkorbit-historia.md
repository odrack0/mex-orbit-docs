# Historia de los recursos de DarkOrbit — y la compresión para MexOrbit

**Propósito:** antes de decidir el set de materiales de MexOrbit, entender para qué existió *realmente* cada recurso del DarkOrbit clásico. La historia revela la función; la función decide si sobrevive. Complementa a [economia-darkorbit-2010-retail.md](economia-darkorbit-2010-retail.md) (§6) y a la decisión de eliminar el uridium como moneda.

## 1. La historia, recurso por recurso

| Recurso | Desde | Cómo se obtenía | Función REAL | Destino típico |
|---|---|---|---|---|
| **Prometium** | 2006 (lanzamiento) | Cajas de carga, cargo de aliens, recolección | Ingreso crudo de volumen | Vender (10–20 C) o refinar |
| **Endurium** | 2006 | Ídem | Ingreso crudo (variante) | Vender (~15 C) o refinar |
| **Terbium** | 2006 | Ídem (mapas medios/altos) | Ingreso crudo (variante alta) | Vender (~25 C) o refinar |
| **Prometid** | 2006 | Refinado: 10 prometium + 10 endurium | **Escalón intermedio** | Vender (trampa: vale menos que sus insumos) o promerium |
| **Duranium** | 2006 | Refinado: 10 endurium + 10 terbium | **Escalón intermedio** (idéntico al anterior) | Ídem |
| **Xenomit** | 2006 | **Solo drops de NPC** (no se recolecta en el espacio) | **Catalizador**: 1 unidad por cada promerium refinado | Consumido al refinar |
| **Promerium** | 2006 | Refinado: 10 prometid + 10 duranium + 1 xenomit | El material "premio": recubrir equipo | Recubrir láseres/cohetes (+daño por carga) o escudos/motores (por minutos); luego, insumo de seprom |
| **Seprom** | **may 2011** (Skylab) | 10 promerium → 1 seprom, solo en Skylab | **El ápice**: el recubrimiento élite (láser "sepromeado" era el estándar UFE) | Recubrir láseres (~+60%) y escudos |
| **Palladium** | **2011** (mapas piratas) | Solo cajas flotantes en 5-1/5-3 (zona de riesgo) | **Boleto de acceso**: 15 palladium = 1 energía de puerta galáctica | Canje en la base desmilitarizada de 5-2 |

Detalles finos que la historia deja ver:

- **El sistema de recubrimiento** (prime ores): cargar promerium/seprom en un láser añadía 10 unidades *que se consumían al disparar* — munición sobre la munición; en escudos/motores daba 10 *minutos* por unidad. Es decir: el material ápice era un **consumible de mantenimiento del poder**, no una compra única. Buen diseño de demanda perpetua, mal comunicado.
- **El xenomit existía para hacer no-gratis el refinado** — y BigPoint mismo lo admitió por diseño: el módulo Xeno del Skylab (2011) existía literalmente para *sustituir* el xenomit. Vendieron la solución a su propia fricción (patrón #7 del doc retail).
- **Prometid y duranium eran una trampa matemática documentada**: se vendían por menos que la suma de sus insumos. Su única función real era ser dos clics más entre el mineral crudo y el promerium.
- **El Skylab (may 2011) industrializó todo**: convirtió el refinado de actividad-de-vuelo en fábrica idle con timers y niveles — y de paso en otro sink de créditos/uridium (ampliaciones). Los intermedios pasaron de "trampa" a "columna de un Excel".
- **La era Reloaded es la advertencia**: BigPoint nunca retiró un material — solo agregó. Hoy conviven los 9 clásicos MÁS Diametrion, Kyhalon, Tetrathrin, Bifenon, Schism Crystals, Osmium… (los vimos en las recompensas de puertas modernas, §14 del doc retail). Inflación de SKUs sin funciones nuevas: el anti-patrón de materiales.

## 2. El análisis funcional: 9 SKUs, 6 funciones

| Función económica | SKUs que la cumplían | ¿Cuántos hacían falta? |
|---|---|---|
| Ingreso crudo de recolección | Prometium, Endurium, Terbium | Los 3 se justifican SOLO por geografía (mapas distintos sueltan mezclas distintas) |
| Escalón intermedio | Prometid, Duranium | **Cero** — eran fricción de clics con disfraz de profundidad |
| Catalizador de refinado | Xenomit | Cero como catalizador (era fricción monetizable) |
| Material-premio del midgame | Promerium | 1 ✓ |
| Ápice consumible | Seprom | 1 ✓ |
| Boleto de acceso a endgame | Palladium | 1 ✓ (cuando el contenido exista) |

## 3. Propuesta de compresión para MexOrbit: de 9 a 6 (y 2 en reserva)

**Set activo del lanzamiento — 6 materiales, cada uno con UN trabajo:**

| Material | Rol | Grifo | Sumidero |
|---|---|---|---|
| Prometium | Crudo común | Cargo/recolección en mapas bajos | Vender / refinar |
| Endurium | Crudo medio | Mapas medios | Vender / refinar |
| Terbium | Crudo alto | Mapas altos/disputados | Vender / refinar |
| **Promerium** | Gate del midgame | **Refinado directo de los 3 crudos** (crudos: única fuente = cajas de carga de aliens) | Crafteo LF-3, recubrimientos medios, insumo de seprom |
| **Seprom** | Ápice | Solo producción: promerium + tiempo de assembly | LF-4, drones élite, recubrimiento élite (consumible) |
| **Xenomit** | Mantenimiento | Drops de combate (NPC y PvP) | **Reparaciones** — demanda perpetua |

**Los cortes y cambios, justificados por la historia:**

1. **Prometid y duranium: eliminados.** Su historia demuestra que nunca fueron una decisión — eran fricción (y una trampa de venta). El refinado queda directo: `X prometium + Y endurium + Z terbium → 1 promerium`. La "profundidad" que aportaban era falsa; la profundidad real está en las proporciones (qué mapa farmeas para qué crudo).
2. **Xenomit: reposicionado.** De catalizador-fricción a **material de mantenimiento** (reparaciones). Hereda la mejor propiedad del sistema de recubrimiento original — demanda perpetua — sin heredar la fricción. Y como solo cae de combate, amarra la economía al conflicto.
3. **Seprom conserva su doble naturaleza histórica**: insumo de crafteo (LF-4, élite) *y* consumible de recubrimiento. Esa segunda función es valiosísima: es el sink que sigue vivo cuando ya estás full — el élite "sepromeado" sigue farmeando para mantenerse sepromeado.
4. **Palladium: en reserva.** Su función (boleto de acceso por zona de riesgo) es excelente, pero un material sin su contenido es una columna muerta en la bodega. Entra el día que entre nuestro equivalente de puertas/contenido endgame, no antes. **Regla general que nos deja la era Reloaded: ningún material se lanza sin su sumidero, y se retira si su sumidero muere.**
5. **Uridium: eliminado** (decidido previamente — ver conversación de monedas). Su función histórica era ser el aparato de cobro; en una economía de una moneda + bono, no tiene trabajo que hacer.

**Resultado:** de 9 SKUs a 6 activos + 1 en reserva, sin perder ninguna función económica — los cortes eran fricción pura. La cadena queda legible en una línea:

> crudos (3, por geografía) → **promerium** (midgame) → **seprom** (ápice consumible), con **xenomit** aparte pagando las reparaciones.

### Fuentes
- [Fandom — Ore](https://darkorbit.fandom.com/wiki/Ore) · [Refining](https://darkorbit.fandom.com/wiki/Refining) · [Prometid](https://darkorbit.fandom.com/wiki/Prometid) · [Duranium](https://darkorbit.fandom.com/wiki/Duranium) · [Promerium](https://darkorbit.fandom.com/wiki/Promerium) · [Seprom](https://darkorbit.fandom.com/wiki/Seprom) · [Xenomit](https://darkorbit.fandom.com/wiki/Xenomit) · [Palladium](https://darkorbit.fandom.com/wiki/Palladium) · [Extra Energy](https://darkorbit.fandom.com/wiki/Extra_Energy)
- [Fandom — Skylab](https://darkorbit.fandom.com/wiki/Skylab) · [Skylab building guide](https://darkorbit.fandom.com/wiki/Skylab_building_guide) · [Reloaded Wiki — Promerium refinery](https://darkorbit-archive.fandom.com/wiki/Promerium_refinery)
