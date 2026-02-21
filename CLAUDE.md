# CoLobster

A single Google Colab notebook (`colobster.ipynb`) that runs an [OpenClaw](https://openclaw.ai) AI gateway with zero setup.

## Project structure

```
colobster.ipynb          # The notebook — this IS the project
README.md                # Project description + Colab badge
assets/colobster_banner.png  # Banner image for README
LICENSE                  # MIT
```

OpenClaw itself is installed at runtime via `npm install -g openclaw@latest` — the source code is NOT in this repo.

## Notebook cell layout (18 cells)

| Index | Type     | Purpose |
|-------|----------|---------|
| 0     | markdown | Title |
| 1     | markdown | Step 1 — API key instructions |
| 2     | markdown | Step 2 intro |
| 3     | code     | Step 2 — Install Node.js + OpenClaw, detect API keys |
| 4     | markdown | Choose a Model intro |
| 5     | code     | Model selection dropdown (ipywidgets) |
| 6     | markdown | Optional Drive persistence intro |
| 7     | code     | Drive persistence setup |
| 8     | markdown | Step 3 intro |
| 9     | code     | Step 3 — Launch gateway (writes config, starts process) |
| 10    | markdown | Step 4 — Connect via ngrok |
| 11    | code     | Step 4 — ngrok tunnel setup |
| 12    | markdown | "You're all set!" — directs user to dashboard |
| 13    | markdown | Utilities section header |
| 14    | code     | View logs |
| 15    | code     | Check status |
| 16    | code     | Stop gateway |
| 17    | code     | Reset WhatsApp (clear stored auth) |

## Key technical details

- **Model config format**: `agents.defaults.model` must be `{ primary: "provider/model-id" }` (Zod-validated object), NOT a plain string
- **Control UI basePath**: Config includes `controlUi.basePath: '/ui'` so dashboard serves at `/ui/` instead of root
- **Dashboard auth**: Token passed via URL hash fragment `#token=<TOKEN>` — auto-extracted by Control UI, stored in settings, then sent inside WebSocket connect frame as `params.auth.token`
- **trustedProxies**: Config includes `trustedProxies: ['127.0.0.1']` — required for ngrok. Without this, the gateway sees proxy headers from an untrusted address and rejects the connection
- **allowInsecureAuth**: Config includes `controlUi.allowInsecureAuth: true` — lets the dashboard use token-only auth without device pairing. Required for ngrok/tunnel access. Without this, the Control UI sends a device identity that needs pairing approval, creating a chicken-and-egg problem (dashboard can't connect to approve its own device)
- **No Gradio**: The notebook does not include a built-in chat UI — the OpenClaw Control UI (via ngrok) provides chat, channel setup, and agent config
- **ngrok is mandatory**: Step 4 sets up ngrok as the primary user interface — the Control UI at `/ui/` has everything users need
- **ANSI stripping**: Log parser uses `re.sub(r'\x1b\[[0-9;]*m', '', ...)` to strip terminal color codes
- **`OPENCLAW_NO_RESPAWN=1`**: Required on Colab (no systemd) — enables in-process restart instead of process exit
- **No channel/gmail skip flags**: `OPENCLAW_SKIP_CHANNELS` and `OPENCLAW_SKIP_GMAIL_WATCHER` are NOT set — all channels (WhatsApp, Telegram, Discord, Slack) and Gmail watcher load normally
- **Gateway shutdown**: Uses `pkill` + poll loop (up to 10s) to ensure the process fully exits before restarting. The `openclaw gateway restart` CLI does NOT work on Colab (no systemd/dbus)
- **Tool restrictions**: Config includes `tools.deny: ['exec', 'bash', 'process']` — blocks the agent from running shell commands, preventing prompt-injection attacks that trick it into leaking API keys via `printenv`, `cat /proc/self/environ`, etc.
- **API keys in memory only**: Keys are set in `os.environ` during cell 3 and inherited by the gateway process — no `.env` file is written to disk
