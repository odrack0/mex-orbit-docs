# El plan del cliente 3D — migrar el v1 a la arquitectura del original

**Dictamen (29-ago-2026)**: el v1 replica el **cliente 3D** de DarkOrbit lo más fielmente posible, con sus mismas escalas de prioridades. No hay Q1 ni lineamiento previo que lo frene: lo que estorbe se reescribe. De los dos clientes del decompilado (2D y 3D), migramos **solo el 3D**. La fuente de todas las constantes es [darkorbit-3d-guidelines.md](darkorbit-3d-guidelines.md) (citado aquí como **G§n**); este documento dice **qué construir, en qué orden, y qué muere**.

La decisión de fondo que lo hace viable: **G§13.1 — el 3D es presentación pura**. El modelo de juego, el protocolo, el server, las ventanas del sistema N y el picking son lógica 2D y **no se tocan**. Lo que se reescribe es la capa de vista: de "mundo de sprites con SubViewports por bicho" a **una sola escena 3D con cámara en perspectiva**, que es como lo hacía el original.

---

## 1. La arquitectura destino (el mapa del original → Godot)

| Pieza DO 3D | En Godot |
|---|---|
| Escena Away3D única + cámara perspectiva | **Un solo `World3D`**: `Node3D` raíz + `Camera3D` en perspectiva. Muere el SubViewport-por-entidad |
| Starling (fondo 2D) bajo el 3D, un solo present | El mundo 3D en el viewport principal; fondo = skybox + geometría 3D (ver §5). Los mapas "planos" pueden conservar un `CanvasLayer` de paralaje detrás durante la transición |
| HUD Flash display list encima | `CanvasLayer` con elementos posicionados por `camera.unproject_position()` — ya es nuestro patrón de ventanas N |
| Coordenadas juego (x,y) → 3D (x, alt, −y), 1:1 | Igual, literal (G§2). 1 unidad 3D = 1 unidad de juego; alturas: naves y=0, ores −70..−130, star dust 0..−300, tilemaps de fondo −3500+capa·550 |

## 2. La cámara (G§3, completa y sin adaptar)

Es la pieza que responde "veo los aliens desde otra perspectiva al hacer zoom": en una escena única, bajar la cámara cambia el escorzo de TODO a la vez.

- `Camera3D` **perspectiva, FOV vertical 30°**, near 10, far 80000.
- Rig orbital: `d = 1740/zoom` · **tilt 135° (elevación 45°)** · pan 0° (25° en mapas con fondo 3D) · `lookAt(héroe.x, 0, −héroe.y, up=+Y)`.
- Zoom **[1, 3]** continuo, rueda ×1.2 / ×0.8, tween 0.3 s Quad ease-out sobre el factor.
- **Acoplamiento tilt↔zoom**: `tiltEfectivo = tilt − clamp((zoom−1)/2·20, 0, 20)` — al acercar, la cámara baja hasta 20° hacia el horizonte. Este es el efecto pedido.
- Seguimiento rígido `int(héroe)`; sin clamp en bordes; zonas de cámara y órbita con botón derecho cuando haya mapas que las declaren.
- Shake: **solo detonaciones** (corrección de G§3); v1 no tiene minas aún → sin shake.
- Proyección/desproyección: `unproject_position` para HUD; `project_ray_origin/normal` + intersección con plano y=0 para click-to-move y para el **trapecio del minimapa** (las 4 esquinas — G§4).

**Qué muere**: el rango de zoom 0.621–1.157 y su calibración (era del mundo sprite), la Camera2D, el `HUD_ZOOM_REF`.

## 3. Entidades (G§5–G§7)

`EntityNode` conserva TODO su modelo (interpolación dist/speed, sombra autoritativa de bichos, reconcile, attack_target, barras absolutas) y cambia su cuerpo:

- **Vista = `Node3D` en la escena única** con el GLB instanciado (los mismos GLB del pipeline actual — `mexorbit-asset-3d` sigue siendo la cadena de producción). Carga asíncrona ya implementada; se conserva.
- **Placeholder y especies sin malla**: un **quad tumbado en el plano** con su PNG cenital — exactamente el placeholder del original (G§5.5). Las especies de atlas de video (skarn, ferox, mordax, gravit, gravon, skarnox) entran así (quad con su atlas animado) **hasta que tengan GLB**; la meta de fase 4 es que todo el bestiario sea malla. Nota honesta: un quad plano a cámara 45° se ve escorzado como cartulina — igual que el placeholder del DO; es transición aceptada, no destino.
- **Giro**: se adopta el del 3D original — **ease exponencial QuadOut d=0.2 s, continuo, sin cuantizar** (G§5.2). El giro 0.1 s/32 pasos era el look del cliente 2D y muere con él. Se recalibra volando si 0.2 se siente lento.
- **Banking real**: roll del modelo ±20°/0.2 s crucero, −2×/±10°/0.08 s combate (ya implementado — se conserva tal cual, ahora en el mundo único).
- **Hover** 5u/5°/ciclo 2 s solo parado (implementado; pasa a mover el Node3D visual). NPCs idle ±180° cada 2–7 s (implementado).
- **Slots** `tobera_*`/`canon_*` del GLB (ya existen vía `marcar-anclajes.py`): en escena única dan posiciones 3D reales — los disparos salen de las bocas incluso con banking (G§6.1).
- **Luz del héroe**: PointLight azul 0x2E7DFF, fallOff 450 — **ahora sí derrama sobre vecinos**, como el original (el aislamiento de SubViewports lo impedía). Energía 0.6, specular alto.
- **Glow ↔ HP** (implementado, se conserva). **Cloak** alpha 0.5/0.2 s (transparencia de material).

## 4. Iluminación y materiales (G§7)

Presupuesto DURO del original, ahora posible en el mundo único:

- **1 DirectionalLight** (dial actual de `AssetDefs`: cian 0xA3FFFF·0.8, elevación −54°) + **1 PointLight del héroe** + **pool de exactamente 3 OmniLight de efectos** (explosiones/disparos, con tween de vida corta). **Sin sombras. Sin post-proceso** (el glow del environment sí, es el equivalente del emisivo por canal).
- Ambiente cálido 0xFF855C·0.5 (dial actual) + tonemap FILMIC.
- Materiales **compartidos por especie** (la escena única lo permite al fin — muere la copia por instancia; el destello por entidad se hace con `instance uniforms` o con el multiplicador de emisión por instancia de Godot).
- El shader `relieve` 2D muere con los sprites: en 3D la luz fija sobre modelo que gira es **gratis por construcción**.

## 5. El fondo (G§10)

- **Skybox de estrellas con twinkle**: esfera/sky shader `stars(uv) · mask(uv+(2,1)t/120) · mask(uv+(−1.5,1)t/120)`, recentrada en el punto de mira.
- **Star dust 3D**: el sistema de 1500 quads en volumen 2000³, clonado en mosaico cada 2000 u, y ∈ [0,−300], bounds por tile — es LA capa que vende la profundidad al volar y al hacer zoom (G§10.4). Sustituye al `Starfield2D` de pantalla.
- **Tilemap 3D**: quads de nebulosa a y = −3500 + capa·550, cada tile con offset aleatorio −500..−200 — **el paralaje lo hace la cámara**, sin fórmula (G§10.2). El `maps/1-1.json` actual se traduce: sus `tiles_far/near` pasan a capas de profundidad, sus `p_factor` se convierten en alturas.
- **Planetas/sol**: planos billboard a distancia (coordenadas esféricas del original) o quads a su profundidad. Lensflares 2D proyectados con oclusión por raycast (la cadena ×3 ya está portada — se re-engancha a la proyección 3D).
- El `MapBackground` 2D actual sobrevive SOLO como fallback de calidad baja durante la transición, y muere en fase 4.

## 6. FX (G§9)

Las recetas ya están mapeadas en G§15 (tabla técnica→Godot). Prioridad de migración:

1. **Beams**: quad/malla de largo fijo + textura patrón repeat + UV scroll (`uv.x += TIME/cycle`, 0.3–0.5 s), estirado `scale.z = dist/largo` + `rotation.y = atan2` — el disparo deja de ser proyectil 2D y pasa a ser el haz del original (muere `projectile_2d.gd`; el modelo de duración 0.15 s era del prototipo 2D).
2. **Explosión multi-capa 3D**: chispas GPUParticles3D radiales + flash + fireballs de atlas + debris en max, con flash del pool de luces (¡ahora hay luces!).
3. **Thrusters** (partícula por tobera, escala 1/0.7/0 por estado — port de lo implementado) + **estela ribbon 12 muestras × 30 ms** (G§6.3) — vuelve, porque en 3D es el trail del original y la objeción de las chispas 2D no aplica.
4. **Impactos** casco/escudo como quads billboard aditivos (los sheets actuales sirven tal cual).
5. Efectos declarativos: cada fx = escena .tscn (el .awp del original era JSON; nuestro .tscn cumple el mismo papel — G§13.6). Additive + unshaded + color ramp + desincronización SIEMPRE (G§9.5).

## 7. Calidad — "sus mismas escalas de prioridades" (G§12)

`Quality` (autoload) se conserva con las claves re-mapeadas al mundo 3D, calcando los cortes del original:

| clave | BAJA | MEDIA | ALTA |
|---|---|---|---|
| `npc` | quad PNG | quad PNG | **malla GLB** |
| `luces` | solo sol | sol + 1 efecto | sol + héroe + pool 3 |
| `emissive`/glow | apagado | encendido | encendido |
| `engine` | sin llamas | llamas | llamas + ribbon |
| `explosion` | flipbook solo | multi-capa | multi-capa + debris |
| `background` | skybox solo | + star dust | + tilemaps 3D |
| `aa` (MSAA viewport) | 0 | 2× | 4–8× (el 16× del original no paga en Vulkan) |

- **Gates por sitio de invocación**, con booleanos, aplicables en vivo (ya es así).
- **Auto-quality por FPS** (implementada): la escalera se re-mapea a estos cortes.
- **Lifetimes de assets** (G§8.4): transitorio / mapa / permanente + refcount con gracia 5 s — se implementa sobre la caché de GLB existente al crecer el catálogo.

## 8. Lo que NO cambia (y es la mitad del valor)

`net/` entero · protocolo y server · `Session`/reconexión/salto de sector · ventanas del sistema N (Nave, Estación, COMMS, Minimapa, Ajustes, taskbar, sysbar) · `data/*.json` (se extienden con los campos 3D, no se sustituyen) · pipeline de arte Meshy→GLB→horno (el horno pasa de "producir el juego" a "producir placeholders y calidad baja") · dev-run, autotest y bestiario (la lógica de fases vive sobre el modelo; solo se re-encuadran las capturas).

## 9. Lo que muere, dicho sin eufemismos

`Camera2D` y su rango calibrado · giro 0.1 s/32 pasos · sprites del mundo como camino principal (quedan de placeholder/baja) · SubViewport por entidad y `VIEWPORT_FACTOR` · shader `relieve` y su prueba del autotest · `Starfield2D` de pantalla · `MapBackground`/paralaje por fórmula (fase 4) · `projectile_2d.gd` · la homologación media↔alta por horneado **cambia de sentido**: media ya no imita a alta — media ES el mundo de quads, como el cliente 2D del original convivía con el 3D.

## 10. Fases (cada una cierra con su gate)

**F1 — El mundo gira** *(la fase que responde al pedido de hoy)*
Escena 3D única + cámara G§3 completa (rig orbital, zoom [1,3], tilt↔zoom) + héroe como malla + click-to-move por ray-plane + HUD proyectado (barras/nombre/números) + skybox básico. Gate: volar el 1-1, zoom cambia la perspectiva de la nave propia, autotest de vuelo verde.

**F2 — El mundo pelea**
Todas las entidades en escena (GLB los que tienen, quad los demás) + giro/banking/hover en mundo único + beams UV-scroll desde las bocas + impactos + explosión multi-capa + luces (sol + héroe + pool) + selección/picking proyectado + minimapa con trapecio. Gate: autotest de combate y loot verde; bestiario re-encuadrado.

**F3 — El mundo es profundo**
Star dust 3D en mosaico + tilemaps de profundidad + planetas/sol/lensflares + estación y portales en escena (la estación ya es malla; el portal pasa de atlas a quad/malla) + cajas. Gate: autotest completo verde; el 1-1 se lee con paralaje real al volar y al hacer zoom.

**F4 — El mundo es el 3D**
Thruster+ribbon + fx declarativos restantes + calidad/AA/auto-quality re-mapeadas + retirar los caminos 2D muertos + GLB para el bestiario completo (una especie por ronda, con la cadena de `mexorbit-asset-3d`). Gate: regresión completa + comparación lado a lado contra DO 3D en vivo (el oráculo que ya demostró valer más que cualquier informe).

**Regla de las fases**: el juego queda jugable al cierre de cada una — nada de ramas eternas; se avanza en `main` con los gates de siempre.

## 11. Riesgos con nombre

1. **Un mundo, muchas mallas**: el banco (`banco_3d`) midió N modelos en un mundo — es EL escenario nuevo, y su cifra era mejor que la de viewports. Medir con población real en F2.
2. **Quads placeholder a 45°** se ven planos — asumido; presiona a completar GLBs, no a suavizar la cámara.
3. **El giro 0.2 s** puede sentirse distinto al gusto ya validado — es dial; se recalibra volando como todo.
4. **El bestiario/autotest** comparan píxeles con encuadres cenitales — re-calibrar umbrales al re-encuadrar (lección: verificar mtime y tamaño de las capturas).
5. **Las cifras del guideline** pueden traer generalizaciones — ante cualquier duda, leer las condiciones en el decompilado y contrastar contra DO 3D en vivo (lección del shake, ya cobrada).
