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
  version: "1.3"
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

A workspace `gabi_` token is required before any create, publish, or Gateway
work. Read `$GABRIEL_TOKEN` or `$GABI_TOKEN`, or MCP
`Authorization: Bearer gabi_…` on `https://gabrieloperator.com/mcp/gateway`.

Use a **workspace** token (`gabi_…` with no twin binding), MCP Gateway preset:

- `api:access`
- `mcp:access`
- `digital-twin:admin`
- `digital-twin:chat`
- `digital-twin:tools`
- `digital-twin:media`
- `automation:read`
- `automation:run`

Never ask for a password or browser session when this token can perform the
work. Do not print the raw token after it is configured. Do not ask the user
to paste the raw token into chat.

### If the token is missing

Do not start building. Stop and tell the user how to get one:

1. Open https://gabrieloperator.com/signup (or **Sign up** on the homepage).
   First visit creates an account when they sign in.
2. Sign in with email code, Google, Apple, or phone.
3. Open **Workspace → Dashboard** (`/workspace/dashboard`).
4. Top-right pill labeled **Gateway API key**: click copy. The first copy
   mints a Dashboard (MCP Gateway) token that starts with `gabi_`.
5. They will not see the full token again unless they generate a new one.
   Never commit it. Never print it.
6. Connect MCP with that token, or set `GABRIEL_TOKEN` / `GABI_TOKEN`, then
   continue.

If the pill is empty or copy fails: **Generate new key**, or
https://gabrieloperator.com/workspace/developer-settings → **API Tokens** →
**Create New Token** → preset **MCP Gateway**.

Wait until MCP or the env var is actually connected before provisioning.

## Git provisioning (ask once)

Repo create on the **caller's GitHub** (`gabriel_create_git_repository`, `useGithubOAuth: true`) needs a connected GitHub account. A `gabi_` token cannot complete GitHub OAuth. The Edit persona **Create Git repository** default is **Gabriel-managed (internal) Git**, which does not need GitHub.

Before any git create/bind:

1. Call `gabriel_git_provisioning_status`.
2. If `provisioningMode` is `"managed"` — do **not** ask. Call `gabriel_provision_managed_git` for the page, each list, the pipeline, and each slash-command workflow (`kind` + id). That creates **and** binds. Skip `gabriel_create_git_repository` and `gabriel_initialize_*_git` for those resources.
3. If `provisioningMode` is `"own"` — if `githubConnected` is false, stop and send them to `connectGithubUrl` (Developer Settings → Connect GitHub). Wait until they confirm, re-check status, then use `gabriel_create_git_repository` + `initialize_*_git` with `useGithubOAuth: true`.
4. If `needsChoice` is true (`provisioningMode` is null), **ask the user once** and wait. Offer all three:

   - **Gabriel-managed (automated / shared internal Git)** — you create the repos now with no GitHub account. Same as Edit persona → Create Git repository → internal Git.
   - **Their own GitHub** — they connect GitHub at `connectGithubUrl`, then you create repos on that account.
   - **Don't ask again** — they can set **AI Resources setup** at https://gabrieloperator.com/workspace/settings?section=preferences (`Ask me each time` / `Always use internal Git` / `Always use my own GitHub`). If they want you to remember for this chat, call `gabriel_set_git_provisioning_preference` with `managed` or `own` after they choose.

Do not invent a GitHub PAT. Do not ask them to paste a GitHub token into chat.

If `gabriel_create_git_repository` still returns `GITHUB_NOT_CONNECTED`, show `askPrompt` from status (or the Preferences URL) instead of retrying.

## Interview loop

Do not dump a blank page and stop. Collect, propose, confirm, then provision.

1. Confirm the workspace token is available (env or MCP). If it is missing,
   follow **If the token is missing** above and wait. Then run **Git provisioning (ask once)** — do not probe with a throwaway `gabriel_create_git_repository`.
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
4. **Git** — follow **Git provisioning (ask once)** first.
   - **Managed:** `gabriel_provision_managed_git` for `kind=page` (`pageId`), each list (`kind=list`, `listId`), the pipeline (`kind=pipeline`, `pipelineId`). After slash commands exist, `kind=workflow` with `pageId`/`agentId` + `actionId`. Do not also create/initialize those repos.
   - **Own GitHub:** `gabriel_create_git_repository` then initialize page, lists, and pipeline (`useGithubOAuth: true`, `repoFullName`). Create slash-command actions **before** binding their workflow repos (`actionId` required).
5. **Slash-command workflows** — `gabriel_create_operator_command` with `pageId` and `trigger` (no leading slash). This mints the action promote looks for (`sourceMetadata.kind = persona_slash_command`) and returns `actionId`. Then bind git: managed `kind=workflow`, or `gabriel_initialize_workflow_git` with `agentId` = `pageId` and that `actionId`. Author `assets/workflow.json` with `workflow-builder`. After promote assigns resource keys, register the slash in `assets/chat-config.json` via `digital-twin-page` (`workflowRef` by resource key, never raw database ids). `gabriel_add_operator_action` with `agentId` = `pageId` now mints the same slash command (uses `trigger`, or derives it from `title`). Do not use `gabriel_list_flows` to check this — that lists page endpoints, not slash commands.
6. **Team agents** — `gabriel_create_team_agent` then `gabriel_initialize_team_agent_git` (own GitHub). Author `assets/team-agent.json` with the `team-agents` skill. These are page endpoints, not team-workspace Page Builder apps.
7. **Chat config** — session + `gabriel_update_twin_config` for name, first message, system prompt, model. Deep git edits use `digital-twin-page`.
8. **Portable workspace** — `gabriel_promote_workspace` (assigns resource keys and writes `references/registry.json`), then `gabriel_validate_workspace` (HTTP 200 with `ok: false` is a failure), then `gabriel_publish_workspace`.
9. **Optional** — `gabriel_publish_twin` for the live page, `gabriel_mint_persona_key` (returned once; `/api/v1` and `/mcp/persona` only).

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
| `gabriel_create_git_repository` | Create a GitHub repo on the **connected** account (fails without OAuth) |
| `gabriel_git_provisioning_status` | GitHub connected? Remembered AI Resources setup? Ask-once prompt copy |
| `gabriel_set_git_provisioning_preference` | Remember managed / own / ask (`null`) |
| `gabriel_provision_managed_git` | Create + bind Gabriel-managed git (no caller GitHub) |
| `gabriel_initialize_page_git` | OAuth-backed page git binding + scaffold |
| `gabriel_initialize_list_git` | Bind a list repo |
| `gabriel_initialize_pipeline_git` | Bind a pipeline repo |
| `gabriel_add_operator_action` | Add a generic operator action, **or** mint a persona slash command when `agentId` is a page |
| `gabriel_create_operator_command` | Create a persona slash command + linked action (required for promote) |
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

Workflow init also requires `actionId` from `gabriel_create_operator_command` (or from `gabriel_add_operator_action` when `agentId` is the page). Team-agent init requires `endpointId`.

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

curl -X GET $BASE/api/gateway/git-provisioning -H "$AUTH"

curl -X PUT $BASE/api/gateway/git-provisioning -H "$AUTH" -H 'Content-Type: application/json' \
  -d '{"provisioningMode":"managed"}'

curl -X POST $BASE/api/gateway/git-binding/provision-managed -H "$AUTH" -H 'Content-Type: application/json' \
  -d '{"kind":"page","pageId":"{pageId}"}'

curl -X POST $BASE/api/gateway/pages/{pageId}/git-binding/initialize -H "$AUTH" -H 'Content-Type: application/json' \
  -d '{"useGithubOAuth":true,"repoFullName":"your-org/sales-persona","defaultBranch":"main"}'

curl -X POST $BASE/api/gateway/pages/{pageId}/operator-commands -H "$AUTH" -H 'Content-Type: application/json' \
  -d '{"trigger":"qualify-a-buyer-business","label":"Qualify a buyer business","description":"Score a buyer inquiry."}'

curl -X POST $BASE/api/gateway/pages/{pageId}/team-agents -H "$AUTH" -H 'Content-Type: application/json' \
  -d '{"name":"Intake agent"}'

curl -X POST $BASE/api/gateway/pages/{pageId}/workspace/promote -H "$AUTH" -H 'Content-Type: application/json' \
  -d '{}'
```

## UI fallback when Gateway or MCP cannot do it

Prefer MCP/REST first. If the tool is missing, returns 404 / not implemented,
needs a browser OAuth or third-party API key, or keeps failing after a real
Gateway attempt, do not invent a workaround and do not ask the user to paste
vendor secrets into chat.

Hand the user the **Edit persona / Configure** screen instead, with a clickable
URL that includes the `pageId` you already have from `gabriel_create_page` or
`gabriel_list_resources`.

Personal workspace (same destination as the **Configure** button on the persona
page):

https://gabrieloperator.com/workspace/edit-persona/{pageId}

Business workspace:

https://gabrieloperator.com/c/{teamSlug}/{unitSlug}/edit-persona/{pageId}

Deep links the app honors:

| Need | URL |
|---|---|
| Composio / Arcade / Nango / Scalekit keys | `.../edit-persona/{pageId}?tab=simulated-world` then expand **MCP connectors** |
| Default LLM | `.../edit-persona/{pageId}?tab=simulated-world&section=llm-model` |
| Features (voice, computer, …) | `.../edit-persona/{pageId}?tab=input` |
| Slash commands / Canvas agents | `.../edit-persona/{pageId}?tab=ai-agents` |
| Lists / pipelines | `.../edit-persona/{pageId}?tab=pipelines-workflows` |
| Experience / output | `.../edit-persona/{pageId}?tab=output` |
| Phone / inbox / chat apps | `.../edit-persona/{pageId}?tab=reach` (`&section=phone`, `inbox`, or `chat-integrations`) |
| GitHub not connected (own GitHub path) | https://gabrieloperator.com/workspace/developer-settings |
| Remember git setup (AI Resources) | https://gabrieloperator.com/workspace/settings?section=preferences |

There is no `?section=mcp` deep link. For Composio, send `tab=simulated-world`
and tell them to expand **MCP connectors**.

### When to use this

- Saving Composio, Arcade, Nango, or Scalekit keys (profile keys, not Gateway)
- Connecting GitHub (`GITHUB_NOT_CONNECTED`)
- Voice/provider BYOK, computer providers, or other credential UIs
- Any configure field with no matching `gabriel_*` tool
- OAuth / “open this site and approve” flows

### Composio keys (common case)

Gateway cannot create the Composio API key. The user must do it in the UI:

1. Open
   https://gabrieloperator.com/workspace/edit-persona/{pageId}?tab=simulated-world
2. Or open the persona page and click **Configure**.
3. On the **Tools** tab, expand **MCP connectors**.
4. Choose **Composio**, add a key (label + API key), save, then enable the
   toolkits they need (Gmail, Sheets, Calendar, …).
5. After they confirm it is saved, continue with MCP/REST.

### How to tell the user

- Give the full `https://` URL, not “go to settings”.
- Name the tab (Tools, AI Operators, Data, …) and the control (MCP connectors,
  Add key, Connect GitHub).
- Wait for them to finish. Then retry the Gateway call.
- Never print or store the Composio/Arcade/vendor secret.

## Promote skipped the workflow (slash command vs generic action)

If `initialize_workflow_git` returned success, `gabriel_list_flows` is `[]`, and promote says **`workflow: no git-bound persona command found for this persona`**, tell the user this — do not retry the same `add_operator_action` + promote loop.

**What happened.** Promote only counts actions with `sourceMetadata.kind = persona_slash_command`. A generic operator action git-binds fine and is still skipped. `gabriel_list_flows` lists page workflow endpoints, not slash-command operator workflows. Empty `[]` is expected unless a team-agent / page flow exists.

**What to do next (token path).**

1. Call `gabriel_create_operator_command` with the persona `pageId` and the trigger (example: `qualify-a-buyer-business`). Keep the returned `actionId`.
2. Call `gabriel_initialize_workflow_git` with `agentId` = that `pageId` and the **new** `actionId`. Point it at the workflow repo already created. Do not reuse the old generic action id.
3. Call `gabriel_promote_workspace` again. The skip should disappear once that action is git-bound.
4. After promote assigns `workflowRef.resourceKey`, register the slash in `assets/chat-config.json` with that key (never raw database ids).

If a generic action was already created, leave it (or delete later). Its git binding will never count for promote.

**UI fallback** if Gateway create is unavailable:

https://gabrieloperator.com/workspace/edit-persona/{pageId}?tab=ai-agents

On **AI Operators** → **Operator Slash Commands**, use **Create command**, then bind git to **that** command’s action and promote.

Resource keys are assigned by promote. Do not write `workflowRef` into chat-config until promote has returned them.

## Safety

- Prefer MCP/REST. If Gateway cannot do it, send the user to Edit persona /
  Configure with the page URL (see **UI fallback when Gateway or MCP cannot do it**).
- Do not claim success unless the gateway returned success. For validate/publish, read `ok`.
- Do not mint or reprint persona keys unless the user asked.
- Do not store prompt bodies in notes you write back to git.
- Team-workspace Page Builder apps (`/teams/:teamId/.../apps`) are out of scope here. Page-scoped team-agent endpoints on the persona are in scope.
