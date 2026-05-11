## 1. LSP Wiring

- [ ] 1.1 Advertise `documentSymbolProvider` in `ServerCapabilities`
- [ ] 1.2 Add `textDocument/documentSymbol` request routing in `compiler-bin/src/lsp.rs`

## 2. Analyzer Implementation

- [ ] 2.1 Add analyzer entry point `analyzer::document_symbols::implementation(engine, files, uri)`
- [ ] 2.2 Collect file-local term/type/class items and map to `SymbolInformation`
- [ ] 2.3 Compute stable `Location` ranges using existing `common::{file_term_location,file_type_location}` helpers
- [ ] 2.4 Ensure deterministic ordering of returned symbols

## 3. Integration Tests

- [ ] 3.1 Extend the LSP fixture harness to query document symbols (add a new cursor marker)
- [ ] 3.2 Add fixture module and snapshot asserting returned symbols and ranges
- [ ] 3.3 Run `cargo check` for affected crates and run the new integration test
