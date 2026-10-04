# Embedded offline runtime contract v1

Use this contract only with a runtime that reports `contractVersion: 1`.

- **Storage:** encrypted SQLite, isolated by account and workspace. Runtime records use stable `kind`, `entityId`, `revision`, `baseRevision`, `status`, `deleted`, and JSON `payload` fields. Files referenced by records must already be present in the prepared local pack.
- **Implemented local operations:** `prompt.local`, `knowledge.search`, `data.read`, `data.write`, `list.read`, `list.write`, `pipeline.transition`, `task.write`, `schedule.write`, `form.collect`, `approval.request`, `condition`, `loop.bounded`, `wait.durable`, `artifact.write`, and `signal.evaluate`.
- **Capability report:** inspect nested `steps`, `body`, `then`, `else`, `nodes`, and `children`. Report the compatible count and every blocking path before execution.
- **Connectivity:** HTTP, REST, MCP, web search, SaaS, remote databases, notifications, cloud models/media, telephony, and meetings require a connection. Desktop browser, coding, and computer-control steps require an installed desktop adapter. Local speech, vision, and media require installed device models.
- **Execution:** unsupported steps become `blocked`; reconnecting or synchronizing never executes them. Runs persist checkpoints, outputs, errors, waits, and approvals.
- **Schedules:** desktop runs while the Electron runtime is running and the computer is awake. Mobile runs only while an app shell is active and resumes after reopening. Missed recurring times coalesce into one catch-up occurrence. Stable occurrence IDs and one owner device prevent duplicate dispatch.
- **Synchronization:** synchronization is manual. A mutation and outbox entry commit together. `Sync now` performs a bounded push/pull session with idempotent operation IDs, tombstones, checkpoints, and retained conflicts. Token expiry blocks synchronization but never local access. Do not start sync from connectivity changes.
- **Separation:** Git sync carries authored definitions only. Runtime sync carries selected account/Persona data. Execution state remains with its owner device. Runner-memory export and cloud uploads remain explicit operations.
- **Current boundary:** chat and operational data have explicit sync paths. The server implements resumable attachment endpoints, but skills must not claim automatic attachment transfer until a client invokes them. External side effects remain blocked offline.

The embedded runtime is distinct from the Docker Portable Runtime appliance. Detect the environment and its capability report; never silently redirect a local request to hosted APIs.

## Local model imports and capability contract

- Custom model manifests use version 1 and pin an immutable Hugging Face revision, artifact format, SHA-256, byte size, runtime engine, context/capability metadata, and opaque installation ID. Accept model data only. Never run repository Python, JavaScript, templates, or install scripts.
- Keep Hugging Face read tokens ephemeral. They may authorize inspection and the requested download, but must never enter manifests, authored Persona assets, logs, exports, runtime synchronization, or Git.
- Desktop accepts verified GGUF through a release-packaged `llama.cpp` protocol sidecar and LiteRT-LM files through its LiteRT sidecar. Mobile catalogs compatible GGUF and LiteRT-LM files, but a model is selectable only when that build reports its native engine available. Apple Foundation Models remain OS-managed.
- Browser-local inference uses WebLLM, WebGPU, a trusted application-pinned runtime library, and compatible MLC repositories from the shipped catalog. OPFS holds model assets. A native GGUF or `.litertlm` file is not browser-compatible by itself.
- Capability states are separate: `chat`, `structured-output`, and verified tool use. Loading a model proves none of the latter two. A playbook with an output schema requires schema validation. A model-driven tool step requires verified tool use and the device/runtime adapter it invokes.
- The embedded desktop runner limits a model to eight tool turns, validates every argument schema, exposes only local registered operations, requires explicit permission for writes, and assigns stable operation IDs. Current phone local chat remains tool-free unless its build supplies and verifies the complete native tool cycle.
- Signals and schedules include nested model requirements in their offline report. Their selected model and execution owner are device runtime state and never belong in authored definitions. Reconnecting or pressing Sync must not replay a completed tool call, Signal action, or scheduled occurrence.
- Show the third-party risk acknowledgement, license, size, memory estimate, format, publisher, runtime availability, and verified/unverified capabilities before acquisition. A downloaded but unavailable model remains installed data, not an executable capability.
