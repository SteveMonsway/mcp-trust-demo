# Vulnerable MCP Server — a live demo for MCP Trust

[![Scanned by MCP Trust](https://img.shields.io/badge/MCP_Trust-scanned-6f42c1)](https://github.com/SteveMonsway/mcp-trust)
[![MCP Trust](https://github.com/SteveMonsway/mcp-trust-demo/actions/workflows/mcp-trust.yml/badge.svg)](https://github.com/SteveMonsway/mcp-trust-demo/actions/workflows/mcp-trust.yml)

> ## ⚠️ This repository is intentionally vulnerable
> It contains a **deliberately insecure** MCP server that exists **only** as a live
> demo target for the [MCP Trust](https://github.com/SteveMonsway/mcp-trust) scanner.
> **Do not install, run, import, or copy this code.** There is no real functionality
> here — every risky pattern is on purpose. The exfiltration URL (`attacker.example`)
> is a reserved, non-routable domain; there are no real secrets.

## What this repo demonstrates

That wiring MCP Trust into a repo's CI takes **one line**, and that it catches real
issues in an MCP server **before** an agent ever connects to it:

```yaml
# .github/workflows/mcp-trust.yml
- uses: actions/checkout@v4
- uses: SteveMonsway/mcp-trust/packages/action@v0.5.3
  with:
    fail-on: high
    upload-sarif: true   # → GitHub Security / Code Scanning alerts
    comment-pr: true     # → summary comment on pull requests
```

That's the entire integration. Full inputs/outputs: the
[MCP Trust GitHub Action](https://github.com/SteveMonsway/mcp-trust#github-action).

## What MCP Trust flags here

Scanning [`server.js`](server.js) + [`package.json`](package.json) produces
**Decision: `BLOCK`** on the following evidence:

| Rule | Severity | What |
|---|---|---|
| `MCP-CODE-001` | critical | `child_process.exec` with caller-controlled input |
| `MCP-META-001` | high | Tool description with an instruction-override phrase ("ignore previous instructions") |
| `MCP-META-002` | high | Tool description with a concealment phrase ("do not tell the user") |
| `MCP-CODE-005` | medium | Arbitrary filesystem read at a caller-supplied path |
| `MCP-SUPPLY-003` | medium | `postinstall` script runs arbitrary code on `npm install` |
| `MCP-CODE-007` | low | Secret-like environment variable access (`GITHUB_TOKEN`) |
| `MCP-SUPPLY-001` | low | No linked source repository |

## See it live

- **Actions** → [latest MCP Trust run](https://github.com/SteveMonsway/mcp-trust-demo/actions/workflows/mcp-trust.yml)
- **Security → Code Scanning** → [the SARIF alerts](https://github.com/SteveMonsway/mcp-trust-demo/security/code-scanning) uploaded by the scan
- **Pull Requests** → the open demo PR shows the **MCP Trust comment** and a **red ✗ check** — that's the scanner gating a change

### Why is the check red on PRs but green on main?

The target is vulnerable, so the scan is always `BLOCK`. On a **pull request** the job
**fails on purpose** (`fail-on: high`) — that's MCP Trust blocking the merge, exactly what
you want in review. On **push to `main`** the step is set to `continue-on-error`, so the
branch badge stays green while the SARIF alerts still populate the Security tab. A green
run here would mean the scanner *missed* the vulnerabilities.

## Try it against your own MCP server

Point the scanner at any public repo, npm package, or local path:

```bash
npx @mcp-trust/cli scan github:owner/repo
```

Or add the Action (the one-liner above) to your own repository's
`.github/workflows/`. More: [MCP Trust](https://github.com/SteveMonsway/mcp-trust)
· [public benchmark of 452 real MCP servers](https://github.com/SteveMonsway/mcp-trust/tree/main/public-reports).
