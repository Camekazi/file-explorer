# File Explorer

Self-hosted file browser that shows all your machines in one tab — browse, preview, download, upload, and edit over HTTP.

```bash
bun install && bun run build
FILE_EXPLORER_ADMIN_TOKEN=your-token bun run start
# open http://localhost:3456
```

The UI loads at `localhost:3456`. Add remote machines from the device switcher.

---

## Add a Machine: URL Mode

Run the same server on the remote, then register it in the hub UI.

```bash
# On the remote machine
bun install && bun run build
PORT=3456 FILE_EXPLORER_ADMIN_TOKEN=remote-token bun run start
```

Device switcher → **Add a Machine** → **Device URL** → paste the host. Accepts:
- `my-host.your-tailnet.ts.net`
- `http://192.168.1.20:3456`

## Add a Machine: SSH Mode

No server needed on the remote. The hub runs SSH commands directly.

Device switcher → **Add a Machine** → **SSH Host** → enter the alias from `~/.ssh/config`.

```bash
# Verify non-interactive key auth works first
ssh my-host 'echo ok'
```

SSH mode supports browse, search, preview, download, upload, create, rename, delete, and save.

## Deploy Helper

Install Bun, rsync code, start server, and register with hub in one command:

```bash
HUB_TOKEN=admin-token DEVICE_AUTH_TOKEN=remote-token ./deploy.sh vps-london
```

Auto-detects the remote's Tailscale DNS, LAN IP, or public IP for registration.

## Read-Only Token

```bash
FILE_EXPLORER_ADMIN_TOKEN=admin \
FILE_EXPLORER_READ_TOKEN=readonly \
bun run start
```

Read tokens can browse, search, preview, and download. Write operations require the admin token.

## Command Palette

Press `⌘K` to fuzzy-search files across the current device (or all devices). Powered by Fuse.js.

## Combo Views

Group any subset of machines into a named view — "work machines", "home servers", "all media". One click to switch.

---

## Environment Variables

| Variable | Default |
|----------|---------|
| `FILE_EXPLORER_ADMIN_TOKEN` | Required (or set `ALLOW_NO_AUTH=true`) |
| `FILE_EXPLORER_READ_TOKEN` | Optional read-only access |
| `FILE_EXPLORER_ROOT` | `$HOME` |
| `FILE_EXPLORER_DATA` | `~/.file-explorer/` |
| `PORT` | `3456` |

## Stack

Bun · Hono · React · Tailwind · Fuse.js

## Known Limitations

- Recent files tracking is in-memory only (clears on restart)
- No directory download as zip — single file only
- No folder upload — file-by-file only
- iOS/Android not addable as devices
