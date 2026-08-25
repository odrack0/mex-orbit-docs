# Pilar 01 — El protocolo

**Repo:** `mex-orbit-protocol` · **Estado:** **diseño v1 escrito en E1** → [`mex-orbit-protocol/docs/protocolo-v1.md`](https://github.com/odrack0/mex-orbit-protocol/blob/main/docs/protocolo-v1.md) (catálogo del slice ~35 mensajes, framing, sincronización, reconexión); pendiente el spike de codegen (iteración I2 de la etapa 1).

## Decisiones tomadas

1. **WebSocket sobre TLS (`wss://`) con payload binario** — framing estándar (adiós a la clase de bugs del framing artesanal), cifrado de fábrica (el protocolo plano fue la puerta de los bots del retail), soporte nativo en Godot 4, puerta abierta a cliente web.
2. **Una sola definición de mensajes → generación para C# y GDScript.** Nada se escribe dos veces.
3. **Versionado negociado en el handshake.**
4. **Anti-cheat como requisito estructural**: el cliente manda intenciones, jamás resultados · validación por rangos en la capa de deserialización · números de secuencia · rate limits por tipo de mensaje declarados en el contrato · cero comandos de debug en el contrato de producción.

## Por definir (el trabajo de este pilar)

- Mecanismo de codegen: **protobuf** (maduro, plugins GDScript de terceros) vs **esquema propio** (control total, generadores simples a medida). Evaluar con un spike.
- El catálogo de mensajes v1 (el mínimo del vertical slice): handshake/auth, spawn/despawn, movimiento, combate, loot/carga, chat.
- Modelo de sincronización: frecuencia de snapshots del server, interpolación en cliente, relevancia por rango (culling).
- Manejo de reconexión y de sesión única por cuenta (la lección de las sesiones zombi del legado).

## Referencias

- Vectores del protocolo viejo: [../02-investigacion/decompilacion/](../02-investigacion/decompilacion/) — la lista de lo que este diseño hace imposible.
- Bugs del legado que se previenen por diseño: `pendientes-server.md` del prototipo (framing, sesiones zombi, Moving fuera de límites).
