# mex-orbit-docs — La Biblia del proyecto

Todo el conocimiento, las decisiones y los planes de MexOrbit v1, **en español y versionado**. Si una decisión no está aquí con su commit, no existe.

> **MexOrbit** es nombre temporal del proyecto. En los repos de código: **código en inglés, comentarios en español**.

## Mapa de la Biblia

| Carpeta | Qué contiene | Cuándo se lee |
|---|---|---|
| **[01-vision/](01-vision/README.md)** | Qué es MexOrbit, las leyes del diseño (el invariante de convergencia y los anti-patrones prohibidos), y las convenciones del proyecto | Primero — y cada vez que una decisión se sienta dudosa |
| **[02-investigacion/](02-investigacion/README.md)** | La evidencia: la autopsia económica del DarkOrbit 2010, el estudio de MMOs vivos en 2026, la historia de los recursos, la decompilación del protocolo legado y los flujos del sistema anterior | Cuando alguien pregunte "¿y por qué así?" — aquí están los números |
| **[03-guidelines/](03-guidelines/README.md)** | **El documento rector**: los Guidelines generales del juego — 20 secciones de diseño cerrado, con su guía de lectura por temas | La referencia de trabajo diaria: todo pilar y todo plan se valida contra él |
| **[04-pilares/](04-pilares/README.md)** | Los documentos de diseño técnico de la reconstrucción: protocolo, base de datos, game server, api, cliente, consola, infraestructura — cada uno con sus decisiones tomadas y sus pendientes | Antes de escribir código de ese pilar |
| **[05-arte/](05-arte/README.md)** | **Espacio reservado**: cómo haremos el arte — pendiente de trabajarse | Próximamente |
| **[06-plan-maestro/](06-plan-maestro/README.md)** | El plan general de implementación por etapas, del que nacen los planes pequeños iterables | Para saber qué sigue y por qué |
| **[ideas/](ideas/lluvia-de-ideas.md)** | La lluvia de ideas — depósito crudo, sin filtro | Cuando caiga una idea |

## Las tres reglas de esta Biblia

1. **Papel antes que código**: cada pilar y cada sistema arranca con su documento aquí, como la economía arrancó con los guidelines.
2. **Una sola verdad**: los documentos se editan aquí y se commitean; no existen copias sueltas fuera de git.
3. **Nada contradice a los Guidelines sin que los Guidelines cambien primero** — con su commit.

## Repos del proyecto

`mex-orbit-protocol` (el contrato) · `mex-orbit-data-base` (esquema y migraciones) · `mex-orbit-game-server` (simulación) · `mex-orbit-api` (plataforma: cuentas, Mercado, almacén) · `mex-orbit-client` (Godot) · `mex-orbit-console` + `mex-orbit-api-admin` (administración) · `mex-orbit-art` (fuente de arte) · **este repo** (la Biblia).
