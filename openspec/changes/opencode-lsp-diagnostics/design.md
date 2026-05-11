## Context

`opencode debug lsp diagnostics <file>` spawns an LSP server for the file type, sends `initialize`, then sends file-watcher notifications (`workspace/didChangeWatchedFiles`) and opens the file (`textDocument/didOpen`). It then waits for diagnostics published via `textDocument/publishDiagnostics`.

Today, `purescript-analyzer`:

- does not register a handler for `workspace/didChangeWatchedFiles`, causing `async_lsp` to error and the server to panic.
- only triggers diagnostics collection on `textDocument/didSave`, which opencode does not send as part of its diagnostics command.

The server already has a diagnostics pipeline via `event::emit_collect_diagnostics` and `event::collect_diagnostics` that converts lowering/resolving/checking errors into LSP diagnostics and calls `publish_diagnostics`.

## Goals / Non-Goals

**Goals:**

- Prevent crashes on common, optional LSP notifications emitted by clients (specifically `workspace/didChangeWatchedFiles`).
- Ensure diagnostics are published for opencode’s open-driven workflow.
- Provide simple configurability for when diagnostics are published, with `diagnostics-on-change` defaulting to off.
- Keep changes minimal and localized to the LSP binary crate.

**Non-Goals:**

- Implement LSP pull-diagnostics (`textDocument/diagnostic`).
- Add throttling/debouncing, incremental diagnostics, or background build orchestration.
- Change typechecker semantics or diagnostics content.

## Decisions

- Handle `workspace/didChangeWatchedFiles` as a no-op notification.
  - Rationale: opencode and editors send this routinely. The analyzer does not currently use file watcher deltas, so ignoring is safe and prevents panics.

- Publish diagnostics using the existing internal `CollectDiagnostics` event.
  - Rationale: keeps the diagnostics generation logic in one place and maintains a single publish path.

- Add CLI flags (not initializationOptions) for diagnostics triggers.
  - Rationale: simplest configuration surface given current code structure. opencode can pass flags in its per-project config; editors can choose defaults.

- Defaults:
  - `--diagnostics-on-open`: enabled by default, so opencode can get diagnostics from `didOpen` without extra configuration.
  - `--diagnostics-on-change`: disabled by default (opt-in), per user preference.
  - `--diagnostics-on-save`: enabled by default to preserve typical editor behavior.

## Risks / Trade-offs

- Diagnostics on open/change can be expensive in large workspaces.
  - Mitigation: keep `diagnostics-on-change` default off; opencode can opt in per-project. Leave room for future debouncing.

- Publishing diagnostics on every `didChange` might produce stale publishes when cancellation occurs.
  - Mitigation: existing query engine cancellation (`engine.request_cancel()`) already attempts to terminate in-flight work. If needed, later add versioning or coalescing.

- Ignoring watched file notifications could miss future incremental workspace update opportunities.
  - Mitigation: treat as no-op for now; future work can translate watched file changes into targeted `on_change`/reload behavior.
