# El pipeline de arte con IA — propuesta

**Estado: propuesta para dictamen** — las decisiones marcadas ⚠️ (Q1–Q8) están abiertas y cambian el diseño del pipeline.

## Mi opinión honesta: viable en 2026, con tres condiciones

Crear todo el arte con IA es alcanzable hoy — las herramientas de 2025-2026 (Gemini con edición por referencia, generadores vectoriales nativos, imagen→3D) cruzaron el umbral. Pero solo funciona con tres condiciones que separan un juego con identidad de una sopa de estilos:

1. **La IA genera; el humano dirige y cura.** El pipeline produce candidatos; cada asset que entra al juego pasó por un ojo humano con criterio (¿es legible a escala de juego?, ¿es de ESTE juego?). El rol del equipo cambia de "dibujar" a "dirección de arte + curaduría + QA".
2. **La Biblia de estilo antes que el primer asset.** Un documento con: paleta maestra, reglas de forma por familia (los aliens comparten ADN visual; las naves de una facción comparten lenguaje), iluminación canónica, nivel de detalle, y **los prompts maestros con imágenes de referencia** que anclan cada generación. Sin esto, el asset 200 no se parece al asset 1.
3. **Derivar antes que generar.** Generar desde cero produce inconsistencia; **derivar de una referencia aprobada** (la fortaleza de Gemini: "esta misma nave, vista superior" / "este mismo alien, versión colosal") produce familias coherentes. Cada familia tiene UN hero aprobado del que desciende todo.

## La decisión que define todo el pipeline ⚠️ Q1: la perspectiva

| Opción | Cómo se produce | Costo por nave/alien |
|---|---|---|
| **A. Top-down puro** (recomendada) | 1 imagen por entidad; **la rotación la hace el motor gratis** (rotar el nodo). La profundidad se simula con sombreado/luz en el sprite | **1 frame** |
| B. Perspectiva 3/4 estilo DO | La rotación cambia la silueta → 32 frames por entidad. Ruta: hero IA → imagen-a-3D (Tripo/Rodin) → limpieza → render de 32 rotaciones en Blender ortográfico | **32 frames + un modelo 3D intermedio** |

Mi recomendación es **A**: el pipeline es ~10× más simple, el estilo top-down limpio es moderno (y más legible en combate), y elimina la mayor fuente de inconsistencia (32 frames coherentes). La B es viable si la identidad visual de DO (naves "de ladito") es irrenunciable — pero cada entidad costará un ciclo de 3D.

## Los cinco carriles del pipeline (cada tipo de asset tiene el suyo)

### Carril 1 — Entidades (naves, aliens, drones, estaciones)
`Biblia de estilo → hero por familia (generación + curaduría) → derivados por edición con referencia (Gemini: variantes de especie, escalas Elite/Titan, facciones) → normalización (alfa, centrado, registro, tamaño estándar) → atlas + catálogo JSON → QA in-engine`

### Carril 2 — Iconografía de items (~110 iconos)
El carril más amigable a IA: estilo flat/semi-flat con **plantilla de prompt única** (mismo encuadre, mismo fondo, misma luz) + generación **vectorial nativa si el estilo lo permite** (⚠️ Q2). Los iconos se nombran por su código (`pb-1`, `df-b2`, `cel-4`) — el catálogo §11 de los Guidelines ES la lista de trabajo.

### Carril 3 — Mundo (fondos, planetas, tiles)
Fondos de mapa por zona (la IA es excelente en esto — nebulosas, campos estelares), planetas/decoraciones como sprites sueltos, tiles de asteroides/nubes. Los **minimapas no son arte**: se generan desde el editor de mapas (modernización sobre el prototipo, que usaba JPGs prerrenderizados).

### Carril 4 — FX: procedural antes que sprites
La lección más importante del censo del prototipo: el Flash necesitaba sprites para TODO (explosiones FLV, humo, escudos, motores). **En Godot 2026 casi nada de eso debe ser un sprite**: láseres = shader + textura base (los tiers CEL-1..4 son parámetros de color/intensidad, no assets), explosiones y motores = GPUParticles con 2-3 texturas de chispa/humo, escudos/cloak/recubrimientos (el glow de Aurorium/Tachyon) = shaders. La IA solo aporta las texturas semilla; el movimiento es del motor. **Esto reduce el inventario de FX de ~60 sheets a ~10 texturas + una librería de shaders.**

### Carril 5 — UI
La IA diseña (mockups de dirección: ⚠️ el prototipo de UI "militar-espacial 2026" es el siguiente entregable) y genera la iconografía de sistema; la implementación es **Godot Theme** (paneles nine-patch, estilos de control) — la UI vive como theme, no como imágenes sueltas. Tipografía: fuente licenciada/libre, no generada.

## Las herramientas candidatas (a validar con spikes)

| Rol | Candidatas | Nota |
|---|---|---|
| Heroes y arte clave | Gemini (Imagen), Midjourney | Calidad tope; curaduría fuerte |
| **Derivación consistente** | **Gemini (edición con imagen de referencia)** | La pieza clave del pipeline — variantes, poses, facciones sin perder identidad |
| Vector nativo | Recraft | Si Q2 = estilo flat: genera SVG real (iconos, UI) |
| Imagen → 3D | Tripo / Rodin / Meta 3D | Solo si Q1 = perspectiva 3/4 |
| Normalización | rembg (alfa), scripts propios (registro, atlas) | Vive en `mex-orbit-art/pipeline/` |
| Audio (⚠️ Q6) | ElevenLabs SFX, Suno/Udio música | Si el audio entra al pipeline IA |

**⚠️ Licencias**: antes de producir en serio, verificar los términos comerciales de cada herramienta elegida (uso comercial de outputs). Va en la Biblia de estilo como checklist.

## Riesgos honestos y sus mitigaciones

| Riesgo | Mitigación |
|---|---|
| Inconsistencia entre sesiones/meses | La Biblia de estilo + derivar siempre del hero aprobado + carpeta de referencias canónicas en `mex-orbit-art/brand/` |
| Animación cuadro-a-cuadro (lo más débil de la IA) | Carril 4: procedural. Lo poco animado por sprites (¿idle de aliens?) se decide por costo |
| "Vectorial" vs detalle | ⚠️ Q2 decide: estilo flat vectorizable vs raster renderizado |
| Legibilidad en juego (assets bellos que no se leen a 64px) | QA in-engine obligatorio: el visor de assets en Godot sobre fondos reales es parte del pipeline, no un extra |
| El nombre temporal | Cero logos/branding definitivo (regla de 01-vision) |

## El flujo resumido

```
Biblia de estilo (una vez, se mantiene viva)
   └→ por familia: HERO aprobado
        └→ derivados (Gemini ref-edit / Recraft / render 3D según carril)
             └→ normalización automatizada (scripts en mex-orbit-art)
                  └→ atlas + catálogo JSON (naming = códigos del juego)
                       └→ QA in-engine (visor Godot)
                            └→ commit en mex-orbit-art → export a mex-orbit-client
```
