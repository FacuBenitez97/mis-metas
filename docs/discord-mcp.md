# Conectar Discord por MCP

Este repo incluye `.mcp.json`, que configura el servidor MCP
[`@pasympa/discord-mcp`](https://github.com/PaSympa/discord-mcp) (MIT) para que Claude Code
pueda administrar tus servidores de Discord.

## Antes de empezar: qué se puede y qué no

**Sí se puede** (con el bot invitado y con permisos):

- crear, editar, mover y borrar canales, categorías, foros e hilos
- crear y editar roles, asignarlos y quitarlos, ordenarlos, setear permisos
- administrar miembros: kick, ban, unban, timeout, nickname, prune, búsqueda
- invitaciones, webhooks, eventos programados, audit log, estadísticas del servidor
- leer y enviar mensajes, embeds, reacciones, DMs

**No se puede: crear servidores nuevos.** Ninguno de los servidores MCP de Discord que hay
publicados expone esa operación, y la API de Discord la restringe fuerte: `POST /guilds` solo
funciona para bots que estén en menos de 10 servidores, y el servidor creado queda a nombre del
bot, no tuyo — transferirte la propiedad después es engorroso y Discord lo desalienta.

En la práctica: **los servidores los creás vos a mano** (toma diez segundos en la app), invitás
al bot, y a partir de ahí Claude te lo administra entero. Si de verdad necesitás creación
automatizada, hay que escribir un servidor MCP propio contra `POST /guilds` y aceptar esas
limitaciones.

## Requisitos

- Node.js 22 o superior (lo exige el paquete)
- Claude Code de escritorio o CLI. **Esto no funciona en sesiones web de claude.ai**: los
  servidores MCP de `.mcp.json` corren como proceso local, y el contenedor de la sesión web es
  efímero.

## Paso 1: crear el bot

1. Entrá a <https://discord.com/developers/applications> y hacé **New Application**.
2. En la pestaña **Bot**, hacé **Reset Token** y copiá el token. Se muestra una sola vez.
3. En esa misma pestaña, activá los **Privileged Gateway Intents** que necesites:
   - *Server Members Intent* — para listar y administrar miembros
   - *Message Content Intent* — para leer el contenido de los mensajes

## Paso 2: invitar el bot a tu servidor

En **OAuth2 → URL Generator**, marcá los scopes `bot` y `applications.commands`, elegí los
permisos que quieras darle, y abrí la URL que se genera para invitarlo.

Sobre permisos: *Administrator* es lo más cómodo y lo más peligroso. Un token con Administrator
que se filtra permite borrar tu servidor entero. Si el servidor te importa, dale permisos
puntuales (Manage Channels, Manage Roles, Kick/Ban Members, Send Messages, Manage Messages) en
vez del combo completo.

## Paso 3: configurar el token localmente

```bash
cp .env.example .env
# editá .env y pegá el token en DISCORD_TOKEN
```

Llená también `DISCORD_ALLOWED_GUILDS` con los IDs de los servidores que querés administrar
(separados por coma). Es lo que evita que un error o un prompt malicioso toque un servidor que no
era. Para copiar un ID: activá **Configuración → Avanzado → Modo desarrollador** en Discord,
después click derecho sobre el servidor → *Copiar ID*.

`.env` está en `.gitignore`. El token nunca se commitea: `.mcp.json` solo referencia
`${DISCORD_TOKEN}` y lo resuelve desde tu entorno al arrancar.

Si tu shell no carga `.env` solo, exportá las variables antes de abrir Claude Code:

```bash
set -a && source .env && set +a
claude
```

## Paso 4: activarlo

Abrí Claude Code en este repo. Va a pedirte aprobar el servidor MCP del proyecto la primera vez
— aceptá. Verificá con `/mcp`, que debería listar `discord` como conectado.

## Si algo falla

- **`discord` no aparece en `/mcp`** — el `.mcp.json` del proyecto no fue aprobado, o Node es
  menor a 22. Revisá con `node --version`.
- **El servidor arranca y se cae** — casi siempre es el token: vacío, mal pegado o revocado.
  Confirmá que `DISCORD_TOKEN` está en el entorno con `echo ${DISCORD_TOKEN:0:10}`.
- **"Missing Access" o "Missing Permissions"** — el bot está invitado pero le falta el permiso
  para esa acción puntual, o el rol del bot está por debajo del rol que intenta modificar.
  Discord no deja que un rol administre roles que estén más arriba que el suyo.
- **No lee mensajes** — falta el *Message Content Intent* del Paso 1.

## Nota sobre confianza

`@pasympa/discord-mcp` es un paquete de la comunidad, sin respaldo oficial de Discord ni de
Anthropic. Lo elegí sobre las alternativas porque es MIT, depende solo del SDK oficial de MCP,
discord.js, dotenv y zod, y porque es el único que deja limitar el alcance por servidor y por
grupo de herramientas. Aun así, va a recibir tu token de bot: si te parece bien, revisá el
código en <https://github.com/PaSympa/discord-mcp> antes de darle permisos amplios, y fijá la
versión (ya está pinneada a `2.1.0` en `.mcp.json`) para que un release nuevo no entre solo.
