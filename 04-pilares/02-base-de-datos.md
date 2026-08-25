# Pilar 02 — La base de datos

**Repo:** `mex-orbit-data-base` · **Estado:** **esquema v1 diseñado en E1** → [`mex-orbit-data-base/docs/esquema-v1.md`](https://github.com/odrack0/mex-orbit-data-base/blob/main/docs/esquema-v1.md) + migración `2026.08.25.1` (25 tablas del slice + semillas); pendiente la corrida viva (iteración I3 de la etapa 1).

## Decisiones tomadas

1. **MySQL** — la mejora es el esquema y la disciplina, no el motor.
2. **UNA base de datos** compartida por game server y api; frontera de escritura por dominio, documentada tabla por tabla.
3. **Migraciones versionadas `AAAA.MM.DD.N/` con `rollout.sql` + `rollback.sql`** — ningún cambio fuera de este mecanismo; sin baselines binarios como despliegue.
4. **Esquema en inglés, comentarios en español.**
5. Desde cero, con las lecciones del legado como lista negra: NPCs separados de naves de jugador · cero JSON-en-varchar para datos estructurados · claves foráneas reales.

## Por definir (el trabajo de este pilar)

- El diseño del esquema v1, dominio por dominio: cuentas/sesiones · naves y equipo (con **durabilidad** e instancias de item — los items usados se comercian) · almacén global · materiales y consumibles · el Mercado de órdenes (compra/venta, emparejamiento, historial de precios) · planos y pity · NPCs/mapas/zonas (con el sesgo de drops) · temporadas y rangos · misiones · perfil de piloto · Starbond/License · configuración calibrable (los "números" del guideline).
- Estrategia de acceso: EF Core vs Dapper por servicio; pooling; transacciones del Mercado.
- Runner de migraciones (aplicación ordenada + registro de versión aplicada).
- Persistencia del estado en vivo: qué se escribe en caliente vs al descargar/logout (la lección del `current_hit_points` que congelaba el server legado).

## Referencias

- El esquema legado como referencia de dominio (y lista negra): [../02-investigacion/implementacion-legado/sql-schema.md](../02-investigacion/implementacion-legado/sql-schema.md) y los guidelines §1–§20 como catálogo de todo lo que la BD debe representar.
