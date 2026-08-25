# Economía del DarkOrbit retail (~2008–2012): cuadros de costos y crítica

**Propósito:** radiografía de la economía del DarkOrbit clásico (la versión que replica nuestro contenido local: mismos items, aliens y mapas), con precios de tienda documentados y un análisis de por qué la estructura de precios produce *tiers muertos*, progresión saltada y pay-to-win. Este documento critica el juego original; las decisiones para MexOrbit se derivarán después en un documento aparte.

**Fuentes:** [DarkOrbit Fandom Wiki](https://darkorbit.fandom.com/), [DarkOrbitWiki.com](https://darkorbitwiki.com/), foros oficiales de BigPoint y elitepvpers. ⚠️ Las wikis actuales mezclan valores *post-rebalanceo* (Reloaded, 2015+: LF-1 a 65 de daño, LF-2 a 140, etc.). Aquí usamos los **stats clásicos 2010** —que son los que nuestra versión local ya implementa (LF-1 40, LF-2 100, LF-3 150, LF-4 200)— y los **precios de tienda clásicos**. Donde las fuentes discrepan se marca con ⚠️.

**Las dos monedas:**
- **Créditos** — se ganan jugando (kills, carga, misiones, subasta). Inflacionarios: un veterano acumula cientos de millones sin nada que comprar.
- **Uridium** — moneda premium. Se compraba con dinero real (referencia tardía documentada: 330,000 U ≈ $99 USD, ~$0.30/1,000 U; en 2010 el orden de magnitud era similar con paquetes de €5 a €100). Un jugador gratuito lo goteaba: ~1–4 U por alien, cajas bonus (~5–30 U), misiones.
- La **subasta** era el único puente créditos→élite: pujas de millones de créditos por items de uridium (G3N-7900, SG3N-B02, LF-3).

---

## 1. Motores (generadores de velocidad)

| Ítem | +Velocidad | Precio 2010 | Créditos por punto | Veredicto |
|---|---|---|---|---|
| G3N-1010 | +2 | 2,000 C | 1,000 | Muerto en horas |
| G3N-2010 | +3 | 4,000 C | 1,333 | Muerto |
| G3N-3210 | +4 | 8,000 C | 2,000 | Muerto |
| G3N-3310 | +5 | 16,000 C | 3,200 | Muerto |
| G3N-6900 | +7 | ⚠️ 256,000 C (clásico) / 1,000 U (moderno) | 36,571 | Trampa: 16× peor ratio que el 3310 |
| G3N-7900 | +10 | 2,000 U (o subasta ~5M C) | — | **El único que existe de facto** |

**Crítica:** la velocidad es el stat más determinante del juego (kiting, escape, perseguir). La escalera de créditos se degrada en *ratio* conforme subes (1,000 → 3,200 C/punto) y se corta en +5; el doble salto de calidad (+7, +10) vive en uridium. Resultado: 4 de 6 tiers no los compra nadie más de una vez, y el 6900 es un señuelo con el peor ratio de toda la tabla. El jugador aprende en la primera semana que equipar 10–15 slots de motor solo tiene una respuesta: 7900, vía cartera o vía subasta. **Seis SKUs, una decisión.**

## 2. Escudos (generadores de escudo)

Stats clásicos 2010 (los de nuestra versión local):

| Ítem | Escudo | Absorción | Precio 2010 | C por punto de escudo | Veredicto |
|---|---|---|---|---|---|
| SG3N-A01 | 1,000 | 40% | 8,000 C | 8 | Primer día |
| SG3N-A02 | 2,000 | 50% | 16,000 C | 8 | Primera semana |
| SG3N-A03 | 5,000 | 60% | 128,000–256,000 C ⚠️ | 26–51 | El techo real de créditos |
| SG3N-B01 | 4,000 | 50% | 2,560,000 C | **640** | **Trampa absoluta: 20× el costo del A03 por menos stats** |
| SG3N-B02 | 10,000 | 80% | 10,000 U (o subasta) | — | **El único endgame** |

**Crítica:** la absorción hace el daño multiplicativo: 80% vs 60% no es "un poco mejor", es *otra categoría* — el B02 aguanta el doble de puntos y desvía el 80% del daño al escudo. No existe ningún puente entre A03 y B02: el B01 cuesta 20 veces más créditos por punto que el A03 siendo *peor* que él en absorción. Es un precio diseñado para que voltees a ver el precio en uridium y digas "10,000 U es razonable". Eso es *decoy pricing* de manual: el tier intermedio no existe para comprarse, existe para vender el premium.

## 3. Láseres — la escalera rota

| Ítem | Daño 2010 | Precio 2010 | Daño por 1,000 C | Veredicto |
|---|---|---|---|---|
| LF-1 | 40 | 10,000 C | 4.0 | Inicio |
| MP-1 | 60 | 40,000 C | 1.5 | Puente barato |
| LF-2 | 100 | 250,000 C | 0.4 | **El tier saltado** |
| LF-3 | 150 | **10,000 U** (o subasta) | — | El estándar élite: "8 Iris Goliath full LF-3" |
| LF-4 | 200 | **No comprable** (2011: Galaxy Gates, booty boxes) | — | El inicio del power creep terminal |

**Crítica (el salto MP-1 → LF-3):** cada tier de créditos es ~3–4× peor en valor que el anterior. El LF-2 pide 6.25× el precio del MP-1 por +67% de daño, y necesitas 10–15. Equipar la nave con LF-2 son ~3–4M de créditos… que rinden más pujados en subasta por LF-3 reales. Así que la ruta racional es exactamente la que describes: MP-1 baratos → acumular → saltar a LF-3. El LF-2 solo lo compra quien no ha entendido la economía todavía.

**Crítica (LF-4, 2011):** con el LF-3 al menos sabías el precio. El LF-4 cambió el modelo de *pagar por el item* a **pagar por probabilidades**: llaves de booty box (~500 U por intento, drop <1%) o Galaxy Gates (miles de U en energía por ~15% de chance). Un full LF-4 costaba en la práctica **millones de uridium en varianza** — el "15 millones de uridium en un año" que los veteranos identifican como el momento en que BigPoint mató su propia era dorada. Es lootbox antes de que se llamaran lootbox.

## 4. Drones — la mitad del poder, detrás de la caja

| Ítem | Slots | Precio | Total el set | Veredicto |
|---|---|---|---|---|
| Flax (×8) | 1 | 100k C, duplica cada uno: 100k→12.8M | ~25.5M C | La ruta gratuita: honesta pero 1 slot |
| Iris (×8) | 2 | Escalera: 15k, 24k, 42k, 60k, 84k, 96k, 126k, 200k U | **647,000 U** | El sink premium más grande del juego base |
| Apis (2012) | 2+bono | 45 planos × 24,000 U | **~1,080,000 U** | Full P2W: planos solo de booty pirata |
| Zeus (2012) | 2+bono | 45 planos × 33,000 U | **~1,485,000 U** | Full P2W, encima del Apis |
| Havoc (diseño) | +10% daño láser en drones (set completo) | **Puerta Zeta**: recompensa aleatoria (45% al completarla) | ~18 Zetas esperadas para el set de 8 | La lotería sobre la lotería |
| Hercules (diseño) | +15% escudo en el dron; set completo: +HP de la nave | **Puerta Kappa**: recompensa aleatoria | Peor que Havoc (chance no publicada) | Ídem |

**Crítica:** los drones duplican tus slots efectivos — son *la mitad del poder del barco* — y su versión buena (Iris, 2 slots) es exclusivamente uridium: 647k U es ~$200–300 USD él solo, o años de goteo gratuito. Y cuando los veteranos por fin completaban sus 8 Iris, llegaron Apis y Zeus (2012): **2.5M U adicionales** vía planos de cajas de azar, más diseños Havoc/Hercules encima. Cada capa nueva reseteaba a "incompleto" al que ya había pagado. Los 2 drones adicionales no expandieron el juego: expandieron la factura.

**Los diseños: la lotería sobre la lotería (Galaxy Gates).** Havoc y Hercules no tenían precio — se "ganaban" en las Galaxy Gates, y ahí está la triple capa de azar:

1. **Armar la puerta es RNG**: cada giro del generador cuesta **100 U** y da piezas aleatorias (con duplicados); completar una puerta grande consumía miles de giros — la comunidad reportaba decenas de miles de uridium por puerta.
2. **Completarla tampoco garantiza el diseño**: la Zeta suelta el Havoc con **45% de probabilidad**; la Kappa suelta el Hercules como "bono aleatorio" sin porcentaje publicado.
3. **Y necesitas 8**: el bono fuerte de ambos diseños exige el set completo en los 8 drones. Valor esperado: **~18 Zetas completadas** para el full Havoc — cientos de miles de uridium adicionales en giros, sin contrato de precio en ningún paso.

La única ruta gratuita era el grind de **palladium en 5-3** (introducido en 2011): farmear palladium → canjear por Extra Energy → giros gratis. Funcionaba, y por eso 5-3 vivía saturado de bots — era legendariamente el grind más desalmado del juego. El diseño de las gates es el mismo movimiento que el LF-4: reemplazar "sé cuánto me falta" por varianza pura, con un botón de 100 U como única constante.

## 5. Munición — el impuesto por disparo

Consumo clásico: 1 unidad × láser equipado × volea (1 volea/segundo).

| Ítem | Multiplicador | Precio/unidad | Costo por hora de farmeo (15 láseres) | Veredicto |
|---|---|---|---|---|
| LCB-10 | x1 | 10 C | ~540,000 C | La única rentable |
| MCB-25 | x2 | 0.5 U | ~27,000 U/h | Quema cartera |
| MCB-50 | x3 | 1 U | ~54,000 U/h | PvP/urgencias |
| UCB-100 | x4 | No en tienda (gates/cajas) | — | Élite de élite |
| SAB-50 | x2 escudo | 1 U | — | Utilidad PvP |
| RSB-75 | x5–x6 | 5 U | — | Cadencia 3s; burst PvP |
| CBO-100 | x3+escudo | 5 U | — | Evento |
| Cohetes | R-310 100 C · PLT-2026 500 C · PLT-2021 5 U · PLT-3030 7 U | | | Misma escalera C→U |

**Crítica:** la munición es la genialidad y el pecado original del modelo. Genialidad: es el único *sink continuo* del juego — el poder se renta, no se compra una vez. Pecado: como el costo escala con tus láseres, **mejorar tu nave encarece tu operación**, y la brecha x1↔x2+ convierte cada pelea seria en una decisión de cartera. El que paga dispara literalmente al doble o triple de daño con el mismo equipo. No hay ninguna munición intermedia ganable a volumen por juego (el UCB-100 existía justo ahí, y lo sacaron de la tienda).

## 6. Minerales — "lo más valioso del espacio" pagado en centavos

Precios de venta clásicos (mejoran marginalmente con honor):

| Mineral | Venta base | Máximo | Uso real en 2010 |
|---|---|---|---|
| Prometium | 10 C | ~20 | Vender / refinar |
| Endurium | 15 C | ~ | Vender / refinar |
| Terbium | 25 C | ~ | Vender / refinar |
| Prometid | 200 C | 400 | Refinado que **vale menos que sus ingredientes** |
| Duranium | 200 C | 400 | Ídem |
| Promerium | 500 C | 1,000 | El "premio" del refinado |
| Xenomit | no venta | — | Reparaciones |
| Seprom / Palladium | — | — | **No existían en 2010** (Skylab: mayo 2011; Palladium: 5-3, 2011) |

**Crítica:** la carga completa de un Kristallon (~1,170 minerales, el alien top del juego) vendida entera daba **~74,000 créditos… contra 409,600 que pagaba el kill mismo**. La recolección — temáticamente "las materias primas más valiosas del universo", mecánicamente la razón de tener bodega, cargo boxes y el botón de recolectar — aportaba ~15% del ingreso y ningún poder. Refinar era además una *estafa matemática* (prometid/duranium valen menos que el mineral crudo que consumen). Solo con el Skylab (2011) el promerium/seprom ganó un uso de poder real (mejorar láseres/escudos +20%/+60%)… y aún entonces, era otro sistema de espera-o-paga. En 2010: los minerales eran créditos con pasos extra.

## 7. Naves — cinco escalones de utilería y una sola decisión real

Los precios en créditos nunca fueron rebalanceados (los de la wiki actual son los de la época); los stats de las naves chicas sí sufrieron reworks posteriores, aquí van los del cliente clásico (⚠️ aproximados en las menores; las élite coinciden con nuestra BD local).

| Nave | HP clásico | Láseres | Gens | Velocidad | Precio | Veredicto |
|---|---|---|---|---|---|---|
| Phoenix | ~4,000 | 1 | 1 | 320 | **Gratis** (+ REP-S gratis) | Buen onboarding, lo único rescatable |
| Yamato | ~8,000 ⚠️ | 2–3 | 2 | ~340 | Retirada ~2010 al "cementerio de naves" | Precio clásico no sobrevive en fuentes |
| Defcom | ~16,000 ⚠️ | 4 | 4 | ~340 | Retirada ~2010 | Ídem (ambas volvieron años después a 120,000 U) |
| Liberator | ~8,000 | 4 | 4 | 360 | 40,000 C | Un fin de semana de vida útil |
| Piranha | ~16,000 | 6 | 6 | 360 | 100,000 C | Una semana |
| Nostromo | ~26,000 | 7 | 7 | 320 | 195,000 C | Días |
| BigBoy | 64,000 | 8 | 10 | 240 | **285,000 C** — el techo de créditos | El "tanque" que muere ante cualquier élite |
| Leonov (2011) | 64,000 | 6 | 6 | 360 | 15,000 U | "La élite barata", ventajas en mapas propios |
| **Vengeance** | **180,000** | **10** | **10** | **380** | **30,000 U** (o subasta) | Mitad de la única decisión real |
| **Goliath** | **256,000** | **15** | **15** | **300** | **50,000 U** (o subasta) | La otra mitad; el estándar élite |
| Spearhead (2012) | 100,000 | 5 | 12 | 370 | Paquetes de pago → uridium | Explorador/marcador de grupo |
| Aegis (2012) | 275,000 | 10 | 15 | 300 | Paquetes de pago → uridium | Sanador de grupo |
| Citadel (2012) | 550,000 | 7 | 20 | 240 | Paquetes de pago → uridium | Tanque/warp de grupo |

**Crítica — la escalera de créditos es un tutorial disfrazado de progresión.** Toda la línea de créditos junta (Liberator→BigBoy) cuesta **620,000 créditos** — menos que 3 láseres LF-2 — y culmina en un BigBoy de 64k HP y 8 láseres que contra un Goliath (256k HP, 15 láseres, y encima más rápido que él) es *cuatro veces menos nave*. Cinco SKUs cuya única función es ser abandonados; nadie en la historia del juego hizo PvP serio en un Nostromo. El patrón es idéntico al de motores y escudos, pero aquí es más grave porque la nave define cuántos slots tienes: **el techo de créditos te deja con la mitad de los slots del juego.**

**La única decisión de build del DarkOrbit clásico — Vengeance vs Goliath — es genuinamente buen diseño** (velocidad y evasión contra tanque y slots, sin que ninguna domine), y está íntegramente detrás del uridium. El juego guardó su única elección interesante para el catálogo premium, con la subasta como válvula para veteranos ricos en créditos.

**Los diseños de nave** escalaron de cosmético (Jade, Adept, Corsair… en paquetes de dinero real, con bonos menores de %) a los **diseños de habilidad** — botones de combate vendidos por euros, tan importantes que tienen su propia sección (§8). Junto con el trío 2012 (Aegis/Citadel/Spearhead, inicialmente solo en paquetes), son la encarnación en naves del "15M de uridium en un año" que rompió a la base veterana: cada nave/diseño nuevo reseteaba el estatus de élite ya pagado.

## 8. Los diseños de habilidad — el día que el P2W se volvió literal (16 dic 2010)

Cinco diseños de Goliath con **habilidad activa exclusiva**, lanzados el 16 de diciembre de 2010 — es decir, *dentro* de la era dorada, no después de ella. Precio: **250,000 uridium cada uno, o ~35€ en pago directo** (años más tarde llegaron a subasta/assembly). Efectos y tiempos según la implementación de época (emulador legacy en nuestro repo, coincide con las descripciones retail):

| Diseño | Habilidad | Números de época | Duración / Cooldown |
|---|---|---|---|
| **Solace** | Nano-reparación instantánea | Cura **35% del HP máximo** propio (~90,000 HP en un Goliath full); +15% a compañeros de grupo cercanos | Instantáneo / 15 min |
| **Spectrum** | Escudo prismático | **−80% de daño recibido**; a cambio tu daño saliente −50% | 30 s / 15 min |
| **Sentinel** | Fortaleza | −30% del daño láser que entra a tus escudos | 2 min / 20 min |
| **Diminisher** | Debilitar escudos | **+50% de tu daño** contra el objetivo marcado | 60 s / 20 min |
| **Venom** | Singularidad | DoT imparable sobre el objetivo: 1,500/s que **crece +200 cada segundo** — hasta ~440,000 de daño acumulado si aguanta los 60 s | 60 s / 20 min |

**Crítica — tres rupturas en un solo item:**

1. **Rompió la simetría del combate, no la velocidad de progresión.** Todo el P2W anterior (LF-3, B02, Iris) compraba *números más grandes en el mismo juego*; el que farmeaba años llegaba al mismo lugar. Los skill designs venden **verbos que el otro no tiene**: un botón que cura 90,000 HP, un DoT que derrite 440k, un −80% de daño a voluntad. Contra un Solace, un Goliath "full élite" gratuito pelea contra una barra de vida de 135%; contra un Venom, un Vengeance (180k HP) está muerto por decreto en menos de 60 segundos. Por primera vez, dos jugadores con el mismo equipo y la misma habilidad manual **no juegan el mismo juego**.
2. **El precio rompió la escala del propio catálogo.** 250,000 U por *un botón* = **cinco Goliaths** = casi el precio del futuro dron Apis completo. Y como solo puedes montar un diseño a la vez, el metajuego élite exigía comprar varios y alternarlos según el rival: el "set" de 5 son **1.25M de uridium** — más que todo lo demás del juego base junto. La alternativa de 35€ directos, además, saltaba el uridium por completo: dinero → poder, sin moneda intermedia siquiera.
3. **Fecha clave para nuestra tesis: 16 de diciembre de 2010.** El power creep que los veteranos del hilo ubican en 2011–2012 (PET, LF-4, Apis/Zeus, Hércules) en realidad empezó *antes*, dentro del periodo que la nostalgia recuerda como puro. La era dorada no fue una época sin abusos: fue la época en que los abusos todavía no alcanzaban a invalidar el grind previo. Los skill designs fueron el primer item que sí lo hizo — y el patrón de ahí en adelante fue solo repetirlo más rápido.

## 9. Extras y CPUs — la capa donde todo lo útil es premium

Precios documentados ([DarkOrbitWiki — CPU](https://darkorbitwiki.com/equipment/cpu/); ⚠️ algunos son valores de la era Reloaded, el patrón de monedas es el histórico):

**Robots de reparación — vendiendo tu propio tiempo muerto:**

| Ítem | Reparación completa en | Precio | Veredicto |
|---|---|---|---|
| REP-S | lentísimo (1,000 HP/min) | Gratis | El ancla |
| REP-1 | 165 s | 10,000 C | Primer día |
| REP-2 | 120 s | 64,000 C | El techo de créditos |
| REP-3 | 105 s | 5,000 U | 15 segundos por 5,000 U |
| REP-4 | 90 s | 20,000 U | **30 s menos que REP-2 por 20,000 U** |

El repbot no da poder: da *menos downtime*. Monetizar los segundos de reparación es literalmente venderle al jugador su propio tiempo de vuelta, y el salto REP-2→REP-4 (30 segundos) cuesta más uridium que una Vengeance entera si lo compras directo.

**CPUs de combate — matemática de combate en la vitrina premium:**

| Ítem | Efecto | Precio | Veredicto |
|---|---|---|---|
| AIM-01 / AIM-02 | −25% / −50% de fallos de láser | 75,000 U / 200,000 U | **Precisión = daño real; solo uridium, sin ruta de créditos** |
| ISH-01 (Insta-shield) | 3 s de inmunidad | 50,000 U | Botón defensivo obligatorio en PvP alto |
| SMB-01 (Smartbomb) | Bomba de área | 50,000 U | Ídem ofensivo |
| ROK-T01 | −50% cooldown de cohetes | 10,000 U | DPS directo |
| MIN-T01 / MIN-T02 | −25% / −50% cooldown minas | **5,000,000 C** / 25,000 U | El señuelo de manual: 5M de créditos por la mitad del efecto |
| DR-01 / DR-02 | Reparar drones (8 usos / 32 usos) | 150,000 C / 15,000 U | Durabilidad + reparación premium: doble cobro |
| CL04K XS/M/XL (cloak) | 1 / 10 / 50 invisibilidades | 500 U / 5,000 U / 22,500 U | **Ventaja táctica como consumible: ~450–500 U por uso, cero ruta de créditos** |

**CPUs de utilidad — el juego vendiendo la automatización de su propio tedio:**

| Ítem | Efecto | Precio |
|---|---|---|
| SLE-01…04 (slots extra) | +2 / +4 / +6 / +10 slots | 600,000 C / 75,000 U / 150,000 U / **250,000 U** |
| JP-01 / JP-02 / AJP-01 | Saltos de mapa (10 / 20 / ilimitado+voucher) | 150,000 C / 2,000 U / **75,000 U + vouchers consumibles** |
| AROL-X / RLLB-X | Auto-cohetes / auto-recarga | 25,000 U c/u |
| RB / AM (auto-compra) | Recompra munición/cohetes sola | 15,000 U c/u |
| NC-serie (auto-mejoras) | Automatizan el laboratorio | 10,000–15,000 U |
| FB-X | Auto-combustible del PET | 5,000 U |
| GEMINEX (compresor) | Duplica bodega | 10,000 U |
| HM7 (dron comercial) | Vender minerales sin volar a base | **Solo paquetes de pago** |

**Crítica en tres capas:**

1. **Los slot extenders son la confesión.** El SLE-04 (+10 slots, 250,000 U) cuesta cinco Goliaths. La nave que compraste viene artificialmente incompleta y te venden su capacidad de regreso por partes. No hay mejor prueba de que el catálogo se diseñó desde la factura y no desde el juego.
2. **Todo lo que toca la matemática de combate (AIM, ISH, SMB, cloak) es uridium sin alternativa.** No son señuelos con tier de créditos malo: directamente **no existe ruta gratuita**. Un jugador con AIM-02 + cloaks + insta-shield juega otro juego contra uno sin ellos, con equipo idéntico.
3. **Las CPUs de automatización son el síntoma más revelador**: si los jugadores pagan 15,000–25,000 U por *no ejecutar* una mecánica (recomprar munición, recargar cohetes, volar a vender), la mecánica es tedio puro — y BigPoint lo sabía, porque a la vez baneaba bots que hacían exactamente eso gratis. Vender la solución al problema que tú mismo diseñaste es el modelo de negocio en una frase.

## 10. Potenciadores (boosters, 2011) — la suscripción que no se atrevió a decir su nombre

Ocho efectos × dos variantes = los 16 SKUs de nuestro catálogo. Todos al mismo precio: **10,000 U por 10 horas**.

| Serie | Efecto (B01, personal) | Efecto (B02, compartido) | Precio c/u |
|---|---|---|---|
| DMG | +10% daño | +10% propio, +1% a aliados cercanos | 10,000 U / 10 h |
| SHD | +10% escudo | ídem estructura | 10,000 U / 10 h |
| HP | +10% vida | ídem | 10,000 U / 10 h |
| REP | +reparación | ídem | 10,000 U / 10 h |
| SREG | +regeneración de escudo | ídem | 10,000 U / 10 h |
| RES | +recursos recolectados | ídem | 10,000 U / 10 h |
| HON | +honor | ídem | 10,000 U / 10 h |
| XP | +experiencia | ídem | 10,000 U / 10 h |

**La trampa está en que B01 y B02 se apilan**: DMG-B01 + DMG-B02 = +20% de daño… pagando dos veces. Mantener el stack estándar de combate (daño + escudo, ambas variantes) cuesta **4,000 U por hora de juego** — más que el ingreso íntegro de uridium de un jugador gratuito activo. Es una suscripción de facto (paga X por hora para existir al nivel del meta), pero cobrada por horas para que nunca se sienta como una cuota mensual… y para que el jugador intenso pague *más* que una cuota mensual. Sumado a la munición, el jugador élite de 2011+ ya no compraba poder: **lo rentaba en dos contadores simultáneos** (por disparo y por hora).

## 11. Formaciones de drones (jul 2012) — un millón de uridium por cambiar de figura

Las 13 originales (exactamente las de nuestro catálogo). Precios documentados:

| Formación | Bono | Castigo | Precio |
|---|---|---|---|
| Turtle | +10% escudo | −7.5% daño | 1,000,000 C |
| Arrow | +20% daño cohete | −3% daño láser | 1,000,000 C |
| Lance | +50% daño minas | — | 75,000 U (assembly) |
| Star | +25% daño cohete, +5% evasión | +33% recarga lanzacohetes | 100,000 U |
| Pincer | +3% daño PvP, +5% honor | −10% penetración de escudo | 100,000 U |
| Double Arrow | +30% daño cohete, +10% penetración | −20% escudo | 75,000 U |
| Diamond | Regenera 1%/s de escudo | −30% HP | 100,000 U |
| Chevron | +65% daño cohete | −20% HP | 75,000 U |
| **Moth** | **+20% penetración de escudo, +20% HP** | −5%/s escudo propio | 100,000 U |
| Crab | +20% absorción | −15% velocidad | 100,000 U |
| **Heart** | **+20% HP, +20% escudo** | −5% daño láser | 100,000 U |
| Barrage | +5% daño PvE, +5% XP | −15% absorción | 100,000 U |
| Bat | +8% daño PvE, +8% XP | −15% velocidad | 125,000 U |

**Crítica:** como *diseño de juego*, las formaciones son de lo mejor que hizo BigPoint — trade-offs reales, contexto (Moth para romper tanques, Heart para aguantar, Crab para huir), decisiones por pelea. Como *economía*, son el patrón señuelo perfeccionado: las dos formaciones de créditos son las **peor valoradas de la tabla** (Turtle y Arrow, bonos pequeños con castigo), y todo lo táctico real está en uridium. El set completo original: **2M de créditos + 1,050,000 U**. Y a diferencia del equipo, aquí no hay "elegir una": el meta PvP exigía alternar Moth/Heart/Crab en combate, así que o tenías el abanico o peleabas con desventaja de verbos — la lección de los skill designs, aplicada en masa 18 meses después.

## 12. El P.E.T. (jul 2011) — la obra maestra del triple cobro

El PET es la pieza más sofisticada de monetización de toda la era, porque cobra **tres veces por el mismo objeto**:

**CAPEX — comprarlo y armarlo:**

| Concepto | Costo |
|---|---|
| P.E.T. 10 base (1 gear, 1 láser, 2 gens, 2 protocolos, 50k HP) | **50,000 U** — el precio de un Goliath |
| Niveles (XP + uridium por nivel) | de miles a cientos de miles de U acumulados |
| Gears: Kamikaze (hasta 50,000 U nv.3), Auto-loot (2,500–37,500 U), Reparador, Localizador, Comerciante… | **~325,000 U** el set completo |
| Protocolos IA (nuestros `ai-*`: escudo, láser, evasión, HP, cargo, radar, puntería…) | **~229,000 U** el set completo |
| Ampliaciones de slots (gears hasta 100,000 U por slot) | cientos de miles más |

**OPEX — mantenerlo encendido:** combustible a **0.25 U por unidad**, con consumo por minuto = `1 + nivel×3 + protocolos×3 + gears×6`. Un PET nivel 10 medianamente equipado quema ~50 unidades/minuto ≈ **700–750 U por hora de vuelo**. El tanque lleno (50,000 unidades) son 12,500 U.

**El seguro — cuando muere:** reparación cobrada en uridium, escalando con el nivel (hasta cientos de miles de U en niveles altos sin premium). Y la guinda: el gear **Kamikaze** — pagas 50,000 U por la habilidad de *hacer explotar tu propio PET* como ataque… y luego pagas la reparación. Es el único item del catálogo cuyo uso previsto es generar su propio costo de reposición.

**Crítica:** un PET completo ronda **~1M de uridium de inversión + ~750 U/hora de operación + franquicia por muerte**. Es una segunda nave en precio, con la diferencia de que la nave se compraba una vez. El PET inauguró en 2011 el modelo CAPEX+OPEX+seguro que ninguna pieza anterior se había atrevido a juntar — y como además hacía el farmeo objetivamente mejor (auto-loot, kamikaze), no era opcional para el jugador serio.

## 13. Módulos de base de clan (CBS, 2013) — monetizar la guerra de clanes

Los 10 módulos de nuestro catálogo son exactamente el set CBS: internos (Hull `hulm`, Deflector `defm`) y externos (torretas láser `ltm-lr/mr/hr` — corto alcance/alto daño, medio/medio, largo/preciso —, arrays de cohetes `ram-la/ma`, reparador `repm`, y boosters de clan de daño/honor/XP `dmgm/honm` que escalan hasta +10% para todo el clan).

Lo documentado de su economía:

- Se construyen en **assembly con recursos + tiempo real** y se despliegan en puntos fijos del mapa; los módulos se **desgastan y expiran** (vida útil ~1 semana, según documentación comunitaria ⚠️), forzando reconstrucción perpetua.
- **Reparación de emergencia: 1,000 U por uso** — cualquier miembro del clan puede pagarla en caliente durante un asedio. Un consumible premium accionado en pánico colectivo.
- Destruir módulos enemigos paga **500–2,600 U** — la guerra de clanes como faucet de uridium… que se reinyecta en reconstruir y reparar.

**Crítica:** el CBS tomó lo único que el dinero no podía comprar en DarkOrbit — la historia social, los clanes, el territorio — y le puso taxímetro. La base no es un logro que se conquista una vez: es una **obligación recurrente de clan** (recursos + reconstrucción semanal + 1,000 U por emergencia) donde el clan que más quema, domina el mapa. El resultado conocido: los clanes chicos dejaron de disputar territorio y el sistema aceleró la concentración de poder que vació los servidores. Convertir el pegamento social del juego en un sink fue, de todas las decisiones de esta lista, la que atacó directamente la razón por la que la gente se quedaba.

---

## 14. Puertas galácticas (2009–2012) — el endgame como máquina tragamonedas

Las puertas de la época: **Alpha, Beta, Gamma** (las originales), **Delta** (2011), **Zeta** (ago 2011, debutó junto al LF-4), **Epsilon** (2011), **Lambda** (2012) y **Kappa** (dic 2012). Hades y Kronos llegaron en 2013, ya en la frontera del periodo.

**La máquina (el costo real).** Las puertas no se compran: se *arman* girando el Generador Galáctico a **100 U por energía** (1, 5, 10 o 100 giros por tirada). Cada giro paga según esta distribución documentada:

| Resultado del giro | Probabilidad |
|---|---|
| Munición | 67% |
| **Pieza de puerta** | **13%** |
| Xenomit | 12% |
| Nano-casco | 4% |
| Voucher de reparación | 3% |
| Log disk | 1% |

Es decir: **87 de cada 100 giros no avanzan la puerta**. A 13% de pieza por giro, cada pieza cuesta en promedio ~770 U brutos; los multiplicadores por duplicado (máx. 6 acumulables) lo bajan a unos ~550–650 U efectivos. Con eso, el costo esperado de armar cada puerta:

| Puerta | Piezas | Costo esperado de armado | Oleadas (referencia Alpha) |
|---|---|---|---|
| Alpha | 34 | **~19,000–26,000 U** | 10 oleadas: de 20 Streuners a 16 Kristallons |
| Beta | 48 | ~26,000–37,000 U | Más densa que Alpha |
| Gamma | 82 | ~45,000–63,000 U | Más densa que Beta |
| Delta | 128 | **~70,000–98,000 U** | La más cara de armar de la era |
| Epsilon | 99 | ~54,000–76,000 U | |
| Zeta | 111 | ~61,000–85,000 U | Oleadas especiales |
| Lambda | 45 | ~25,000–35,000 U | |
| Kappa | 120 | ~66,000–92,000 U | |
| **Set completo de la era** | **667 piezas** | **~370,000–510,000 U** | …y son contenido repetible |

La ruta gratuita: **palladium en 5-3** (15 palladium = 1 energía), el grind más denso del juego — miles de cajas de palladium por puerta. Y una capa más de consumible adentro: entras con **5 vidas**; las extra cuestan **5,000 U duplicándose por cada una**.

**Las recompensas, puerta por puerta.** ⚠️ Las tablas que sobreviven en las wikis son las post-buff (2015+); en la época los montos eran menores, pero la estructura y los premios firma son los de entonces. Junto al costo de armado, el balance en uridium queda así:

| Puerta | Costo armado | Uridium | UCB-100 (x4) | XP | Honor | Log disks | Balance U | Premio firma |
|---|---|---|---|---|---|---|---|---|
| Alpha | ~19–26k U | 20,000 | 20,000 | 4M | 100k | 2 | ≈ tablas | Anillo dorado (1.ª vez) |
| Beta | ~26–37k U | 40,000 | 40,000 | 8M | 200k | 4 | **positivo** | |
| Gamma | ~45–63k U | 60,000 | 60,000 | 12M | 300k | 10 | ≈ tablas / positivo | |
| Delta | ~70–98k U | 45,000 | 45,000 | 9M | 225k | 8 | **negativo** | |
| Epsilon | ~54–76k U | 25,000 | 20,000 | 5M | 150k | 10 | **muy negativo** | Chance de **LF-4** |
| Zeta | ~61–85k U | 35,000 | 25,000 | 6M | 200k | **50** | **muy negativo** | **Havoc (45%)** |
| Lambda | ~25–35k U | 15,000 | 10,000 | 2.75M | 100k | 3 | negativo | |
| Kappa | ~66–92k U | 15,000 | 30,000 | 9M | 325k | 10 | **el más negativo** | **Hercules (37.5%) + LF-4 (25%) + MultiBooster (25%)** |

Y el UCB-100 cuenta doble: es munición x4 que **no se vende en tienda** — valorada contra la MCB-50 (x3 a 1 U el tiro), cada carga de 20,000–60,000 UCB equivale a decenas de miles de uridium en poder de fuego que no tuviste que comprar.

**El patrón del balance no es casualidad — es el tarifario de la lotería.** Las tres puertas viejas (A/B/Γ), que solo pagan moneda, quedan en equilibrio o en positivo: eran **el motor económico del jugador gratuito serio** — armarlas con energía de palladium y cosechar uridium + UCB era la única ruta libre hacia el equipo élite, y por eso 5-3 vivía saturado. Las puertas nuevas (2011–2012) invierten el signo exactamente en proporción al premio firma que sortean: Delta pierde poco (no sortea nada), Epsilon y Zeta pierden mucho (LF-4, Havoc), y Kappa — la del triple sorteo — es la más deficitaria del juego. **El déficit de uridium de cada puerta es, literalmente, el precio implícito del boleto.** BigPoint no puso precio al LF-4 ni al Hercules: puso un impuesto de varianza en la puerta que los sortea. Detalle fino: Zeta paga 50 log disks (10-25× lo normal) — el cebo de puntos de piloto colocado justo en la puerta del Havoc.

**Crítica — el lazo perfecto de la casa.** Las puertas son, mecánicamente, el mejor contenido que DarkOrbit tuvo jamás: oleadas cooperativas, dificultad real, el único PvE que exigía equipo élite y coordinación. Y su economía es un casino de tres pisos: pagas en varianza para *armar* (87% de giros muertos), pagas vidas para *intentar*, y el premio gordo es otra varianza (45% el Havoc, ~15% el LF-4). El ROI en uridium es negativo por diseño — recuperas una fracción de lo girado — y la diferencia se paga a cambio de *chances* de los items que no tienen precio. Nótese la jugada maestra: la munición UCB-100 (x4, la mejor del juego) **solo** sale de aquí y de cajas — el jugador élite farmeaba puertas para financiar la munición con la que farmeaba puertas. El endgame entero era un bucle cerrado donde la casa cobraba comisión en cada vuelta; Kronos (2013) lo remató exigiendo **17 puertas completadas** (1×A, 3×B, 1×Γ, 1×Δ, 4×Ε, 1×Ζ, 2×Κ, 5×Λ) solo para poder armarla.

## 15. Cofres piratas y el precio real de los items "sin precio"

**El sistema.** Los cofres aparecen en los mapas piratas (5-1 a 5-3) y se abren con llaves. La **llave verde** se vende en tienda a **1,500 U** (con descuentos por paquete); las llaves **roja, azul y dorada** solo existían en eventos y paquetes de pago. Cada cofre da 1–4 items: casi siempre munición y morralla, con chances de extras caros, energía de puertas, **partes de Apis** (verdes/rojos/azules), **partes de Zeus** (solo dorados) y **LF-4**.

**BigPoint jamás publicó las probabilidades** — en un sistema donde el item central del juego (LF-4) sale de cajas de azar, la tasa era secreto comercial. Lo que existe son muestras comunitarias, y son elocuentes:

| Muestra documentada (foros/fandom) | Resultado | Tasa implícita |
|---|---|---|
| 671 cofres verdes (~1M U en llaves) | **0 LF-4**, 25 partes de Apis | LF-4 < 0.45%; parte Apis ≈ 3.7% (1 cada 27 llaves ≈ **40,000 U por parte**) |
| ~80 llaves (mixtas) | 2 LF-4, 12 partes Apis, 1 LF-3, 6 B02 | ~2.5% LF-4 (muestra chica, con suerte) |
| 30 azules + 50 rojos (paquetes de pago) | **8 LF-4** + 8 partes Apis | **~10% LF-4 por cofre de evento** |

**El costo esperado de cada item élite, ruta por ruta:**

| Item | Ruta | Costo esperado |
|---|---|---|
| **LF-4** (1 unidad) | Kappa (25% actual; ⚠️ ~15% en la época) | 4 Kappas ≈ **265,000–370,000 U** brutos (época: 440,000–615,000 U) |
| | Epsilon (~15%) | ~360,000–505,000 U brutos |
| | Cofres verdes (<0.45%) | **>330,000 U** — y la muestra de 671 dice que puede ser mucho peor |
| | Cofres rojo/azul de evento (~10%) | **~15,000–20,000 U por LF-4** … pero las llaves solo llegaban en paquetes de dinero real |
| **Full LF-4** (~30 láseres, nave+drones) | La mejor ruta de juego (Kappa) | **~8–11 millones de U** — la cifra exacta de la narrativa veterana del "15M de uridium" |
| **Havoc** (set de 8) | Zeta al 45% → ~17.8 Zetas | **1.1M–1.5M U** brutos; ~480,000–890,000 U netos tras recompensas |
| **Hercules** (set de 8) | Kappa al 37.5% → ~21.3 Kappas | **1.4M–2.0M U** brutos |
| **Apis** (45 partes) | Assembly a 24,000 U/parte | **1,080,000 U** — precio fijo |
| | Cofres verdes (≈40,000 U/parte) | ~1,800,000 U — más caro Y con varianza |
| **Zeus** (45 partes) | Assembly a 33,000 U/parte | **1,485,000 U** — precio fijo |
| | Cofres dorados | Partes solo ahí; llaves solo de eventos; tasa jamás conocida — **incalculable por diseño** |

**Crítica — la asimetría es el producto.** Léase la fila del LF-4 de nuevo: la mejor ruta *jugando* costaba ~300–600k U por láser; la ruta de *paquetes de evento* costaba ~15–20k U por láser. **El mismo item era 20–30 veces más barato pagando** — no porque las cajas fueran generosas, sino porque las llaves que sí tenían buena tasa no se podían obtener jugando. Eso convierte los cofres en un mecanismo de segmentación perfecta: el jugador gratuito veía LF-4s salir de cajas (los streamers de la época abriendo paquetes) y compraba llaves verdes cuya tasa real era ~20 veces peor. Con las probabilidades secretas, ni siquiera podía saber que estaba en otra lotería. Y la tabla del assembly (Apis/Zeus a precio fijo) confirma que BigPoint sabía perfectamente ponerle precio a las cosas — elegía no hacerlo exactamente donde la opacidad rendía más.

## 16. Los aliens de la época y sus recompensas

Fuente primaria: el FAQ oficial de aliens del foro de BigPoint (board-es), cuya tabla ya está replicada en nuestro [rollout de NPCs](../../MexOrbit.Server/Scripts/2025.12.03.2/npcs/rollout_npcs.sql); piratas verificados contra las fichas de Fandom. **eHP** = vida + escudo; **eHP/U** = cuánto hay que destruir por cada uridium — la métrica de farmeo (menor = mejor).

### Aliens normales (mapas x-1 a x-8)

| Alien | HP / Escudo | Daño | Vel. | Carga | XP | Honor | Créditos | Uridium | eHP/U |
|---|---|---|---|---|---|---|---|---|---|
| Streuner | 800 / 400 | 20 | 270 | 20 | 400 | 2 | 400 | 1 | **1,200** |
| Lordakia | 2,000 / 2,000 | 100 | 320 | 60 | 800 | 4 | 800 | 2 | 2,000 |
| Saimon | 6,000 / 3,000 | 250 | 320 | 124 | 1,600 | 8 | 1,600 | 4 | 2,250 |
| Mordon | 20,000 / 10,000 | 400 | 125 | 257 | 3,200 | 16 | 6,400 | 8 | 3,750 |
| StreuneR | 40,000 / 30,000 | 4,200 | 280 | 515 | 6,000 | 30 | 12,000 | 15 | 4,667 |
| Sibelonit | 40,000 / 40,000 | 1,500 | 320 | 317 | 3,200 | 16 | 12,800 | 12 | 6,667 |
| Kristallin | 50,000 / 40,000 | 1,650 | 320 | 333 | 6,400 | 32 | 12,800 | 16 | 5,625 |
| Protegit | 50,000 / 40,000 | 1,500 | 380 | 334 | 6,400 | 32 | 12,800 | 16 | 5,625 |
| Devolarium | 100,000 / 100,000 | 1,200 | 200 | 334 | 6,400 | 32 | 51,200 | 16 | **12,500** |
| Sibelon | 200,000 / 200,000 | 3,000 | 100 | 668 | 12,800 | 64 | 102,400 | 32 | **12,500** |
| Lordakium | 300,000 / 200,000 | 3,600 | 230 | 1,036 | 25,600 | 128 | 204,800 | 64 | 7,813 |
| Kristallon | 400,000 / 300,000 | 5,000 | 250 | 1,172 | 51,200 | 256 | 409,600 | 128 | 5,469 |
| Cubikon | 1,600,000 / 1,200,000 | **0** | 30 | 4,752 | 512,000 | 4,096 | 1,638,400 | 1,024 | **2,734** |

### Bosses (×4 la recompensa del normal)

| Boss | HP / Escudo | XP | Honor | Créditos | Uridium |
|---|---|---|---|---|---|
| Boss Streuner | 3,200 / 1,600 | 1,600 | 8 | 1,600 | 4 |
| Boss Lordakia | 8,000 / 8,000 | 3,200 | 16 | 3,200 | 8 |
| Boss Saimon | 24,000 / 12,000 | 6,400 | 32 | 6,400 | 16 |
| Boss Mordon | 80,000 / 40,000 | 12,800 | 64 | 25,600 | 32 |
| Boss StreuneR | 80,000 / 60,000 | 12,800 | 64 | 25,600 | 32 |
| Boss Sibelonit | 160,000 / 160,000 | 12,800 | 64 | 51,200 | 48 |
| Boss Kristallin | 200,000 / 160,000 | 25,600 | 128 | 51,200 | 64 |
| Boss Devolarium | 400,000 / 400,000 | 25,600 | 128 | 204,800 | 64 |
| Boss Sibelon | 800,000 / 800,000 | 51,200 | 256 | 409,600 | 128 |
| Boss Lordakium | 1,200,000 / 800,000 | 102,400 | 512 | 819,200 | 256 |
| Boss Kristallon | 1,600,000 / 1,200,000 | 204,800 | 1,024 | 1,638,400 | 512 |

### Ubers (×8 la recompensa del normal)

| Uber | HP / Escudo | XP | Honor | Créditos | Uridium |
|---|---|---|---|---|---|
| Uber Streuner | 6,400 / 3,200 | 3,200 | 16 | 3,200 | 8 |
| Uber Lordakia | 16,000 / 16,000 | 6,400 | 32 | 6,400 | 16 |
| Uber Saimon | 48,000 / 24,000 | 12,800 | 64 | 12,800 | 32 |
| Uber Mordon | 160,000 / 80,000 | 25,600 | 128 | 51,200 | 64 |
| Uber StreuneR | 320,000 / 240,000 | 48,000 | 240 | 96,000 | 120 |
| Uber Sibelonit | 320,000 / 320,000 | 25,600 | 128 | 102,400 | 96 |
| Uber Kristallin | 400,000 / 320,000 | 51,200 | 256 | 102,400 | 128 |
| Uber Devolarium | 800,000 / 800,000 | 51,200 | 256 | 409,600 | 128 |
| Uber Sibelon | 1,600,000 / 1,600,000 | 102,400 | 512 | 819,200 | 256 |
| Uber Lordakium | 2,400,000 / 1,600,000 | 204,800 | 1,024 | 1,638,400 | 512 |
| Uber Kristallon | 3,200,000 / 2,400,000 | 409,600 | 2,048 | 3,276,800 | 1,024 |

### Piratas de los mapas 5-x (2011)

| Pirata | HP / Escudo | Daño | Vel. | XP | Honor | Créditos | Uridium | Habilidad |
|---|---|---|---|---|---|---|---|---|
| Interceptor | 60,000 / 40,000 | 375–500 | 600 | 7,500 | 40 | 25,000 | 20 | Enjambre veloz (tipo Protegit) |
| Barracuda | 180,000 / 100,000 | 4,500–6,000 | 430 | 15,000 | 56 | 90,000 | 25 | Kamikaze al 20% de vida |
| Saboteur | 200,000 / 150,000 | 3,000–4,000 | 400 | 22,500 | 72 | 125,000 | 45 | Orbe que te frena a 120 |
| Annihilator | 300,000 / 200,000 | 11,250–15,000 + HST | 350 | 75,000 | 128 | 250,000 | 65 | Hellstorm |
| Battleray | 330,000 / 260,000 | 15,000–16,500 | 450 | 83,300 | 190 | **1,160,000** | 116 | Nodriza: invoca Interceptors, ISH, enrage, repara |

**Tres lecturas económicas de las tablas:**

1. **La aritmética es binaria y perfectamente plana entre tiers.** Recompensas en potencias de 2; Boss = ×4 y Uber = ×8 del normal — pero como el HP también escala ×4/×8, **la eficiencia eHP/U es idéntica dentro de cada familia** (Streuner: 1,200 en normal, Boss y Uber; Kristallon: 5,469 en los tres). Matar un Boss no era "mejor farmeo": era el mismo farmeo en un solo bocado. La elección real del jugador era *entre familias*, donde la eficiencia varía 10×.
2. **La tabla codifica dos rutas de farmeo distintas.** Por *uridium*, los eficientes son los extremos: Streuner (1,200 eHP/U — por eso los novatos progresaban) y Cubikon (2,734 — el mejor blanco grande, encima con daño 0). Por *créditos*, los tanques: Devolarium, Sibelon y Kristallon pagan 0.26–0.59 créditos por eHP siendo pésimos en uridium (12,500 eHP/U los dos primeros). El "spot" de cada quien era una decisión real — probablemente el mejor diseño económico silencioso del juego.
3. **Los piratas de 2011 inflaron el faucet junto con los sinks.** El Battleray paga 1.16M de créditos (3× un Kristallon) y los piratas traen mecánicas reales (kamikaze, frenos, enjambres) — la dificultad justificaba el pago. No es casualidad que el mismo parche que trajo el PET (sink de ~1M U) trajera los mapas 5-x (faucet de créditos y palladium): cada capa de gasto nueva venía con su zanahoria de ingreso para que el jugador *sintiera* que podía costearla. La economía entera de 2011 vivía en 5-x: palladium para puertas, piratas para créditos, cofres para el azar.

## 17. La cuenta final: cuánto costaba "terminar" el juego

La pregunta que resume todo el documento: ¿cuánto le costaba a un jugador el **full UFE de la era** — Goliath, 10 drones (8 Iris + Apis + Zeus), ~35 LF-4 (nave + drones + repuestos), 10 Havoc, 10 Hercules, escudos élite completos, extras completos, PET full, los 5 skill designs y el resto?

### La lista de compras (rutas de juego, uridium bruto)

| Concepto | Ruta | Costo |
|---|---|---|
| Goliath | Tienda | 50,000 U |
| 8 Iris | Tienda (escalera) | 647,000 U |
| Apis + Zeus | Assembly (90 partes) | 2,565,000 U |
| **35 LF-4** | ~140 Kappas (al 25%; ⚠️ con el ~15% de la época: ~235) | **9.2M–12.9M U** |
| 10 Hercules | **Gratis**: subproducto de esas 140 Kappas (37.5% → ~52 caen solos) | 0 U |
| 10 Havoc | ~22 Zetas (al 45%) | 1.4M–1.9M U |
| Escudos élite (~25 B02) + motores (7900) | Tienda/subasta | ~270,000 U |
| Extras completos (AIM-02, ISH, SMB, SLE-04, AJP, cloaks, autos…) | Tienda | ~800,000 U |
| PET completo (nave, niveles, gears, protocolos, slots) | Tienda | ~1,000,000 U |
| 5 skill designs | Tienda | 1,250,000 U |
| Naves de rol (Aegis/Citadel/Spearhead) y diseños de casco | Paquetes | ~200,000 U equiv. |
| *Devolución de las ~162 puertas voladas (uridium de recompensa)* | | *−2.9M U* |
| **TOTAL NETO** | | **~15–19 millones de U** |

(La sinergia de Kappa es el único descuento del sistema: farmear los LF-4 ahí regala los Hercules y ~35 MultiBoosters. Nótese también lo que la cifra confirma: la narrativa veterana del **"15 millones de uridium"** no era hipérbole — es la aritmética.)

A eso súmale lo no-uridium: ~162 puertas **voladas** (a 1–3 h por puerta: ~300–450 horas solo de PvE de puertas), los créditos menores (formaciones, Flax, subasta) y la hoja de piloto (log disks + uridium, varios millones más para quien la quería completa).

### El tiempo, por perfil de jugador

Base de ingreso gratuito de la época: cajas bonus (~300–600 U/h), palladium en 5-3 (~1,000 U/h equivalente en energía), aliens y misiones — un efectivo de **~700–1,200 U/h** jugando bien.

| Perfil | Ingreso anual aprox. | Tiempo hasta el full UFE (~15–19M U) |
|---|---|---|
| Gratuito casual (1–2 h/día) | ~400,000 U | **~40–50 años** — es decir: nunca, por diseño |
| Gratuito hardcore (4–6 h/día, eventos dobles, palladium) | ~2M U | **~7–9 años** |
| Pagador moderado (premium + ~20–30€/mes) | ~4M U | **~4–5 años** |
| Ballena (paquetes + llaves de evento) | — | **~$2,500–4,500 USD** y **3–6 meses** de calendario — el piso ya no es el dinero sino los eventos de llaves y las horas de vuelo de las puertas |

### La demostración del treadmill

Y aquí está el teorema que explica la muerte de la era dorada mejor que cualquier testimonio: el jugador gratuito hardcore generaba **~2M de uridium al año**… y BigPoint lanzó capas nuevas por **~2.5–4M de uridium al año** (2010: designs 1.25M; 2011: PET ~1M + puertas nuevas; 2012: formaciones 1.05M + Apis/Zeus 2.5M). **El déficit era estructural: el jugador más dedicado del mundo perdía terreno cada año jugando a diario.** El grind de DarkOrbit no era largo — era *divergente*. La única forma de converger era pagar, y la única forma de mantenerse era seguir pagando. Cuando los veteranos dicen "el juego se murió cuando ya no se podía alcanzar", esta tabla es exactamente eso, en números.

## 18. Diagnóstico: no son errores, es el diseño — y por eso envejeció mal

Juntando los cuadros, el patrón es idéntico en **todas** las familias:

1. **Escalera de créditos con ratio degradante** (cada tier es peor valor que el anterior) que se corta justo antes de la calidad competitiva.
2. **Un tier señuelo** carísimo en créditos (G3N-6900, SG3N-B01, LF-2) cuyo trabajo es hacer ver "barato" el precio en uridium.
3. **El item real en uridium**, precio fijo alto (2k / 10k / 10k U), comprable también por veteranos vía subasta — la válvula que mantenía a los ricos-en-créditos enganchados.
4. **Un sink continuo** (munición, boosters) que renta el poder en vez de venderlo, y que escala con tu propio progreso.
5. Desde 2011: reemplazo del precio fijo por **azar** (booty boxes, gates, planos) — más rentable, pero destruyó el contrato psicológico "sé cuánto me falta" que hacía funcionar el grind.

Y con naves y extras a la vista, se suman dos patrones más:

6. **La capa de capacidad sin ruta gratuita**: slots (SLE), precisión (AIM), invisibilidad (cloak), inmunidad (ISH) — lo que toca la matemática de combate o la capacidad de la nave no tiene ni siquiera un tier señuelo de créditos: es uridium o nada.
7. **El tedio como producto**: las CPUs de automatización (auto-munición, auto-cohetes, auto-venta) monetizan no-jugar las mecánicas aburridas del propio juego. Cuando tu tienda vende la solución a tu propio diseño, el diseño es el problema.
8. **La venta de verbos** (skill designs, dic 2010): el escalón terminal — ya no vender números más grandes sino *acciones que el otro jugador no puede ejecutar*. Es la línea que separa "paga para llegar antes" de "paga para jugar otro juego", y una vez cruzada no hay regreso: cada item posterior tuvo que competir contra botones de 35€.
9. **El poder rentado** (boosters, munición, combustible del PET): dos contadores corriendo siempre — por disparo y por hora. El stack estándar de boosters (4,000 U/h) superaba el ingreso total de un jugador gratuito: una suscripción encubierta que al jugador intenso le cobraba más que cualquier cuota.
10. **CAPEX + OPEX + seguro** (el PET): comprar el objeto, pagar por operarlo y pagar cuando muere. El gear Kamikaze como símbolo: un item cuyo uso previsto genera su propio costo de reposición.
11. **Monetizar lo social** (CBS, 2013): la base de clan como obligación recurrente — desgaste semanal, reparación de emergencia a 1,000 U el botonazo — convirtió el pegamento social del juego en el sink que aceleró la concentración de poder y vació los mapas.

**La cronología de las capas** deja ver el modelo completo: cada año, un sistema nuevo del tamaño de una economía entera — dic 2010: skill designs (1.25M U el set); 2011: PET (~1M U + 750 U/h), boosters (renta perpetua) y la expansión de puertas Delta/Zeta/Epsilon (~200k U de armado + LF-4 por varianza); 2012: formaciones (1.05M U), Apis/Zeus (2.5M U) y Lambda/Kappa; 2013: CBS (sink de clan sin fondo) y Kronos (17 puertas de peaje). El jugador que se puso al corriente en cualquier año amanecía atrasado al siguiente. Eso es lo que los veteranos llaman "el año en que murió el juego" — solo que ocurrió todos los años.

La consecuencia de diseño más importante para nosotros: **de ~40 items de progresión de equipo y naves, el jugador solo toma ~5 decisiones reales en toda su carrera** (MP-1 → LF-3, A02 → B02, 3310 → 7900, Flax → Iris, y la única buena: Vengeance vs Goliath). Todo lo demás es utilería de tienda. El "midgame" del DarkOrbit clásico no es una etapa del juego: es una sala de espera con precios inflados donde decides si vas a pagar o a farmear durante meses para el mismo único destino. Y cuando todos llegan al mismo destino (UFE), la única palanca que le quedó a BigPoint fue subir el techo (LF-4, Apis, Zeus, Havoc…), reseteando a su base de pagadores — la espiral que mató la era dorada.

Las **áreas de oportunidad** son el espejo de cada cuadro: tiers intermedios que sean *paradas reales* (decisiones con trade-offs, no señuelos), la calidad competitiva alcanzable por la moneda que se gana jugando, los minerales como insumo de poder y no como centavos, drones como progresión y no como catálogo de cartera, y munición premium ganable a volumen dentro del juego. Ese es el punto de partida del documento de economía de MexOrbit.

---

### Fuentes consultadas
- [DarkOrbitWiki — Lasers and Ammunition](https://darkorbitwiki.com/equipment/lasers-and-ammunition/)
- [DarkOrbitWiki — Generators](https://darkorbitwiki.com/equipment/generators/)
- [DarkOrbitWiki — Drones](https://darkorbitwiki.com/drones/)
- [Fandom — LF-1](https://darkorbit.fandom.com/wiki/LF-1) · [LF-2](https://darkorbit.fandom.com/wiki/LF-2) · [LF-3](https://darkorbit.fandom.com/wiki/LF-3) · [LF-4](https://darkorbit.fandom.com/wiki/LF-4) · [MP-1](https://darkorbit.fandom.com/wiki/MP-1)
- [Fandom — G3N-6900](https://darkorbit.fandom.com/wiki/G3N-6900) · [G3N-7900](https://darkorbit.fandom.com/wiki/G3N-7900) · [SG3N-B02](https://darkorbit.fandom.com/wiki/SG3N-B02) · [SG3N-A03](https://darkorbit.fandom.com/wiki/SG3N-A03)
- [Fandom — Drone](https://darkorbit.fandom.com/wiki/Drone) · [Iris](https://darkorbit.fandom.com/wiki/Iris) · [Flax](https://darkorbit.fandom.com/wiki/Flax) · [Apis](https://darkorbit.fandom.com/wiki/Apis) · [Zeus](https://darkorbit.fandom.com/wiki/Zeus)
- [Fandom — Ore](https://darkorbit.fandom.com/wiki/Ore) · [Promerium](https://darkorbit.fandom.com/wiki/Promerium) · [Prometium](https://darkorbit.fandom.com/wiki/Prometium) · [Terbium](https://darkorbit.fandom.com/wiki/Terbium) · [Duranium](https://darkorbit.fandom.com/wiki/Duranium)
- [Fandom — Buy now (in-game)](https://darkorbit.fandom.com/wiki/Buy_now(in-game)) · [Auction](https://darkorbit.fandom.com/wiki/Auction) · [Uridium](https://darkorbit.fandom.com/wiki/Uridium)
- [Fandom — Galaxy Gates](https://darkorbit.fandom.com/wiki/Galaxy_Gates) · [Reloaded Wiki — Zeta Gate](https://darkorbit-archive.fandom.com/wiki/Zeta_Gate) · [Foro oficial — Zeta reward FAQ](https://board-en.darkorbit.com/threads/zeta-galaxy-gate-reward-faq-update.119344/) · [Foro oficial — Havoc design](https://board-en.darkorbit.com/threads/gi-questions-about-drone-design-havoc.122167/)
- [FAQ oficial de aliens (board-es de BigPoint)](https://board-es.darkorbit.com/threads/faqs-aliens.135697/#post-992911) — replicado en `MexOrbit.Server/Scripts/2025.12.03.2/npcs/rollout_npcs.sql` · piratas: [Interceptor](https://darkorbit.fandom.com/wiki/Interceptor) · [Barracuda](https://darkorbit.fandom.com/wiki/Barracuda) · [Saboteur](https://darkorbit.fandom.com/wiki/Saboteur) · [Annihilator](https://darkorbit.fandom.com/wiki/Annihilator) · [Battleray](https://darkorbit.fandom.com/wiki/Battleray)
- [Fandom — Pirate chests & keys](https://darkorbit.fandom.com/wiki/Pirate_chests_%26_keys) · [Green Pirate Chest](https://darkorbit.fandom.com/wiki/Green_Pirate_Chest) · [Blue pirate chest](https://darkorbit.fandom.com/wiki/Blue_pirate_chest) · [Reloaded Wiki — Pirate booty](https://darkorbit-archive.fandom.com/wiki/Pirate_booty) · muestras comunitarias: [Booty Chest FAQ](https://board-en.darkorbit.com/threads/booty-chest-faq.799/) · [Chances of winning boxes](https://board-en.darkorbit.com/threads/ui-chances-of-winning-boxes.119496/) · [hilo de drops en fandom](http://darkorbit.wikia.com/wiki/Thread:21998)
- [DarkOrbitWiki — Galaxy Gates (piezas y distribución del generador)](https://darkorbitwiki.com/galaxy-gates/) · recompensas por puerta: [Alpha](https://darkorbitwiki.com/galaxy-gates/alpha-gate/) · [Beta](https://darkorbitwiki.com/galaxy-gates/beta-gate/) · [Gamma](https://darkorbitwiki.com/galaxy-gates/gamma-gate/) · [Delta](https://darkorbitwiki.com/galaxy-gates/delta-gate/) · [Epsilon](https://darkorbitwiki.com/galaxy-gates/epsilon-gate/) · [Zeta](https://darkorbitwiki.com/galaxy-gates/zeta-gate/) · [Lambda](https://darkorbitwiki.com/galaxy-gates/lambda-gate/) · [Kappa](https://darkorbitwiki.com/galaxy-gates/kappa-gate/) · [Fandom — Galaxy gate/Alpha](https://darkorbit.fandom.com/wiki/Galaxy_gate/Alpha)
- [elitepvpers — Total Item Upgrading Cost](https://www.elitepvpers.com/forum/darkorbit/1975888-total-item-upgrading-cost.html)
- [DarkOrbitWiki — CPU](https://darkorbitwiki.com/equipment/cpu/) · [Ships](https://darkorbitwiki.com/ships/) · [P.E.T.](https://darkorbitwiki.com/p-e-t/) · [Clan Battle Station](https://darkorbitwiki.com/guides/clan-battle-station/) · [Extras/Boosters](https://darkorbitwiki.com/equipment/extras-boosters/)
- [Fandom — Booster](https://darkorbit.fandom.com/wiki/Booster) · [DMG-B01](https://darkorbit.fandom.com/wiki/DMG-B01) · [DMG-B02](https://darkorbit.fandom.com/wiki/DMG-B02) · [Drone formation](https://darkorbit.fandom.com/wiki/Drone_formation) · [Pet 10](https://darkorbit.fandom.com/wiki/Pet_10) · [P.E.T Fuel](https://darkorbit.fandom.com/wiki/P.E.T_Fuel) · [Clan Battle Stations](https://darkorbit.fandom.com/wiki/Clan_Battle_Stations)
- [Fandom — Solace](https://darkorbit.fandom.com/wiki/Solace) · [Designs](https://darkorbit.fandom.com/wiki/Designs) · [Goliath-Designs (wiki DE: fecha 16-dic-2010 y precio 250k U / 35€)](https://darkorbit.fandom.com/de/wiki/Goliath-Designs) · efectos verificados contra el emulador de época en `Legacy/DarkOrbit 10.0/Game/Objects/Players/Skills/`
- [Fandom — Goliath](https://darkorbit.fandom.com/wiki/Goliath) · [Vengeance](https://darkorbit.fandom.com/wiki/Vengeance) · [BigBoy](https://darkorbit.fandom.com/wiki/BigBoy) · [Nostromo](https://darkorbit.fandom.com/wiki/Nostromo) · [Piranha](https://darkorbit.fandom.com/wiki/Piranha) · [Liberator](https://darkorbit.fandom.com/wiki/Liberator) · [Leonov](https://darkorbit.fandom.com/wiki/Leonov) · [Yamato](https://darkorbit.fandom.com/wiki/Yamato) · [Defcom](https://darkorbit.fandom.com/wiki/Defcom) · [Designs](https://darkorbit.fandom.com/wiki/Designs) · [Yamato and Defcom return](https://darkorbit.fandom.com/wiki/Yamato_and_Defcom_return)
