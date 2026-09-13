# Coolify / Dokploy

Both consume the repository's `docker-compose.yml` directly.

1. New resource → Docker Compose → paste or point at the file.
2. Environment: fill the variables from `env.example` (generate the password hash with
   `docker run -i --rm --entrypoint /app/backend/greenlight-server ghcr.io/wpsoul/greenlight-dash:<version> hash-password`).
3. Storage: keep the `./data:/data` bind mount or replace it with a managed volume.
4. Domains: attach `boards.example.com` **and** `preview.boards.example.com` to the
   service on port 8080; both platforms provision TLS. Set `GL_TRUSTED_PROXIES` to the
   platform proxy network (default `172.16.0.0/12` covers Docker networks).
