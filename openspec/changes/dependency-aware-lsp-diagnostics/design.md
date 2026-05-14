## Context

The current LSP diagnostic path computes diagnostics for one file on open/save/change, while `purescript.analyzerRefresh` computes diagnostics for every refreshable workspace PureScript source file. This gives users either narrow diagnostics or an expensive full-workspace operation.

The compiler core already has a query engine with dependency tracking, cancellation, work deduplication, and structural equality for cached query results. The LSP layer does not yet maintain a workspace module graph or a scheduler that can choose the affected diagnostic set after a file changes.

## Goals / Non-Goals

**Goals:**

- Refresh diagnostics for files affected by a saved module through the workspace import/dependant graph.
- Report errors in open dependant files quickly after a save.
- Keep full-workspace analyzer refresh available, but make it progressive and client-friendly.
- Avoid publishing diagnostics when a file's merged diagnostic set has not changed.
- Preserve the existing `--diagnostics-on-open`, `--diagnostics-on-save`, and `--diagnostics-on-change` controls.

**Non-Goals:**

- Replacing the query engine or changing PureScript checking semantics.
- Implementing file-system watching for newly created or deleted modules beyond the current loaded-file model.
- Guaranteeing minimal invalidation using exact query reverse dependencies in the first implementation.

## Decisions

1. **Add an LSP-side diagnostic scheduler**

   Decision: Introduce scheduler state that owns queued/running diagnostic jobs, stale-result generations, and last-published analyzer diagnostics.

   Rationale: Diagnostic policy belongs in the LSP layer. The query engine should remain responsible for incremental computation and correctness, while the LSP decides which files to ask about and when to publish results.

   Alternative: Make `collect_diagnostics` recursively publish dependant diagnostics. Rejected because recursive event dispatch makes prioritization, cancellation, deduplication, and bounded concurrency harder to reason about.

2. **Maintain a workspace import/dependant graph from indexed imports**

   Decision: Track `file -> imported files` and `file -> dependant files` for refreshable workspace files. Rebuild or update affected graph entries after file content changes and indexing succeeds.

   Rationale: Import graph closure is conservative enough for rename/export changes and cheaper than checking every file. It also works with the existing module-file mapping and indexing pipeline.

   Alternative: Use the query engine's internal dependency trace directly. Rejected for the initial implementation because those traces are private implementation detail and include query-level dependencies that are more precise but more invasive to expose safely.

3. **Prioritize diagnostics by editor usefulness**

   Decision: On save, enqueue the saved file first, then open direct dependants, open transitive dependants, closed direct dependants, and closed transitive dependants.

   Rationale: The user needs immediate feedback in the file they saved and files visible in the editor. Closed-file diagnostics can complete in the background without blocking interaction.

   Alternative: Process files in arbitrary graph order. Rejected because it can delay visible diagnostics behind irrelevant closed files.

4. **Use bounded background work**

   Decision: Run diagnostic jobs in the background with bounded concurrency and generation tokens. New edits/saves cancel or stale-suppress older diagnostic batches.

   Rationale: The existing query engine supports cancellation and work deduplication, but the LSP must avoid spawning one blocking job per workspace file without limits.

   Alternative: Keep spawning one task per file. Rejected because it can saturate CPU and produce notification bursts that make clients such as Emacs feel locked.

5. **Publish only changed merged diagnostics**

   Decision: Before publishing, compare the new merged diagnostic vector against the last analyzer/build merge for that URI and skip publication if unchanged.

   Rationale: LSP clients still pay for processing notifications even when diagnostics are identical. Suppressing unchanged notifications reduces editor latency.

   Alternative: Always publish after each check. Rejected because it preserves the current notification flood behavior.

6. **Keep full refresh semantics but make them progressive**

   Decision: `purescript.analyzerRefresh` remains a full refresh over refreshable workspace source files, but it should enqueue diagnostics through the scheduler rather than synchronously emitting all per-file jobs.

   Rationale: Full refresh is still useful for startup/session validation. Routing it through the scheduler gives cancellation, prioritization, bounded concurrency, and publication dedupe.

   Alternative: Replace `purescript.analyzerRefresh` with a narrower command. Rejected because the command is already advertised and useful as an explicit full-workspace operation.

## Risks / Trade-offs

- **Conservative dependant closure may check more files than strictly necessary** -> Start with import graph closure for correctness and add public-interface fingerprints later to prune implementation-only edits.
- **Graph entries can become stale when module names or imports change** -> Update module-file mappings and graph edges during file-change processing; fall back to wider refresh if graph update fails.
- **Bounded concurrency can delay closed-file diagnostics** -> Prioritize open files and expose progress/status for full refreshes.
- **Comparing diagnostics can hide desired republish behavior for some clients** -> Compare complete merged diagnostics and still publish when build or analyzer diagnostics change, including clearing previously published diagnostics.
- **Cycles in imports or queries can complicate dependant traversal** -> Use visited sets for graph traversal and rely on existing query errors for semantic cycles.

## Migration Plan

1. Introduce graph/scheduler state behind existing diagnostic triggers.
2. Route save-triggered diagnostics through the scheduler while preserving current flags.
3. Route `purescript.analyzerRefresh` through the scheduler as a full-workspace batch.
4. Add tests for dependant diagnostics, prioritization, unchanged-publication suppression, and refresh scoping.
5. Keep existing command names and CLI flags unchanged so users do not need configuration changes.

## Open Questions

- Should full refresh report LSP work-done progress, status messages, or remain silent?
- Should public-interface fingerprinting be included in the first implementation or follow after import-closure scheduling is working?
