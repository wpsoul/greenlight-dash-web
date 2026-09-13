# Railway (prepared, not yet exercised by us)

Railway can run the image, but its filesystem is **ephemeral**: without a Volume every
redeploy wipes your boards.

1. New project → **Deploy a Docker image** → `ghcr.io/wpsoul/greenlight-dash:<version>`.
2. Service → **Volumes** → add a volume mounted at `/data` (start with 50 GB).
3. Service → **Variables**: everything from `env.example` except `GREENLIGHT_IMAGE`.
   Railway injects `PORT`; the image reads it. Set `GL_INSTANCE_URL` to the service's
   public HTTPS domain, and `GL_PREVIEW_ORIGIN` to a second custom domain attached to the
   same service (Railway allows several domains per service).
4. Railway mounts volumes owned by root while the image runs as uid 1000. If the first
   boot logs "GL_DATA_ROOT … is not writable", set the variable `RAILWAY_RUN_UID=0`
   (Railway's documented workaround) or use a startup command that chowns `/data`.
5. Health check path: `/health`. Memory: choose a plan with ≥ 8 GB for the full image.
6. Chromium inside Railway has no `/dev/shm` control; the launcher already passes
   `--disable-dev-shm-usage`.

`railway.toml` in this repository carries the health check and restart policy for
template-based deploys.
