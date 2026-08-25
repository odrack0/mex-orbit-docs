# Inventario de assets de v1

**Estado: CERRADO** con el dictamen Q1–Q8 (ver [pipeline](01-pipeline-ia.md)): top-down 1 frame · estilo vectorial (fuente SVG) · naves sin variantes de facción · Elite/Titan por tinte+escala+VFX · Imperators con arte único **renovados por temporada** · grafo de mapas heredado del prototipo (~17 fondos) · audio con IA · export a 1080p con soporte 1440 (el SVG hace el 4K futuro un re-export).

**Fuentes:** el censo real del prototipo (~330 piezas que un cliente funcional consume — `inventario-assets.md` del repo legado) cruzado con el roster de los Guidelines (§10–§11, naves, taxonomía, contenido §13–§15).

**Prioridades:** 🟥 P1 = vertical slice (E2) · 🟧 P2 = economía y carril solitario (E3–E4) · 🟨 P3 = grupo y lanzamiento (E5–E6).

---

## 1. Naves (9)

| Asset | Prioridad | Notas |
|---|---|---|
| Phoenix | 🟥 | La primera nave del slice |
| Lynx, Taurus, Perseus | 🟧 | Tier bajo |
| Aquila, Cygnus, Ursa | 🟨 | Trío de rol (siluetas MUY distintas entre sí: se leen en combate) |
| Pegasus, Orion | 🟨 | Las metas; Orion = la silueta más imponente del juego |

- Formato ✅: **1 frame top-down por nave, fuente SVG** — la rotación la hace el motor.
- Motores encendidos, estelas, daño acumulado: **procedural** (carril 4), no arte por nave.
- Facciones ✅: **las naves son idénticas para todas las facciones** — la identidad de facción vive en estaciones y fondos, no en los cascos.
- Sin variantes Veteran/Elite al lanzamiento (decidido, Guidelines §11).
- PET: fase 2 — fuera de este inventario.

## 2. Aliens (13 especies + 3 Imperators)

| Grupo | Piezas | Prioridad | Notas |
|---|---|---|---|
| Vex | 1 | 🟥 | El alien del slice |
| Vexor, Skarn, Ferox, Mordax | 4 | 🟧 | Zona baja/media |
| Skarnox, Vorax, Gravit, Gravon, Vitrin, Vitron, Custit | 7 | 🟧 | Media/alta |
| Hexon | 1 | 🟧 | El cubo colosal — landmark del mundo |
| **Elite / Titan por especie** ✅ | 0 arte nuevo | 🟧 | **Escala + tinte + VFX de corona** (shader compartido) |
| Imperators (T1: Vexor, Vorax, Vitron) ✅ | 3 **por temporada** | 🟨 | Arte único — los villanos del ciclo, renovados con cada trío de incursiones (§18) |
| Disparo firma por especie | ~6 texturas | 🟧 | Shader + textura (el prototipo usaba 5 sprites de disparo NPC) |

## 3. Drones y sus diseños

| Asset | Piezas | Prioridad |
|---|---|---|
| Comet (créditos) | 1 | 🟧 |
| Meteor (élite) | 1 | 🟨 |
| Pulsar, Singularity (legendarios) | 2 | 🟨 |
| Diseños Nova y Magnetar | 2 overlays/skins sobre dron | 🟨 |
| Formaciones (13) | 0 arte — son posiciones + 1 icono c/u (ver §6) | 🟨 |

## 4. El mundo

| Asset | Piezas | Prioridad | Notas |
|---|---|---|---|
| Fondos de mapa ✅ | ~17 (grafo heredado del prototipo: bajos, altos, PvP 4-x) | 🟥 1 → resto 🟧/🟨 | Por bioma con identidad de facción en zonas natales (Q3) |
| Tiles: campo estelar, nubes, asteroides | ~12 | 🟥 básico | Paralaje |
| Planetas y decoraciones | ~20–30 (prototipo: 53) | 🟧 | Sprites sueltos, la IA brilla aquí |
| Portal/jumpgate | 1 set (+ variante Eclipse) | 🟥 | Base, activo, vórtice — animación procedural/shader |
| Estaciones de facción ✅ | 1 estación modular **×3 facciones con identidad propia** | 🟥 (1) | El prototipo: 4 edificios ×3 facciones |
| **El Materializador** | 1 estructura | 🟧 | Edificio nuevo — no existe en DO: identidad propia |
| Pods (cápsulas) | 1 (+variante rica de zona alta) | 🟧 | |
| Caja de carga de alien | 1 | 🟥 | El loot del slice |
| **Black Box** | 1 | 🟧 | Pieza icónica del PvP — merece diseño memorable |
| Iconos de minimapa | ~15 (naves, aliens, Titans, portal, estación, pod, black box, convoy, evento) | 🟥 básico | 9×9–16×16, legibilidad extrema |
| Minimapas | 0 — se generan del editor de mapas | — | Modernización sobre el prototipo |

## 5. FX (filosofía procedural — ver carril 4 del pipeline)

| Asset | Piezas de arte reales | Prioridad |
|---|---|---|
| Láser: textura base por familia | 2–3 texturas (los tiers CEL-1..4/OVC = parámetros de shader) | 🟥 |
| Cohetes TOR-1..4 y SVR | 2 modelos + tintes | 🟧 |
| Minas MIN-* | 5 sprites pequeños | 🟨 |
| Explosiones, humo, chispas | 3–4 texturas de partícula (el movimiento es GPUParticles) | 🟥 |
| Escudo, impacto, insta-shield (SHELL) | shaders + 2 texturas | 🟥 |
| Cloak (VEIL), PEM, recubrimientos Aurorium/Tachyon (el glow del "taquionizado"), salto WARP | shaders | 🟧 |
| Rayo recolector, aro de selección, telegraphs de jefe (áreas de daño) | 3–4 | 🟥/🟨 |

## 6. Iconografía de items (~110 iconos, nombrados por código)

La lista exacta sale del roster §11 de los Guidelines:

| Familia | Iconos | Detalle |
|---|---|---|
| Láseres | 4 | PB-1/2/3, DR-1 |
| Escudos | 5 | DF-A1/A2/A3, DF-B1/B2 |
| Motores | 6 | ION-1…6 |
| Repbots | 5 | NAN-0…4 |
| Lanzacohetes | 2 | SV-1/2 |
| Munición láser | 6 | CEL-1/2/3/4, OVC-1, DRN-1 |
| Especiales | 5 | EMP-1, STS-1/2, JAM-1, CRY-1 |
| Cohetes | 7 | TOR-1…4, SVR-1/2/X |
| Minas | 5 | MIN-1/2/3/D/E |
| CPUs | 6 | SHELL-1, SHOCK-1, VEIL-1, WARP-X, TOR-A, SV-A |
| Cargas y energía | 6 | SHELL-C, SHOCK-C, VEIL-C, WARP-C, Fuel, **Flux** |
| Materiales | 5 | Asterium, Nebulium, Coronium, Aurorium, Tachyon |
| Partículas | 3 | Ion, Photon, Boson |
| AMPs | 9 (+badge de calidad I/II/III como overlay, no ×3 arte) | AMP-DMG…AMP-CRG |
| Planos legendarios | 5 (o 1 diseño + color por item) | Quasar, Nova, Magnetar, Pulsar, Singularity |
| Legendarios (item completo) | 5 | |
| Drones | 4 + 2 diseños | |
| Formaciones | 13 | Geometrías — casi se autogeneran |
| Naves (retratos de tienda/hangar) | 9 | |
| Monedas y metas | ~6 | Credits, Starbond, Star License, Jump Core, XP, honor |
| Sistema (durabilidad, reparación, muerte, buff/debuff…) | ~10 | |

## 7. UI (el design system — ver prototipo de UI, siguiente entregable)

| Grupo | Piezas | Prioridad |
|---|---|---|
| **Design system base**: paleta, tipografía (fuente licenciada), set nine-patch de paneles/ventanas, botones/inputs/tabs/sliders/scroll, tooltips, cursores | 1 theme de Godot | 🟥 |
| HUD: barras (HP/escudo), frame de objetivo, action bar + slots, minimapa frame, chat, indicadores de zona/riesgo | ~15 componentes | 🟥 |
| Ventanas (del prototipo + sistemas nuevos): login, hangar/equipo, **almacén**, tienda NPC, **Mercado de órdenes**, **Materializador/crafteo**, perfil de piloto, misiones, clan, grupo, mapa estelar, killscreen (con Black Box), opciones, **lobby de Eclipses/incursiones**, **cola de Arena**, **calendario de eventos**, **tracker de planos ("me faltan 3")**, notificaciones/toasts | ~18 layouts sobre el theme | 🟥 (5) → 🟨 |
| Iconos de sistema UI | ~30 | 🟥 básico |
| Pantalla de carga, splash (sin logo definitivo — nombre temporal) | 2 | 🟧 |

## 8. Audio ✅ (con IA: ElevenLabs para SFX, Suno para música)

| Grupo | Piezas (referencia: prototipo usó 55 SFX + 1 pista) |
|---|---|
| SFX combate (láser ×tiers, cohetes, explosiones, escudo, impactos) | ~20 |
| SFX mundo/UI (recolección, pod, refinado, mercado, ventanas, alertas, muerte, level-up) | ~25 |
| Música: menú, ambiente por bioma (~3), combate, incursión/Imperator, Arena | ~7 pistas |

---

## Totales (cerrados con el dictamen)

| Bloque | Piezas |
|---|---|
| 🟥 P1 — el vertical slice completo | **~45 piezas + theme base + 4 shaders + ~10 SFX** |
| v1 completo (E2→E6) | **~320–350 piezas + librería de ~12 shaders + ~45 SFX + ~7 pistas** |
| Por temporada (recurrente, desde T2) | 3 Imperators + arte de 3 incursiones + cosméticos del ciclo |

**Convención de nombres**: cada archivo se llama por su código del juego (`pb-1`, `vex`, `orion`, `flux`) — el catálogo JSON del cliente los referencia igual que los Guidelines. Un nombre, tres lugares: diseño, arte, código.
