# Pilar 07 — Infraestructura y operación

**Transversal a todos los repos** · **Estado:** deudas identificadas; por diseñar.

## Decisiones tomadas

1. **Los backups son la deuda #1 y no se negocian**: BD con respaldo automático diario + copia fuera de la máquina, desde el primer entorno con datos que duelan. (La lección viva: el prod del prototipo operó sin respaldo.)
2. Entornos separados: dev → staging → producción.
3. GitHub como plataforma de v1 (repos `odrack0/mex-orbit-*`); los repos legados quedan en Azure DevOps como archivo del prototipo.

## Por definir

- CI/CD por repo (build + tests + despliegue), y el runner de migraciones de `mex-orbit-data-base` integrado al despliegue.
- Hosting de v1: ¿el servidor actual (74.208.108.67) alberga también los entornos nuevos o se separa? Destino del prototipo en producción (¿laboratorio congelado?).
- Monitoreo: métricas del tick del game server, alertas (el "server congelado sin log" del legado no puede repetirse — health checks reales), logs estructurados.
- TLS/dominios para `wss://` y las APIs (los dominios turname.mx existentes, ¿se reutilizan?).
- Estrategia de despliegue del cliente Godot (builds, canal de actualización).
