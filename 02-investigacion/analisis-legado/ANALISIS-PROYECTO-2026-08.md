# MexOrbit — Análisis del proyecto y del entorno de producción

**Fecha del análisis:** 12 de agosto de 2026
**Alcance:** código local (`C:\Source\MexOrbit`), servidor de producción (`74.208.108.67`, por SSH) y documentación de deploy existente.

---

## 1. Resumen ejecutivo

MexOrbit es un emulador privado de DarkOrbit compuesto por un **GameServer TCP** y una **API REST** en .NET 8, un **CMS web** en Vue 3 y un **cliente de escritorio Electron** con Flash PPAPI. Todo está desplegado y **funcionando en producción** en una VM Ubuntu 24.04 (IONOS), estable desde el 1 de agosto sin errores en logs.

Estado general:

- ✅ **Producción sana**: API y GameServer activos 11+ días, memoria y CPU holgadas, certificados vigentes, sin warnings/errores en `journalctl` en los últimos días.
- ⚠️ **Sin respaldos automáticos de la BD `darkorbit`** — el único dump es el baseline de abril. Es el riesgo operativo nº 1.
- 🚧 **Trabajo local sin commitear**: fix de reensamblado de tramas TCP en el GameServer + utilidad de trazado de paquetes (`GameNetPacketTrace.cs`). Producción NO tiene este fix.
- 📉 Deuda técnica conocida y acotada: doble capa de acceso a datos, SQL interpolado en el GameServer, docs con referencias rotas, restos legacy (`src/`, scripts de arranque viejos del Client).

---

## 2. Mapa del proyecto

```
C:\Source\MexOrbit\
├── MexOrbit.Server\      Backend .NET 8 (repo git propio, rama init-code)
│   ├── Api\              MexOrbit.Api.External (REST) + MexOrbit.Api.Health (vestigial)
│   ├── Service\          MexOrbit.GameServer (emulador TCP, proyecto aislado)
│   ├── Domain\ Common\ Infrastructure\   Clean/Onion architecture de la API
│   ├── Scripts\          Migraciones SQL versionadas (YYYY.MM.DD.N) + baseline
│   ├── deploy\           Guías VM, systemd, nginx, env (fuente de verdad del deploy)
│   └── src\              ☠️ MUERTO: solo obj/ huérfanos, sin código (borrable)
├── MexOrbit.CMS\         Frontend Vue 3 + TS + Vite (repo git propio, rama init)
├── MexOrbit.Client\      Electron 11 + Flash PPAPI → carga el CMS remoto
├── MexOrbit.Tools\       CompareBase64Json, do-swf-decryptor
├── Tools\                AirAtfProbe, scripts de spacemap
├── Decompile\            SWFs decompilados (spacemap, inventory, space, images)
├── docs\                 Base de conocimiento (protocolo, flujos, decompilado)
└── Legacy\               Código original de referencia (.NET Framework + CMS PHP)
```

Notas:

- La **raíz no es un repo git**; `MexOrbit.Server` y `MexOrbit.CMS` son repos independientes en Azure DevOps (`evolabsmx/MexOrbit`).
- El README raíz menciona `MexOrbit.UnityClient` y `docs/knowledge-database.md` referencia `MexOrbit.GodotClient`; **ninguna de las dos carpetas existe** en este equipo. Si esos clientes viven en otro repo/máquina, conviene documentarlo; si se abandonaron, limpiar las referencias.

---

## 3. Producción (VM `74.208.108.67`)

### 3.1 Infraestructura

| Elemento | Valor |
|---|---|
| SO | Ubuntu 24.04.4 LTS (kernel 6.8), uptime 81 días |
| Recursos | 24 GB RAM (3.3 usados), disco 697 GB (3% usado) |
| Acceso | SSH `root@74.208.108.67` (clave, puerto 22) |
| Runtime | .NET SDK 8.0.129, Node v20.20.2, MySQL 8.0.46 |
| Multi-tenant | La VM también aloja **Gallos** (puertos 3000/5093) y **operacionacarreo.com** (5150) |

### 3.2 Servicios MexOrbit

| Servicio | Unit / mecanismo | Puerto | Estado |
|---|---|---|---|
| API REST | `mexorbit-api.service` → `/opt/mexorbit/api` | 127.0.0.1:5007 (solo local, tras Nginx) | ✅ activo desde 01-ago |
| GameServer | `mexorbit-gameserver.service` → `/opt/mexorbit/gameserver` | 8080 (juego), 9338 (chat), 4301 (IPC API) | ✅ activo desde 01-ago |
| CMS | estático Nginx → `/var/www/mexorbit-cms` | 443 | ✅ responde 200 |
| MySQL | `mysql.service` | 127.0.0.1:3306 | ✅ |

Dominios y certificados (Let's Encrypt, renovación automática con `certbot.timer`):

- `https://mex-orbit.turname.mx` → CMS (expira 20-sep-2026)
- `https://api.turname.mx` → API (expira 20-sep-2026)

Conectividad externa verificada: **8080 y 9338 abiertos**; **4301 bloqueado desde fuera por el firewall del proveedor** (IONOS), aunque UFW lo permite. No es un problema real: 4301 solo lo usa la API localmente (`127.0.0.1`). La regla UFW de 4301 podría eliminarse por higiene.

### 3.3 Versiones desplegadas vs. local

| Repo | Rama | Commit en PROD | Local |
|---|---|---|---|
| MexOrbit.Server | `init-code` | `c4f8608` (26-abr) "Agregando creditos y uridium a la administracion" | mismo commit **+ cambios sin commitear** (§7) |
| MexOrbit.CMS | `init` | `cf1eafc` (30-abr) "3D" | mismo commit |

En la VM el working tree del CMS tiene `base_563.xml` modificado (host del chat inyectado en build) y ambos repos tienen su `.env.production` local sin versionar — esperado según el flujo de release.

### 3.4 Base de datos

- Esquema `darkorbit`: **40 tablas, ~3.3 MB, 25 cuentas de jugador**.
- Migraciones: carpetas SQL manuales en `MexOrbit.Server/Scripts/` (§4.4); no hay tabla de control de qué se aplicó.
- **Respaldos: NO automatizados.** Solo existe `/home/source/dbdumps/darkorbit_baseline_20260422_225115.sql` (abril). El cron de la VM solo respalda la BD de Gallos.

---

## 4. Backend .NET (`MexOrbit.Server`)

### 4.1 Solución

11 proyectos net8.0 en arquitectura onion. La API (`MexOrbit.Api.External`) usa EF Core 8 + Pomelo MySQL, con DI centralizada en `Common/MexOrbit.Dependencies`. El GameServer (`Service/MexOrbit.GameServer`) es un **ejecutable aislado sin referencias a proyecto**: emulador legacy portado (~393 .cs) con su propio acceso a datos por SQL crudo (`MySql.Data`).

La carpeta `MexOrbit.Server/src/` es un resto muerto (solo `obj/` de NuGet) y el `README.md` del server describe esa estructura antigua — ambos a limpiar.

### 4.2 GameServer

- **Entry point** `Program.cs`: chequeo MySQL con reintentos → carga de catálogos (`QueryManager.LoadClans/LoadShips/LoadMaps`, catálogos de recursos) → arranque de listeners → keep-alive (soporta consola y systemd). Comandos de consola: `/restart <segundos>`, `/list_players`.
- **Puertos hardcodeados en código**: 8080 (`Net/GameServer.cs`), 9338 (`Net/ChatServer.cs`), 4301 (`Net/SocketServer.cs`). El `GameServer:Port` de `appsettings.json` es **configuración muerta** — nadie la lee.
- **Estado global** en `Managers/GameManager.cs` (`ConcurrentDictionary` de sesiones, mapas, clanes…).
- **`Net/SocketServer.cs`**: IPC JSON con la API (~23 acciones: BuyItem, ChangeShip, clanes, diplomacia, pilot skills, bans…).
- **Ticks**: `TickManager` con bucle async; la constante `TICKS_PER_SECOND = 84` en realidad son **milisegundos de delay** (~11.9 ticks/s reales) y la `List<Tick>` no es thread-safe.
- **Zona horaria**: usa `DateTime.Now` extensivamente → la lógica de juego depende del reloj del SO de la VM (documentado en DEPLOY-VM-SIMPLE.md §2).

### 4.3 API REST

- Controllers: `api/players` (registro/login/perfil), `api/v1/{shop,equipment,clans,pilot-skills,auctions,assembly,bonuses}` con `[Authorize]`, `api/v1/admin/*` con política `CmsAdmin`, y `flashAPI/*` + `flashinput/*` para el cliente Flash.
- **Auth JWT stateful**: en `OnTokenValidated` se compara el claim `sessionId` contra la BD — el logout invalida tokens. Generación en `JwtService.cs` (HMAC-SHA256, exp. 1440 min).
- **Ojo**: el esquema `FlashAuth` (middleware de sesiones Flash) está registrado pero **ningún controller lo usa** — los endpoints `flashAPI/*` son públicos hoy.
- Background service de liquidación de subastas (`AuctionSettlementHostedService`).
- `MexOrbit.Api.Health` no está desplegado ni referenciado — vestigial.

### 4.4 Migraciones SQL (`Scripts/`)

Convención: carpeta `YYYY.MM.DD.N/` con `rollout.sql` + `rollback.sql` + `README.md`, aplicadas **a mano en orden** con `mysql -u root -p darkorbit < .../rollout.sql`. Baseline "oro" generable con `Scripts/baseline/generate-from-dev.ps1`.

Estado real: 33 carpetas (2025.12.03 → 2026.04.25), pero **18 sin rollback**, 6 sin `rollout.sql` canónico y solo 8 con README. No hay tabla de control de versiones aplicadas: saber "qué falta en PROD" depende del operador.

### 4.5 Secretos

- `appsettings.json` versionados llevan credenciales **de desarrollo** (root/root a 127.0.0.1) y el placeholder de `Jwt:SecretKey`. En producción los valores reales viven en `/etc/mexorbit/api.env` (chmod 600) y `/opt/mexorbit/gameserver/appsettings.Production.json` (chmod 600), fuera de git. Correcto, pero el `.gitignore` **no** protege `appsettings.Production.json` ante un despiste.

---

## 5. CMS (`MexOrbit.CMS`)

- **Rutas**: públicas (`/`, `/login`, `/register`) y autenticadas — `/dashboard`, `/hangar`, `/shop`, `/clan`, `/bonuses`, `/pilot-skills`, `/assembly`, `/auction`, `/company-select`, `/profile`, `/map-revolution` (embebe el juego Flash) y panel `/admin/*` (guard `isCmsAdmin`).
- **Estado**: solo 2 stores Pinia (`auth`, `equipment`, 996 líneas); el resto es estado local por vista. Vistas grandes: `ShopView` (1937 líneas), `ClanView` (1752), `AdminBonusesView` (1697).
- **API**: base `VITE_API_BASE_URL` (fallback `http://localhost:5007/api`). Hay **3 instancias axios duplicadas** (`authService`, `equipmentService`, `shopService`); la mayoría de servicios importan el cliente de `shopService` (acoplamiento raro).
- **Flash**: `MapRevolutionView.vue` monta `spacemap/preloader.swf` vía `flashembed` con flashvars (`userID`, `sessionID`, `pid=563`, `host` sin esquema…). Assets pesados en `public/` (spacemap 347 MB, do_img 101 MB).
- **Chat**: `scripts/inject-gamechat-base563.mjs` sobrescribe `public/gamechat/as3/cfg/base_563.xml` con `VITE_CHAT_TCP_HOST:9338` antes de cada build. El XML commiteado apunta a `127.0.0.1` — sin la variable, el chat muere en producción.
- **Build/deploy**: `azure-pipelines.yml` compila y publica artefacto `cms-dist` pero **no despliega** (y su variable por defecto es un placeholder); el deploy real es manual (`pnpm build` en la VM + copia a `/var/www/mexorbit-cms`). La carpeta `MexOrbit.CMS/deploy/` está **vacía** aunque `deploy/README.md` de la raíz dice que ahí vive `DEPLOY-CMS.md`.
- Pendientes funcionales: `checkBans()` siempre devuelve "no baneado", `allowChat`/`autoStartEnabled` hardcodeados, menú URIDIUM deshabilitado.
- Higiene: `public.zip` de **494 MB** y `dist/` construido dentro del árbol del repo.

---

## 6. Cliente Electron (`MexOrbit.Client`)

- Electron **11.5.0** (obligado: Flash muere en versiones modernas) + `pepflashplayer.dll` 32.0.0.465 (presente en disco, **ignorado por git** — quien clone no tendrá Flash).
- No hospeda nada: carga `https://mex-orbit.turname.mx` (configurable por env `MEXORBIT_CLIENT_URL` o menú CMS con presets).
- Wallet de cuentas con `electron-store` cifrado con clave derivada de la ruta de userData (ofuscación, no seguridad real); autofill envía la contraseña **en claro por IPC** al renderer.
- Empaquetado `electron-builder` → portable x64 (`dist/MexOrbit-Client-1.0.0-portable.exe`, también copiado a `MexOrbit.CMS/public/client/` para el botón DOWNLOAD GAME). La carpeta `build/` (iconos) referenciada por el builder **no existe en el repo**.
- Restos a limpiar: scripts de arranque del modelo viejo ("el cliente levanta el CMS"): `start-dev.bat`, `start-dev.js`, `start-manual.bat`, `dev-simple.js`; rama muerta `game.html` en `setWindowOpenHandler` (la ruta real es `/map-revolution`); logging forense de red masivo en `main.js`.

---

## 7. Trabajo en curso (sin commitear, solo local)

En `MexOrbit.Server` (rama `init-code`, encima de `c4f8608`):

| Archivo | Cambio |
|---|---|
| `Net/GameClient.cs` | **Fix de framing TCP**: acumulador `_receiveBuffer` + `DrainFramedMessages()` — reensambla tramas Netty `[ushort BE len][body]` partidas o coalescidas por TCP. Antes, el excedente se perdía y el stream quedaba desalineado (`cmd=0`, tamaños absurdos). Incluye guardas de trama inválida y límite de 40 MB. |
| `Net/netty/Handler.cs` | Ajustes del dispatcher acordes al framing. |
| `Net/netty/handlers/LoginRequestHandler.cs` | Ajustes de login (trazado). |
| `Utils/Logger.cs` | Refactor de logging. |
| `Utils/GameNetPacketTrace.cs` | **Nuevo, sin trackear**: utilidad de trazado de paquetes de red (drops sin sesión, diagnóstico de protocolo). |

Este trabajo encaja con el objetivo de portar clientes nuevos (Unity/Godot) que estresan el protocolo más que el cliente Flash. **Producción no tiene este fix**: el siguiente paso natural es terminarlo, commitear y desplegar.

---

## 8. Hallazgos y riesgos priorizados

| # | Prioridad | Hallazgo | Recomendación |
|---|---|---|---|
| 1 | 🔴 Alta | **Sin backup automático de `darkorbit`** (solo baseline de abril; el cron de la VM solo respalda Gallos) | Añadir cron diario de `mysqldump` con rotación (igual que el de Gallos) y, si se puede, copia fuera de la VM |
| 2 | 🔴 Alta | **Trabajo sin commitear** en el server local (fix de framing TCP) — riesgo de pérdida | Terminar, commitear y push a `init-code` |
| 3 | 🟠 Media | Endpoints `flashAPI/*` sin autenticación (el esquema FlashAuth existe pero no se aplica) | Aplicar `[Authorize(AuthenticationSchemes = "FlashAuth")]` o validar `sid` en los controllers Flash |
| 4 | 🟠 Media | Enlace de terceros `http://wolffr.ddns.net/Wolf_Fr Browser.zip` en `MapRevolutionView.vue:70` (DDNS ajeno que sirve un ZIP ejecutable a jugadores) | Eliminarlo o apuntar al portable propio (`VITE_CLIENT_DOWNLOAD_URL`) |
| 5 | 🟠 Media | Todo corre **como root** en la VM (servicios, repos, nginx estático) | Endurecer con usuario dedicado cuando haya ventana (los propios docs lo reconocen) |
| 6 | 🟠 Media | SQL interpolado por strings en `QueryManager.cs` del GameServer | Migrar gradualmente a parámetros (`MySqlParameter`), empezando por inputs de jugador |
| 7 | 🟡 Baja | `TICKS_PER_SECOND=84` usado como delay en ms (~11.9 ticks/s reales); `List<Tick>` no thread-safe | Renombrar a `TICK_INTERVAL_MS` y usar colección concurrente |
| 8 | 🟡 Baja | Sin registro de migraciones aplicadas; 18/33 carpetas de Scripts sin rollback | Tabla `schema_migrations` sencilla + disciplina de rollback en scripts nuevos |
| 9 | 🟡 Baja | EF Core declara `MySqlServerVersion(10.4.27)` (MariaDB) pero PROD es MySQL 8.0.46 | Cambiar a `MySqlServerVersion(new Version(8,0,46))` |
| 10 | 🟡 Baja | Referencias rotas en docs: `DEPLOY-CMS.md`, `AZURE-DEVOPS-PIPELINES.md`, `Scripts/test-api-login.ps1`, `DEPLOY-UBUNTU.md` no existen; READMEs desactualizados (server, raíz); clientes Unity/Godot referenciados pero ausentes | Pasada de limpieza documental |
| 11 | 🟡 Baja | Higiene de repos: `public.zip` 494 MB, `dist/` versionable, `src/` muerto, scripts legacy del Client, regla UFW 4301 innecesaria | Limpieza incremental |
| 12 | ⚪ Nota | `webSecurity: false` + CSP permisiva en el cliente Electron; contraseñas por IPC en claro; `crossdomain.xml` abierto a `*` | Inherente al soporte Flash/PPAPI; aceptable para servidor privado, pero documentado aquí para tenerlo presente |

---

## 9. Runbook rápido (producción)

```bash
# Estado de servicios
ssh root@74.208.108.67 "systemctl status mexorbit-api mexorbit-gameserver --no-pager"

# Logs en vivo
ssh root@74.208.108.67 "journalctl -u mexorbit-gameserver -f"

# Release del server (en la VM)
cd /home/source/MexOrbit.Server && git checkout init-code && git pull
dotnet publish Api/MexOrbit.Api.External/MexOrbit.Api.External.csproj -c Release -o /opt/mexorbit/api
dotnet publish Service/MexOrbit.GameServer/MexOrbit.GameServer.csproj -c Release -o /opt/mexorbit/gameserver
systemctl restart mexorbit-api mexorbit-gameserver

# Release del CMS (en la VM)
cd /home/source/MexOrbit.CMS && git checkout init && git pull
pnpm install --frozen-lockfile && pnpm run build
rm -rf /var/www/mexorbit-cms/* && cp -a dist/. /var/www/mexorbit-cms/ && chmod -R a+rX /var/www/mexorbit-cms

# Backup manual de BD
ssh root@74.208.108.67 "mysqldump --single-transaction --routines --triggers darkorbit | gzip > /home/source/dbdumps/darkorbit_$(date +%Y%m%d_%H%M%S).sql.gz"
```

Guías completas: `MexOrbit.Server/deploy/DEPLOY-VM-SIMPLE.md` y `MexOrbit.Server/deploy/RELEASE-GUIDE.md`.

---

*Documento generado a partir del análisis del 12-ago-2026. Para ampliaciones de protocolo/cliente Flash, ver `docs/knowledge-database.md` y `docs/decompile/`.*
