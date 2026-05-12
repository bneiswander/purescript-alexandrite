## 1. Server Wiring

- [x] 1.1 Advertise `documentHighlightProvider` in `InitializeResult.capabilities`
- [x] 1.2 Add router handler for `textDocument/documentHighlight`

## 2. Analyzer Implementation

- [x] 2.1 Add `compiler-lsp/analyzer/src/document_highlight.rs` implementing document highlight logic
- [x] 2.2 Filter reference locations to the requested document and convert to `DocumentHighlight` with `kind: None`

## 3. Tests

- [x] 3.1 Add an LSP integration test fixture that asserts highlights for a repeated identifier in one file
- [x] 3.2 Add an LSP integration test ensuring cross-file references are not returned as highlights
