# MarketNow MCP — Free Trust Layer for AI Agents

> Verify **who** you're talking to and **what** you're about to call — before your agent executes anything. 100% free. No fees. No signup. No API keys.

**Remote endpoint (zero install):** `https://www.marketnow.site/api/mcp` — `marketnow-mcp v1.13.0`, 9 tools, JSON-RPC over streamable HTTP.

```bash
curl -s https://www.marketnow.site/api/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"curl","version":"1.0"}}}'
```

## Why

- AI agents connect to MCP servers **blind**. Tool poisoning, prompt injection via tool descriptions, and scam domains are live attack surfaces (see the [OWASP MCP Cheat Sheet](https://genai.owasp.org/)).
- Existing answers put a **price tag on trust** (per-call fees, paywalled ledgers). MarketNow trust tooling is **free forever**: the endpoint, the credential format, the registry lookups, and the 14 npm packages.
- Everything is **fail-closed**: if verification can't complete, the answer is "unknown", never "trusted".

## The 9 remote tools

| Tool | What it does |
|---|---|
| `marketnow_verify_trust` | Verify any agent credential: JWT, W3C VC, MCP Card, ATC v3, A2A, EAT-AI, ZTA, X.509 (8 formats) |
| `marketnow_translate_credential` | Translate credentials between the 8 formats |
| `marketnow_list_formats` | List supported formats + algorithms + status |
| `marketnow_get_pipeline` | Details of the 12-stage verification pipeline |
| `marketnow_check_domain` | Scam checker: risk score + reasons (WHOIS age, TLS, reputation) |
| `marketnow_search_skills` | Search 68k+ indexed MCP servers (GitHub, npm, PyPI + more) |
| `marketnow_check_revocation` | Revocation status of an Agent Trust Card (card_id) or CA key (kid) |
| `marketnow_fingerprint_tool` | Cryptographic fingerprint of MCP tool definitions (OWASP: "verify tool definitions before execution") |
| `marketnow_submit_skill` | Publish a skill to the MarketNow catalog (validated + Sentinel-scanned) |

## 30-second setup

**Remote (recommended — nothing to install):**

```json
{
  "mcpServers": {
    "marketnow": {
      "url": "https://www.marketnow.site/api/mcp"
    }
  }
}
```

Works with Cursor, Cline, Claude, Gemini CLI, and any MCP 2025-03-26+ client. See [`mcp.json`](mcp.json) for a drop-in config and [`llms-install.md`](llms-install.md) for a guided install.

**Local stdio (npm):**

```bash
npx marketnow-mcp
```

## Client configs

<details><summary><b>Cursor / Windsurf</b></summary>

```json
{
  "mcpServers": {
    "marketnow": { "url": "https://www.marketnow.site/api/mcp" }
  }
}
```
</details>

<details><summary><b>Claude Desktop / Claude Code</b></summary>

```json
{
  "mcpServers": {
    "marketnow": { "type": "http", "url": "https://www.marketnow.site/api/mcp" }
  }
}
```
</details>

<details><summary><b>Cline</b></summary>

```json
{
  "mcpServers": {
    "marketnow": { "url": "https://www.marketnow.site/api/mcp", "disabled": false, "autoApprove": [] }
  }
}
```
</details>

<details><summary><b>Gemini CLI</b></summary>

`gemini-extension.json` is already in this repo with the `gemini-cli-extension` topic — the extensions gallery crawler indexes it automatically.
</details>

## REST API (no MCP needed)

```bash
# Scam-check any domain — free, no key
curl -s 'https://www.marketnow.site/api/scam-check?url=example.com'

# Current CA public key for Agent Trust Card verification
curl -s 'https://www.marketnow.site/api/atc?action=ca-key'
```

Also live: `/api/trust`, `/api/interceptor`, `/api/audit-report.json`, `/api/owasp.json`, and the ATC schemas at `/atc/unified` and `/atc/schema-3.0`.

## npm packages (14, ~6,300 downloads/month)

| Package | Role |
|---|---|
| [`marketnow-mcp`](https://www.npmjs.com/package/marketnow-mcp) | This MCP server (stdio + remote) |
| [`agent-trust-card`](https://www.npmjs.com/package/agent-trust-card) | Mint/verify Agent Trust Cards (ATC v3, Ed25519, RFC 8785 canonicalization) |
| [`@marketnow/trust-core`](https://www.npmjs.com/package/@marketnow/trust-core) | Core verification engine |
| [`@marketnow/trust-gateway`](https://www.npmjs.com/package/@marketnow/trust-gateway) | MCP middleware gateway + receipts |
| [`@marketnow/trust-mcp-middleware`](https://www.npmjs.com/package/@marketnow/trust-mcp-middleware) | Drop-in `tools/call` wrapper: verify → allow/deny → receipt |
| [`@marketnow/uta-conformance`](https://www.npmjs.com/package/@marketnow/uta-conformance) | 14 signed conformance vectors |
| [`@marketnow/sentinel-rules`](https://www.npmjs.com/package/@marketnow/sentinel-rules) | 29 MCP security rules (semgrep) |
| [`@marketnow/cline-trust-plugin`](https://www.npmjs.com/package/@marketnow/cline-trust-plugin) | Trust plugin for Cline |
| [`marketnow-install-stack`](https://www.npmjs.com/package/marketnow-install-stack) | Multi-source installer (5 stacks) |
| +5 more | [`marketnow-audit`](https://www.npmjs.com/package/marketnow-audit), [`@marketnow/uta-verify`](https://www.npmjs.com/package/@marketnow/uta-verify), [`@marketnow/trust-adapters`](https://www.npmjs.com/package/@marketnow/trust-adapters), [`@marketnow/uts`](https://www.npmjs.com/package/@marketnow/uts), [`@marketnow/trust-observability`](https://www.npmjs.com/package/@marketnow/trust-observability) |

One-shot installer for the whole stack:

```bash
curl -fsSL https://marketnow.site/install.sh | bash
```

## Agent Trust Card (ATC) — the free credential

ATC v3 is an open trust credential for agents: Ed25519-signed, RFC 8785-canonicalized, revocable. Mint one from npm, verify any of the 8 formats through the endpoint. **Free. No per-card fee. No ledger rent.** The verification pipeline runs 12 stages (signature → canonicalization → expiry → revocation → CA rotation → ...); get the full list via the `marketnow_get_pipeline` tool.

## Facts

- Registry: **68,388 indexed MCP servers** (deduped, multi-source) — search it with `marketnow_search_skills`
- Credential formats supported: **8** (ATC v3, JWT, W3C VC, A2A, EAT-AI, ZTA, MCP Card, X.509)
- Cost: **$0** — endpoint, packages, registry search, CA key, revocation checks
- This repo = distribution storefront (docs + manifests for marketplaces). Canonical development: [alicelabs-llc/universal-trust-adapter](https://github.com/alicelabs-llc/universal-trust-adapter)

## Manifests in this repo (agent-first)

| File | Consumer |
|---|---|
| [`llms.txt`](llms.txt) / [`llms-full.txt`](llms-full.txt) | LLM agents crawling the repo |
| [`AGENTS.md`](AGENTS.md) | Coding agents (Codex, Cursor, Jules...) |
| [`llms-install.md`](llms-install.md) | Cline marketplace installer |
| [`mcp.json`](mcp.json) / [`.mcp.json`](.mcp.json) | Generic MCP + Claude Code |
| [`.claude-plugin/plugin.json`](.claude-plugin/plugin.json) | Claude plugin indexers |
| [`.cursor-plugin/plugin.json`](.cursor-plugin/plugin.json) | Cursor plugin indexers |
| [`gemini-extension.json`](gemini-extension.json) | Gemini CLI extensions gallery |
| [`.well-known/agent.json`](.well-known/agent.json) | A2A Agent Card discovery |

## License

Docs and manifests in this repo: **MIT**. The npm packages carry their own licenses (AL-1.0 / MNNC-1.0) — see each package page.
