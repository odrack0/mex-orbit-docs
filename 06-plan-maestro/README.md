# El Plan Maestro de implementación

El plan general de v1: **siete etapas, cada una con su meta comprobable (gate)**. De cada etapa nacen planes pequeños e iterables que se documentan en esta misma carpeta (`etapa-N-plan-X.md`) y se ejecutan en ciclos cortos con entregable observable.

> Reglas del plan: papel antes que código (el pilar se diseña antes de implementarse) · los Guidelines mandan · el invariante de convergencia se valida en cada etapa que toque economía · el prototipo legado (Azure) queda congelado como laboratorio de referencia.

## Las etapas

### E0 — Fundaciones ✅ (esta)
Biblia estructurada, 9 repos con propósito y fronteras, convenciones (español/inglés, mex-orbit, migraciones fechadas). **Gate: cumplido.**

### E1 — Contratos 🔄 (en progreso — [plan de etapa](etapa-1-contratos.md))
Los dos pilares fundacionales pasan de decisiones a diseño completo:
- Protocolo v1: catálogo de mensajes del slice, codegen elegido (spike protobuf vs propio), modelo de sincronización.
- Esquema BD v1: dominios del slice (cuentas, naves/equipo, mundo, materiales) → migración `.1` en `mex-orbit-data-base`.
- Diseño de auth (api emite, game server valida).

**Gate:** los tres documentos revisados + el contrato compilando en C# y GDScript + la migración inicial aplicable y reversible.

### E2 — El vertical slice
El corazón de la fase 1, a través de TODOS los repos:

> **login → conectar → volar → matar un Vex → recoger su carga → volver a base → refinado automático → almacén → vender al NPC**

Con: cliente de UI mínima nueva (login, HUD, almacén), game server con tick fijo y un mapa, api con cuentas, la migración de datos semilla, y el arte mínimo del slice (definido en la sesión de arte).

**Gate:** el slice jugable end-to-end en dev, con reconexión limpia y sin ninguna pieza legada en el camino.

### E3 — La economía circular
Lo que convierte el slice en economía: el Mercado de órdenes (api) · muerte completa (gradiente 0/50/100, Black Box, durabilidad y taller) · pods y Flux · tienda NPC T0–T2 con la pirámide de tiers · misiones de tutorial y diarias · el sesgo de drops por zona.

**Gate:** dos jugadores pueden comerciar, arriesgar carga, morir, reponerse y progresar — el loop económico completo vivo.

### E4 — El mundo del carril solitario
T3 y el contenido individual: zonas altas y world bosses (Titans, el Hexon) · el Materializador y los Eclipses (Penumbra/Umbra/Antumbra) · recubrimientos (Aurorium/Tachyon) · los AMPs y las partículas · perfil del piloto.

**Gate:** el jugador solitario tiene su semana completa (farmeo → Eclipses → craft → mejora → Titans) y el carril de consumibles circula.

### E5 — El juego en grupo
Los cinco legendarios obtenibles: incursiones (NIDUS primero, luego FAUCES y VITRUM) con la composición 1-1-1-2 y el loot de dos capas · la Arena Minor/Major · La Escolta y El Asedio · grupos y clanes base.

**Gate:** un grupo de 5 completa NIDUS; los planos de los 5 legendarios caen de sus fuentes; el algoritmo ponderado reparte.

### E6 — Temporada 1 y lanzamiento
- **La pasada de balanceo general** (el pendiente §21 del guideline): todos los números en una pasada integral, validados contra los dos ritmos (casual 4.5–5 meses / tryhard 2–2.5).
- Starbond y Star License en modo administrado; temporada 1 (full farmeo) configurada; consola con tablero y tuning operativos; infraestructura de producción (backups, monitoreo, TLS).
- Beta cerrada → ajustes → lanzamiento.

**Gate:** el invariante demostrado con datos de beta, no con fe.

## Transversales (corren junto a las etapas)

- **Arte** ([05-arte](../05-arte/README.md)): su plan propio está pendiente de la sesión de arte; alimenta a E2 (slice mínimo) y crece por etapa.
- **Consola**: crece con cada etapa (lo que cada etapa parametriza, la consola lo expone).
- **Infraestructura**: backups desde el primer entorno con datos; CI por repo desde E1.
- **Fase 2 (después de v1)**: PET · PvPvE de La Escolta · sistema pirata (5-x) · cosméticos/skins · CBS/clanes avanzados · app móvil.

## Cómo se usa este plan

1. Al arrancar una etapa, se escribe aquí su plan detallado (`etapa-N-*.md`): alcance, iteraciones, responsables de pilar.
2. Cada iteración termina en algo **observable** (demo, test, métrica) — nada de "avancé en…".
3. El plan maestro se ajusta con commit cuando la realidad enseñe algo — es mapa, no contrato de mármol.
