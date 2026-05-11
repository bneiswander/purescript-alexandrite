## 1. LSP Wiring

- [x] 1.1 Advertise `documentSymbolProvider` in `ServerCapabilities`
- [x] 1.2 Add `textDocument/documentSymbol` request routing in `compiler-bin/src/lsp.rs`

## 2. Analyzer Implementation

- [x] 2.1 Add analyzer entry point `analyzer::document_symbols::implementation(engine, files, uri)`
- [x] 2.2 Collect file-local term/type/class items and map to `SymbolInformation`
- [x] 2.3 Compute stable `Location` ranges using existing `common::{file_term_location,file_type_location}` helpers
- [x] 2.4 Ensure deterministic ordering of returned symbols

## 3. Integration Tests

- [x] 3.1 Extend the LSP fixture harness to query document symbols (add a new cursor marker)
- [x] 3.2 Add fixture module and snapshot asserting returned symbols and ranges
- [x] 3.3 Run `cargo check` for affected crates and run the new integration test
