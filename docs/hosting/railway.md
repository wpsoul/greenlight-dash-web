# Railway (verified 2026-09-13)

Railway runs the image well; the one rule is that its filesystem is **ephemeral**:
without a Volume every redeploy wipes your boards.

1. New project → **Deploy a Docker image** → `ghcr.io/wpsoul/greenlight-dash:<version>`.
2. Service → **Volumes** → add a volume mounted at `/data`. Railway's default volume is
   5 GB; enlarge it before uploading real media.
3. Service → **Variables**: everything from `env.example` except `GREENLIGHT_IMAGE`.
   Railway injects `PORT`; the image reads it. Set `GL_INSTANCE_URL` to the service's
   public HTTPS domain (`https://<service>.up.railway.app` or your custom domain), and
   `GL_PREVIEW_ORIGIN` to a second domain attached to the same service — Railway allows
   several domains per service; without it HTML preview isolation is reported as off.
   Also set `GL_TRUSTED_PROXIES=0.0.0.0/0` (Railway's edge proxy terminates TLS) and
   `GL_CHROMIUM_NO_SANDBOX=1` (Railway blocks Chromium's user-namespace sandbox).
4. Volumes are mounted owned by root while the image runs as uid 1000. Set the variable
   `RAILWAY_RUN_UID=0` (Railway's documented workaround) so `/data` is writable.
5. Health check path: `/health`. Memory: 4 GB is enough for browsing and short renders;
   pick 8 GB for effect-heavy 1080p renders.
6. Chromium inside Railway has no `/dev/shm` control; the launcher already passes
   `--disable-dev-shm-usage`.

A typical deploy builds and starts in 2–3 minutes; renders take a few seconds for a
short clip on the shared CPU.

Building from a Dockerfile instead of the image (only meaningful with the private source
repository): Railway ignores the Dockerfile unless the service variable
`RAILWAY_DOCKERFILE_PATH` points at it, and it rejects any `VOLUME` instruction.

`railway.toml` in this repository carries the health check and restart policy for
template-based deploys.
