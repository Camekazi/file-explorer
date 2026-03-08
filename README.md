# File Explorer

> **When** you need to get at a file on another machine, **you want** to browse, preview, and download it without setting up a VPN or remembering SSH paths — **so you can** just open a browser tab and find it.

![File Explorer](screenshot.png)

Built with [droid](https://factory.ai). Stack: Bun, Hono, React, Tailwind.

---

## Why not just use...

**Cloud sync (Dropbox, iCloud)**: Requires you to have synced the file in advance. Doesn't cover servers, VPS, or NAS. Doesn't give you the full filesystem.

**SSH + terminal**: Works but you need to know the exact path, can't preview images or code in-place, and requires a terminal open for every machine you want to check.

**VPN + file sharing**: Heavy to set up, requires the remote machine to advertise shares, and it's still a separate app experience.

File Explorer is a single-process web server. Run it on each machine, point them at a hub, and all your machines appear in one browser tab. Browse, search, preview, download, upload, edit — all over HTTP, with token auth you set yourself.

---

## What Makes It Different

**Self-hosted**: No cloud account, no sync, no data leaving your machines. The hub is just another instance you control.

**Two connection modes**: For machines you can reach over a network (VPS, NAS, home server on Tailscale), use URL mode — the hub proxies requests to the remote's HTTP server. For machines where you only have SSH access and don't want to run a server, SSH mode works without installing anything on the remote.

**Combo views**: Group any subset of your machines into a named view. "Work machines", "home servers", "all media" — one click to switch between them.

**Full file operations**: Browse, fuzzy-search, preview text/images/code, download, upload (drag-and-drop), create files/folders, rename, duplicate, delete, edit text files in-browser. All operations work the same whether the file is local or on a remote device.

**Command palette**: ⌘K to search across the current device (or all devices). Fuzzy-matched via Fuse.js.

---

## Requirements

- [Bun](https://bun.sh) (`curl -fsSL https://bun.sh/install | bash`)
- SSH key access to any remote machines you want to add via SSH mode

---

## Setup: Hub (your main machine)

```bash
git clone https://github.com/<your-org-or-user>/file-explorer.git
cd file-explorer
bun install
bun run build
FILE_EXPLORER_ADMIN_TOKEN=<strong-token> bun run start
```

Open `http://localhost:3456`. The hub serves the UI and acts as a proxy for all remote devices.

To add a read-only token for shared access:

```bash
FILE_EXPLORER_ADMIN_TOKEN=<admin-token> \
FILE_EXPLORER_READ_TOKEN=<read-only-token> \
bun run start
```

Read tokens can browse, search, preview, and download. Write operations (create, rename, delete, upload, edit) require the admin token.

Tokens are passed as `Authorization: Bearer <token>` — the UI prompts for one on first load.

---

## Add a Machine: URL Mode (recommended)

Run file-explorer on the remote machine the same way:

```bash
# On the remote machine
git clone ...
bun install && bun run build
PORT=3456 FILE_EXPLORER_ADMIN_TOKEN=<remote-token> bun run start
```

Then in the hub UI:

- Device switcher → **Add a Machine** → **Device URL**
- Paste the host or full URL. Host-only input auto-adds `http://` and `:3456`.
- If the remote has its own token, fill in **Remote token**.

Good URL examples:
- `my-host.your-tailnet.ts.net`
- `http://192.168.1.20:3456`
- `http://my-vps.example.com:3456`

---

## Add a Machine: SSH Mode

No server needed on the remote. The hub runs SSH commands directly.

- Device switcher → **Add a Machine** → **SSH Host**
- Enter the SSH host alias from your `~/.ssh/config`.
- Hub must be able to SSH non-interactively (key auth, `BatchMode=yes`).

Quick check from the hub machine:
```bash
ssh my-host 'echo ok'
```

SSH mode supports all the same operations: browse, search, preview, download, upload, create, rename, delete, save. Recent files tracking is not available over SSH.

---

## Deploy Helper

For URL mode, `deploy.sh` handles the full remote setup in one command: installs Bun if needed, rsyncs the code, starts the server, and registers it with the hub.

```bash
HUB_TOKEN=<hub-admin-token> \
DEVICE_AUTH_TOKEN=<remote-token-or-empty> \
./deploy.sh <ssh-host> [hub-url] [device-url]
```

Examples:
```bash
./deploy.sh vps-london
./deploy.sh mini http://hub.local:3456
./deploy.sh mini http://hub.local:3456 http://mini.your-tailnet.ts.net:3456
```

The script auto-detects the remote's Tailscale DNS name, Tailscale IP, LAN IP, or public IP — in that order — to pick the best URL to register. Pass `device-url` explicitly if the detection is wrong.

---

## Environment Variables

| Variable | Description |
|---|---|
| `FILE_EXPLORER_ADMIN_TOKEN` | Full read/write access |
| `FILE_EXPLORER_READ_TOKEN` | Read-only access (browse, search, preview, download) |
| `FILE_EXPLORER_ROOT` | Root directory to expose (default: `$HOME`) |
| `FILE_EXPLORER_DATA` | Where to store device registry and settings (default: `~/.file-explorer/`) |
| `FILE_EXPLORER_ALLOW_NO_AUTH=true` | Disable auth — local/dev only |
| `FILE_EXPLORER_CORS_ORIGINS` | Comma-separated CORS allowlist |
| `PORT` | Server port (default: 3456) |
| `FILE_EXPLORER_API_TOKEN` | Legacy alias for admin token |

Auth is required by default. The server refuses to start without a token unless `ALLOW_NO_AUTH=true` is set.

---

## Troubleshooting

**`Cannot reach ...`**: Remote not running, wrong host, wrong port, or no route from hub. Check: `curl http://<remote>:3456/api/files?path=`

**`Remote returned 401/403`**: Remote has token auth. Fill **Remote token** when adding a URL device.

**SSH add fails**: Validate from the hub shell: `ssh <alias> 'echo ok'`. The hub needs non-interactive key auth.

**Server fails to start**: Check auth env vars — the server throws if neither `FILE_EXPLORER_ADMIN_TOKEN` nor `FILE_EXPLORER_ALLOW_NO_AUTH=true` is set.

---

## What "any machine" means in practice

Addable via **URL mode**: macOS, Linux, VPS, NAS, home servers — anything that can run Bun and be reached over HTTP from the hub.

Addable via **SSH mode**: Any machine you can already SSH into non-interactively from the hub. No server install needed on the remote.

Not practical to add directly: iOS/Android (no Bun server or SSH daemon).

---

## Current State and Limitations

- Recent files tracking is per-device and in-memory only (restarts clear it)
- No directory download as zip — single file download only
- No folder upload (file-by-file only)
- Port 3456 is hardcoded as the default; URL-mode remotes are assumed to be on the same port unless you specify a full URL
- Device registry is stored as flat JSON (`~/.file-explorer/devices.json`) — no migration tooling
