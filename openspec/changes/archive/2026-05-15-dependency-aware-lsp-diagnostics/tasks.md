## 1. Workspace Graph

- [x] 1.1 Add LSP-side workspace graph state for file-to-imports and file-to-dependants relationships.
- [x] 1.2 Derive graph edges from indexed imports and module-file mappings for refreshable workspace source files.
- [x] 1.3 Update graph entries when files are loaded or changed, including module name/import changes.
- [x] 1.4 Add unit tests for graph construction, reverse dependant lookup, transitive closure, and excluded URI handling.

## 2. Diagnostic Scheduler

- [x] 2.1 Introduce a diagnostic scheduler/batch abstraction with generation tokens for stale-result suppression.
- [x] 2.2 Add bounded background execution for diagnostic jobs instead of spawning one unbounded task per file.
- [x] 2.3 Implement priority ordering for saved files, open direct dependants, open transitive dependants, closed direct dependants, and closed transitive dependants.
- [x] 2.4 Preserve cancellation/reset behavior so older diagnostic batches cannot republish stale results.

## 3. Publication Deduplication

- [x] 3.1 Track last published merged diagnostics per URI.
- [x] 3.2 Skip `publishDiagnostics` when the merged diagnostics for a URI are unchanged.
- [x] 3.3 Ensure clearing previously published diagnostics still publishes an empty diagnostic set.
- [x] 3.4 Add tests for unchanged diagnostics, changed diagnostics, and cleared diagnostics publication behavior.

## 4. Trigger Integration

- [x] 4.1 Route `textDocument/didSave` diagnostics through dependency-aware scheduling when diagnostics on save are enabled.
- [x] 4.2 Keep `textDocument/didOpen` behavior focused on the opened file unless full-workspace diagnostics are explicitly requested.
- [x] 4.3 Keep `textDocument/didChange` diagnostics opt-in and scoped to the changed file when enabled.
- [x] 4.4 Add integration tests showing save of an exported function rename refreshes dependant diagnostics at call sites.

## 5. Analyzer Refresh Integration

- [x] 5.1 Route `purescript.analyzerRefresh` through the scheduler as a progressive full-workspace diagnostic batch.
- [x] 5.2 Preserve refreshable source filtering for workspace `.purs` files and excluded directories.
- [x] 5.3 Prioritize open files during full-workspace refresh.
- [x] 5.4 Add tests that full refresh publishes changed diagnostics progressively and does not publish unchanged diagnostics.

## 6. Verification

- [x] 6.1 Run `cargo check -p purescript-analyzer --tests`.
- [x] 6.2 Run relevant LSP integration tests with `just t lsp` filters for diagnostics and execute-command behavior.
- [x] 6.3 Manually validate in an editor or LSP test harness that saving a module updates diagnostics in dependant open files without invoking full analyzer refresh.
