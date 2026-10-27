# Cat Rack

Cat Rack is a browser dashboard for monitoring and managing Minecraft servers on
a host running the [CatRack Host Agent](https://github.com/unknowncodr/catrack-backend).
The dashboard is a static website: it connects directly to the agent through a
WebSocket and does not run a backend of its own.

## Features

- Discover server folders and view server status, Minecraft version, and port.
- Start, stop, and restart servers.
- View live player count, CPU and memory use, uptime, and (where supported) TPS.
- Review recent console output and send console commands.
- Inspect online and previously known players, their available saved data and
  statistics, and manage player permissions.
- Change a server's RAM override and port for its next start.
- View recent CPU, memory, and TPS history in the server detail view.

## Requirements

- A modern browser with JavaScript, WebSocket, and `sessionStorage` support.
- A running CatRack Host Agent that the browser can reach.
- The agent's tunnel or reverse proxy must support WebSocket upgrades. Use HTTPS
  for the dashboard and a secure WebSocket (`wss://`) for the agent when
  connecting over the public internet.

The dashboard uses Google Fonts and loads player head images from
`api.mcheads.org`; those optional external services require an internet
connection.

## Connect to a host

1. Start and configure the Host Agent by following its
   [setup instructions](https://github.com/unknowncodr/catrack-backend#setup).
   Keep its `AGENT_TOKEN` private.
2. Open the Cat Rack dashboard.
3. Enter the host's reachable **HTTP or HTTPS base URL** (for example,
   `https://your-tunnel.example.com`) and the same token configured as
   `AGENT_TOKEN`, then select **Connect**. Do not include `/ws` or a token in the
   URL; the dashboard builds the WebSocket URL itself.
4. Select a server to open its details. Use **Rescan** on the server list to
   refresh discovery, and **Disconnect** to clear the saved connection and
   return to the login screen.

The URL must route `/ws` to the agent's WebSocket endpoint. The dashboard
automatically uses `wss://` for an HTTPS URL and `ws://` for an HTTP URL.
Connection details are stored in the current tab's `sessionStorage` so the
dashboard can reconnect after a refresh; they are cleared when the tab's
session ends or when you disconnect.

### Server controls

- **Start / Stop / Restart** control the selected server.
- **Console** displays recent output and accepts commands while the server is
  running.
- **RAM** accepts an override from 512 to 65,536 MB. Leave it blank to clear
  the override and use the server's own start script when available.
- **Port** accepts a port from 1 to 65,535. Port and RAM changes apply when the
  server next starts.
- **Players** shows online players and players previously found in the server's
  user cache. Selecting a player opens available saved details and actions.

TPS/MSPT data depends on support in the Minecraft server version; the Host Agent
uses the `tick query` command for Minecraft 1.20.3 and newer.

## Run locally

No package installation, compilation, or build step is needed. From the
frontend directory, start a static file server:

```powershell
py -m http.server 8080
```

Then open <http://localhost:8080>. Connect using the agent's reachable base
URL and token. Stop the local web server with `Ctrl+C`.

Opening `index.html` directly from the filesystem may cause browser restrictions
and is not the recommended way to preview the dashboard.

## Deploy

Publish `index.html`, `caticon.png`, and `CNAME` from the repository root using
a static hosting provider. For GitHub Pages, configure the repository to publish
from the root of the selected branch; `CNAME` contains the custom domain
`catrack.chunkkeeper.com`. Configure that domain's DNS and HTTPS in the hosting
provider as required.

The deployed dashboard still needs network access to the Host Agent. For a
publicly hosted HTTPS dashboard, expose the agent through a trusted HTTPS
tunnel or reverse proxy that supports WebSockets and use its HTTPS base URL in
the dashboard.

## Security

The dashboard can send arbitrary commands to Minecraft servers and perform
administrative player actions. The connection token grants access to those
controls. Use a strong private token, share it only with trusted users, and
expose the Host Agent only through a network or tunnel you trust. The token is
sent to the agent as a WebSocket query parameter and is stored in the current
browser tab's session storage; avoid connecting from shared or untrusted
devices.