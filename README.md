# GreenLight Dash Server

> Self-hosting package for GreenLight Dash Server: the Compose file, environment template and docs.
> The application image is published at `ghcr.io/wpsoul/greenlight-dash`; this repository carries no application source.

The self-hosted web app: your boards, media and video editor on your own Linux server,
opened from a browser, with AI agents working in the same workspace through the API.
One owner per installation. Data never leaves your server unless you configure a
provider.

## Quickstart (Docker Compose, ~5 minutes)

Requirements: a Linux host with Docker Compose, 2 vCPU / 4 GB (4 vCPU / 8 GB for
server-side rendering), a hostname with TLS in front (Caddy, Traefik, or your
platform's proxy).

```bash
mkdir greenlight && cd greenlight
curl -fsSLO https://raw.githubusercontent.com/wpsoul/greenlight-dash-web/main/docker-compose.yml
curl -fsSL  https://raw.githubusercontent.com/wpsoul/greenlight-dash-web/main/env.example -o .env
# 1. owner password → paste the printed scrypt$… line into .env as GL_AUTH_PASSWORD_HASH
docker compose run --rm -i greenlight hash-password
# 2. set GL_INSTANCE_URL (https://boards.example.com) and GL_PREVIEW_ORIGIN (https://preview.boards.example.com)
# 3. start
docker compose up -d
```

Put TLS in front of `127.0.0.1:8080`. Caddyfile:

```
boards.example.com, preview.boards.example.com {
    reverse_proxy 127.0.0.1:8080
}
```

`preview.boards.example.com` is a second hostname for the same container: HTML
cards are shown from it so a card's script can never act as you (it runs with no
session, on an opaque origin). Without `GL_PREVIEW_ORIGIN` the server still works
and `Settings ▸ Server` reports preview isolation as off.

## Hosting

See [docs/hosting](docs/hosting/README.md): a Linux VPS with Docker + Caddy is recommended;
Coolify/Dokploy, Railway, Fly.io and Render are prepared; Vercel, Netlify and Cloudflare
Workers/Pages cannot run a persistent server with a disk.

## The image

`ghcr.io/wpsoul/greenlight-dash:<version>` (~4.3 GB — Chromium, its browser dependencies
and ffmpeg are most of it). It runs the whole app: boards, media, HTML cards, the video
editor, the agent API, AI providers with your keys, and server-side rendering — stills,
previews and final videos for agents.

Rendering needs Chromium, ffmpeg and free disk; `GET /api/ready` reports each of them,
and Settings ▸ Server shows the result. Routes that reach the host's filesystem, launch
local applications or open a shell do not exist on a server at all.

Rendering works on a CPU-only server (open-source Chromium, frames encoded by
ffmpeg). Expect minutes, not seconds, for effect-heavy 1080p on a small VPS.

## Agents

- **External agents** are the way to work with agents on the web version: there is no
  terminal in the browser. Open **GLAI** (board) or **GLEA** (video editor / mockups) in
  the header and pick Claude Code, Codex, Gemini CLI or OpenCode: the connection modal
  mints an API token (owner password), downloads the bundled `greenlight-dash` skill
  and shows one copyable block to run on your own computer —
  `export GREENLIGHT_API_BASE_URL=…`, `export GREENLIGHT_API_TOKEN=glt_…`, the skill
  download (`GET /api/agent-skills/greenlight-dash.zip`) and the launch command. Any
  HTTP client works the same way: `Authorization: Bearer <token>` on every request to
  `https://boards.example.com/api/…`. Tokens are listed and revoked in
  `Settings ▸ API Keys ▸ Server`. Long operations are jobs: `GET /api/jobs?active=true`,
  `DELETE /api/jobs/{id}`, `POST /api/jobs/download`.

## Security model

- Owner password (scrypt), HttpOnly session cookie bound to the instance hostname,
  CSRF header on writes, bearer tokens for agents, login rate limit.
- Default-deny route policy: desktop-only routes (host paths, native apps, terminal,
  Telegram, sounds) do not exist on a server.
- Outbound policy at connect time: private, loopback, link-local and metadata
  addresses are refused for every fetch and for yt-dlp/curl through an internal
  egress proxy. `GL_OUTBOUND_ALLOW_HOSTS` allow-lists internal providers.
- Chromium runs with `--no-sandbox` inside the container (Docker's seccomp blocks its
  user-namespace sandbox); the container itself is non-root with
  `no-new-privileges`.

## Backup, restore, upgrade

- Online backup: `POST /api/admin/backup` with the owner
  session: writes pause for a few seconds, the SQLite backup API snapshots the DB and
  the durable directories are archived to `/data/backups/…tar.gz`. Download via
  `GET /api/admin/backups/<name>`.
- Offline: stop the container, `docker compose run --rm greenlight backup /data/backups/manual.tar.gz`.
- Restore into an EMPTY volume: `docker compose run --rm greenlight restore /data/backups/<file>.tar.gz`
  (mount the archive inside `/data` first). Restore never overwrites a live root.
- Upgrade: back up, change the image tag, `docker compose up -d`. A data root written by
  a newer server refuses to start on an older image.
- Health: `GET /health` (public), `GET /api/ready` (authenticated diagnostics).
  Uploads, downloads, renders and tasks return 507 below `GL_MIN_FREE_BYTES` (1 GiB).

## Licence

The same licence and the same gates as the desktop app: free by default, and the
features that are PRO on the desktop (AI Shorts, PRO effects and templates, Telegram,
the GLAI/GLEA agents) are PRO here. Server rendering is not
gated. Activate in the app (header ▸ Account ▸ "Already have a License Key?": key +
email, owner session) — the key is stored under `/data/secrets` — or set `GL_LICENSE_KEY`
(and `GL_LICENSE_EMAIL`) on the server, which then manages it. Either way it is validated
against greenlightdash.pro at activation and daily, with a 7-day offline grace. One
activation per domain: the instance id derives from the hostname, so a redeploy on the
same domain reuses its activation.

## Troubleshooting

- `docker compose logs -f greenlight` shows startup; `/data/logs/server.log` the app log.
- "refusing to start": a required variable is missing; the message names it.
- Renders fail on a CPU host: check `/api/ready` → `chromium`; mount a 1 GB `shm_size`.
- Behind Cloudflare/other proxies set `GL_TRUSTED_PROXIES` to the proxy's network so
  rate limits see real client addresses.
