## 1. Server Wiring

- [ ] 1.1 Advertise `documentHighlightProvider` in `InitializeResult.capabilities`
- [ ] 1.2 Add router handler for `textDocument/documentHighlight`

## 2. Analyzer Implementation

- [ ] 2.1 Add `compiler-lsp/analyzer/src/document_highlight.rs` implementing document highlight logic
- [ ] 2.2 Filter reference locations to the requested document and convert to `DocumentHighlight` with `kind: None`

## 3. Tests

- [ ] 3.1 Add an LSP integration test fixture that asserts highlights for a repeated identifier in one file
- [ ] 3.2 Add an LSP integration test ensuring cross-file references are not returned as highlights
