# La investigación — la evidencia detrás de cada decisión

Todo lo que se estudió antes de diseñar. Cuando alguien pregunte "¿por qué decidimos X?", la respuesta con números está aquí.

## Los tres estudios madre

| Documento | Qué demuestra |
|---|---|
| **[economia-darkorbit-2010-retail.md](economia-darkorbit-2010-retail.md)** | La autopsia completa del DarkOrbit clásico: 18 secciones, los 11 anti-patrones con precios reales, el costo del full UFE (~15–19M de uridium), la cronología de capas anuales y la demostración del treadmill divergente. **El documento que justifica la existencia de este proyecto.** |
| **[economias-mmo-2026.md](economias-mmo-2026.md)** | Cómo lo hacen los MMOs que sobreviven: el puente de monedas (PLEX/Bonds/Token), pity como estándar, la regulación UE de loot boxes, temporadas, horizontal antes que vertical |
| **[recursos-darkorbit-historia.md](recursos-darkorbit-historia.md)** | La historia de los 9 recursos del retail y la compresión a nuestro set de 5 |

## El sistema legado, documentado (referencia de dominio)

Material producido durante el desarrollo del prototipo — la "spec de comportamiento" que ningún proyecto nuevo tiene el lujo de tener:

- **decompilacion/** — los mapas del protocolo Flash: command IDs, crossref cliente↔server. La lápida del protocolo viejo y la lista de vectores (bots, injection) que `mex-orbit-protocol` hace imposibles.
- **flujos-legado/** — cómo funcionaba el sistema anterior: cálculo de daño y su aleatorización, sincronización de equipo, cálculo de stats, inicialización del cliente.
- **definiciones-legado/** — el catálogo completo de los 202 items del retail (con los `loot_id` que la §10–11 de los Guidelines mapea a los nombres nuevos) y las skills de piloto.
- **analisis-legado/** — el análisis del proyecto anterior (ago-2026) y su base de conocimiento.
- **implementacion-legado/** — los planes de modernización que se intentaron sobre el server viejo (historia de qué se aprendió).

⚠️ Las rutas a código que aparecen en estos documentos (`MexOrbit.Server/...`, `MexOrbit.GodotClient/...`) refieren a los repos **legados en Azure DevOps** (evolabsmx/MexOrbit) — el prototipo/laboratorio, no los repos de v1.
