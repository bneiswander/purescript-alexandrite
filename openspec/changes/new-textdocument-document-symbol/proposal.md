## Why

PureScript editing workflows need a document outline to support quick navigation within a module (jump to top-level values, types, and classes) and to populate editor UI like “Outline” and breadcrumbs.

## What Changes

- Add support for the LSP request `textDocument/documentSymbol`.
- Advertise `documentSymbolProvider` in server capabilities.
- Provide a flat list of symbols (initially) for a PureScript document, including values, types, data constructors, classes, and class members.
- Add an integration test fixture/snapshot covering the new request.

## Capabilities

### New Capabilities
- `lsp-document-symbol`: Provide per-document symbols via `textDocument/documentSymbol` for PureScript source files.

### Modified Capabilities

<!-- None. -->

## Impact

- `compiler-bin` LSP wiring: new request handler and capability advertisement.
- `compiler-lsp/analyzer`: new analysis pass producing `DocumentSymbolResponse`.
- `tests-integration`: new fixture/snapshot and harness support to query document symbols.
