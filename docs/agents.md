# Agents on GreenLight Dash Server

The web version has no terminal and runs no agent itself. Agents run on your own computer
and connect to the instance through the API — the same GLAI (board) and GLEA (video editor,
mockups) you know from the desktop app, just started by you instead of by the app.

## Connect an agent (in the app)

1. Open **GLAI** in the board header, or **GLEA** in the video editor / mockup generator,
   and pick your agent: Claude Code CLI, Codex, Gemini CLI or OpenCode.
2. The connection guide mints an API token (it asks for the owner password; the token is
   shown once) or accepts one you already have.
3. Copy the block it shows and run it in a terminal on your computer:

       export GREENLIGHT_API_BASE_URL="https://boards.example.com"
       export GREENLIGHT_API_TOKEN="glt_…"
       mkdir -p "$HOME/greenlight-skills" && curl -fsSL -H "Authorization: Bearer $GREENLIGHT_API_TOKEN" \
         "$GREENLIGHT_API_BASE_URL/api/agent-skills/greenlight-dash.zip" -o "$HOME/greenlight-skills/greenlight-dash.zip" \
         && unzip -oq "$HOME/greenlight-skills/greenlight-dash.zip" -d "$HOME/greenlight-skills"
       claude --dangerously-skip-permissions "You are GLAI - GreenLight Dash AI agent. Use skill $HOME/greenlight-skills/greenlight-dash/SKILL.md."

   The first two lines can live in your shell profile; after that only the last line is
   needed. Needs `curl` and `unzip` (Git Bash or WSL on Windows).
4. Work as on the desktop: "add the uploaded images as a grid", "make a 10-second promo and
   render it". Uploads go through the API, renders run on the server, results appear on the
   board while you watch.

Tokens are listed and revoked in `Settings ▸ API Keys ▸ Server`. The desktop app needs no
token: its local backend has no login.

## Any HTTP client

Every request carries `Authorization: Bearer $GREENLIGHT_API_TOKEN`. The `greenlight-dash`
skill (`GET /api/agent-skills/greenlight-dash.zip`, also shipped with the desktop app)
documents the endpoints. Long operations are jobs: `GET /api/jobs?active=true`,
`DELETE /api/jobs/{id}` cancels, `POST /api/jobs/download` downloads in the background.
