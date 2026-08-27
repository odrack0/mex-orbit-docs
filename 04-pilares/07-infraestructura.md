# Pilar 07 — Infraestructura y operación

**Transversal a todos los repos** · **Estado:** v1 publicada el 27-ago-2026; CI/CD y monitoreo por diseñar.

## Decisiones tomadas

1. **Los backups son la deuda #1 y no se negocian**: BD con respaldo automático diario + copia fuera
   de la máquina, desde el primer entorno con datos que duelan. (La lección viva: el prod del
   prototipo operó sin respaldo.) **Sigue sin cumplirse**: el despliegue de v1 dejó un volcado
   manual, no un `cron`. Se anota aquí y no en una lista de tareas porque es la única decisión de
   este pilar que llevamos incumpliendo desde que se tomó.
2. Entornos separados: dev → staging → producción. **Hoy solo hay dev y producción**; staging no
   existe todavía.
3. GitHub como plataforma de v1 (repos `odrack0/mex-orbit-*`); los repos legados quedan en Azure
   DevOps como archivo del prototipo.
4. **v1 comparte servidor con el prototipo** (`74.208.108.67`), y con dos proyectos que no son
   nuestros. No se separa porque no hace falta: 12 núcleos y 23 GB dan de sobra, y tener las dos
   versiones vivas a la vez deja comparar. La consecuencia es una regla de operación, no una
   arquitectura: nombres propios (`astrion-*`) para todo, y **`nginx -t` antes de cada `reload`**.
5. **Los dominios de turname.mx se reutilizan**, con subdominios propios: el prototipo se queda en
   `mex-orbit.turname.mx` y v1 vive en `astrion.turname.mx`.
6. **El cliente se distribuye como exportación Web.** Los testers abren un enlace; actualizar es
   reexportar. Se exporta **en el servidor**, no en el PC: el paquete pesa 120 MB y subirlo desde
   una conexión doméstica se atasca.

## La topología de v1

| Dominio | Sirve | Detrás |
|---|---|---|
| `astrion.turname.mx` | el juego y **la API** | estático en `/var/www/astrion`; `/api/` proxy a `127.0.0.1:5110` |
| `astrion-api.turname.mx` | herramientas y depuración | proxy a `127.0.0.1:5110` |
| `astrion-gs.turname.mx` | el mundo | `/ws` proxy a `127.0.0.1:5210` con *upgrade* de WebSocket |

**La API se sirve desde el mismo origen que el juego, y esa es la decisión menos obvia de todas.**
La API de v1 no tiene CORS configurado, así que un navegador que cargue el juego en un dominio y
llame a otro se lo bloquea. Mismo origen lo resuelve sin escribir código, y de paso deja la API sin
superficie pública que endurecer. El subdominio propio sigue existiendo porque es cómodo para
`curl`, no porque el juego lo use.

**v1 habla WebSocket nativo.** El prototipo necesita dos puentes Node (`wsbridge`, `wschat`) para
que el navegador llegue a un servidor que solo entiende TCP; aquí nginx proxya directo y esos dos
procesos no existen. Es la diferencia más visible entre las dos infraestructuras.

## Dónde vive cada cosa

| Cosa | Ruta | Por qué ahí |
|---|---|---|
| API publicada | `/opt/astrion/api` | la reescribe cada despliegue |
| Game server publicado | `/opt/astrion/gs` | ídem |
| Configuración real | `…/appsettings.Production.json` | fuera del repo: lleva credenciales |
| Claves Ed25519 del ticket | `/var/lib/astrion/keys` | **fuera** de la carpeta de publicación: un redespliegue que se las llevara por delante invalidaría todos los tickets sin avisar |
| Cliente web | `/var/www/astrion` | el `.pck` se publica con `mv` atómico |
| Código fuente | `/home/astrion/mex-orbit-v1/` | los repos clonados **como hermanos**: los `.csproj` compilan el protocolo por ruta relativa |

## Cómo se despliega

Cada repo trae lo suyo y todo sale de `main` — **nada llega a producción sin pasar por un commit**.
Los detalles y las trampas de cada pieza están en el README de su repo.

```bash
ssh root@74.208.108.67 'bash -s' < mex-orbit-api/deploy/deploy.sh
ssh root@74.208.108.67 'bash -s' < mex-orbit-game-server/deploy/deploy.sh
ssh root@74.208.108.67 'bash -s' < mex-orbit-client/tools/deploy-web.sh
```

Y **tras cualquier migración que toque `map` o `map_server`**, correr
`mex-orbit-data-base/deploy/produccion.sql`. Si se olvida, el juego entra bien y falla justo al
**saltar de sector**, porque el host del salto sale de la BD y no del que usó el cliente para
entrar. Es el fallo más caro de diagnosticar del despliegue: no aparece hasta el segundo mapa.

## Lo que enseñó el primer despliegue

Cuatro fallos, y los cuatro de la misma familia: **cosas que en desarrollo no pueden verse**.

- **El registro público daba cuentas sin nave.** Insertaba la fila de `account` y nada más; esa
  cuenta hace login, abre el socket y rebota. En dev no se ve nunca porque las cuentas de dev nacen
  por el sembrador, que sí monta nave, estado, cartera y equipo. La puerta por la que entran los
  testers era la única que nadie había cruzado.
- **La URL de la API estaba escrita a mano** como `127.0.0.1`, que en el navegador de un tester
  apunta a su propia máquina.
- **`HttpClient.BaseAddress` se comía el prefijo `/api`**: una ruta relativa que empieza por `/`
  reemplaza el camino de la base. Solo falla cuando la API se sirve bajo un prefijo — o sea, solo en
  producción.
- **Un guardián de despliegue que se saltaba a sí mismo.** Comprobaba con `strings`, que no está
  instalado en el servidor, y al ir dentro de un `if` el fallo se leyó como «todo bien».

La conclusión operativa no es «revisar mejor», es que **la prueba de humo tiene que correrse contra
producción con una cuenta recién creada**. Las cuatro se cazan así y ninguna se caza en dev.

## Por definir

- CI/CD por repo (build + tests + despliegue), y el runner de migraciones integrado al despliegue.
- **Respaldo automático**: hoy solo hay un volcado manual. Es la decisión #1 y sigue incumplida.
- Staging: hoy se salta de dev a producción.
- Monitoreo: métricas del tick del game server, alertas (el "server congelado sin log" del legado no
  puede repetirse — health checks reales), logs estructurados.
- Destino del prototipo en producción (¿laboratorio congelado?).
- Un solo proceso sirve los 29 mapas. El diseño ya contempla repartirlos —el salto *siempre* negocia
  conexión, aunque hoy sea contra sí mismo, y el directorio está en `map_server`— pero hoy si se cae
  el proceso se caen los 29.
