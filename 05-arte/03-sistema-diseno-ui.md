# Sistema de diseño UI — dirección N (aprobada)

**Estado: aprobado** (2026-08-25). La dirección oficial de la interfaz del juego es la **Propuesta N**: merge de la estructura/UX del cliente Godot actual (Propuesta M) con la identidad visual holo de la Propuesta I (heredera de la B, la favorita).

- **Referencia viva**: [`prototipo-ui-n.html`](prototipo-ui-n.html) — abrir en navegador; toda duda visual se resuelve ahí, no en capturas.
- Historia del proceso: `prototipo-ui.html` (A) → `-b` → `-c` → `-d` → `-e` → `-h` → `-i` → `-m` → **`-n`**. Los anteriores son archivo, no referencia.
- Cambios registrados después de la aprobación: reparto del teclado (§6), selector segmentado (§7),
  trazo blanco y rediseño del glifo de Ajustes (§10), desviación del cristal en Godot (§11),
  **un solo botón de ventana** (§1.1 y §4), la comprobación medida del ratio del minimapa (§8),
  las **ventanas contextuales** (§1.11) y el **aviso de peligro persistente** (§9, aprobado 2026-09-01
  en vivo con la zona radiactiva).
- Este documento es **la fuente de verdad** para cualquier interfaz nueva (mockups HTML, UI en Godot, CMS). Si un caso no está cubierto, se propone, se aprueba con el usuario y **se registra aquí**.

---

## 1. Leyes de UX (dictámenes aprobados)

1. **Todo es ventana.** Cada elemento del HUD (nave, usuario/general, sector, objetivo, chat, registro, minimapa, boosters, misiones…) es una ventana: se minimiza y se reabre desde su icono. **Un solo botón, `–`**: en un juego donde toda ventana vuelve desde la taskbar, "cerrar" y "minimizar" no se distinguen en nada —ni en lo que hacen ni en cómo se vuelve—, y dos botones para una acción solo obligan al jugador a preguntarse cuál es cuál. El `×` era herencia del escritorio y queda retirado (2026-08-26).
2. **Un solo menú de iconos.** Todas las ventanas viven en la taskbar superior izquierda, con el **mismo estilo** de botón para todas. Nada de dos familias de iconos.
3. **Un solo código de estado: ámbar = abierta.** Ventana abierta → icono ámbar (borde superior + punto). Cerrada o minimizada → icono neutro. Sin cian para estados, sin doble código.
4. **Solo iconos, cero texto en el layout.** Los nombres son cadenas localizables y viven **exclusivamente en tooltips**. Un idioma nunca puede romper el ancho de una barra.
5. **Las ventanas se abren desde su icono** del menú; clic sobre una abierta la cierra/minimiza (toggle).
6. **Todas las ventanas se arrastran** con clic sostenido en su barra de título (la ventana General, desde cualquier punto). La posición se clampa al viewport y se persiste (en juego real: server + local).
7. **La pantalla debe poder quedar limpia**: minimizando todo solo quedan las barras de acción, el menú y la sysbar.
8. **Al menos dos barras de acción** (I: teclas 1–0; II: F1–F10).
9. **Sysbar** arriba a la derecha, fuera del menú de ventanas: ajustes ⚙, ayuda ?, pantalla completa ⛶, salir ⏻ (salir con hover hostil). Ajustes es ventana y también marca ámbar al abrir.
10. **Clic en una ventana la trae al frente**; z-index incremental.
11. **La cercanía condiciona las ACCIONES, no la ventana.** Una ventana que depende del contexto —la Estación, que solo sirve atracado— se abre y se cierra desde su icono como todas, y lo que cambia con el contexto es qué se puede **hacer** dentro: fuera de rango sus botones quedan bloqueados (§6) y una línea dice por qué. Un botón apagado enseña que ahí hay algo; una ventana que desaparece no enseña nada. Puede **abrirse sola** al entrar en contexto, pero si el jugador la cierra estando dentro, no se le vuelve a abrir hasta que salga y vuelva: una ventana que se reabre sola tras cerrarla no es cómoda, es terca.

## 2. Tokens de color

| Token | Hex/valor | Uso |
|---|---|---|
| `--bg` | `#07070F` | fondo del espacio |
| `--glass` | `rgba(13,17,29,.74)` | fondo de ventana (con blur 12px) |
| `--glass-2` | `rgba(13,17,29,.55)` | fondo de barras (taskbar, sysbar, botones sueltos) |
| `--edge` | `rgba(0,229,255,.35)` | borde activo / contenedores con identidad |
| `--edge-soft` | `rgba(120,140,180,.22)` | borde de reposo, separadores |
| `--cyan` | `#00E5FF` | acento primario: esquinas L, banda de título, hover, selección |
| `--violet` | `#A78BFA` | acento secundario: portales, especiales |
| `--warn` | `#FFC85C` | **números/valores** (firma heredada de M) y **estado "abierta"** |
| `--hostile` | `#FF3D6E` | enemigos, peligro, salir, killscreen |
| `--hp` | `#3DF58C` | vida |
| `--shield` | `#4DA6FF` | escudo |
| ~~`--nano`~~ | ~~`#F5F03E`~~ | ~~nanocasco~~ — **v1 no tiene nano-casco** (ver abajo); el token queda retirado |
| `--txt` | `#E8F0FF` | texto principal |
| `--muted` | `#8A97B8` | etiquetas frías (la pareja del ámbar: etiqueta fría + número ámbar) |
| `--faint` | `#5A6784` | texto terciario, teclas |

**Colores de contador** (munición/consumibles, herencia del cliente): azul `#587bbd` (default), rojo `#ff5d5d` (bajo), verde `#3DF58C`, ámbar `#FFC85C`, cian `#00E5FF`, naranja `#ff8a4d`.

**Regla**: no inventar colores. Si hace falta uno nuevo, se propone y se registra aquí.

## 3. Tipografía

| Familia | Rol | Tamaños típicos |
|---|---|---|
| **Michroma** | títulos de ventana, etiquetas de sistema, tooltips, botones de identidad | 6.5–10px, `letter-spacing .08–.28em`, UPPERCASE |
| **Exo 2** | cuerpo, chat, descripciones | 10–13px |
| **JetBrains Mono** | **todos los números**, códigos de item, coordenadas, inputs técnicos | 8–12px, `tabular-nums` |

- Separador de miles con **punto**: `8.421.900` (herencia del cliente).
- Los números van en `--warn`; sus etiquetas en `--muted`. Esa pareja es la firma.

## 4. Chrome de ventana

Anatomía (ver `.fp` en el prototipo):

- Contenedor: `--glass` + `backdrop-filter: blur(12px)`, borde 1px `--edge-soft`, sombra `0 0 26px rgba(0,229,255,.06), 0 18px 44px rgba(0,0,0,.5)`.
- **Esquinas en L**: superior izquierda e inferior derecha, 13px, 1.5px `--cyan`.
- **Header 26px**: banda izquierda de 3px cian con glow; degradado horizontal `rgba(0,229,255,.12) → transparente`; chip de icono 16px (SVG cian); título Michroma 9px `.16em`; botón **`–`** de 17px (borde `--edge-soft`, hover cian) — uno solo, ver §1.1. Cursor `move`; es la zona de arrastre.
- **Grip diagonal** en la esquina inferior derecha (rayas 135°, `--edge`), señal de redimensionado.
- Ventanas de peligro (killscreen): mismas reglas con `--hostile` en esquinas y borde.

## 5. Taskbar (menú de ventanas)

- Contenedor `--glass-2` + blur, borde `--edge`, esquinas L, **cap "MENÚ" vertical** (Michroma 7px, `.28em`).
- Botones **44×44**, solo icono SVG 21px (stroke 1.6, `round`), color reposo `#A9B6CC`.
- Estados: hover = blanco + glow cian + fondo `rgba(0,229,255,.07)`; **abierta = ámbar** (borde superior 2px, glifo con glow ámbar, punto inferior).
- Tooltip bajo el botón: Michroma 7.5px cian sobre `rgba(7,10,18,.94)`.
- Separadores verticales 1px por grupos: *estado del piloto* (Nave, Usuario, Boosters, Misiones) · *economía* (Hangar, Tienda, Ensamblaje, Laboratorio) · *social* (Clan, Grupo, Piloto, P.E.T.) · *información* (Chat, Registro, Minimapa, Sistema estelar).
- Punto de notificación (`ndot`): 6px ámbar parpadeante, esquina superior derecha del botón.

## 6. Barras de acción

- Dos barras centradas abajo, etiquetadas **I** y **II** (Michroma 8px), separación 5px.
- **Slot hexagonal** 46×52 (`clip-path` hexágono), fondo degradado oscuro, brillo cian superior.
- Interior: tecla arriba (JetBrains 7.5px `--faint`) · código Michroma 7px centro · contador abajo (JetBrains 8px, **color de contador** según estado).
- Estados: hover `brightness(1.35)`; **selección de munición** = glow cian; **bloqueado** = 45% opacidad; **cooldown** = velo negro que baja + segundos `#ffcc33` (el código se oculta mientras).
- Activación al soltar el clic (no al presionar) — herencia del cliente, evita gastar ítems al iniciar un arrastre.
- **Paleta de categorías**: flecha ▲ a la derecha de la barra II; abre fila de categorías encima, y al elegir una, la sub-fila de ítems encima de esa.

**Reparto del teclado.** Las dos barras se quedan **1–0 y F1–F10**, y eso es todo el teclado que el
juego reparte por ahora. **Ninguna ventana puede colgarse de una tecla de esas dos series**: los
Ajustes estuvieron en F1 y se habrían comido un slot en cuanto existan las barras. Las ventanas se
abren desde su icono (§1.5); si además necesitan atajo, sale de fuera de las series. **`Escape`
cierra la ventana enfocada** y es la única tecla reservada al margen de las barras.

## 7. Ventanas del juego (inventario)

Visibles al entrar: **Nave** (vida/escudo/bodega en barras segmentadas + velocidad/configuración), **Usuario** (experiencia, nivel, honor, jackpot, créditos, Flux, bonos de salto, llaves), **Boosters** (filas icono + nombre/código + `+N%` ámbar), **Laboratorio** (pestañas Refinamiento/Potenciación; Asterium/Nebulium/Coronium → Aurorium), **Grupo**, **P.E.T.** (retrato + badge de nivel + 4 barras finas + acciones), **Chat** (pestañas Global/Facción/Clan, nombres clicables, input con susurros), **Registro** (log con clases ok/warn), **Minimapa**.

Cerradas al entrar: **Misiones** (tracker de una misión con page-dots), **Hangar** (pestañas + ranuras equipadas + inventario en grid 42×42), **Tienda/Mercado** (tarjetas con chips y fila roja si no alcanza), **Ensamblaje**, **Clan**, **Piloto**, **Sistema estelar**, **Ajustes** (desde ⚙).

- **Barras segmentadas**: 96×11, relleno a rayas verticales de 4px del color de la stat sobre negro.
- **Fila insuficiente/roja**: fondo `rgba(255,61,110,.1)`, borde `rgba(255,61,110,.35)`.

### Selector segmentado (elegir una de pocas opciones)

Para un ajuste con **dos a cuatro** opciones excluyentes —la calidad gráfica es el primer caso—, en
vez de un desplegable va una fila de segmentos a la derecha de su fila `.r`, con el mismo estilo que
las pestañas `.ltab`: Michroma 7px `.1em` UPPERCASE, `padding 4px 10px`, hueco de 3, reposo en
`--muted` sobre `rgba(0,229,255,.04)` con borde `--edge-soft`.

**El elegido va en `--cyan`**, sobre `rgba(0,229,255,.12)` y borde `--edge` — igual que `.ltab.on`,
y **no en ámbar**. El ámbar del §1.3 significa "esta ventana está abierta" y ese código no se
comparte con nada: un segmento elegido es una pestaña activa, no un estado de ventana. Mezclarlos
deja al jugador sin saber qué le está diciendo el color.

Con más de cuatro opciones deja de caber en la fila y toca otra cosa; cuando aparezca el caso, se
propone aquí.

### Solo dos barras de estado: casco y escudo

**v1 no tiene nano-casco.** El original apilaba tres barras (vida, escudo y la
amarilla del nano-casco); aquí son **dos y solo dos** — `--hp` para el casco y
`--shield` para el escudo — tanto en la ventana Nave como sobre las naves en el
mundo. El token `--nano` queda retirado de la paleta.

Sobre la nave, el orden es **escudo arriba, casco abajo**, y el nombre va debajo
del casco. Cada stat se lee contra **su propio máximo**: nunca se suman casco y
escudo en una sola barra, porque esconde cuál de los dos se está gastando.

## 8. Minimapa

- Canvas con borde `--edge`; anchos por pasos `[180, 238, 300, 380, 460]` (botones − / + en el header); alto = ancho / ratio del mapa (20800×12800 → 1.625). **Nunca se deforma**, y eso hay que **medirlo**: un mapa estirado sigue pareciendo un mapa en una captura, así que el ratio se compara contra un número en la prueba, no a ojo. En Godot el fallo llegó por ahí — la ventana no encogía al bajar el zoom, el contenedor estiraba el canvas y el mapa salía con el ancho viejo y el alto nuevo (640×260 contra el 1,625 que toca).
- **Título dinámico**: `"<Sector> · (x, y)"` con coordenadas vivas.
- Contenido: rejilla cian tenue, portales en violeta, hostiles `--hostile` con **anillo pulsante**, aliados `--hp`, neutros `--muted`, héroe cian con anillo respirando.
- **El encuadre de la cámara** (herencia del cliente 3D de DarkOrbit, guidelines §4): las cuatro esquinas del viewport llevadas al mapa, dibujando **solo el 12.5% de cada lado desde cada esquina** — un marco entero taparía el contenido. Trazo `--txt` al 45%, 1px. Crece y encoge con el zoom: es la única lectura de "cuánto estoy viendo".
- **Clic = autopiloto**: línea punteada ámbar animada hacia el destino con **X** de 8px, y línea en el Registro.

## 9. Overlays y flujos

- **Toast** centro-superior: Michroma 10px cian sobre panel oscuro con borde `--edge`, 2.6s.
- **Pila de recompensas** bajo el toast: líneas ámbar `+ N × recurso`, fade ~3s, máximo 8.
- **Aviso de peligro persistente** (2026-09-01, primer caso: la **zona radiactiva**). Es la anatomía
  del toast con la sustitución del §4 para peligro: Michroma 10px `.14em` UPPERCASE, panel
  `rgba(7,10,18,.85)`, borde 1px, centro-superior a 88px — pero texto y borde en `--hostile`. Dos
  diferencias con el toast, y las dos son lo que significa "persistente": **no caduca** (vive
  mientras dure la condición) y **late** (opacidad entre 0,55 y 1 cada 1,2 s — el pulso es el
  idioma de peligro del sistema, el mismo del anillo hostil del minimapa §8). Debajo, una **viñeta**
  `--hostile` en los bordes del viewport (radial, transparente al centro, 18% en el borde) latiendo
  en fase: la lectura periférica, para que se sienta sin leer. Entra y sale con el fundido del toast
  (.25s). No es ventana (§1 no aplica: no se abre ni se cierra, la condición manda) y no lleva
  chrome. Al entrar y salir, una línea en el Registro (hostil / fría). En Godot: `ui/radiation_warning.gd`.
- **Killscreen**: velo `rgba(5,7,13,.97)` que tapa todo + ventana hostil con 4 opciones (base gratis en verde `--hp`; portal / en este punto / reparación completa en créditos). Menciona el costo de durabilidad.
- **Logout**: cuenta atrás JetBrains 30px ámbar, botón cancelar hostil.
- **Números de combate**: JetBrains **24px** con contorno negro (6), flotando sobre la entidad golpeada (suben 42 px en 1 s y se desvanecen). Viven en el **mundo**, no en el HUD, así que el zoom de juego (0,621) los reduce — 24 se ven como ~15 reales; con los 15 originales el golpe pasaba desapercibido. Daño que haces `#FF0000`, daño que recibes `#DB63E2` (herencia del cliente clásico, como los contadores de las barras); golpes seguidos del mismo color sobre el mismo blanco se acumulan en el número vivo.
- **Login**: logo Michroma + card glass con banda cian; campos API/GameServer/usuario/contraseña; secuencia de estados real ("Autenticando…" → "Sesión HTTP OK…" → "Socket abierto…").
- **Alta de cuenta**: **el mismo card que el login**, no una pantalla aparte. Los dos modos se eligen con el **selector segmentado del §7** (`ENLACE` / `ALTA`) en la cabecera del card; en `ALTA` aparecen dos campos más (correo y nombre de piloto) y el botón cambia de rótulo. Al crear la cuenta **entra solo**: el jugador acaba de teclear esos datos y volver a pedírselos no comprueba nada.

  Una pantalla aparte habría significado un segundo logo, un segundo card y un botón de "volver" — tres piezas nuevas para un formulario que comparte dos de sus cuatro campos con el que ya existe. El selector segmentado ya estaba en el sistema para justo esto: elegir entre dos y cuatro opciones excluyentes.

  **Los errores dicen cuál es el caso**, no "datos inválidos": "ese usuario ya existe" se arregla cambiando el nombre y "el registro está cerrado" no se arregla de ninguna manera. Juntarlos en un mensaje genérico hace que el jugador pruebe diez nombres contra una puerta cerrada. Fallo en `--hostile`, cuenta creada en `--hp`.

  Las pantallas de entrada (login y alta) **no son ventanas** y no llevan el chrome del §4. El §1 dice "todo es ventana" del juego; antes de entrar no hay escritorio del que formar parte, ni taskbar desde la que reabrir.

## 10. Iconografía

- SVG `viewBox="0 0 24 24"`, `stroke` 1.6–1.7, `fill:none`, `stroke-linecap/linejoin: round`.
- Un `<symbol>` por concepto, reutilizado vía `<use>`. Glifos simples de una idea (nave, matraz, bandera, grafo…), sin relleno ni detalle interno.
- **El trazo va en blanco puro**, nunca en el color final. En el prototipo lo pone `currentColor`; en
  Godot lo pone `modulate` sobre la textura, y un icono ya coloreado **no se puede teñir de ámbar**
  al abrir su ventana. Es la regla que hace funcionar el §1.3 en el cliente.
- **Prohibido el relleno, no la forma.** El engranaje de Ajustes era un círculo con ocho rayos
  sueltos y a 16px se leía como un **sol**: los rayos separados de la corona no dicen "mecanismo".
  Se redibujó como silueta dentada cerrada —ocho dientes de 20° a radio 9,1 sobre valles a 6,3, más
  un buje de 2,9— y ahí sí se lee. Cuando un glifo necesita su contorno para significar algo, el
  contorno **es** la idea y no cuenta como detalle interno.

## 11. Aplicación en Godot (cuando toque)

- Resolución lógica base 1280×720; la UI no escala con el zoom del mundo.
- Tokens → `Theme` central (StyleBoxFlat con los colores de §2; sin texturas 9-slice heredadas).
- Fuentes a incluir en el proyecto: Michroma, Exo 2, JetBrains Mono (hoy el cliente usa la default de Godot; se reemplaza).
- El chrome de `FloatingPanel` se re-estiliza a §4 conservando su comportamiento (drag en `_process`, clamp, persistencia de layout en server + `user://ui_state.cfg`).
- El chrome vive en **una sola pieza reutilizable** (`NWindow`), no copiado por ventana: tres
  cabeceras hechas a mano son la forma de que este documento deje de ser la fuente de verdad.

### Desviación conocida: el cristal no lleva desenfoque

`--glass` va con `backdrop-filter: blur(12px)` y **Godot no lo da gratis**: haría falta un
`BackBufferCopy` con shader por ventana. Sin él el color es el correcto, pero el fondo se transparenta
**nítido**, así que sobre una nave o un planeta la ventana se lee más ruidosa que en el prototipo.

**Es la única desviación conocida del §4**, y se deja anotada en vez de disimularla subiendo la
opacidad: cambiar el token por una ventana rompería el token para todas. Si el ruido llega a molestar,
la salida es el shader, no la paleta.

## 12. Checklist para toda interfaz nueva

- [ ] ¿Usa solo tokens de §2 y tipografías de §3? (nada inventado)
- [ ] ¿Números en JetBrains ámbar con miles con punto, etiquetas frías?
- [ ] ¿Es ventana? → chrome §4 completo (esquinas L, banda cian, `–`/`×`, drag, grip)
- [ ] ¿Tiene icono en la taskbar con tooltip y estado ámbar al abrir?
- [ ] ¿Cero texto fijo en barras/botones de icono? (i18n solo en tooltips)
- [ ] ¿Estados de peligro en `--hostile`, éxito en `--hp`?
- [ ] ¿Se probó minimizar/reabrir/arrastrar y que la pantalla pueda quedar limpia?
- [ ] ¿Lo nuevo que no estaba cubierto quedó registrado en este documento?
