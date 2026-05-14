## Why

Manual `purescript.analyzerRefresh` currently approximates project-wide diagnostics by checking every refreshable workspace source file, which can create long latency and client-side diagnostic notification floods in large projects. The LSP should instead provide efficient, dependency-aware diagnostics so saving a changed module quickly reports errors in affected dependants without requiring a full refresh.

## What Changes

- Add dependency-aware diagnostic scheduling for PureScript workspace files.
- On save, refresh diagnostics for the saved file and affected dependant files rather than only the saved file or the entire workspace.
- Prioritize open files and direct dependants before closed/transitive dependants.
- Keep full workspace refresh available for explicit session startup or manual validation, but run it progressively and avoid unnecessary diagnostic publications.
- Publish diagnostics only when the merged diagnostic set for a file changes.
- Preserve existing configurable diagnostic triggers for open/save/change.

## Capabilities

### New Capabilities
- `lsp-dependency-aware-diagnostics`: Covers dependency graph tracking, dependant diagnostic refresh, prioritization, cancellation, and progressive full-workspace diagnostics.

### Modified Capabilities
- `lsp-diagnostics-publish-triggers`: Saving a document should refresh diagnostics for affected dependant files, not only the saved document.
- `lsp-execute-command`: `purescript.analyzerRefresh` should remain a full workspace diagnostic command but avoid blocking/flooding clients by publishing progressively and suppressing unchanged diagnostics.

## Impact

- Affects `compiler-bin/src/lsp.rs` and `compiler-bin/src/lsp/event.rs` diagnostic publication paths.
- May require LSP-side workspace graph state derived from indexed imports and module-file mappings.
- May require query-engine or helper APIs for efficient module import/dependant discovery.
- Does not change PureScript language semantics or external compiler behavior.
