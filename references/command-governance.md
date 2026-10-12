# Command governance and estimated cost

When authoring a persona slash command, inspect the actual workflow for irreversible
and unknown effects. Add concrete previews and authored confirmation boundaries
where useful. Governance findings are warnings and do not block activation;
runtime still requires an exact human approval before irreversible effects.
Never mark opaque code/computer execution as safe using a prompt or read-only hint.
Use controlled tools for effects; unsupported providers are held for review.

Use `gabriel_validate_command_governance` and `gabriel_estimate_command_cost` with
the page and command ids. Include the findings, effective review preset, selected
worker/reviewer models, estimated provider USD spend, Gabriel credits, unknown
fees, and comparisons with/without optional AI reviews in the build handoff.
Do not run a real command merely to calculate its first-run quote.

Defaults: `every_stage` with correctness and policy reviewers enabled. Authors may
choose `protected_and_final` or `rules_only`, select models, enable content checks,
and configure stage overrides. Persona defaults inherit into commands. Store only
portable author settings (`governanceConfig` on the persona; `governance` on the
slash command); do not commit quote/run/proposal ids or approval records to Git.
Mandatory human approval, authorization and audit are never optional settings.

Read settings with `gabriel_get_command_governance`. Save optional overrides with
`gabriel_update_command_governance` using its returned `expectedRevision`.
Observe runs with `gabriel_get_governance_run` and `gabriel_list_governance_events`.
`gabriel_request_action_approval` returns a human review URL for a server-owned
proposal; API/MCP agents cannot approve by submitting `approved: true`.

Quotes are advisory. Prefer authorized comparable history; first runs use labelled
stage/token allowances. Current verified prices take precedence over display
fallbacks. Unknown browser/media/tool fees remain unknown, not zero. A known
subtotal is not a full run total. Rules-only comparisons retain human approvals
and audit. Never silently remove checks because they cost money. Reconcile uncertain
external effects rather than retrying them. AI review assesses configured policies
and observed evidence; it cannot guarantee ethical correctness.
