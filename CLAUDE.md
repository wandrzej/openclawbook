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

## Notebook cell layout (19 cells)

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
| 10    | markdown | "What now?" + troubleshooting |
| 11    | markdown | Chat UI intro |
| 12    | code     | Chat UI (Gradio ChatInterface) |
| 13    | markdown | ngrok tunnel intro + comparison tables |
| 14    | code     | ngrok tunnel setup |
| 15    | markdown | Utilities section header |
| 16    | code     | View logs |
| 17    | code     | Check status |
| 18    | code     | Stop gateway |

## Key technical details

- **Model config format**: `agents.defaults.model` must be `{ primary: "provider/model-id" }` (Zod-validated object), NOT a plain string
- **Control UI basePath**: Config includes `controlUi.basePath: '/ui'` so dashboard serves at `/ui/` instead of root
- **Dashboard auth**: Token passed via URL hash fragment `#token=<TOKEN>` — auto-extracted by Control UI, stored in settings, then sent inside WebSocket connect frame as `params.auth.token`
- **trustedProxies**: Config includes `trustedProxies: ['127.0.0.1']` — required for ngrok. Without this, the gateway sees proxy headers from an untrusted address and rejects the connection
- **allowInsecureAuth**: Config includes `controlUi.allowInsecureAuth: true` — lets the dashboard use token-only auth without device pairing. Required for ngrok/tunnel access. Without this, the Control UI sends a device identity that needs pairing approval, creating a chicken-and-egg problem (dashboard can't connect to approve its own device)
- **Gradio 6**: `type='messages'` parameter was removed — do NOT add it back to `gr.ChatInterface()`
- **ANSI stripping**: Log parser uses `re.sub(r'\x1b\[[0-9;]*m', '', ...)` to strip terminal color codes
- **`OPENCLAW_NO_RESPAWN=1`**: Required on Colab (no systemd) — enables in-process restart instead of process exit
- **Tool restrictions**: Config includes `tools.deny: ['exec', 'bash', 'process']` — blocks the agent from running shell commands, preventing prompt-injection attacks that trick it into leaking API keys via `printenv`, `cat /proc/self/environ`, etc. This is the primary security boundary; the Chat UI's `_redact_secrets()` filter is a secondary defense-in-depth layer
- **API keys in memory only**: Keys are set in `os.environ` during cell 3 and inherited by the gateway process — no `.env` file is written to disk
