---
name: persona-builder
description: >
  Interview a user and provision a Gabriel AI Persona end to end from a workspace
  Gabi token: page, lists, pipeline/machine, operator workflows, git repos and
  bindings, Page Builder team-agent endpoints, workspace promote/validate/publish,
  and optional persona-scoped key. Use this skill when the user wants to create
  a new persona from a description rather than edit an already-bound git repo.
metadata:
  author: gabriel-operator
  version: "1.0"
---

# Persona Builder

Orchestrator skill for coding agents. Given a workspace `gabi_` token and a
description of what the AI Persona should do, interview the user and create the
full stack through the Gabriel Gateway.

This skill **provisions** resources. After git bindings exist, hand off to
child skills to author the JSON definitions.

This skill ships only from [`Gabriel-Operator/persona-builder`](https://github.com/Gabriel-Operator/persona-builder).
That is the **create-from-scratch** pack.

[`Gabriel-Operator/gabriel-operator-coding-agent-plugin`](https://github.com/Gabriel-Operator/gabriel-operator-coding-agent-plugin)
is the **edit-existing-resources** pack (`workflow-builder`, `list-builder`, `pipeline-builder`, `team-agents`, …).
Installing only that repo does **not** load this interview flow unless `skills/persona-builder` is present.

The gateway bootstrap skill is a third pack:
`npx github:Gabriel-Operator/gabriel-operator-skills add ./gabriel-operator`.

## Using this skill in coding agents

Install **this** pack, then connect MCP. Do not stop after adding the authoring plugin.

| Agent | Install this skill |
|-------|---------|
| **NPX / Cursor / Windsurf** | `npx skills add Gabriel-Operator/persona-builder` |
| **Claude Code** | `/plugin marketplace add Gabriel-Operator/persona-builder` then `/plugin install persona-builder@persona-builder` |
| **Codex** | `codex plugin marketplace add Gabriel-Operator/persona-builder --sparse .agents/plugins` then install **Persona Builder** |
| **Grok Build** | `grok plugin marketplace add Gabriel-Operator/persona-builder` then `grok plugin install persona-builder --trust` |
| **OpenClaw** | `npx skills add Gabriel-Operator/persona-builder` then `openclaw gateway connect` |
| **Runtime fallback** | MCP `gabriel_get_skill_instructions` with `{ "topic": "persona-builder" }` |
| **Child JSON authoring (after git exists)** | `Gabriel-Operator/gabriel-operator-coding-agent-plugin` |
| **Gabriel Operator monorepo** | `cp -R server/skills/persona-builder ./your-workspace/` |

Required MCP (workspace `gabi_` token):

```json
{
  "mcpServers": {
    "gabriel": {
      "url": "https://gabrieloperator.com/mcp/gateway",
      "headers": {
        "Authorization": "Bearer gabi_<token>"
      }
    }
  }
}
```

Local origin example: `http://localhost:3000/mcp/gateway`.

## Authentication

Use a **workspace** token (`gabi_…` with no twin binding), MCP Gateway preset:

- `api:access`
- `mcp:access`
- `digital-twin:admin`
- `digital-twin:chat`
- `digital-twin:tools`
- `digital-twin:media`
- `automation:read`
- `automation:run`

Read `$GABRIEL_TOKEN` or `$GABI_TOKEN` if set. Never ask for a password or
browser session when this token can perform the work. Do not print the raw
token after it is configured.

The caller's Gabriel account must have **GitHub connected**. Repo create and
OAuth git-init fail with `GITHUB_NOT_CONNECTED` otherwise.

## Interview loop

Do not dump a blank page and stop. Collect, propose, confirm, then provision.

1. Confirm the workspace token is available and GitHub is connected (`gabriel_create_git_repository` with a throwaway name is enough to learn `GITHUB_NOT_CONNECTED`; prefer asking once).
2. Ask what the persona does if the user did not already describe it. Derive a title, short description, slug, and visibility (`private` default). Confirm before creating.
3. Propose the data model: which lists (columns), which pipeline/machine (stages + transitions), which operator workflows / slash commands, and whether Page Builder event team agents are needed. Confirm the proposal.
4. Provision in the order below. Create lists and pipelines **before** any config that stores their ids.
5. After git bindings exist, fetch child skill markdown (`gabriel_get_skill_instructions`) and author definitions.
6. Promote the workspace to a portable registry, validate, then publish. Optionally mint a persona-scoped key.
7. Summarize with user-facing names, page URL/slug, and what to do next. Do not dump internal ids unless asked.

Ask before publishing or minting keys.

## Build order

Gateway REST lives under `https://gabrieloperator.com/api/gateway`. Prefer MCP tools when connected.

1. **Page** — `gabriel_create_page` / `POST /pages` (`title`, `description`, `pageSlug`, `visibility`). Keep `pageId`. The persona operator id is this `pageId`.
2. **Lists** — `gabriel_create_data_list` / `POST /data-lists` with `pageId` and `columns`. Do this before any twin config that names a list.
3. **Pipeline** — `gabriel_create_pipeline` / `POST /pipelines` with `pageId` and `stages`. Then `gabriel_update_pipeline_stages` / `PUT /pipelines/{pipelineId}/stages` with **both** `stages` and `transitions` (create accepts stages but drops transitions).
4. **Git repos** — `gabriel_create_git_repository` / `POST /git-repositories` for the page, each list, the pipeline, each workflow action, and each team agent. Idempotent: an existing name returns `{ created: false }`.
5. **Git bindings** — initialize page, lists, pipeline, then workflow actions (`actionId` required). Use `useGithubOAuth: true` and `repoFullName`.
6. **Operator actions** — `gabriel_add_operator_action` with `agentId` = `pageId`. Bind each action's workflow repo. Author `assets/workflow.json` with `workflow-builder`. Register slash commands in `assets/chat-config.json` via `digital-twin-page` (`workflowRef` by resource key, never raw database ids in portable files).
7. **Team agents** — `gabriel_create_team_agent` then `gabriel_initialize_team_agent_git`. Author `assets/team-agent.json` with the `team-agents` skill. These are page endpoints, not team-workspace Page Builder apps.
8. **Chat config** — session + `gabriel_update_twin_config` for name, first message, system prompt, model. Deep git edits use `digital-twin-page`.
9. **Portable workspace** — `gabriel_promote_workspace` (assigns resource keys and writes `references/registry.json`), then `gabriel_validate_workspace` (HTTP 200 with `ok: false` is a failure), then `gabriel_publish_workspace`.
10. **Optional** — `gabriel_publish_twin` for the live page, `gabriel_mint_persona_key` (returned once; `/api/v1` and `/mcp/persona` only).

A config that points at a missing list or pipeline id does not raise. The feature skips silently. If the persona chats but never acts, check those ids first.

## MCP tools for this skill

| Tool | Purpose |
|---|---|
| `gabriel_create_page` | Create the AI Persona page |
| `gabriel_create_session` | Target-bound session for config/chat |
| `gabriel_update_twin_config` | Patch safe chat/config fields |
| `gabriel_publish_twin` | Publish the live persona page |
| `gabriel_mint_persona_key` | Mint a persona-scoped `/api/v1` key |
| `gabriel_create_data_list` | Create a list + collection |
| `gabriel_create_pipeline` | Create a pipeline/machine |
| `gabriel_update_pipeline_stages` | Replace stages **and** transitions |
| `gabriel_create_git_repository` | Create a GitHub repo on the connected account |
| `gabriel_initialize_page_git` | OAuth-backed page git binding + scaffold |
| `gabriel_initialize_list_git` | Bind a list repo |
| `gabriel_initialize_pipeline_git` | Bind a pipeline repo |
| `gabriel_add_operator_action` | Add a workflow action under the persona operator |
| `gabriel_initialize_workflow_git` | Bind one action's workflow repo (`actionId` required) |
| `gabriel_create_team_agent` | Create a page-scoped event team-agent endpoint |
| `gabriel_initialize_team_agent_git` | Bind that endpoint to git and scaffold `team-agents` |
| `gabriel_promote_workspace` | Assign portable resource keys + write registry v2 |
| `gabriel_validate_workspace` | Check the bundle (`ok` field, not HTTP status) |
| `gabriel_publish_workspace` | Pin submodule revisions on the persona root |
| `gabriel_get_skill_instructions` | Load child skill markdown by topic |

Git-init bodies:

```json
{
  "useGithubOAuth": true,
  "repoFullName": "your-org/persona-lists",
  "defaultBranch": "main"
}
```

Workflow init also requires `actionId`. Team-agent init requires `endpointId`.

## Child skills (after repos exist)

Request `gabriel_get_skill_instructions` with:

| Topic | When |
|---|---|
| `digital-twin-page` | `assets/chat-config.json`, registry, slash-command registration |
| `list-builder` | `assets/list.json` |
| `pipeline-builder` | `assets/pipeline.json` |
| `workflow-builder` | `assets/workflow.json` |
| `team-agents` | `assets/team-agent.json` + `assets/task-orchestration.json` |
| `digital-twin-embed` | `assets/embed-config.json` |
| `overview` | Central gateway skill |

Never write `pageId`, `userId`, `actionId`, `listId`, or other database ids into portable git definitions. Use `resourceKey` / `workflowRef`.

## REST fallback

When MCP is not connected, curl the same operations:

```bash
AUTH='Authorization: Bearer gabi_<token>'
BASE=https://gabrieloperator.com

curl -X POST $BASE/api/gateway/pages -H "$AUTH" -H 'Content-Type: application/json' \
  -d '{"title":"Sales Persona","description":"Qualifies leads.","visibility":"private"}'

curl -X POST $BASE/api/gateway/data-lists -H "$AUTH" -H 'Content-Type: application/json' \
  -d '{"pageId":"{pageId}","name":"Leads","columns":[{"key":"email","label":"Email","type":"text"}]}'

curl -X POST $BASE/api/gateway/pipelines -H "$AUTH" -H 'Content-Type: application/json' \
  -d '{"pageId":"{pageId}","name":"Lead lifecycle","stages":[{"id":"new","name":"New"}]}'

curl -X PUT $BASE/api/gateway/pipelines/{pipelineId}/stages -H "$AUTH" -H 'Content-Type: application/json' \
  -d '{"stages":[{"id":"new","name":"New"},{"id":"qualified","name":"Qualified"}],"transitions":[{"id":"qualify","name":"Qualify","fromStageId":"new","toStageId":"qualified","trigger":"manual"}]}'

curl -X POST $BASE/api/gateway/git-repositories -H "$AUTH" -H 'Content-Type: application/json' \
  -d '{"name":"sales-persona","isPrivate":true}'

curl -X POST $BASE/api/gateway/pages/{pageId}/git-binding/initialize -H "$AUTH" -H 'Content-Type: application/json' \
  -d '{"useGithubOAuth":true,"repoFullName":"your-org/sales-persona","defaultBranch":"main"}'

curl -X POST $BASE/api/gateway/pages/{pageId}/team-agents -H "$AUTH" -H 'Content-Type: application/json' \
  -d '{"name":"Intake agent"}'

curl -X POST $BASE/api/gateway/pages/{pageId}/workspace/promote -H "$AUTH" -H 'Content-Type: application/json' \
  -d '{}'
```

## Safety

- Prefer MCP/REST over driving the Gabriel browser UI.
- Do not claim success unless the gateway returned success. For validate/publish, read `ok`.
- Do not mint or reprint persona keys unless the user asked.
- Do not store prompt bodies in notes you write back to git.
- Team-workspace Page Builder apps (`/teams/:teamId/.../apps`) are out of scope here. Page-scoped team-agent endpoints on the persona are in scope.
