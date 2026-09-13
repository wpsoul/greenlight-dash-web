# Render (prepared, not yet exercised by us)

Use a **Web Service** from a Docker image with a **Persistent Disk** mounted at `/data`
(disks are only available on paid plans and force a single instance, which is what the
app needs). `render.yaml` in this repository describes it; set the secret variables in
the dashboard after the first sync. Health check path `/health`. Choose a plan with
≥ 8 GB RAM for the full image.
