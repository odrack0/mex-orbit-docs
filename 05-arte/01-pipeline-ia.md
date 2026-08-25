# El pipeline de arte con IA

**Estado: DISEÑO CERRADO** (dictamen Q1–Q8 del 25-ago-2026):

| Decisión | Dictamen |
|---|---|
| Q1 Perspectiva | **Top-down puro: 1 frame por entidad, la rotación la hace el motor** |
| Q2 Estilo | **Vectorial** (flat/estilizado, fuente SVG) |
| Q3 Facciones | Identidad por facción en **estaciones y fondos de mapa**; las **naves son idénticas** para todos |
| Q4 Tiers de alien | Elite/Titan = **escala + tinte + VFX** (cero arte nuevo); **Imperators con arte único, renovados por temporada** (3 por ciclo, junto a sus incursiones — §18 de los Guidelines) |
| Q5 Mapas | **Se hereda el grafo del prototipo** (bajos, altos, PvP 4-x) con sus ~17 fondos |
| Q6 Audio | **También con IA** (SFX y música) |
| Q7 Herramientas | Set recomendado abajo (Recraft + Gemini + ElevenLabs + Suno) |
| Q8 Resolución | **1080p nativo con soporte 1440p**; 4K después |

## Mi opinión honesta: viable en 2026, con tres condiciones

Crear todo el arte con IA es alcanzable hoy — las herramientas de 2025-2026 (Gemini con edición por referencia, generadores vectoriales nativos, imagen→3D) cruzaron el umbral. Pero solo funciona con tres condiciones que separan un juego con identidad de una sopa de estilos:

1. **La IA genera; el humano dirige y cura.** El pipeline produce candidatos; cada asset que entra al juego pasó por un ojo humano con criterio (¿es legible a escala de juego?, ¿es de ESTE juego?). El rol del equipo cambia de "dibujar" a "dirección de arte + curaduría + QA".
2. **La Biblia de estilo antes que el primer asset.** Un documento con: paleta maestra, reglas de forma por familia (los aliens comparten ADN visual; las naves de una facción comparten lenguaje), iluminación canónica, nivel de detalle, y **los prompts maestros con imágenes de referencia** que anclan cada generación. Sin esto, el asset 200 no se parece al asset 1.
3. **Derivar antes que generar.** Generar desde cero produce inconsistencia; **derivar de una referencia aprobada** (la fortaleza de Gemini: "esta misma nave, vista superior" / "este mismo alien, versión colosal") produce familias coherentes. Cada familia tiene UN hero aprobado del que desciende todo.

## La perspectiva (Q1 ✅ cerrada): top-down puro

**1 frame por entidad; la rotación la hace el motor.** La profundidad se simula con sombreado/luz dentro del sprite. Consecuencias: pipeline ~10× más simple que la alternativa de 32 frames, máxima legibilidad en combate, cero paso de 3D intermedio, y el estilo vectorial (Q2) encaja natural — un SVG top-down rota sin perder nitidez a cualquier resolución (Q8 resuelta de paso: el vector es resolution-independent; 4K será re-export, no re-trabajo).

## Los cinco carriles del pipeline (cada tipo de asset tiene el suyo)

### Carril 1 — Entidades (naves, aliens, drones, estaciones)
`Biblia de estilo → hero por familia (generación + curaduría) → derivados por edición con referencia (Gemini: variantes de especie, escalas Elite/Titan, facciones) → normalización (alfa, centrado, registro, tamaño estándar) → atlas + catálogo JSON → QA in-engine`

### Carril 2 — Iconografía de items (~110 iconos)
El carril más amigable a IA: estilo flat/semi-flat con **plantilla de prompt única** (mismo encuadre, mismo fondo, misma luz) y **generación vectorial nativa en Recraft** (Q2 ✅: SVG como fuente). Los iconos se nombran por su código (`pb-1`, `df-b2`, `cel-4`) — el catálogo §11 de los Guidelines ES la lista de trabajo.

### Carril 3 — Mundo (fondos, planetas, tiles)
Fondos de mapa por zona (la IA es excelente en esto — nebulosas, campos estelares), planetas/decoraciones como sprites sueltos, tiles de asteroides/nubes. Los **minimapas no son arte**: se generan desde el editor de mapas (modernización sobre el prototipo, que usaba JPGs prerrenderizados).

### Carril 4 — FX: procedural antes que sprites
La lección más importante del censo del prototipo: el Flash necesitaba sprites para TODO (explosiones FLV, humo, escudos, motores). **En Godot 2026 casi nada de eso debe ser un sprite**: láseres = shader + textura base (los tiers CEL-1..4 son parámetros de color/intensidad, no assets), explosiones y motores = GPUParticles con 2-3 texturas de chispa/humo, escudos/cloak/recubrimientos (el glow de Aurorium/Tachyon) = shaders. La IA solo aporta las texturas semilla; el movimiento es del motor. **Esto reduce el inventario de FX de ~60 sheets a ~10 texturas + una librería de shaders.**

### Carril 5 — UI
La IA diseña (mockups de dirección: ⚠️ el prototipo de UI "militar-espacial 2026" es el siguiente entregable) y genera la iconografía de sistema; la implementación es **Godot Theme** (paneles nine-patch, estilos de control) — la UI vive como theme, no como imágenes sueltas. Tipografía: fuente licenciada/libre, no generada.

## El set de herramientas (Q7 ✅ — las suscripciones recomendadas)

| Herramienta | Rol en el pipeline | Suscripción |
|---|---|---|
| **Recraft** | **La herramienta principal**: generación vectorial nativa (SVG real) — naves, aliens, iconos, elementos de UI. Estilos custom entrenables (nuestra Biblia de estilo como estilo propio) | Sí (Pro) — la inversión #1 |
| **Gemini** (ya la tienes) | Exploración de conceptos, derivación por referencia ("este mismo alien, colosal"), composiciones para fondos, iteración rápida antes de vectorizar | Ya cubierta |
| **ElevenLabs** | SFX (láseres, explosiones, UI) — generación por descripción | Sí |
| **Suno** | Música (menú, biomas, combate, Imperator) | Sí |
| Inkscape + SVGO | Limpieza/edición de SVG y optimización — **gratis** | — |
| Scripts propios | Normalización, registro, export SVG→texturas 1080/1440, atlas, catálogo JSON | `mex-orbit-art/pipeline/` |

Sin herramienta 3D (Q1 la eliminó), sin Midjourney por ahora (con estilo vectorial, Recraft+Gemini cubren; sumable después para arte de marketing). **Total: 3 suscripciones nuevas.**

**Checklist de licencias (antes de producir en serie):** verificar los términos de uso comercial de Recraft, ElevenLabs y Suno en sus planes de pago — documentar en la Biblia de estilo qué plan otorga qué derechos sobre los outputs.

## Riesgos honestos y sus mitigaciones

| Riesgo | Mitigación |
|---|---|
| Inconsistencia entre sesiones/meses | La Biblia de estilo + derivar siempre del hero aprobado + carpeta de referencias canónicas en `mex-orbit-art/brand/` |
| Animación cuadro-a-cuadro (lo más débil de la IA) | Carril 4: procedural. Lo poco animado por sprites (¿idle de aliens?) se decide por costo |
| "Vectorial" vs detalle | Q2 ✅ cerrada: estilo flat/estilizado con fuente SVG — el detalle se gana con forma y luz, no con textura fotográfica |
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
