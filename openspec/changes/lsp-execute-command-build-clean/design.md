## Context

The current LSP server (Rust, `async_lsp`) supports push diagnostics for a single document via an internal event (`CollectDiagnostics`) triggered on `didOpen`/`didSave`/optional `didChange`. It does not currently implement `workspace/executeCommand`.

Many editor integrations (notably `purescript-language-server`) expose `purescript.build` / `purescript.clean` as execute-commands. We want to provide a compatible subset while keeping this server’s default diagnostics behavior unchanged.

Constraints:

- Only advertise commands we actually implement.
- `purescript.build` must run a real build (spago or purs) and publish diagnostics derived from compiler JSON errors.
- Build diagnostics should be tracked separately from analyzer diagnostics and merged for publication, suppressing stale duplicates by preferring build diagnostics at the same range.
- `purescript.clean` must delete only the workspace `output/` directory.
- Provide `purescript.reset` (fast diagnostic/caches reset) and `purescript.analyzerRefresh` (manual analyzer diagnostics run for workspace source files).

## Goals / Non-Goals

**Goals:**

- Implement `workspace/executeCommand` request handling and advertise `executeCommandProvider`.
- Implement the four commands: `purescript.build`, `purescript.clean`, `purescript.reset`, `purescript.analyzerRefresh`.
- Support build tool selection for `purescript.build`: spago or purs (plus optional escape hatch command string if needed).
- Publish build diagnostics by parsing JSON error output and mapping to `publishDiagnostics`, without broadcasting empty diagnostics for every workspace file.
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
   
   Decision: Use the existing internal event pattern (`state.client.emit(...)` + `router.event_ext(...)`) for long-running work (build, clean, workspace refresh). Keep `purescript.reset` inline in the execute-command handler so diagnostic clears happen immediately and deterministically for the request.
   
   Rationale: Matches current architecture for blocking work, while reset is intentionally lightweight and benefits from direct access to current diagnostic/open-file state.
   
   Alternatives:
   
   - Do every command directly in the executeCommand handler. Rejected because build/clean/refresh can block request handling.

3. **Diagnostics publication model**
   
   Decision: Track build diagnostics and analyzer diagnostics separately. When publishing diagnostics for a URI, merge build diagnostics first, then analyzer diagnostics, suppressing analyzer diagnostics at ranges already covered by build diagnostics.
   
   Rationale: Build output should not leave stale diagnostics behind, but analyzer diagnostics for distinct ranges remain useful. Separating sources also lets clean/reset remove only the relevant diagnostic source.
   
   Trade-off: After a build, users can still see analyzer diagnostics that do not overlap build diagnostics. This is intentional to preserve useful analyzer feedback while avoiding duplicate/stale diagnostics.

4. **Build publication scope**

   Decision: On build, publish diagnostics only for files that previously had build diagnostics and files reported by the current build. Do not publish empty diagnostics for every known workspace file.

   Rationale: Avoids flooding clients with large numbers of diagnostic notifications on large workspaces.

   Trade-off: Analyzer diagnostics remain visible on files untouched by build output unless reset, clean, or a later per-file analyzer publication updates them.

5. **`purescript.clean` semantics**
   
   Decision: Delete only `<workspaceRoot>/output` recursively. Treat a missing output directory as success. Clear stored build diagnostics and republish merged diagnostics for files that previously had build diagnostics.
   
   Rationale: Explicit requirement and safest interpretation of “clean compiled output”.
   
   Safety: Validate the target path is exactly `root/output` (canonicalize/absolutize) and refuse otherwise.

6. **`purescript.reset` semantics**
   
   Decision: Implement reset as a fast diagnostic reset. Cancel in-flight analyzer work, bump diagnostics generation to suppress stale background publications, clear stored build/analyzer diagnostics, publish empty diagnostics for known diagnostic URIs and currently open file URIs, and invalidate workspace-symbol/suggestion caches. Keep loaded file contents and do not rediscover/reload workspace source files.
   
   Rationale: Provides a deterministic, low-latency way to clear stale diagnostics without triggering an expensive project reload on large workspaces. Users can explicitly recompute analyzer diagnostics with `purescript.analyzerRefresh` or run `purescript.build`.

   Trade-off: Reset does not pick up new/deleted source files by itself; that remains tied to initialization/source loading and later file events.

7. **Analyzer refresh scope**

   Decision: `purescript.analyzerRefresh` recomputes analyzer diagnostics only for refreshable workspace PureScript source files: `file://` `.purs` files under the workspace root, excluding `.spago`, `output`, `.git`, and `node_modules`. It clears stored analyzer diagnostics for non-refreshable file URIs while preserving any build diagnostics for those URIs.

   Rationale: Manual refresh should cover user source/test files while avoiding dependency, generated, external, and non-file diagnostics that confuse clients or create noisy output.

8. **Build tool support**
   
   Decision: Support spago and purs directly. Prefer spago by default when `spago.lock` is used to discover sources; otherwise use purs if configured.
   
   Spago invocation: `spago build --json-errors` (plus optional user args forwarded through `--purs-args`).
   
   Purs invocation: `purs compile --json-errors <workspace sources...>` (plus optional user args).
   
   Rationale: Mirrors common workflows. JSON errors are required to turn build output into file-scoped diagnostics.

9. **Diagnostics parsing**
   
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

- **Fast reset scope**: Reset no longer reloads workspace sources.
  
  Mitigation: Keep reset focused on diagnostics/caches. Use analyzer refresh or build for recomputation; rely on normal file events/source discovery for loaded file updates.
