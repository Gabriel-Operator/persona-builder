# Setup

This plugin bundles a remote MCP server named `gabriel`
(`https://gabrieloperator.com/mcp/gateway`). It authenticates with a workspace
**Gabi token** read from the `GABRIEL_TOKEN` environment variable, so it stays
disconnected until that variable is set.

```bash
export GABRIEL_TOKEN='gabi_...'
```

Mint the token at **Workspace → Dashboard → Gateway API key** on
https://gabrieloperator.com, or via **Developer Settings → API Tokens → Create
New Token → preset MCP Gateway**. It must be a workspace token (not twin-bound).

Full walkthrough, required scopes, and troubleshooting:
[`skills/gabriel-mcp-setup/SKILL.md`](skills/gabriel-mcp-setup/SKILL.md).

Set `GABRIEL_MCP_URL` only to target a non-production origin, e.g.
`http://localhost:3000/mcp/gateway`.
