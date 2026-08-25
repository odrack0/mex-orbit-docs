# MexOrbit — Guidelines generales del juego

**Qué es este documento:** el diseño completo de economía y sistemas de MexOrbit — monedas, materiales, items, naves, aliens, contenido endgame, temporadas, mercado, misiones y perfil de piloto. Nació como las decisiones de economía y creció hasta ser el guideline general: toda decisión de diseño nueva se valida contra él.

**Estado:** diseño completo, 24-ago-2026; pendiente la pasada de balanceo (números). Consolida todo lo decidido en las sesiones de diseño económico. Documentos de soporte: [crítica del retail 2010](../02-investigacion/economia-darkorbit-2010-retail.md) (los anti-patrones), [economías MMO 2026](../02-investigacion/economias-mmo-2026.md) (los patrones modernos), [historia de los recursos](../02-investigacion/recursos-darkorbit-historia.md) (la compresión de materiales).

**La regla de oro (el invariante de convergencia):** el ingreso anual del jugador gratuito dedicado debe ser **mayor o igual** al costo del contenido nuevo de ese año. DarkOrbit murió por violar esto todos los años (~2M de ingreso contra 2.5–4M de capas nuevas). Toda decisión futura de contenido/precios se valida contra este invariante.

---

## 1. Monedas ✅

**Una sola moneda de cartera: Credits.**

- El uridium **se elimina por completo** — ni moneda ni material. Su función histórica era ser el aparato de cobro; en esta economía no tiene trabajo.
- No hay moneda premium comprable: cero exposición a la regulación de monedas virtuales (guías CPC UE 2025, Digital Fairness Act), cero *breakage*, cero ofuscación de precios.
- Honor y experiencia siguen existiendo como puntajes (rango/nivel), no como monedas.

## 2. El puente con el dinero real ✅

**El Starbond es un item (token) desde el día uno** — comprable solo con dinero real, vendible en la subasta por Credits. El que lo tiene elige: **canjearlo por License (premium) o venderlo y ganar Credits.** Se implementa una sola vez; lo que evoluciona es el modo de precio:

1. **Fase administrada** (lanzamiento): el valor en Credits del Starbond lo **fija y configura el administrador** (parámetro en `server_settings`, revisable con los datos en vivo). La subasta lo lista a ese precio único.
2. **Fase libre** (cuando haya liquidez): se libera a **oferta y demanda** — el precio lo pone el mercado. El cambio es un flag de configuración, no un desarrollo nuevo.

Fricción anti-especulación (aplica en ambas fases): **un solo salto** — al comprarse con Credits queda ligado al comprador; re-listarlo cuesta 10% de su valor (que además es sink de Credits). En fase libre, el precio del Starbond se convierte en nuestro termómetro público de la economía.

**El Starbond canjea únicamente:**
- **Star License** (el premium) — en 7 / 30 / 90 días.
- Cosméticos.
- **Nunca** poder directo, ni items, ni materiales.

**Star License (premium, UN solo SKU):**
- Incluye el **duplicador de pods** integrado (internamente son dos flags separados, para poder regalar "fin de semana de duplicador" en eventos).
- Reparación/QoL y conveniencias por definir.
- **Venta remota de carga** (herencia rediseñada del HMD-07): vender materiales al NPC sin volar a base, con penalización por zona — natal −15%, media −30%, alta **−50%**. Regla de calibración: *la penalización siempre debe superar el riesgo actuarial del viaje* — el que arriesga ir a base gana más en promedio, siempre. Solo canal NPC y solo carga; **la subasta exige el item físicamente en base** (el mercado de jugadores queda reservado para quien voló). Sin tope diario: la penalización se autorregula.
- El "rebate/descuento" del DO original **se elimina**: era una mecánica de incentivo de gasto (patrón oscuro bajo mira regulatoria). Lo reemplazan precios por volumen transparentes del Starbond.

**Meta aspiracional gratuita:** "pagar tu License con Credits" — el rito de pasaje del jugador dedicado (el "plexear" de EVE).

**El circuito completo:** la ballena compra Starbonds → los vende por Credits → le compra items élite al jugador que corrió el contenido. El dinero compra tiempo ajeno; **jamás inyecta poder al mundo**. El RMT queda capturado por la vía oficial.

## 3. Materiales ✅

**Compresión: de los 9+1 del retail a 5 activos.** Regla: *ningún material sin sumidero real; un material = un trabajo.* Nombres universales (pseudo-latín científico; los tres del cielo son elementos hipotéticos reales de la astronomía del s. XIX):

| Material | Rol | Zona | Equivalente DO |
|---|---|---|---|
| **Asterium** | Crudo común | Baja | Prometium |
| **Nebulium** | Crudo medio | Media | Endurium |
| **Coronium** | Crudo escaso | Alta/disputada | Terbium |
| **Aurorium** | Refinado medio — recubrimientos | (se refina de los 3 crudos) | Promerium |
| **Tachyon** | Ápice — recubrimiento élite | **Drops: aliens Uber y eventos especiales** (no se refina) | Seprom |

**Eliminados:** prometid y duranium (escalones-fricción; se vendían por menos que sus insumos), xenomit (catalizador-fricción; su rol de "reparaciones" también se descartó — ver §7), palladium (buen diseño de boleto de acceso, pero sin su contenido aún), uridium.

**En reserva con nombre listo:** **Ferrium** (mantenimiento, si algún día hay un trabajo real), **Jump Core** (acceso a contenido endgame, cuando ese contenido exista).

**Localización:** los nombres propios (materiales, Starbond, Tachyon) **no se traducen en ningún idioma** — son marca, como PLEX. Los descriptivos (pods/cápsulas, black box/caja negra, license/licencia) sí se localizan.

## 4. Refinado — "el clic desaparece, la decisión se queda" ✅

**Receta única, automática y gratis en base** (jamás se monetiza ni se premiumiza el refinado — lección del auto-refine del DO):

> **30 Asterium + 20 Nebulium + 10 Coronium → 1 Aurorium**

**El sesgo geográfico es el corazón del comercio** — ninguna zona suelta la mezcla de la receta (50/33/17):

| Zona | Mezcla de drops (A/N/C) | Le sobra | Le falta |
|---|---|---|---|
| Baja (natal) | 60% / 30% / 10% | Asterium | Coronium |
| Media | 30% / 45% / 25% | Nebulium | — (la más autosuficiente) |
| Alta/disputada | 15% / 30% / 55% | Coronium | Asterium |

Nadie refina solo: **comercian o viajan**. El novato tiene por primera vez algo que el élite necesita comprarle.

**El ápice NO se refina:**

> El **Tachyon** cae exclusivamente de **aliens Uber y eventos especiales** — comerciable. El cazador de gigantes es su productor; la subasta, su distribuidor.

Con esto el sistema de materiales no tiene **ninguna** mecánica idle: sin colas, sin timers, sin Skylab, sin edificios. Todo el material se gana volando. (Refinerías como infraestructura de clan quedan, si acaso, para fase 2.) Sinergia con §10: los Ubers pagan a la vez los planos del Pulsar y el Tachyon — "cazar gigantes" es la profesión completa del ápice. Palanca de calendario: eventos de "lluvia de Tachyon".

**El almacén (hangar de base):** todo jugador tiene su almacén/hangar donde guarda sus materias (y su equipo). Es el ancla de la logística:

- Lo almacenado está **siempre a salvo** — jamás en riesgo de muerte (§7: solo la carga volante se pierde).
- Ahí ocurre el **refinado automático** al descargar.
- La **subasta exige el item físicamente en el almacén** (§2/§6): listar en el mercado siempre requiere haber llevado la mercancía.
- La **venta remota** de la License (§2) es precisamente la excepción de conveniencia a este ancla — por eso penaliza.
- Capacidad: **por definir** — puede ser ilimitada o con progresión de ampliación (la única monetización/progresión de "espacio" aceptable según §12/slots es la de *organización*, nunca la de combate).

## 5. Las dos capas: items vs mejora ✅

**Los materiales NO craftean items.** (Fiel al DO clásico: el assembly con recetas es invento de la era Reloaded.)

| Capa | Se obtiene con | Comportamiento |
|---|---|---|
| **Items** (láseres, escudos, motores, naves, drones) | Credits (tienda/subasta) + contenido | Se compran una vez — la escalera vertical |
| **Mejora** (recubrimientos de daño/escudo/velocidad) | Materiales | **Se consumen** — el estado, no la posesión |

- Recubrimientos: Aurorium = tier medio, Tachyon = tier élite. Cargas que se gastan al disparar (láseres) / minutos (escudos, motores) — herencia directa del sistema de prime ores del DO clásico.
- **Por qué así:** la demanda de crafteo muere cuando el server se equipa (deflación de materiales); la demanda de mejora es **eterna** — el élite full sigue comprando Tachyon para *mantenerse* al máximo. El mercado de materiales nunca muere.
- **La dinámica igualadora:** los items escalan con eficiencia (Credits); el recubrimiento viene de farmear y acarrear carga — y circula por subasta, así que el jugador medio dedicado, full recubierto, le compite a un élite flojo sin recubrir. *El poder de estar activo contra el poder de tener cosas.*
- **Items élite (LF-4 y equivalentes):** salen de **contenido determinista con pity** ("completa N, llévate el item" — nunca lotería opaca) y son comerciables → el corredor de contenido es el productor; la subasta redistribuye.

## 6. Canales de venta ✅

**Dos canales, dos funciones:**

1. **Venta al NPC (el piso):** instantánea, precio fijo bajo (provisional: Asterium 10 C, Nebulium 15 C, Coronium 25 C). Garantiza liquidez al novato con el server vacío, ancla el mercado por abajo, y es un grifo de Credits deliberado y controlable.
2. **Subasta entre jugadores (el mercado):** el comercio del sesgo geográfico, los items élite, los Starbonds. No crea Credits — transfiere. La comisión de la casa es sumidero.

**Rediseño de la subasta ✅**: mercado de órdenes de compra/venta — diseño completo en §17.

Regla de lectura: *el NPC te compra por lástima, los jugadores por necesidad.* Si el precio de subasta del Coronium se pega al piso NPC, el sesgo de zonas está flojo.

**Reglas universales de la venta al NPC (confirmadas):**
- **Todo lo comerciable tiene precio piso NPC** — materiales, Aurorium, Tachyon, Flux, Cargas, partículas y AMPs (montos en el balance). Un principio, cero excepciones.
- **El NPC recompra equipo** a ~25–30% del precio de tienda, solo a reparación completa — la liquidez del novato a las 3 am; el mercado de segunda mano siempre paga más.
- La venta remota de la License aplica **solo a carga/materiales** (§2); el equipo se vende en persona o en el Mercado (§17).

## 7. Ciclo de muerte ✅

**Gradiente por zona — la penalización es parte de la geografía económica:**

| Zona | Carga que sueltas | Lógica |
|---|---|---|
| Natal/inicio | **0%** | Protección estructural del novato: el mapa lo protege, sin reglas de nivel |
| Media | **50%** | El estándar: duele sin arruinar |
| Alta/disputada | **100%** | Donde vive el Coronium: riesgo total, recompensa total |

- Lo soltado cae como **Black Box**: el vencedor la recoge — **transferencia, no destrucción** (ni un mineral desaparece del mundo; el PvP es un canal de distribución). Despawn en 2–3 min.
- **Solo la carga volante está en riesgo**: lo almacenado/refinado en base, jamás; los recubrimientos aplicados, tampoco. Pérdida máxima = un viaje, por diseño.
- **Revivir gratis e instantáneo** — sin fricción de reingreso (un server de 200 necesita MÁS peleas, no menos). Se descartó el xenomit-reparación: bloqueaba (sin material = no vuelas) y era fricción inventada para justificar un SKU.
- **Durabilidad de equipo con reparación en Credits (estilo WoW): TODA muerte desgasta el equipo — PvE y PvP por igual** (~1–3% del valor del item); se repara con Credits en cualquier base, instantáneo, en la misma parada donde ya descargas carga. Es **el sumidero perpetuo de Credits** que le faltaba a la economía — proporcional al equipo (el élite paga más: impuesto progresivo contra la acumulación del veterano). La regla uniforme además cierra un hueco: el peleador PvP con bodega vacía no soltaba nada — el taller es su única apuesta real. Calibración: una muerte cuesta *minutos* de ingreso, jamás sesiones. Durabilidad en cero = stats reducidos, **la nave siempre vuela** (nunca bloqueante).
- La regla completa en una frase: *"si explotas, pagas taller; y lo que traigas encima se cae según la zona".*
- Palanca de emergencia: si la participación PvP cayera por miedo al taller, el primer ajuste es reducir el % de desgaste en muertes PvP (un número en BD, no un rediseño).
- Calibración del gradiente: si nadie pisa la zona alta, 100%→75%; si nadie va a base, subir presión.
- *Implementación pendiente: campo de durabilidad por item equipado (hoy el equipo vive en JSON de `player_equipment`) — sistema nuevo de tamaño moderado.*

## 8. Pods (cajas bonus) ✅

- **Los pods NO sueltan materiales**: las materias primas salen **única y exclusivamente de las cajas de carga de los aliens** — la caja de carga recupera su relevancia clásica (matar → recoger → acarrear → Black Box en juego).
- Rendimiento **decreciente por día** (las primeras ~200 pagan bien, luego cae): tope de inflación y anti-bot en el mismo mecanismo.
- La License **duplica** su contenido (2× de un goteo acotado sigue acotado: compra tiempo, no techo).
- Pods más ricos en zonas disputadas: geografía de riesgo-recompensa.
- **Los pods sueltan Flux** — la energía del Materializador (§13) — y esa es su carga especial única. El loop completo: *volar → Flux → construir Eclipses → consumibles*. Los consumibles (Cargas, Fuel, munición media) ya no se regalan directo: se fabrican construyendo. Heredero del palladium→energía del retail, sin la zona-purgatorio.
- Cero problema regulatorio: recogibles gratuitos, lo opuesto de una loot box.

## 9. Mapa de grifos y sumideros (resumen)

| Flujo | Grifos (crean valor) | Sumideros (lo destruyen) |
|---|---|---|
| **Credits** | Kills de aliens · venta al NPC | Tienda de items · munición x1 · **reparación de equipo (toda muerte — el sink perpetuo)** · comisión de subasta · 10% de re-listado del Starbond |
| **Materiales** | **Cajas de carga de aliens — única fuente de crudos** (sesgado por zona) · **Tachyon de Ubers y eventos** | Recubrimientos consumidos · (transferencia vía Black Box no destruye) |
| **Consumibles** | Pods → **Flux** → Materializador → Cargas, Fuel, CEL-2/3, DRN-1 · Eclipses → CEL-4, OVC-1 | Usos del kit activo · munición · combustible PET (fase 2) · Flux quemado al construir |
| **Dinero real** | Starbond | → License + cosméticos, nunca poder |

## 10. Items legendarios ✅

**Mecanismo unificado: planos.** Todos los legendarios se arman con planos coleccionables, **comerciables** y de cuenta fija — la mejor mecánica del retail (las partes de Apis/Zeus: progreso visible) sin lo tóxico (el azar opaco). Cada clear paga planos **garantizados**; opcionalmente un drop directo del item completo con probabilidad **publicada** (~5%). El pity no es un parche: es el sistema.

**Nomenclatura propia** — regla de familia: los legendarios llevan nombres de los fenómenos más extremos del universo (los materiales son elementos hipotéticos; los legendarios, fenómenos límite). Nombres propios: no se traducen en ninguna localización.

| Item | Tipo | Fuente exclusiva | Planos | Identidad | Ref. DarkOrbit (implementación) |
|---|---|---|---|---|---|
| **Quasar** | Láser legendario | Incursiones (oleadas PvE grupales; entrada con Jump Core — esto activa el material en reserva) | 10 | La emisión más potente del universo: el arma se gana en la guerra PvE | LF-4 · `equipment_weapon_laser_lf-4` |
| **Nova** | Diseño de dron ofensivo | PvP: **la Arena** (§14), objetivos disputados, rango de temporada | 8 por dron | La explosión estelar: quien lo porta, peleó contra gente | Havoc · `drone_designs_havoc` |
| **Magnetar** | Diseño de dron defensivo | **La Escolta y El Asedio** (§15) — contribución defensiva | 8 por dron | El campo magnético más fuerte que existe = el escudo, literal — simetría con Nova | Hercules · `drone_designs_hercules` |
| **Pulsar** | Dron élite | World bosses (Ubers, Cubikon-análogo — contenido existente) | 30 partes | El faro de precisión: el cazador de gigantes | Apis · `drone_apis` |
| **Singularity** | Dron ápice | Solo la versión más difícil de CADA contenido | 30 partes | Donde todo converge: el completista — nuestro "Kronos" bien hecho | Zeus · `drone_zeus` |

Reglas:
- **El endgame es un menú, no una rutina**: cinco actividades distintas, corren en paralelo. Cada élite cuenta cómo juega con lo que porta.
- Planos por **participación en grupo** (no por golpe final).
- Progreso siempre visible en UI ("Planos LF-4: 7/10").
- Como los planos son comerciables, la economía de productores llega al nivel de pieza: el corredor de contenido vende planos por Credits en subasta — el circuito del Starbond funcionando en cada eslabón.
- Números provisionales (recalibrados por las temporadas, §18): incursión ≈ 45–60 min; **el set élite completo de la temporada se alcanza en ~4.5–5 meses jugando casual y en ~2–2.5 meses como tryhard** — la pasada de balance ajusta planos/clear y drops a estos objetivos.

## 11. Las tiers no legendarias ✅

| Tier | Origen | Tiempo objetivo |
|---|---|---|
| **T0 — Inicio** | Tienda NPC trivial + primeros gratis por tutorial | Día 1 |
| **T1 — Temprano** | Tienda NPC barata | Semana 1 |
| **T2 — Midgame** | **Solo tienda NPC**, precios con peso — el grind honesto clásico | Días–semanas por pieza |
| **T3 — Competitivo** | **Origen dual**: drops comerciables de contenido superior + tienda NPC a precio ancla | Semanas–meses el set |
| **T4 — Legendario** | Planos (§10) | El set de la temporada: ~4.5–5 meses casual / ~2–2.5 tryhard (§18) |

### Roster completo por tier (nombres DO de referencia; nomenclatura propia pendiente)

**T0 — Inicio (día 1):**
- Láser: **LF-1** · Escudo: **SG3N-A01** · Motor: **G3N-1010**
- Dron: **1er Flax** · Repbot: **REP-S** (gratis)
- Munición: **LCB-10** (x1) · Cohete: **R-310**
- Nave: **Phoenix** (gratis)
- Cosméticos: fuegos artificiales (FWX-S/M/L, créditos triviales)

**T1 — Temprano (semana 1):**
- Láser: **MP-1** (sidegrade: perfil PvE/PvP distinto) · Escudo: **SG3N-A02** · Motores: **G3N-2010, G3N-3210**
- Repbot: **REP-1** (el radar de diplomacia pasa a ser UI gratuita del minimapa)
- Cohete: **PLT-2026** · Mina básica: **RB-02**
- Nave: **Lynx**

**T2 — Midgame (el grind honesto):**
- Láser: **LF-2** · Escudos: **SG3N-A03** (eje absorción) / **SG3N-B01 rediseñado** (eje pool — deja de ser señuelo) · Motor: **G3N-3310**
- Drones: **Flax 2–8** (escalera de créditos completa)
- Repbot: **REP-2** · Lanzacohetes: **HST-1** · CPUs de conveniencia: **TOR-A** (auto-cohetes), **SV-A** (auto-lanzamisiles)
- Munición: **MCB-25** (x2) · Cohete: **PLT-2021** · Munición de lanzacohetes: **CBR**
- Minas: **ACM-01** (área), **SABM-01** (escudo)
- Formaciones básicas (8): Turtle, Arrow, Star, Pincer, Double Arrow, Chevron, Barrage, Bat
- Naves: **Taurus, Perseus**

**T3 — Competitivo (origen dual):**
- Láser: **LF-3** · Escudo: **SG3N-B02** · Motores: **G3N-6900, G3N-7900**
- Drones: **Iris ×8** (escalera)
- Repbots: **REP-3, REP-4** · Lanzacohetes: **HST-2**
- Extras de combate — **el kit activo, 4 piezas, alimentadas por Cargas que caen de pods**: **ISH-01** (escudo instantáneo), **SMB-01** (bomba), **CL04K** (cloak único consolidado), **AJP-01** (salto avanzado: cooldown + carga)
- Munición: **MCB-50** (x3), **SAB-50** (roba-escudo) · Cohete: **PLT-3030** · Lanzacohetes: **UBR-100**
- Municiones especiales (el kit táctico, todas con contrajuego): **EMP-01** (el contra del cloak), **DCR-250** (freno), **PLD-8** (debuff), **WIZ-X**, **R-IC3** (congelante)
- Minas avanzadas: **DDM-01** (daño directo), **EMPM-01** (EMP)
- Formaciones avanzadas (5): Lance, Diamond, Moth, Crab, Heart
- Naves: **Aquila, Cygnus, Ursa** (el trío de rol) y **Pegasus, Orion** — las dos metas grandes de créditos del juego

**T4 — Legendario (§10):** Quasar, Nova, Magnetar, Pulsar, Singularity — más la **munición-premio de contenido** (comerciable, jamás en tienda): **UCB-100** (x4), **RSB-75** (ráfaga lenta), **CBO-100**, **HSTRM-01** (lanzacohetes élite).

### Nomenclatura propia de las tiers no legendarias

Tercera familia de nombres (materiales = elementos hipotéticos; legendarios = fenómenos extremos; equipo regular = **códigos funcionales**): la regla es *el código dice lo que hace* — raíces cortas universales + número de marca. Los drones usan fenómenos cósmicos *menores* (la línea completa de drones queda ordenada por magnitud cósmica: Comet → Meteor → Pulsar → Singularity).

| MexOrbit (sugerido) | Tier | Categoría | Equivalente DarkOrbit |
|---|---|---|---|
| **PB-1** | T0 | Láser | LF-1 |
| **DR-1** (Dual Ray — perfil PvE/PvP) | T1 | Láser sidegrade | MP-1 |
| **PB-2** | T2 | Láser | LF-2 |
| **PB-3** | T3 | Láser | LF-3 |
| **DF-A1 / DF-A2 / DF-A3** | T0/T1/T2 | Escudos eje absorción | SG3N-A01 / A02 / A03 |
| **DF-B1 / DF-B2** | T2/T3 | Escudos eje pool | SG3N-B01 / B02 |
| **ION-1 … ION-6** | T0→T3 | Motores | G3N-1010 / 2010 / 3210 / 3310 / 6900 / 7900 |
| **NAN-0 … NAN-4** | T0→T3 | Repbots (nanobots) | REP-S / REP-1…4 |
| **Comet** (×8) | T0–T2 | Dron de créditos | Flax |
| **Meteor** (×8) | T3 | Dron élite | Iris |
| **SV-1 / SV-2** | T2/T3 | Lanzacohetes (salvo) | HST-1 / HST-2 |
| **CEL-1 / CEL-2 / CEL-3** | T0/T2/T3 | Munición láser x1/x2/x3 (celdas) | LCB-10 / MCB-25 / MCB-50 |
| **CEL-4** · **OVC-1** | T4 premio | Munición de contenido: x4 (las 3 puertas solitarias) / ráfaga (Beta y Gamma) | UCB-100 / RSB-75 |
| ~~CEL-X~~ | reserva | Descartada por ahora (posible munición de evento) | CBO-100 |
| **DRN-1** | T3 | Munición roba-escudo | SAB-50 |
| **EMP-1** | T3 | Especial: anti-cloak/anti-CPU | EMP-01 |
| **STS-1 / STS-2** | T3 | Especiales: estasis (freno) | DCR-250 / WIZ-X |
| **JAM-1** | T3 | Especial: debuff de daño | PLD-8 |
| **CRY-1** | T3 | Cohete congelante | R-IC3 |
| **TOR-1 … TOR-4** | T0→T3 | Cohetes (torpedos) | R-310 / PLT-2026 / PLT-2021 / PLT-3030 |
| **SVR-1 / SVR-2** | T2/T3 | Munición de lanzacohetes | CBR / UBR-100 |
| **SVR-X** | T4 premio | Munición élite de lanzacohetes | HSTRM-01 |
| **MIN-1 / MIN-2 / MIN-3** | T1/T2/T3 | Minas (básica / área / daño directo) | RB-02 / ACM-01 / DDM-01 |
| **MIN-D / MIN-E** | T2/T3 | Minas tácticas (drena escudo / EMP) | SABM-01 / EMPM-01 |
| **SHELL-1** | T3 | CPU activo: escudo instantáneo (consume SHELL-C) | ISH-01 |
| **SHOCK-1** | T3 | CPU activo: bomba inteligente (consume SHOCK-C) | SMB-01 |
| **VEIL-1** | T3 | CPU activo: cloak único (consume VEIL-C) | CL04K — consolidado de XS/M/XL |
| **WARP-X** | T3 | CPU activo: salto avanzado (cooldown + consume WARP-C) | AJP-01 — JP-01/JP-02 eliminados |
| **TOR-A** | T2 | CPU de conveniencia: auto-lanzado de cohetes | AROL-X |
| **SV-A** | T2 | CPU de conveniencia: auto-lanzamisiles | RLLB-X |
| **SHELL-C / SHOCK-C / VEIL-C / WARP-C** | Consumible | **Cargas** de los CPUs activos (familia "-C") — subproducto del **Materializador**, comerciables | — (nuevos) |
| **Fuel** | Consumible (fase 2) | Combustible del PET — subproducto del Materializador; se activa cuando el PET entre | pet-fuel |
| **Flux** | Consumible | Energía del Materializador — cae **solo de pods**, comerciable | palladium / Extra Energy |
| **PYRO-S/M/L** | T0 | Cosméticos (pirotecnia) | FWX-S/M/L |
| Formaciones: **se conservan los nombres geométricos** (Turtle, Arrow, Moth…) — son descriptivos de la forma, se localizan por idioma | T2/T3 | Formaciones | las 13 |

### Nomenclatura de naves — la familia de las constelaciones

Cuarta familia: **las naves son constelaciones** (latín astronómico: idéntico en todos los idiomas, simbolismo gratis). Roster según la lluvia de ideas (tier alto de 5 + tier bajo de 3 + inicial); orden de precio del tier alto: Orion > Pegasus > Ursa > Cygnus > Aquila; del bajo: Perseus > Taurus > Lynx.

Stats de referencia heredados de nuestra BD (los clásicos de la era; balance final pendiente en el documento de números):

| MexOrbit | Rol | HP | Láseres | Gens | Vel. | Habilidades | Equivalente DO (shipID) |
|---|---|---|---|---|---|---|---|
| **Phoenix** | Inicial gratuita — el ave que renace (la nave del respawn) | ~4,000 | 1 | 1 | 320 | — | Phoenix (1) |
| **Lynx** | Tier bajo — especialista de mapas bajos | 64,000 | 6 | 6 | 360 | **Pulso rastreador** (revela pods, cajas y camuflados en radio; cooldown largo — preview del Aquila) + ventaja territorial: +velocidad y +evasión **solo en los mapas bajos de su facción** | Leonov (3) |
| **Taurus** | Tier bajo — tanque/carga | 64,000 | 8 | 10 | 240 | **Blindaje** (reduce el daño recibido unos segundos — preview de la Ursa) | BigBoy (9) |
| **Perseus** | Tope del tier bajo — el héroe en ascenso | **90,000** | **9** | **9** | 320 | **Sobrecarga** (breve ráfaga de +daño — preview de las naves de batalla) | Nostromo (7) |
| **Aquila** | Rol: reconocimiento/marcador | 100,000 | 5 | 12 | 370 | Kit de 4: **Ultimate Cloak, Target Marker, Jam-X, Double Minimap** (0/4 implementadas hoy) | Spearhead (70) |
| **Cygnus** | Rol: sanador/soporte | 275,000 | 10 | 15 | 300 | Kit de 5: **HP Repair, Shield Repair, Repair Pod** (3/3 implementadas) + **Evasión** y **PEM** (elegidas; el escudo instantáneo se descartó por duplicar el SHELL-1 que cualquiera puede equipar) | Aegis (49) |
| **Ursa** | Rol: tanque/warp | 550,000 | 7 | 20 | 240 | Kit de 4: **Draw Fire** (implementada), **Fortify, Protection, Travel Mode** (1/4 hoy) | Citadel (69) |
| **Pegasus** | Batalla: velocidad | 180,000 | 10 | 10 | 380 | **Ninguna — batalla pura** (lluvia de ideas) | Vengeance (8) |
| **Orion** | Batalla: poder de fuego — el buque insignia; calco mitológico de Goliath (ambos gigantes) | 256,000 | 15 | 15 | 300 | **Ninguna — batalla pura** | Goliath (10) |

**Roster confirmado: 9 naves.** Yamato, Liberator, Piranha y Defcom quedan **fuera del roster jugable** (su arte es reciclable como NPCs o naves de evento; sus nombres de constelación — Lyra, Corvus, Dorado, Scutum — quedan reservados). **Al lanzamiento las naves no tienen variantes de diseño**: los sprites Veteran/Elite/Super Elite no entran; las skins cosméticas (renombradas, sin "Elite") serán tema de una fase posterior de cosméticos.

**Compensación de las naves de batalla (formalizada):** Orion y Pegasus no llevan bonus inventado — su compensación es estructural: los mayores slots de láser del juego (15/10), extras 3, y son **el único rol que puede duplicarse en la composición de incursión** (2 de 5 plazas = el doble de demanda que cualquier otro rol).

**Slots (decisión cerrada):** todos los slots son **fijos por nave** — la nave viene completa y los SLE desaparecen. Slots de **extras** escalonados por tier: tier bajo **1** · naves de rol **2** · Orion/Pegasus **3**. Con 6 CPUs en el juego y solo 1–3 ranuras, el loadout de extras es una decisión de build real por pelea. Legibilidad PvP: ves la nave, sabes qué puede ser. El único espacio ampliable del juego es el **almacén** (§4 — organización, jamás combate).

Notas de la lluvia de ideas integradas:
- **Las naves de batalla (Orion/Pegasus) no llevan habilidades**: su trabajo es el daño. Pendiente definir qué las compensa frente a las naves de rol (¿más slots, más daño base, más HP?) — hoy la compensación implícita es que Orion tiene 15/15 slots (los máximos del juego) y Pegasus la velocidad tope.
- **Las naves pequeñas también tendrán habilidad propia** (una por nave, por definir): identidad desde el tier bajo, suaviza la brecha con el tier alto.
- **La composición de instancia usa este roster**: 1 Cygnus + 1 Ursa + 1 Aquila + 2 de batalla (Orion/Pegasus en cualquier combinación) — los roles no son adorno: son la llave del contenido endgame.
- Estado de implementación de los kits: ver `MexOrbit.Server/Docs/pendientes-naves-2012.md` (Cygnus completa, Ursa 1/4, Aquila 0/4, y variantes Veteran/Elite con datos rotos en BD).

Reservados por si el roster conserva las 4 naves restantes del catálogo: **Lyra** (Yamato), **Corvus** (Liberator), **Dorado** (Piranha — constelación real de un pez), **Scutum** (Defcom — constelación real del *escudo*). La dirección de la lluvia de ideas sugiere comprimir el roster a 9.

### Nomenclatura de aliens — la familia de la taxonomía

Quinta familia: **taxonomía de enjambre en latín oscuro** — raíces latinas amenazantes con morfología que enseña el tier sola (heredamos el único acierto del nombrado de BigPoint — Kristallin→Kristallon — con sistema propio). Reglas: raíz por especie · sufijo **-it/-in** = forma menor, **-on/-or/-ox** = forma mayor · escalera de amenaza: base → **Elite** (Boss, ×4) → **Titan** (Uber, ×8). Los piratas/5-x quedan fuera de esta fase.

⚠️ Nota de desambiguación: al adoptar "Elite" para los aliens, las variantes de nave del arte (Aegis *Veteran/Elite/Super Elite*, etc.) deberán renombrarse con otra palabra cuando se definan los diseños de nave, para que "Elite" signifique una sola cosa en el juego.

| MexOrbit | Raíz/lógica | Equivalente DO |
|---|---|---|
| **Vex** | *vexare*, molestar — la plaga inicial | Streuner |
| **Vexor** | Vex evolucionado | StreuneR |
| **Skarn** | roca dura (término geológico real) | Lordakia |
| **Skarnox** | forma mayor | Lordakium |
| **Ferox** | *ferox*, feroz | Saimon |
| **Mordax** | *mordax*, el que muerde | Mordon |
| **Vorax** | *vorax*, devorador | Devolarium |
| **Gravit** | pesado, forma menor | Sibelonit |
| **Gravon** | pesado, forma mayor | Sibelon |
| **Vitrin** | *vitrum*, cristal — forma menor | Kristallin |
| **Vitron** | cristal, forma mayor | Kristallon |
| **Custit** | *custos*, guardián — el escolta del Hexon | Protegit |
| **Hexon** | hexaedro = el cubo colosal | Cubikon |

Ejemplos de uso: "Vitron Elite" = Boss Kristallon · "Hexon Titan" = el Cubikon supremo · "farmeo Vitrins para Coronium". El Tachyon y los planos del Pulsar caen de los **Titans** (§3/§10).

### Transformados y eliminados (veredicto explícito del resto del catálogo)

| Grupo | Items | Veredicto |
|---|---|---|
| Auto-CPUs de tedio | NC-AGB/AWL/AWR/AWB/RRB, RB-X, FB-X, ALB-X, G3X-AMRY/CRGO (×4) | **Eliminados como items**: sus funciones (recompra automática, recarga, gestión) son **features gratis** del juego — el tedio se elimina, no se vende |
| Slot extenders | SLE-01…04 | **Eliminados (decisión cerrada)**: slots fijos por nave, extras 1/2/3 por tier — la capacidad es identidad de la nave, no producto |
| Boosters | los 16 (DMG/SHD/HP/REP/RES/SREG/HON/XP × B01/B02) | **Eliminados como renta**: renacen como los **AMPs crafteables** (§16) — las 9 familias con calidades y duración por tiempo de juego |
| Llaves y cofres | booty-key verde/roja/azul | **Eliminados para siempre**: sin lootboxes. El azar opaco no existe en MexOrbit |
| Consumibles de salto | jumpvoucher | Eliminado (AJP-01 pasa a cooldown) |
| Dron comercial | HMD-07 | **Transformado en ventaja de la License**: venta remota con penalización por zona (−15/−30/−50%) — ver §2. Deja de ser item |
| CPUs recortados (dictamen 24-ago) | AIM-01/02, ROK-T01, MIN-T01/02, DR-01/02, JP-01/02, RD-X, ALB-X, RB-X | **Eliminados.** Filosofía elegida: **crafteo y potenciación en vez de CPUs pasivos** — los stats se mejoran con recubrimientos/potenciadores crafteables, no con chips comprados. El radar es función gratuita del juego (confirmado). Sobreviven: el **kit activo de 4** (SHELL/SHOCK/VEIL/WARP-X) alimentado por **Cargas** (familia "-C", caen solo de pods), y los 2 CPUs de conveniencia de cohetes (**TOR-A**, **SV-A**). El minijuego de química queda reservado para los *potenciadores* (lluvia de ideas) |
| Variantes redundantes de munición | ECO-10 (×2), SAR-01/02, SHG-01/02, BDR-1211/1212, RB-E01/E02, SL-01, SLM-01, JOB-100 | **Consolidar**: una escalera clara por tipo; las variantes de evento se reservan como recompensas de temporada |
| Reliquias de evento | lottery-euro-2012, wordpuzzle (×12), logfile/deal | Eliminados (log disks se evalúan con el sistema de puntos de piloto, pendiente) |
| PET completo | P.E.T., aiprotocol (×13), petgear (×11), pet-fuel | **Diseño cerrado en §12** — implementación fase 2. Sobreviven 8 gears y 4 protocolos de utilidad; eliminados g-tra1, g-pd1 y los 7 protocolos de combate |
| Módulos de base | los 10 CBS (defm, hulm, ltm-×3, ram-×2, repm, dmgm, honm) | **Diferidos a fase de clanes** — sin taxímetro cuando lleguen |
| Naves de rol | Spearhead, Aegis, Citadel | **En el roster T3** (Aquila/Cygnus/Ursa) con sus kits incluidos; sus variantes de arte NO entran al lanzamiento |
| Naves cortadas del roster | Yamato, Liberator, Piranha, Defcom | **Fuera del roster jugable** — arte reciclable (NPCs/eventos); nombres Lyra/Corvus/Dorado/Scutum reservados |
| Hangar | slot de hangar | Diferido (multi-nave: fase posterior) |

Reglas de la pirámide:
1. **Origen dual del T3**: el precio NPC es el **techo** del mercado (anti-especulación en server chico); los drops crean oferta por debajo → la ruta divertida (contenido) siempre es más barata que la aburrida (solo créditos). Ambas existen.
2. **T2 puro-tienda a propósito**: la acumulación simple tiene su lugar — la memoria feliz del "farmeé una semana para mi LF-2".
3. **Cada tier con decisión interna** (MP-1 sidegrade; escudos A=absorción vs B=pool; orden de extras en T3) — contra el diagnóstico retail de "40 items, 5 decisiones".
4. **Todo comerciable usado**: segunda mano en subasta = catch-up natural del novato tardío; la durabilidad convive (reparar antes de vender).

**Munición:** x1 tienda barata (farmear siempre rentable) · x2 tienda cara (equilibrio) · x3 tienda muy cara (burst PvP, costo neto) · **x4 solo de contenido** (las incursiones la pagan a volumen, comerciable, jamás en tienda). Cohetes/minas: misma lógica. **Formaciones:** tienda accesible (son decisiones, no poder) + 2-3 de logros. **Naves:** escalera de créditos hasta Vengeance y Goliath (las dos metas grandes de créditos, ambas jugando); el trío de rol depende de la decisión pendiente de verbos.

## 12. El P.E.T. ✅ (diseño cerrado; implementación en fase 2)

El anti-diseño de referencia es el §12 del doc retail (CAPEX+OPEX+seguro). El nuestro:

- **Compra única en Credits** (T3). Sin niveles comprados, sin slots comprados.
- **Progresión solo jugando:** el PET recibe un % de la XP del dueño y sube de nivel; **los slots aumentan únicamente con el nivel** — coherente con la decisión de slots (§11): la capacidad jamás se vende.
- **Fuel: subproducto del Materializador** (§13) — se acumula desde ya; sin costo de tienda, sin renta.
- **Reparación gratuita**: el PET muere y vuelve, sin factura especial.
- **Utilidad pura — el PET no toca los stats de la nave, ni la nave los del PET.**

**Gears que sobreviven (los verbos del PET):**

| Gear | `loot_id` | Función |
|---|---|---|
| Auto-Looter | g-al1 | Recoge cajas/carga en radio |
| Auto-Resource | g-ar1 | Recolecta recursos |
| Localizador de enemigos | g-el1 | Señala NPCs |
| Localizador de recursos | g-rl1 (consolida a g-ex1) | Marca recursos en minimapa |
| P.E.T. Repairer | g-rep1 | Regenera al PET |
| Kamikaze | g-kk1 | El PET se inmola (verbo del PET, no stat de nave) |
| Guard Mode | cgm-02 | Desvía daño hacia el PET |
| Ship Repair | csr-02 | Repara la nave del dueño en vuelo |

**Eliminados:** Cargo Trader (g-tra1 — el conflicto con la venta remota se resuelve **a favor de la License**), g-pd1 (función indocumentada).

**Protocolos: sobreviven solo los de utilidad** — Cargo (ai-cr1), Radar (ai-r1), Salvage (ai-s1), Economy (ai-eco1). **Eliminados los de combate/stats**: ai-aim1, ai-lm1, ai-sm1, ai-hp1, ai-e1, ai-al1, ai-pm1 — el principio es el mismo que mató a los AIM: la matemática de combate no se compra por chip.

## 13. Incursiones — el PvE instanciado ✅ (diseño; contenidos concretos por producir)

Dos ramas (de la lluvia de ideas, unificadas con el sistema de planos de §10):

**A. Los Eclipses — Penumbra, Umbra, Antumbra (las puertas solitarias):**

El sistema hereda a las Galaxy Gates y se llama **los Eclipses**: sus tres puertas son las regiones reales de la sombra de un eclipse, en escalera de profundidad.

| | **Penumbra** | **Umbra** | **Antumbra** |
|---|---|---|---|
| Lore | La sombra parcial | La sombra total | Lo que hay más allá de la sombra total |
| Dificultad | Entrada (T2) | Media (T3) | Alta (T3 recubierto) |
| Recompensas | **CEL-4** + insumos de potenciadores básicos | **CEL-4 + OVC-1** + insumos medios + materiales | **CEL-4 + OVC-1** + insumos altos |

- Las tres pagan **CEL-4** (x4); Umbra y Antumbra añaden **OVC-1**. **CEL-X descartada por ahora** (reserva de eventos).
- **Sin planos legendarios** (exclusivos del grupo): el Eclipse abastece el *consumo*; la incursión grupal construye la *leyenda*. El jugador sin clan tiene carril digno.
- **El Materializador:** los Eclipses se *construyen* aquí, alimentado con **Flux** (que cae de pods, §8). Como subproducto de la construcción dispensa **CEL-2, CEL-3, DRN-1, las cuatro Cargas (SHELL-C/SHOCK-C/VEIL-C/WARP-C) y Fuel** — los consumibles no se regalan: se fabrican construyendo, y el loop de Eclipses queda incentivado de punta a punta. Es el generador de puertas del retail (67% munición por giro) vuelto **determinista y sin casino**.
- Límite diario de corridas (2–3 por puerta) — control de inflación de consumibles.

**B. Incursiones grupales (la fuente del Quasar):**

- **Composición fija de 5**: 1 Cygnus + 1 Ursa + 1 Aquila + 2 de batalla (Orion/Pegasus).
- **Entrada: Jump Core** (activa el material de reserva: se junta en zona de riesgo, se quema al abrir).
- **Encuentros diseñados por rol**: zonas que solo el Aquila revela, NPCs que solo la Ursa sostiene, daño que solo el Cygnus mitiga. Si un rol no hace su trabajo, la incursión se complica o se pierde.
- Duración objetivo: 45–60 min.
- **Loot en dos capas** (síntesis §10 + lluvia):
  1. **Piso garantizado**: 1 plano personal por participante por clear (pity visible: "me faltan 3").
  2. **Botín mayor**: 2 drops legendarios por instancia, repartidos por **algoritmo ponderado por rol** con pesos rotativos (incentiva roles escasos) y **visibles antes de entrar** — plano extra, munición-premio, Tachyon, o item completo (5% publicado).
- **Temporadas**: 3 incursiones por temporada, temporadas de **6 meses** (§18); las viejas rotan a un banco con loot reducido (sin planos de temporada).

**Las tres incursiones de la Temporada 1** — familia de nombres: *dominios* en latín oscuro (extensible por temporada: Sepulcrum, Abyssus, Vorago…). Cada incursión tiene un **rol protagonista** (que el algoritmo de loot pondera por defecto), un checkpoint insustituible por rol, y culmina en un **Imperator** — la cúspide formalizada de la taxonomía: base → Elite → Titan → **Imperator** (único de su especie, solo como jefe final de incursión).

| Incursión | Dominio | Protagonista | Mecánica central | Jefe final |
|---|---|---|---|---|
| **NIDUS** | La colmena Vex | Naves de batalla | Oleadas de enjambre con generadores camuflados (solo el Aquila los revela); DPS check contra reloj — un generador vivo duplica la siguiente oleada | **Vexor Imperator** |
| **FAUCES** | La garganta de los devoradores | Ursa (+Cygnus) | Corredores con Vorax/Gravon Elite que solo la Ursa puede sostener (Draw Fire por diseño); Cygnus mitiga daño inevitable; ventanas de ejecución marcadas por el Aquila | **Vorax Imperator** |
| **VITRUM** | El laberinto de cristal | Aquila | Enemigos que reflejan daño salvo en ángulos que el marcado del Aquila expone; rutas falsas sin reconocimiento = emboscada; disparar sin marca te daña | **Vitron Imperator** |

- Entrada: **1 Jump Core por tripulante**.
- Estructura: 2 zonas + miniboss + Imperator · 45–60 min · recompensas según el sistema de dos capas de esta sección.

El **TDM** (lluvia de ideas: dos complejidades, inscripción en Credits, espejo PvP y fuente de los Nova) queda con diseño propio pendiente.

## 14. La Arena (TDM) ✅

El PvP instanciado — espejo de las incursiones y fuente de los **Nova**. Nombre: **la Arena** (latín puro, idéntico en ES/EN), con dos brackets cuyo eco es el de las constelaciones (Ursa Minor/Major):

| | **Arena Minor** | **Arena Major** |
|---|---|---|
| Naves | Las pequeñas: Lynx, Taurus, Perseus | Las T3: trío de rol + Pegasus/Orion |
| Formato | **3v3** | **5v5** |
| Recompensas | **Items T3** (drops comerciables — una de las fuentes del origen dual del T3) | **Planos Nova** + munición-premio + Tachyon (la mesa legendaria PvP de §10) |

Reglas:
1. **Separación por tier de nave**: PvP justo por construcción; las naves pequeñas ganan su propia escena competitiva (el meta de Sobrecarga/Blindaje/Pulso rastreador).
2. **Inscripción en Credits = sumidero puro** (la casa se la queda): filtro anti-partidas-vacías y anti-granjeo de alts. Major cuesta varias veces la Minor.
3. **Partidas de 10–15 min** — caben en la sesión de 20 minutos del contrato de tiempo.
4. **Los 2 items por partida van al equipo ganador, repartidos por contribución** (daño + asistencias + objetivos, no solo kills). El perdedor gana progreso menor de rango. Pity visible vía rango de temporada.
5. **Anti-abuso**: límite diario de partidas premiadas · recompensas degradadas contra oponentes repetidos · la inscripción encarece el win-trading · rango con revisión de anomalías.
6. **Munición y desgaste reales** (regla uniforme §7): la Arena es el mayor consumidor de munición del juego — sink natural.
7. **Rango de temporada** (mismo calendario semestral que las incursiones, §18): hitos de rango pagan planos Nova adicionales y la cosmética de temporada futura.

## 15. Los eventos de defensa ✅

La fuente del **Magnetar** y la herramienta de densidad del server: eventos de cita colectiva en mundo abierto.

| | **La Escolta** (convoy) | **El Asedio** (estación) |
|---|---|---|
| Qué pasa | Caravana NPC cruza mapas (baja→alta) escoltada contra oleadas de asalto | Oleadas alien atacan una estación de facción (las bases existentes) |
| Éxito | % de carga del convoy que sobrevive | % de estructuras salvadas |

Reglas confirmadas:
1. **Calendario publicado**: 2–3 eventos diarios de 20–30 min en horarios rotativos. El evento programado concentra la población — el evento ES el prime time del server.
2. **Participación abierta** (sin composición requerida), pero los **planos Magnetar se ganan por contribución DEFENSIVA** (daño absorbido junto al objetivo, curación, derribos cerca del convoy) — el scoring favorece por diseño a Ursa/Cygnus/Taurus: el Magnetar se gana *defendiendo*, simetría exacta con el Nova.
3. **Umbral de contribución anti-AFK**; recompensas escalan con el éxito (defensa perfecta = plano + bonus; parcial = proporcional; fallo = solo progreso de rango).
4. **Sin castigo al mapa por fallar** — castigaría a los desconectados (el anti-patrón FOMO de Lost Ark). El costo de fallar es no ganar.
5. **El atacante tiene cara**: cada evento lo lidera un **Titan** de la especie agresora, anunciado en el calendario ("20:00 — Escolta contra Mordax"). Los Titans como villanos recurrentes.
6. **PvPvE en zona disputada** (facciones rivales asaltan el convoy y saquean su carga como Black Box): **diseñado, activación en fase 2** — cuando se conozca la distribución real de facciones.
7. Aritmética del set: 8 planos × 10 drones = 80; ~1 plano por defensa exitosa × 2–3 eventos diarios → set completo en ~2–3 meses para el defensor constante — dentro de la ventana casual de la temporada (§18).

## 16. El Módulo de Crafteo — los AMPs ✅

El heredero del "minijuego de química" de la lluvia de ideas, montado sobre la **infraestructura de assembly que ya existe** (cliente: `assembly_window.gd` contra `/v1/assembly`; server: `assembly_recipe`) como categoría nueva de recetas — implementación barata.

**El producto — los AMPs (amplificadores), las 9 familias** (los 16 boosters del retail renacen crafteables):

| AMP | Efecto | Heredero de |
|---|---|---|
| **AMP-DMG** | +% daño | DMG-B01/02 |
| **AMP-SHD** | +% escudo | SHD-B01/02 |
| **AMP-HP** | +% vida | HP-B01/02 |
| **AMP-REP** | +% reparación | REP-B01/02 |
| **AMP-SREG** | +% regeneración de escudo | SREG-B01/02 |
| **AMP-XP** | +% experiencia | XP-B01/02 |
| **AMP-HON** | +% honor | HON-B01/02 |
| **AMP-LOOT** | +% botín (carga y pods) | RES-B01/02 |
| **AMP-CRG** | +% capacidad de bodega | — (nuevo: el amigo del carguero) |

Reglas confirmadas:
1. **Insumos — la familia de partículas** (caen de los Eclipses): **Ion** (Penumbra) · **Photon** (Umbra) · **Boson** (Antumbra). Receta tipo: `partículas + Aurorium + Credits → 1 AMP` — toca tres economías en un crafteo.
2. **Calidades I/II/III + factor suerte**: la mezcla de partículas define las probabilidades de calidad (publicadas); el AMP rolea su % final en un rango min–max según calidad.
3. **Crafteo instantáneo en base** — sin timers, coherente con "cero idle".
4. **Duración 5–10 h de tiempo de juego** al activarse (según tipo/calidad); **sin caducidad en inventario** (anti-FOMO): la demanda recurrente la garantiza el consumo.
5. **Todo comerciable** (partículas y AMPs): nace el químico como profesión.
6. La app móvil de la lluvia (crafteo/comercio remoto) queda para fase posterior.
7. ⚠️ Nota para el balanceo general: los AMPs de combate **apilan** con recubrimientos y formaciones — la pasada de números debe fijar el presupuesto total de % apilable (AMP + recubrimiento + formación) para que el techo de poder quede controlado.

## 17. El Mercado (subasta de órdenes) ✅

El libro de órdenes que reemplaza a la subasta de pujas a ciegas — la infraestructura del puente, del comercio geográfico y del termómetro:

1. **Modelo Grand Exchange**: órdenes de compra Y de venta con emparejamiento por precio. Las órdenes de compra dan liquidez asíncrona — crítico con 200 jugadores en husos distintos: el minero de las 3 am le vende a la orden parada del élite dormido.
2. **Mercado global único, anclado al almacén** (que se hace explícito: **el almacén es único y global por jugador**, accesible desde cualquier base de su facción). Listar exige el item en almacén; lo comprado llega al almacén; lo vendido es saldo. Todo opera **offline**.
3. **Comisión de la casa** sobre venta ejecutada (% en la pasada de números; referencia 4–5%) — el sink del mercado. Sin tarifa de listado en v1. El Starbond conserva su régimen (un salto + 10% re-listado).
4. **Órdenes de 7 días**, retorno automático al expirar.
5. **Todo lo comerciable se comercia**; items con durabilidad se listan **solo a reparación completa**.
6. **Anti-manipulación por transparencia**: piso = venta NPC · techo T3 = tienda NPC · **historial de precios público** (promedio diario por item — el termómetro del jugador y el tablero del administrador) · límite de órdenes activas: **8 base, 16 con License** (conveniencia de organización — monetización permitida por las reglas).
7. **El Starbond vive aquí desde el día uno**: precio fijo por `server_settings` en fase administrada; soltar a mercado = quitar el candado. Un desarrollo, dos modos.
8. **Se retira el código de pujas a ciegas** (los `auction.*` actuales): era un modelo para multitudes; el nuestro necesita liquidez y confianza.

## 18. Las Temporadas ✅

El contenedor de todo el calendario del juego: **temporadas de 6 meses**.

**El invariante de convergencia, reformulado por temporada** (la regla de oro operativa):

> *Todo jugador **casual** alcanza el full de la temporada en ~4.5–5 meses (antes de que termine); el **tryhard** lo alcanza en ~2–2.5 meses.*

Nadie termina una temporada "atrasado por diseño". El tryhard llega antes y disfruta su cima 3–4 meses (estatus, Arena, mercado); el casual llega — siempre llega. La pasada de balance calibra todos los drops/planos a estos dos ritmos.

**El ciclo de cada temporada:**
- **Temporada 1 — full farmeo**: se lanza con el juego. La persecución es el set élite base (Quasar, Nova, Magnetar, Pulsar, Singularity) vía NIDUS/FAUCES/VITRUM, Arena, defensa y Titans. Sin items nuevos a mitad de temporada.
- **Temporada 2 en adelante**: cada temporada trae **3 incursiones grupales nuevas con mecánicas diferentes** y **novedad élite**: items élite nuevos **o** la posibilidad de **upgradear los élite existentes**.
- **Regla de la inversión (anti-BigPoint)**: la novedad élite jamás invalida lo ganado — **tu Quasar se mejora, no se tira**. Los upgrades son rutas sobre el item que ya posees; el poder previo es el prerequisito, no la basura de la temporada pasada. (El contraste exacto con el "15M de uridium que te reseteaba a noob" del retail.)
- Las incursiones viejas rotan al banco con loot reducido (§13); el rango de Arena y las cosméticas de temporada cierran y premian al final de cada ciclo.

## 19. Misiones ✅

El sistema ya tiene fases 1–3 implementadas y probadas; esto define su economía. **Tres capas, cada una con un trabajo:**

| Capa | Qué es | Paga |
|---|---|---|
| **Campaña/tutorial** (una vez) | Enseña los sistemas jugándolos: recolectar, refinar, vender, craftear, la primera Penumbra, pods, la Black Box | El equipo T0/T1 gratis (§11) + Credits semilla |
| **Diarias** (3–5 rotativas) | "Mata 20 Vex · junta 200 Asterium · corre 1 Eclipse · participa en 1 evento" | Credits + Flux + partículas menores — **el ingreso plano por día**, dial controlable |
| **Semanales/de temporada** | Metas mayores: las 3 incursiones, N Arenas | Planos bonus + cosmética de temporada |

**La regla anti-chore:** las misiones **nunca son la fuente principal de nada** — aceleran y *direccionan* hacia donde vive la economía real, y se completan jugando normal. Si el jugador siente que "tiene que hacer las diarias antes de jugar", fallamos.

## 20. El Perfil del Piloto ✅

La hoja de 25 skills con árbol de prerequisitos ya existe en BD; su economía se re-fundamenta (estaba preciada en uridium y log disks, ambos eliminados):

1. **Los puntos de piloto se ganan jugando, jamás se compran** — por hitos de XP/nivel de cuenta. Log disks eliminados definitivamente.
2. **Los niveles de skill cuestan Credits** — la hoja es **EL sink de Credits del endgame** (hereda su rol de mega-sumidero, en la moneda que se gana jugando). Progresión **permanente, cross-temporada**: la identidad del piloto no se resetea.
3. **Rework de skills muertas**: Luck I/II y las de cajas bonus se re-especifican hacia Flux y pods; el resto del árbol (HP, escudo, minas, honor, XP, cargo, reparación, Greed) sigue válido.
4. **Respec permitido con costo en Credits** — otro sink; equivocarse de build no es condena.

## 21. Pendientes (ejecución) 🔶

1. ~~Verbos~~ ✅ **CERRADA**: los skill designs del Goliath/Vengeance clásicos (Solace, Venom, Spectrum, Sentinel, Diminisher) **se descartan por ahora** — Orion y Pegasus son batalla pura (lluvia de ideas) y los verbos viven solo en los kits de las naves de rol (Cygnus/Ursa/Aquila) y las habilidades propias de las naves pequeñas. El código legacy de los 5 skill designs queda en el server sin uso, disponible si algún día se reevalúa.
2. ~~PET~~ ✅ **CERRADA**: diseño completo en §12; implementación en fase 2.
3. ~~Slots~~ ✅ **CERRADA**: slots fijos por nave; extras escalonados 1/2/3 por tier; SLE eliminados; el almacén es el único espacio ampliable (ver §11).
4. **Matriz de origen item×tier**: estructura CERRADA (§10 legendarios, §11 pirámide de tiers). Falta solo el detalle fino: asignación item-por-item del catálogo completo a su tier con precio, en el documento de números finales.
   - **Todo el contenido endgame está diseñado**: Incursiones ✅ (§13), Arena ✅ (§14), eventos de defensa ✅ (§15). Falta solo la producción (mapas, oleadas, stats de NPCs, arenas, rutas de convoy).
5. **Nomenclatura**: equipo ✅ · naves ✅ · aliens ✅ — las 5 familias completas. **Roster de 9 naves confirmado** (§11). Pendiente: rama pirata de la taxonomía (cuando 5-x entre) y familia de skins cosméticas (fase posterior).
6. **Balanceo general (el documento de números)**: precios, stats finales, drops, recompensas y tiempos — se hace en UNA pasada integral cuando todos los puntos de diseño estén definidos, validando contra el invariante de convergencia.
6. **Números finales**: precios de tienda, recompensas por alien, tiempos objetivo por etapa (validados contra el invariante), % de recubrimientos.
7. ~~Rediseño de subasta~~ ✅ **CERRADA**: el Mercado de órdenes (§17).
8. ~~Temporadas~~ ✅ **CERRADA**: sistema completo en §18 — 6 meses, invariante por temporada, novedad élite desde la T2 con regla de la inversión.
