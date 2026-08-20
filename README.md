# Persona Builder

Coding-agent skill that interviews you and provisions a Gabriel AI Persona end to end from a workspace `gabi_` token: page, lists, pipeline, workflows, git bindings, team agents, and publish.

The canonical development copy lives in the Gabriel Operator monorepo at `server/skills/persona-builder/`. This repo is the public install pack.

## Install

### NPX (Cursor, Claude Code, Windsurf, OpenClaw)

```bash
npx skills add Gabriel-Operator/persona-builder
```

### Codex / ChatGPT

```bash
codex plugin marketplace add Gabriel-Operator/persona-builder --sparse .agents/plugins
```

Then install **Persona Builder**.

### Claude Code

```text
/plugin marketplace add Gabriel-Operator/persona-builder
/plugin install persona-builder@persona-builder
```

The Claude Code plugin bundles the `gabriel` MCP server, so there is no MCP JSON
to write by hand. Export your token and start a new session:

```bash
export GABRIEL_TOKEN='gabi_...'
```

### Grok Build

```bash
grok plugin marketplace add Gabriel-Operator/persona-builder
grok plugin install persona-builder --trust
```

## Setup

Every install needs a workspace `gabi_` token for the MCP Gateway. If you do not
have one: sign up at https://gabrieloperator.com/signup, open **Workspace →
Dashboard**, and copy the **Gateway API key** pill (`gabi_...`).

Do not paste the raw token into chat. See [`SETUP.md`](SETUP.md) for the full
walkthrough, required scopes, and troubleshooting.

## What this plugin contains

| Component | Purpose |
|---|---|
| `skills/persona-builder` | The interview + provisioning flow |
| `skills/gabriel-mcp-setup` | Obtaining and configuring the `GABRIEL_TOKEN` |
| `.mcp.json` | The `gabriel` remote MCP server (`https://gabrieloperator.com/mcp/gateway`) |

## Privacy Policy

This plugin connects to Gabriel Operator's hosted MCP Gateway using a token you
supply. Requests you make through it, and the resources it creates, are handled
under the Gabriel Operator privacy policy:
https://gabrieloperator.com/privacy

Terms of service: https://gabrieloperator.com/terms

The plugin itself stores no data locally. Your `GABRIEL_TOKEN` stays in your own
environment and is sent only to the MCP Gateway host as a bearer token.

## Support

- Email: support@gabrieloperator.com
- Issues: https://github.com/Gabriel-Operator/persona-builder/issues

## Related

- This repo is the create-from-scratch skill. Do not install only the authoring plugin and expect this flow.
- Child JSON authoring after git exists: [`Gabriel-Operator/gabriel-operator-coding-agent-plugin`](https://github.com/Gabriel-Operator/gabriel-operator-coding-agent-plugin)
- Gateway bootstrap: [`Gabriel-Operator/gabriel-operator-skills`](https://github.com/Gabriel-Operator/gabriel-operator-skills)
- Product: [gabrieloperator.com](https://gabrieloperator.com)
