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

### Grok Build

```bash
grok plugin marketplace add Gabriel-Operator/persona-builder
grok plugin install persona-builder --trust
```

Also connect MCP Gateway with a workspace `gabi_` token. See [`SKILL.md`](SKILL.md).

## Related

- Full authoring plugin: [`Gabriel-Operator/gabriel-operator-coding-agent-plugin`](https://github.com/Gabriel-Operator/gabriel-operator-coding-agent-plugin)
- Product: [gabrieloperator.com](https://gabrieloperator.com)
