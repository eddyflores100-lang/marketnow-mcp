# MarketNow MCP — install for Cline

MarketNow is a free trust layer for MCP agents: verify credentials, check domains for scam signals, fingerprint tools, search a 68k+ MCP server registry. No API keys, no signup, no fees.

## Remote server (recommended — no install)

Add to `cline_mcp_settings.json`:

```json
{
  "mcpServers": {
    "marketnow": {
      "url": "https://www.marketnow.site/api/mcp",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

That's the whole install. The endpoint is stateless JSON-RPC over streamable HTTP (protocol 2025-03-26). No environment variables, no secrets, no local dependencies.

## Local stdio alternative (npm)

Requires Node.js 18+.

```bash
npx marketnow-mcp
```

Config for the stdio variant:

```json
{
  "mcpServers": {
    "marketnow": {
      "command": "npx",
      "args": ["marketnow-mcp"],
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

## Verify it works

Ask the agent: "list the marketnow tools" — expect 9 tools (`marketnow_verify_trust`, `marketnow_translate_credential`, `marketnow_list_formats`, `marketnow_get_pipeline`, `marketnow_check_domain`, `marketnow_search_skills`, `marketnow_check_revocation`, `marketnow_fingerprint_tool`, `marketnow_submit_skill`).

Or test without any client:

```bash
curl -s 'https://www.marketnow.site/api/scam-check?domain=github.com'
```

A JSON risk report with score and reasons means the service is healthy.

## Full local stack (optional)

```bash
curl -fsSL https://marketnow.site/install.sh | bash
```

Installs the 14 npm packages (trust engine, middleware, ATC tooling) with 5 supported stacks.

## Fail-closed note

If any verification tool returns "unknown", that is the correct answer — the design never defaults to "trusted". Do not retry until a check passes.
