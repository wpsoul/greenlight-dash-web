# Platforms that cannot run GreenLight Dash Server

- **Vercel / Netlify / Cloudflare Pages** host static files and short serverless
  functions. The server is a persistent Python process with SQLite on disk, background
  jobs, WebSockets and, in the full image, a headless Chromium. None
  of that fits a function runtime, and there is no persistent disk.
- **Cloudflare Workers / Containers**: Workers are JavaScript isolates; Cloudflare
  Containers have no persistent volume and no GPU, and server rendering needs a headless Chromium
  inside one long-lived container.
- **GitHub Pages**: static only.
- **Shared PHP hosting**: no Docker, no long-running processes.

What does work everywhere: a small Linux VPS with Docker (see [vps-caddy.md](vps-caddy.md)).
You can still put Cloudflare **in front** of that VPS as a proxy/DNS (set
`GL_TRUSTED_PROXIES` to Cloudflare's ranges or terminate TLS at your own Caddy).
