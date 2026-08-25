# Pilar 03 — El game server

**Repo:** `mex-orbit-game-server` · **Estado:** principios definidos; arquitectura por diseñar.

## Decisiones tomadas

1. **.NET moderno, proyecto desde cero** — el server legado (linaje de emulador, namespaces `Ow.`) es referencia de comportamiento, jamás base de código.
2. **Loop de simulación de tick fijo** con tiempo inyectado (cero `DateTime.Now` disperso) y estado testeable sin red.
3. **Autoridad total del servidor**: el cliente pide, el server decide (contrato del pilar 01).
4. La **lista negra estructural** (los 9 grupos de bugs documentados del legado no pueden existir por diseño): excepción de un tick jamás mata el loop · colecciones de simulación sin carreras · sesión única por cuenta con expulsión limpia · clamps de entrada en la puerta · estados derivados de datos persistidos coherentes (nave muerta al login).

## Por definir (el trabajo de este pilar)

- Arquitectura de simulación: modelo de entidades (composición), particionado por mapa, scheduling del tick, presupuesto de tick (ms) y métricas.
- Concurrencia: hilos de red vs hilo(s) de simulación — colas de mensajes hacia la simulación (nunca mutación directa desde red).
- Persistencia: write-behind al descargar/eventos clave; qué es autoritativo en memoria.
- Los sistemas del guideline en orden del plan maestro: combate y muerte (gradiente/Black Box/durabilidad) · NPCs y taxonomía · pods/Flux · zonas con sesgo · Eclipses/Materializador · incursiones · Arena · eventos de defensa.
- Testeo: simulación ejecutable en tests (el tick como función pura de entrada→estado), harness de bots de carga.

## Referencias

- Comportamiento esperado: [../02-investigacion/flujos-legado/](../02-investigacion/flujos-legado/) · reglas de juego: los guidelines · bugs a imposibilitar: `pendientes-server.md` del prototipo.
