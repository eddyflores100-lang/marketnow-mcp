# Agent quickstart — MarketNow free trust stack (2026-09)

Everything here is live-verified (2026-09-20). Zero cost: no fees, no API keys, no signup.

## 1. Trust-checking before your agent calls an MCP tool

Add the remote MCP endpoint (nothing to install):

```json
{
  "mcpServers": {
    "marketnow": { "url": "https://www.marketnow.site/api/mcp" }
  }
}
```

Server: `marketnow-mcp v1.13.0` (protocol 2025-03-26, streamable HTTP). 9 tools:
`marketnow_verify_trust` · `marketnow_translate_credential` · `marketnow_list_formats` ·
`marketnow_get_pipeline` · `marketnow_check_domain` · `marketnow_search_skills` ·
`marketnow_check_revocation` · `marketnow_fingerprint_tool` · `marketnow_submit_skill`

Pre-call checklist (OWASP MCP guidance: verify tool definitions before execution):

1. `marketnow_check_domain` on the server host (risk score + reasons: RDAP domain age, TLS cert, reputation)
2. `marketnow_fingerprint_tool` on the tool definition — store it, re-check on reconnect (drift = poisoning alarm)
3. `marketnow_verify_trust` if the peer presents a credential (8 formats: ATC v3, JWT, W3C VC, MCP Card, A2A, EAT-AI, ZTA, X.509 — 12-stage fail-closed pipeline)
4. Only then execute. "Unknown" is a final answer, not an error to retry.

## 2. Mint an Agent Trust Card (free, Ed25519, RFC 8785)

```bash
npm install agent-trust-card
npx atc init > keys.json
```

```javascript
import { generateKeyPair, issueATC } from 'agent-trust-card';

const ca = generateKeyPair();
const agent = generateKeyPair();
const atc = issueATC(ca, agent, {
  card_id: 'ATC-2026-0000001',
  identity: { agent_id: 'my-bot', agent_name: 'My Bot', agent_owner: 'My Org' },
  capabilities: {
    filesystem: { read: 'own_dir', write: 'own_dir' },
    network:    { egress: 'allowlist', ingress: 'none' },
    shell:      { exec: 'sandboxed', spawn: 'none' },
    credentials:{ read_env: 'none', read_files: 'none' },
    process:    { subprocess: 'none', signals: 'own' },
  },
  evidence: { /* see spec: 10 controls, 8 required */ },
  risk: { trust_score: 9, risk_level: 'low', score_explanation: 'Clean audit',
          scored_at: new Date().toISOString() },
});
```

Verify any card: `npx agent-trust-card verify card.json` or the `marketnow_verify_trust` tool.
Revocation (public, free): `marketnow_check_revocation` with `card_id` or CA `kid`.

## 3. Free REST (no MCP client needed)

```bash
# scam-check any domain
curl -s 'https://www.marketnow.site/api/scam-check?domain=example.com'
# current CA public key (rotate-aware verifiers fetch this live)
curl -s 'https://www.marketnow.site/api/atc?action=ca-key'
```

## 4. The 14 npm packages (~6,300 downloads/month)

marketnow-mcp · agent-trust-card · @marketnow/trust-core · @marketnow/trust-gateway ·
@marketnow/trust-adapters · @marketnow/uts · @marketnow/uta-verify · @marketnow/uta-conformance ·
@marketnow/trust-mcp-middleware · @marketnow/cline-trust-plugin · @marketnow/sentinel-rules ·
@marketnow/trust-observability · marketnow-audit · marketnow-install-stack

Full stack: `curl -fsSL https://marketnow.site/install.sh | bash`

## 5. Free vs paid (why this exists)

| Capability | MarketNow | Per-call fee protocols |
|---|---|---|
| Credential verification | free, 8 formats | metered per call |
| Revocation lookup | free, public | paywalled ledger |
| Domain risk check | free | paid add-on |
| Registry search | free, 68,388 servers | paid tier |
| Minting a trust card | free | per-card fee |

## Links

- Storefront (manifests for Cline/Claude/Cursor/Gemini/A2A): https://github.com/eddyflores100-lang/marketnow-mcp
- Canonical dev repo: https://github.com/alicelabs-llc/universal-trust-adapter
- Site: https://www.marketnow.site (catalog /catalog · submit /submit · embed /embed)
- Spec: docs/atc-spec/SPEC.md in the canonical repo (ATC/1.0: 10 controls, 8 required)

Fail-closed by design: verification failure stops the call. Never default to trusted.
