# Agents on GreenLight Dash Server

## External agents (Claude Code, scripts)

1. In the app: `Settings ▸ API Keys ▸ Server ▸ Agent tokens ▸ Create`. Copy the token once.
2. Give the agent two environment variables:

       GREENLIGHT_API_BASE_URL=https://boards.example.com
       GREENLIGHT_API_TOKEN=glt_…

3. Every request carries `Authorization: Bearer $GREENLIGHT_API_TOKEN`. The `greenlight-dash` skill
   (shipped with the desktop app and inside the server image) documents the endpoints.

Long operations are jobs: `GET /api/jobs?active=true`, `DELETE /api/jobs/{id}` cancels,
`POST /api/jobs/download` downloads in the background.

## The bundled runner (`GL_PROFILE=full`)

Assign a brief on a board (header ▸ Agent tasks). The server runs Claude Code in a scratch
workspace with a task-scoped token, verifies the outcome on the board (an editable project and a
completed render) and marks the task completed or needs-attention. Credentials, ONE of:

- `GL_RUNNER_CLAUDE_OAUTH_TOKEN` — run `claude setup-token` where Claude Code is logged in.
- `GL_RUNNER_ANTHROPIC_API_KEY` — an API key from console.anthropic.com.

Budgets: `GL_RUNNER_MAX_TURNS` (80), `GL_RUNNER_TIMEOUT_SEC` (3600), `GL_RUNNER_ALLOWED_TOOLS`.
