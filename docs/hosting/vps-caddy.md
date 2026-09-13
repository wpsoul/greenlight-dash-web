# VPS + Docker Compose + Caddy (recommended)

1. A Linux VPS (Ubuntu 22.04/24.04, 4 vCPU / 8 GB for the full image), Docker Engine and
   the Compose plugin installed, two DNS records pointing at it:
   `boards.example.com` and `preview.boards.example.com`.
2. Install:

   ```bash
   mkdir -p ~/greenlight && cd ~/greenlight
   curl -fsSLO https://raw.githubusercontent.com/wpsoul/greenlight-dash-web/main/docker-compose.yml
   curl -fsSL  https://raw.githubusercontent.com/wpsoul/greenlight-dash-web/main/env.example -o .env
   docker compose run --rm -i greenlight hash-password    # paste the scrypt$… line into .env
   nano .env                                              # GL_INSTANCE_URL, GL_PREVIEW_ORIGIN, GL_AUTH_PASSWORD_HASH, runner credential
   mkdir -p data && sudo chown 1000:1000 data             # the container runs as uid 1000
   docker compose up -d
   ```

3. Caddy (`apt install caddy`), `/etc/caddy/Caddyfile`:

   ```
   boards.example.com, preview.boards.example.com {
       reverse_proxy 127.0.0.1:8080
       request_body { max_size 8GB }
   }
   ```

   `systemctl reload caddy`. Caddy obtains certificates automatically.

4. Open `https://boards.example.com`, sign in, create an agent token under
   `Settings ▸ API Keys ▸ Server`.

Backups: `POST /api/admin/backup` from your browser session, or stop the stack and copy
`data/`. Upgrade: back up, change `GREENLIGHT_IMAGE` in `.env`, `docker compose pull && docker compose up -d`.
