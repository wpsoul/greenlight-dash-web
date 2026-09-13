# Where to host GreenLight Dash Server

The server is one long-running Linux container with a persistent disk: a Python API,
SQLite, your media, ffmpeg, and (full image) headless Chromium plus the agent runner.
That rules out platforms built for static sites and short-lived functions.

| Host | Status | Notes |
| --- | --- | --- |
| Any Linux VPS with Docker (Hetzner, DigitalOcean, Linode, OVH…) | **Supported, recommended** | Compose + Caddy. Cheapest and fastest for rendering. See [vps-caddy.md](vps-caddy.md). |
| Coolify / Dokploy (self-managed PaaS on your VPS) | Supported | Import `docker-compose.yml`. See [coolify-dokploy.md](coolify-dokploy.md). |
| Railway | Prepared, needs a volume | Image deploy + Volume at `/data`. See [railway.md](railway.md). |
| Fly.io | Prepared | Image deploy + volume. See [fly.md](fly.md). |
| Render | Prepared | Docker web service + persistent disk. See [render.md](render.md). |
| Vercel, Netlify, Cloudflare Workers/Pages, GitHub Pages | **Not possible** | No persistent process, no disk, no Chromium. See [not-supported.md](not-supported.md). |
| Shared PHP hosting (cPanel, Plesk shared plans) | Not possible | No Docker, no long-running processes. |

Sizing: the `core` image (browser app + agent API) runs on 2 vCPU / 4 GB. Server rendering
and the bundled runner (full image) want 4 vCPU / 8 GB; on a CPU-only host a 30-second
1080p effect-heavy clip takes minutes. Disk: your media plus renders — start at 50 GB.

Whatever the host: TLS in front, the app bound to the host's private network only, a
second hostname for `GL_PREVIEW_ORIGIN`, and backups of `/data`.
