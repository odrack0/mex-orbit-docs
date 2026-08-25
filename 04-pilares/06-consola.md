# Pilar 06 — La consola de administración

**Repos:** `mex-orbit-console` (frontend) + `mex-orbit-api-admin` (backend, ver pilar 04) · **Estado:** alcance definido.

## Decisiones tomadas

1. **El heredero del CMS, reducido a su único trabajo legítimo**: administrar y calibrar. Sin portal de jugadores, sin tienda web, sin assets del juego.
2. Tres áreas: **tablero económico** (el termómetro: precios del Mercado, grifos/sumideros, Starbond, actividad) · **tuning en caliente** (los números calibrables del guideline, editables sin deploy) · **operación** (cuentas/moderación, temporadas, calendario de eventos, anuncios).

## Por definir

- Stack del frontend (el conocimiento previo es Vue/TS — razonable reutilizar el stack, no el código).
- El catálogo v1 de métricas del tablero y de parámetros calibrables (sale del pilar 02: la tabla de configuración).
- Roles de administración y auditoría (con api-admin).
- Qué necesita la consola en cada etapa del [plan maestro](../06-plan-maestro/README.md) — crece con el juego, no al final.
