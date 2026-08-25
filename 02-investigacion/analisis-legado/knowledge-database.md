# Knowledge database (MexOrbit)

Base de conocimiento breve: hechos verificados en código o protocolo, útiles para portar clientes (Flash, Godot, Unity) o alinear el servidor.

---

## Cliente Flash — coordenadas del minimapa (`206 / 127`)

### Qué muestra

En el HUD del minimapa, el texto con formato `N/M` (por ejemplo `206 / 127`) **no** son píxeles dentro de la textura del minimapa ni índices de “casilla” del PNG.

Son la **posición de la nave del jugador** (`hero.position`) en el **mismo espacio lógico** que usa el mapa para movimiento y scroll, expresada de forma compacta: **cada número es la coordenada dividida entre 100 y truncada a entero** (hacia cero, como `int()` en ActionScript).

- **Primer número:** `int(hero.position.x / 100)`
- **Segundo número:** `int(hero.position.y / 100)`

Ejemplo: `206 / 127` implica aproximadamente **X ∈ [20600, 20699]** e **Y ∈ [12700, 12799]** en unidades internas (el rango exacto del entero mostrado depende del truncado; la posición real puede ser cualquier valor en ese intervalo de centenas).

### Fuente en el decompilado

Clase del minimapa: `§_-Nr§` (paquete `§_-H2N§`), método que actualiza el `TextField` del contador:

```286:286:Decompile/spacemap/main/scripts/§_-H2N§/§_-Nr§.as
            this.§_-d3J§.text = int(this.hero.position.x / 100) + "/" + int(this.hero.position.y / 100);
```

### Implicaciones para otros clientes

Para **paridad con Flash**, cualquier HUD que muestre “coordenadas de minimapa” debería usar la misma fórmula sobre la posición del jugador en **unidades de mundo** ya reconciliadas con el servidor/cliente Flash, no coordenadas de textura del minimapa.

---

## Cliente Flash / `maps-config` — `scaleFactor` y tamaño lógico del mapa

### Qué es

Cada `<map>` puede declarar el atributo **`scaleFactor`**. Si falta, se trata como **1**. No cambia solo la “sensación visual”: en el cliente Flash las **dimensiones lógicas del mapa** (ancho/alto jugables en las mismas unidades que `hero.position` y el minimapa `/100`) se obtienen multiplicando la base **21000 × 13100** por ese factor.

- **Ancho lógico:** `21000 * scaleFactor`
- **Alto lógico:** `13100 * scaleFactor`

La base **21000 × 13100** es el mapa “estándar” (`scaleFactor = 1`).

### Ejemplos en datos del proyecto

- **`scaleFactor="2"`** — mapa el doble en X e Y respecto al estándar → **42000 × 26200**. En `MexOrbit.CMS/public/spacemap/graphics/maps-config.xml` aparece en mapas como **4-4** (`id="16"`) y **4-5** (`id="29"`).
- **`scaleFactor="0.5"`** — la mitad del estándar → **10500 × 6550**. Ejemplo: entradas **JP** (`id="101"`, etc.) en el mismo XML.

### Relación con el minimapa

Las coordenadas `N/M` del minimapa siguen siendo `int(x/100)` e `int(y/100)` sobre **`hero.position` en ese espacio ya escalado**. En un mapa con `scaleFactor="2"`, los valores mostrados pueden llegar hasta el doble (en centenas) que en un mapa 1×1.

### Referencias

- Convención y nombres Flash (`§_-t32§`, `§_-82Q§`): [`MexOrbit.GodotClient/docs/cliente-flash-spacemap-mapa.md`](../MexOrbit.GodotClient/docs/cliente-flash-spacemap-mapa.md).
- Valores por mapa: [`MexOrbit.CMS/public/spacemap/graphics/maps-config.xml`](../MexOrbit.CMS/public/spacemap/graphics/maps-config.xml) (atributo `scaleFactor` en `<map>`).
- En Godot, alinear **`logicalWidth` / `logicalHeight`** (o equivalente) con `21000 * scaleFactor` y `13100 * scaleFactor` si se busca paridad con Flash.

---

## Cómo ampliar este documento

Añadir entradas autocontenidas con: fenómeno observado, significado, referencia a archivo/ruta y, si aplica, consecuencia para implementación.
