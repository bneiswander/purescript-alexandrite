## ADDED Requirements

### Requirement: Provide document symbols
The language server MUST handle the `textDocument/documentSymbol` request for PureScript source documents.

#### Scenario: Document contains top-level declarations
- **WHEN** a client requests `textDocument/documentSymbol` for a PureScript document containing values, types, data constructors, classes, and class members
- **THEN** the server MUST return a `DocumentSymbolResponse` containing symbols for those declarations

#### Scenario: Symbol locations are navigable
- **WHEN** the server returns a symbol with a location range
- **THEN** the range MUST refer to the same document and MUST cover the corresponding declaration syntax

#### Scenario: Deterministic ordering
- **WHEN** the underlying document content is unchanged
- **THEN** the server MUST return symbols in a deterministic order (no unstable reordering between requests)
