# Los pilares de la reconstrucción

Un documento por pilar técnico. Cada uno registra **las decisiones ya tomadas** y **lo por definir** — y debe estar razonablemente completo antes de que su repo reciba código de verdad (papel antes que código).

| Pilar | Repo(s) | Estado |
|---|---|---|
| [01-protocolo.md](01-protocolo.md) | `mex-orbit-protocol` | Decisiones base tomadas; diseño de mensajes por hacer |
| [02-base-de-datos.md](02-base-de-datos.md) | `mex-orbit-data-base` | Decisiones base tomadas; esquema por diseñar |
| [03-game-server.md](03-game-server.md) | `mex-orbit-game-server` | Principios definidos; arquitectura por diseñar |
| [04-api.md](04-api.md) | `mex-orbit-api`, `mex-orbit-api-admin` | Fronteras definidas; diseño por hacer |
| [05-cliente.md](05-cliente.md) | `mex-orbit-client` | Herencia del prototipo definida; UI por diseñar |
| [06-consola.md](06-consola.md) | `mex-orbit-console` | Alcance definido |
| [07-infraestructura.md](07-infraestructura.md) | (transversal) | Deudas identificadas; por diseñar |

**Orden de dependencia**: protocolo y base de datos primero (todo se apoya en ellos) → game server y api → cliente → consola. Infraestructura y [arte](../05-arte/README.md) corren en paralelo.
