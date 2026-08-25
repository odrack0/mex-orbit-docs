# Pilar 05 — El cliente

**Repo:** `mex-orbit-client` · **Estado:** herencia del prototipo definida; UI por diseñar.

## Decisiones tomadas

1. **Godot 4 / GDScript** — código en inglés, comentarios en español.
2. **Nace del aprendizaje del prototipo** (`MexOrbit.GodotClient` en Azure): su arquitectura de escenas, ventanas y patrones demostraron el camino; el código se trae **selectivamente y adaptado a las nuevas bases** — jamás la capa de red legada, jamás la UI calcada del Flash, jamás los sprites `do_img`.
3. Red contra `mex-orbit-protocol` (WebSocket binario) para el tiempo real; `mex-orbit-api` (REST) para lo demás.
4. Assets exclusivamente del pipeline de `mex-orbit-art`.

## Por definir (el trabajo de este pilar)

- **El design system de UI propio** — la identidad visual de las ventanas, HUD, tipografía, iconografía (en conjunto con el [pilar de arte](../05-arte/README.md)).
- Arquitectura de escenas v1 y el mapa de pantallas del vertical slice (login → hangar/almacén → mundo → base).
- La capa de red nueva: consumo del contrato generado, interpolación/predicción según el modelo de sincronización del pilar 01.
- Qué módulos del prototipo se portan y en qué orden (candidatos obvios: gestión de escenas del mundo, sistemas de ventana; descartados: `net/`, réplicas del Flash).
- Estrategia de testeo (el prototipo ya tiene cultura de e2e — conservarla).

## Referencias

- El prototipo como catálogo de lecciones: sus `docs/` (protocol-spec, ui-original, naves-y-aliens) en el repo legado de Azure.
