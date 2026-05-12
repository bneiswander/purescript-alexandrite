## Context

The current LSP server (Rust, `async_lsp`) supports push diagnostics for a single document via an internal event (`CollectDiagnostics`) triggered on `didOpen`/`didSave`/optional `didChange`. It does not currently implement `workspace/executeCommand`.

Many editor integrations (notably `purescript-language-server`) expose `purescript.build` / `purescript.clean` as execute-commands. We want to provide a compatible subset while keeping this server’s default diagnostics behavior unchanged.

Constraints:

- Only advertise commands we actually implement.
- `purescript.build` must run a real build (spago or purs) and publish diagnostics derived from compiler JSON errors.
- Build diagnostics should replace analyzer diagnostics (clear analyzer diagnostics for known files before publishing build results).
- `purescript.clean` must delete only the workspace `output/` directory.
- Provide `purescript.reset` (clear all diagnostics and reset/reload analyzer state) and `purescript.analyzerRefresh` (manual workspace analyzer diagnostics run).

## Goals / Non-Goals

**Goals:**

- Implement `workspace/executeCommand` request handling and advertise `executeCommandProvider`.
- Implement the four commands: `purescript.build`, `purescript.clean`, `purescript.reset`, `purescript.analyzerRefresh`.
- Support build tool selection for `purescript.build`: spago or purs (plus optional escape hatch command string if needed).
- Publish build diagnostics by parsing JSON error output and mapping to `publishDiagnostics`.
- Keep existing automatic analyzer diagnostics triggers unchanged.

**Non-Goals:**

- Implement other `purescript-language-server` commands (case split, add clause, etc.).
- Introduce automatic project-wide diagnostics refresh on save/change.
- Fully emulate IDE server behavior.

## Decisions

1. **Command surface and compatibility**
   
   Decision: Implement and advertise only `purescript.build`, `purescript.clean`, `purescript.reset`, `purescript.analyzerRefresh`.
   
   Rationale: Keeps UI clean, avoids advertising unsupported functionality, still supports the common build/clean workflow.
   
   Alternatives:
   
   - Advertise all upstream `purescript-language-server` commands and error at runtime. Rejected because it clutters command palettes and breaks expectations.

2. **Execution model**
   
   Decision: Use the existing internal event pattern (`state.client.emit(...)` + `router.event_ext(...)`) for long-running work (build, reset, workspace refresh).
   
   Rationale: Matches current architecture, runs work in `spawn_blocking`, and benefits from existing query cancellation behavior.
   
   Alternatives:
   
   - Do work directly in the executeCommand handler. Rejected because it would block request handling and complicate cancellation.

3. **Diagnostics “source of truth” for build**
   
   Decision: For `purescript.build`, clear diagnostics for all known workspace files first, then publish build diagnostics parsed from compiler JSON errors. Do not attempt to merge with analyzer diagnostics.
   
   Rationale: Matches requested behavior (“build should replace any analyzer diagnostics”).
   
   Trade-off: Analyzer diagnostics can reappear on subsequent `didOpen`/`didSave` triggers; that is acceptable given the requirement to keep default behavior unchanged.

4. **`purescript.clean` semantics**
   
   Decision: Delete only `<workspaceRoot>/output` recursively.
   
   Rationale: Explicit requirement and safest interpretation of “clean compiled output”.
   
   Safety: Validate the target path is exactly `root/output` (canonicalize/absolutize) and refuse otherwise.

5. **`purescript.reset` semantics**
   
   Decision: Clear diagnostics for all known workspace files, reset analyzer state by recreating `QueryEngine` + `Files` (including `prim::configure`), then reload workspace source files using the same source discovery as initialization (`spago.lock` or `--source-command`).
   
   Rationale: Provides a deterministic “back to clean slate” action that also clears build diagnostics.

6. **Build tool support**
   
   Decision: Support spago and purs directly. Prefer spago by default when `spago.lock` is used to discover sources; otherwise use purs if configured.
   
   Spago invocation: `spago build --purs-args "--json-errors"` (plus optional user args).
   
   Purs invocation: `purs compile --json-errors <workspace sources...>` (plus optional user args).
   
   Rationale: Mirrors common workflows. JSON errors are required to turn build output into file-scoped diagnostics.

7. **Diagnostics parsing**
   
   Decision: Implement a small JSON parser for PureScript compiler JSON errors (shared by both spago and purs). Convert to `lsp_types::Diagnostic` with `source = build/spago` or `build/purs`.
   
   Alternative: shell out and attempt to scrape text output. Rejected because it is fragile and hard to map to per-file diagnostics.

## Risks / Trade-offs

- **Parsing format drift**: PureScript JSON error format may change across compiler versions.
  
  Mitigation: Parse only the fields we need (path, message, position/range). Add unit tests with captured fixtures.

- **Workspace file coverage**: `purs compile` requires an explicit file list; deriving this from current workspace sources might differ from user’s preferred build inputs.
  
  Mitigation: Provide configuration for additional purs args; rely on existing source discovery (spago.lock or `--source-command`).

- **Cancellation**: Killing external build processes reliably is OS-dependent.
  
  Mitigation: Best-effort cancellation (track child process and kill on a new build request). Always avoid blocking the LSP main loop.

- **File deletion safety**: `purescript.clean` deletes directories.
  
  Mitigation: Restrict deletion to `<root>/output` only after path validation. Treat missing output dir as success.
