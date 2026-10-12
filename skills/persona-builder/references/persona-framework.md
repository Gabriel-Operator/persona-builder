# Source-to-outcome contract

## Three connected layers

The data layer supplies facts with provenance and an ontology projection. The
execution layer records why a signal fired, the prepared playbook and the
human-approved effects. The feedback layer records committed results, accepted
outcomes, cost and reviewable calibration proposals. These are linked evidence,
not three independently invented success stories.

## Authored definitions

| Definition | Maintained location |
| --- | --- |
| Shared semantic contract | Parent Persona `assets/ontology.json` |
| Chat App layout/actions and disabled Signal presets | `assets/chat-app.json`, mirrored exactly into `publishedConfig.chatApp` |
| Portable Data Feed catalog | `assets/data-feeds/catalog.json`, referenced by `publishedConfig.dataFeedCatalog` |
| Feed definitions/source workflows | Catalog-declared JSON files under `assets/data-feeds/` |
| Operator workflow | Bound child `assets/workflow.json` |
| State-machine persistence | Bound child `assets/pipeline.json` and referenced List schema |
| Public ROI assumptions | Authored landing calculator model |
| Outcome mappings/valuation | `publishedConfig.roiMonitoring`, using the actual calculator path and ontology |

Preserve resource keys, command IDs and schema keys. Country, authenticated
audience and language select definitions/vocabulary without changing actor
identity or permission. Each execution captures the selected semantic revision;
downstream mapping and historical ROI reuse that snapshot.

## Acquisition and persistence

A feed declares one CRUD operation, input/output schemas, exact domains,
credential-slot descriptors and optional acquisition/ontology contracts. It may
perform its authorized source operation; it returns JSON and never chooses a
destination List or silently persists its result into the Persona's storage.
Invoke the exact feed revision from an authorized parent playbook. Mobile
Guardians and scoped grants supply credentials outside authored definitions.
Do not expose a direct ordinary-caller invocation path or put secret values in
inputs, URLs, examples or model repositories.

Validate JSON against its schema and selected ontology. A following approved
`list_mapping` stage chooses an authorized `listRef`/List and uses existing
mutation paths. Feed provenance, cursors and observation identities belong to
the consuming playbook's saved output/state. Retry persisted results rather
than repeating a source mutation. Uncertain external effects require recovery
review. Source discovery returns candidates; a separate reviewed mapping can
register/evaluate them and make approved sources available to the next loop.

## Signals and playbooks

Author disabled presets only on an automations data point. They reference
portable Lists, declared command actions and optional ontology projections.
The runner configures, saves and explicitly enables a runtime Signal. Record
the event, prior/current values, condition result and selected action. A signal
can prepare or request a playbook; it is not human approval for spending,
introductions, publication or other irreversible effects.

Keep authored operator workflows distinct from captured Canvas playbooks.
Captured IDs, completed stages and replay evidence are runtime data. Replay
checks the stored definition/revision, parameter contract, current access and
approval policy. Saving a playbook does not certify that it ran successfully.
Lists and state-machine transitions retain their existing ownership rules.

## ROI and feedback

Public calculator inputs are disclosed planning assumptions. Actual ROI uses
accepted, committed outcomes with deduplicated identities and captured ontology
projections. A generated row, gross transaction amount, sandbox action or prose
claim is not realized value. Time value requires the runner's explicit baseline,
paired preparation/review effort, hourly value and matching currency. Monetary
savings require comparable committed baseline/actual amounts at the declared
accepted boundary. Preserve withdrawals, negative values and coverage gaps.

Keep provider expense, net credit consumption and credit top-up funding
separate. Comparable known positive cost permits
`ROI% = (eligible value - cost) / cost * 100`; unknown/mismatched/zero cost does
not yield a fabricated percentage. Outcome feedback may inform a proposal for
new assumptions or thresholds. Activation remains an authorized, revision-
checked action; negative ROI does not create an implicit self-modification rule.

## Worked design example

A household persona can acquire inventory observations, project Food/Stock
attributes, detect depletion, prepare a meal or restock plan, request the
parent's exact-plan approval, and record the confirmed result. A retail persona
uses Store/Product/Supplier/PurchaseOrder concepts and manager approval. The
domain determines the entities, sources, conditions and accepted outcome; the
same evidence chain applies. Fictional calculator amounts remain assumptions
until an actual accepted result supports their valuation.
