## 1. Workspace Graph

- [ ] 1.1 Add LSP-side workspace graph state for file-to-imports and file-to-dependants relationships.
- [ ] 1.2 Derive graph edges from indexed imports and module-file mappings for refreshable workspace source files.
- [ ] 1.3 Update graph entries when files are loaded or changed, including module name/import changes.
- [ ] 1.4 Add unit tests for graph construction, reverse dependant lookup, transitive closure, and excluded URI handling.

## 2. Diagnostic Scheduler

- [ ] 2.1 Introduce a diagnostic scheduler/batch abstraction with generation tokens for stale-result suppression.
- [ ] 2.2 Add bounded background execution for diagnostic jobs instead of spawning one unbounded task per file.
- [ ] 2.3 Implement priority ordering for saved files, open direct dependants, open transitive dependants, closed direct dependants, and closed transitive dependants.
- [ ] 2.4 Preserve cancellation/reset behavior so older diagnostic batches cannot republish stale results.

## 3. Publication Deduplication

- [ ] 3.1 Track last published merged diagnostics per URI.
- [ ] 3.2 Skip `publishDiagnostics` when the merged diagnostics for a URI are unchanged.
- [ ] 3.3 Ensure clearing previously published diagnostics still publishes an empty diagnostic set.
- [ ] 3.4 Add tests for unchanged diagnostics, changed diagnostics, and cleared diagnostics publication behavior.

## 4. Trigger Integration

- [ ] 4.1 Route `textDocument/didSave` diagnostics through dependency-aware scheduling when diagnostics on save are enabled.
- [ ] 4.2 Keep `textDocument/didOpen` behavior focused on the opened file unless full-workspace diagnostics are explicitly requested.
- [ ] 4.3 Keep `textDocument/didChange` diagnostics opt-in and scoped to the changed file when enabled.
- [ ] 4.4 Add integration tests showing save of an exported function rename refreshes dependant diagnostics at call sites.

## 5. Analyzer Refresh Integration

- [ ] 5.1 Route `purescript.analyzerRefresh` through the scheduler as a progressive full-workspace diagnostic batch.
- [ ] 5.2 Preserve refreshable source filtering for workspace `.purs` files and excluded directories.
- [ ] 5.3 Prioritize open files during full-workspace refresh.
- [ ] 5.4 Add tests that full refresh publishes changed diagnostics progressively and does not publish unchanged diagnostics.

## 6. Verification

- [ ] 6.1 Run `cargo check -p purescript-analyzer --tests`.
- [ ] 6.2 Run relevant LSP integration tests with `just t lsp` filters for diagnostics and execute-command behavior.
- [ ] 6.3 Manually validate in an editor or LSP test harness that saving a module updates diagnostics in dependant open files without invoking full analyzer refresh.
