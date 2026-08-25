# El arte

**Estado: en progreso.** El pipeline de producción con IA, el inventario de assets y el **sistema de diseño de UI (dirección N, aprobado)** ya están definidos; la dirección de estilo del arte del mundo (naves, aliens, fondos) sigue pendiente de su sesión propia.

## Contenido de esta carpeta

- [`01-pipeline-ia.md`](01-pipeline-ia.md) — pipeline de producción de arte con IA.
- [`02-inventario-assets.md`](02-inventario-assets.md) — inventario completo de assets del juego.
- [`03-sistema-diseno-ui.md`](03-sistema-diseno-ui.md) — **sistema de diseño de UI aprobado** (tokens, chrome, leyes de UX). Fuente de verdad para toda interfaz; existe el skill `mexorbit-ui` que lo carga.
- `prototipo-ui-n.html` — **la referencia viva de la UI** (Propuesta N = merge M × I).
- `prototipo-ui*.html` (A, B, C, D, E, H, I, M) — propuestas archivadas del proceso.

## Lo poco que ya está decidido (para no partir de cero)

- **Cero assets heredados de BigPoint** — identidad visual 100% propia.
- **Base vectorial**: el diseño base de naves vectoriales quedó aprobado en fase 0 y es el punto de partida del estilo.
- La fuente vive en `mex-orbit-art`; el cliente consume exportaciones del pipeline.
- Las cinco familias de nombres (materiales, legendarios, códigos, constelaciones, taxonomía) esperan su identidad visual.
- El nombre del juego es temporal: **nada de logos ni branding definitivo todavía**.

## Las preguntas que esta sesión deberá responder

1. **¿Quién produce?** — ¿arte propio (herramientas, tiempo), comisionado (artistas externos), asistido por herramientas generativas como base para vectorizar, o un híbrido? ¿Con qué presupuesto?
2. **La dirección de estilo**: paleta, silueta, nivel de detalle, legibilidad en combate (un Vex debe distinguirse de un Vexor a un vistazo, y un Titan imponer).
3. **El pipeline técnico**: formato fuente (SVG/proyecto), exportación (¿spritesheets prerrenderizados como el prototipo, o vectorial nativo en Godot?), animaciones (rotaciones de nave, efectos).
4. ~~**El design system de UI**~~ — **resuelto**: ver `03-sistema-diseno-ui.md` (dirección N).
5. **Alcance del vertical slice**: la lista mínima de arte para la Etapa 2 del plan maestro (1 nave, 1 alien, 1 mapa, HUD básico…).
