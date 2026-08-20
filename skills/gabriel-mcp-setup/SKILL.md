---
name: gabriel-mcp-setup
description: >
  Connect the bundled `gabriel` MCP server by obtaining and configuring a
  workspace Gabi token (GABRIEL_TOKEN). Use this skill when the gabriel MCP
  server is failing to connect, when gabriel_* tools are unavailable, or when
  the user has just installed the persona-builder plugin and has no token yet.
metadata:
  author: gabriel-operator
  version: "1.3.0"
---

# Gabriel MCP Setup

The persona-builder plugin bundles a remote MCP server named `gabriel`. It reads
its credentials from the environment, so it stays disconnected until a workspace
token is present.

```json
{
  "type": "http",
  "url": "${GABRIEL_MCP_URL:-https://gabrieloperator.com/mcp/gateway}",
  "headers": { "Authorization": "Bearer ${GABRIEL_TOKEN}" }
}
```

`GABRIEL_MCP_URL` is optional and only needed to point at a non-production
origin (for example `http://localhost:3000/mcp/gateway`).

## Check before you ask

1. Is `GABRIEL_TOKEN` already set in the environment? If yes, the server should
   connect on the next session — tell the user to restart their session rather
   than minting a new token.
2. Are `gabriel_*` tools already listed? If yes, setup is done. Stop here and
   hand off to `persona-builder`.

Only run the steps below when neither is true.

## Getting a workspace token

The token must be a **workspace** token (`gabi_…` with no twin binding), minted
with the **MCP Gateway** preset. A twin-bound persona key will be rejected by
the gateway.

1. Open https://gabrieloperator.com/signup (or **Sign up** on the homepage).
   The first sign-in creates the account.
2. Sign in with email code, Google, Apple, or phone.
3. Open **Workspace → Dashboard** (`/workspace/dashboard`).
4. Click the copy button on the top-right pill labeled **Gateway API key**. The
   first copy mints a Dashboard (MCP Gateway) token starting with `gabi_`.
5. If the pill is empty or the copy fails, use **Generate new key**, or go to
   https://gabrieloperator.com/workspace/developer-settings → **API Tokens** →
   **Create New Token** → preset **MCP Gateway**.

The full token is shown once. It cannot be retrieved again — only regenerated.

Required scopes on the token: `api:access`, `mcp:access`, `digital-twin:admin`,
`digital-twin:chat`, `digital-twin:tools`, `digital-twin:media`,
`automation:read`, `automation:run`.

## Configuring the token

Ask the user to export it in the shell they launch their coding agent from,
then start a new session:

```bash
export GABRIEL_TOKEN='gabi_...'
```

To persist it, add that line to `~/.zshrc` or `~/.bashrc`.

**Never** ask the user to paste the raw token into chat, never write it into a
file inside the repository, and never echo it back after it is configured. If a
token has already been pasted into chat, tell the user to regenerate it.

## Verifying

Start a new session and confirm `gabriel_*` tools are listed. A cheap read-only
check is `gabriel_git_provisioning_status`.

If the server still fails to connect:

- `401` / `403` — the token is wrong, expired, twin-bound, or missing scopes.
  Mint a fresh one with the **MCP Gateway** preset.
- No `gabriel_*` tools at all — `GABRIEL_TOKEN` was not exported into the
  environment the agent was launched from. Re-export and restart the session.

Once tools are available, hand off to the `persona-builder` skill.
