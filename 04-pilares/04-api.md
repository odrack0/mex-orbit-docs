# Pilar 04 — La API de plataforma (y la de administración)

**Repos:** `mex-orbit-api`, `mex-orbit-api-admin` · **Estado:** fronteras definidas; diseño por hacer.

## Decisiones tomadas

1. **`mex-orbit-api` es dueña de los sistemas asíncronos**: cuentas y autenticación (emite los tokens que el game server valida) · **el Mercado de órdenes** · **el almacén global** · Starbond/License · perfil del piloto · misiones fuera de sesión.
2. **Una sola BD compartida** con el game server (pilar 02); frontera de escritura por dominio.
3. **`mex-orbit-api-admin` es una superficie separada** (despliegue y credenciales aparte): tablero económico, tuning en caliente, operación. Jamás alcanzable desde un cliente de juego.

## Por definir (el trabajo de este pilar)

- Auth: mecanismo de tokens (emisión api → validación game server), registro, recuperación, sesión única.
- El motor del Mercado: emparejamiento de órdenes, transaccionalidad, comisión, historial de precios diario, los dos modos del Starbond (precio administrado / libre).
- Contratos REST para el cliente (login, Mercado, almacén, perfil, misiones) — ¿tipos compartidos vía `mex-orbit-protocol` o contrato OpenAPI propio?
- Coordinación api ↔ game server para lo que cruza (ej. el almacén se llena al descargar en base: ¿quién escribe?, ¿eventos?).
- api-admin: modelo de roles, auditoría de acciones de administración (quién cambió qué número y cuándo).

## Referencias

- Guidelines §2 (Starbond/License), §17 (Mercado), §19–20 (misiones, piloto), §4 (almacén).
