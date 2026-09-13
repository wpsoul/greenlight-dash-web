# Fly.io (prepared, not yet exercised by us)

```bash
fly launch --no-deploy --image ghcr.io/wpsoul/greenlight-dash:<version> --name my-greenlight
fly volumes create greenlight_data --size 50 --region <region>
fly secrets set GL_AUTH_PASSWORD_HASH='scrypt$…'
fly deploy
```

Use the `fly.toml` from this repository: it mounts the volume at `/data`, exposes port
8080, sets `GL_INSTANCE_URL`/`GL_PREVIEW_ORIGIN` placeholders you must edit, and asks
for an 8 GB machine. Add the preview hostname with `fly certs add preview.<host>`.
One machine only — the app keeps state in SQLite and process memory.
