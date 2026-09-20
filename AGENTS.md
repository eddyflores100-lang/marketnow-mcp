# AGENTS.md — instructions for AI coding agents

You are working with the MarketNow MCP distribution storefront: docs + marketplace manifests for a free trust layer for MCP agents. This is NOT the development repo.

## What lives here

- `README.md` — human-facing overview (keep in sync with `llms.txt` / `llms-full.txt`)
- `llms.txt`, `llms-full.txt` — machine-readable docs for LLM agents
- `llms-install.md` — Cline marketplace installer doc (Cline reads this; keep it minimal and exact)
- `mcp.json`, `.mcp.json` — MCP client configs (remote endpoint)
- `.claude-plugin/plugin.json`, `.cursor-plugin/plugin.json` — plugin marketplace manifests
- `gemini-extension.json` — Gemini CLI extensions gallery manifest (crawler indexes repos with topic `gemini-cli-extension`)
- `.well-known/agent.json` — A2A Agent Card
- `logo-400.png` — 400×400 marketplace logo

## Source of truth

All version numbers, tool lists, and endpoints must match the live deployment:

- Remote MCP endpoint: `https://www.marketnow.site/api/mcp` (serverInfo.version is the version number to mirror everywhere)
- Canonical development repo: `https://github.com/alicelabs-llc/universal-trust-adapter`
- npm: `marketnow-mcp` (dist-tag latest)

**Do not invent tool names or numbers.** Verify by sending an MCP `tools/list` to the endpoint before editing any tool table. Verify npm download numbers via `https://api.npmjs.org/downloads/point/last-month/marketnow-mcp`.

## Rules

1. Never commit secrets, keys, or tokens. The CA private key never leaves the operator's environment — only the public key is public.
2. Fail-closed is the design philosophy: never edit docs to suggest "retry until trusted" behavior.
3. When bumping a version here, update ALL of: README.md, llms.txt, llms-full.txt, gemini-extension.json, .claude-plugin/plugin.json, .cursor-plugin/plugin.json, .well-known/agent.json — they must stay consistent.
4. Keep marketing claims verifiable: "free", "no API keys", "8 formats", "12 stages", "68k servers" — all must be live-checkable. If a claim stops being true, fix the claim, not the checker.
5. Don't reformat tables or expand whitespace in `llms-install.md` — Cline's installer agent reads it raw.

## Verification commands

```bash
# endpoint alive + version
curl -s https://www.marketnow.site/api/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"check","version":"1"}}}'

# tool list (after initialize)
curl -s https://www.marketnow.site/api/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}'

# scam check works
curl -s 'https://www.marketnow.site/api/scam-check?url=github.com'
```
