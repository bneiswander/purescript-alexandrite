## Purpose
Define server behavior for `textDocument/documentHighlight`.

## Requirements

### Requirement: Server provides document highlights
When the client requests `textDocument/documentHighlight` for a document position, the server SHALL return highlight ranges for the symbol at that position within the same document.

#### Scenario: Highlights returned for symbol references in current file
- **WHEN** the client sends `textDocument/documentHighlight` at a position that resolves to a symbol with references
- **THEN** the server returns a list of `DocumentHighlight` entries whose ranges are all within the requested document

#### Scenario: No highlights when target cannot be resolved
- **WHEN** the client sends `textDocument/documentHighlight` at a position that does not resolve to a supported symbol
- **THEN** the server returns `null`
