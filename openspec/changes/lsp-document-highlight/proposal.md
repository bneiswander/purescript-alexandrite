## Why

Editors currently lack a fast, built-in way to highlight all occurrences of the symbol under the cursor within the current document. This makes local navigation and quick refactoring harder than it needs to be.

## What Changes

- Add LSP support for `textDocument/documentHighlight`.
- Highlight ranges are derived from the existing reference-finding logic and restricted to the active document.
- Advertise `documentHighlightProvider` in server capabilities.

## Capabilities

### New Capabilities
- `lsp-document-highlight`: Provide `textDocument/documentHighlight` results for symbols at a cursor position.

### Modified Capabilities

## Impact

- `compiler-bin/src/lsp.rs`: advertise capability; route new request.
- `compiler-lsp/analyzer`: implement document highlight mapping/filtering.
- `tests-integration`: add/extend LSP integration coverage for document highlights.
