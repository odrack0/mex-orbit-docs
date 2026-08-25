# Etapa 1 — Contratos (plan de etapa)

**Estado: en progreso** (arrancada 2026-08-25). Papel antes que código: los dos pilares fundacionales y el auth pasan de decisiones a diseño completo y revisable.

## Alcance

Tres entregables, cada uno con documento propio en el repo que le corresponde:

| # | Entregable | Vive en | Gate propio |
|---|---|---|---|
| 1 | **Diseño del protocolo v1** | `mex-orbit-protocol/docs/protocolo-v1.md` | catálogo de mensajes del slice cerrado + decisión de codegen tomada + modelo de sincronización definido |
| 2 | **Esquema BD v1 (slice)** | `mex-orbit-data-base/docs/esquema-v1.md` + migración `2026.08.25.1/` | migración `rollout.sql` aplicable y `rollback.sql` reversible sobre MySQL limpio |
| 3 | **Diseño de auth** | `mex-orbit-api/docs/auth-v1.md` | flujo api-emite / game-server-valida cerrado, con sesión única y reconexión |

**Gate de la etapa** (del plan maestro): los tres documentos revisados por el usuario + el contrato compilando en C# y GDScript (spike de codegen) + la migración inicial aplicable y reversible.

## Insumos

- Catálogo real del prototipo: `MexOrbit.GodotClient/docs/protocol-spec.md` y `net/` — qué mensajes existen hoy y qué lecciones dejó (framing, sesiones zombi, validaciones ausentes).
- Esquema legado como lista negra: `02-investigacion/implementacion-legado/sql-schema.md`.
- Guidelines §1–§20: qué debe representar la BD; el slice E2 define el subconjunto v1.
- Decisiones ya tomadas en los pilares [01](../04-pilares/01-protocolo.md) y [02](../04-pilares/02-base-de-datos.md).

## Iteraciones

1. ✅ **I1 — Los tres documentos** (2026-08-25): los tres docs en sus repos, con push. Pendiente la revisión del usuario.
2. ✅ **I2 — Spike de codegen** (2026-08-25): ambas rutas alcanzaron roundtrip byte-exacto C#↔GDScript; **decisión: esquema propio**, porque rangos y rate limits viven en el contrato (inexpresables en protobuf). Evidencia reproducible en `mex-orbit-protocol/spike/README.md`.
3. **I3 — Migración viva**: aplicar `2026.08.25.1` al MySQL local de dev (`mexorbit_dev`), verificar rollout→rollback→rollout. Requiere credenciales del MySQL local del usuario. Entregable: registro de la corrida en el doc del esquema.

## Fuera de alcance (E3+)

Mercado de órdenes, Eclipses/Materializador, incursiones, temporadas, Starbond/License, perfil de piloto: la BD los lista como dominios futuros, el protocolo les reserva rangos de IDs, y nada más.
