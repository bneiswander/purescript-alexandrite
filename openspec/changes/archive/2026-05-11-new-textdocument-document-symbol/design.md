## Context

The repository already has an LSP server (`compiler-bin/src/lsp.rs`) and an analyzer crate (`compiler-lsp/analyzer`) that provides analysis for several LSP requests (definition/hover/completion/references/workspaceSymbol).

We want to add `textDocument/documentSymbol` using the same split:
- `compiler-bin`: LSP protocol wiring and capability advertisement.
- `compiler-lsp/analyzer`: PureScript-specific symbol extraction given a `QueryEngine`, `Files`, and a document URI.

Constraints:
- Keep the initial implementation minimal and correct.
- Prefer stable, deterministic output for snapshots and client UX.

## Goals / Non-Goals

**Goals:**
- Handle LSP `textDocument/documentSymbol` requests for `.purs` files.
- Return a useful set of top-level symbols (values, types, constructors, classes, class members).
- Provide correct `Range`s to allow editor navigation.
- Add an integration test fixture + snapshot.

**Non-Goals:**
- Produce a nested outline tree (`DocumentSymbolResponse::Nested`).
- Implement symbol tags (deprecated) or container names.
- Provide full semantic classification for every PureScript construct.

## Decisions

- Return `DocumentSymbolResponse::Flat(Vec<SymbolInformation>)`.
  - Rationale: the analyzer already has helpers to compute `Location` for indexed items (`common::{file_term_location,file_type_location}`), and the LSP harness already works well with flat results.
  - Alternative: `Nested(Vec<DocumentSymbol>)` offers richer structure, but requires defining parent/child relationships (e.g. constructors under a data type), which is more logic and more edge cases.

- Derive symbols from `engine.resolved(file_id)?.locals` and the file-local `engine.indexed(file_id)?`.
  - Rationale: `resolved.locals` provides stable name-to-item IDs, and `indexed` provides `TermItemKind`/`TypeItemKind` for basic categorization.
  - Alternative: walk the CST directly; this is more flexible but duplicates existing indexing work.

- Stable sort by start position.
  - Rationale: deterministic output helps snapshots and reduces flicker in clients.

## Risks / Trade-offs

- [Ranges include more than the identifier] → Mitigation: accept initially; refine later by extracting identifier token ranges if needed.
- [Flat list lacks hierarchy] → Mitigation: plan a follow-up to build nested symbols for type declarations and class members.
- [Incomplete symbol coverage] (e.g. instances/derives/operators may be imperfect) → Mitigation: expand coverage incrementally and add fixtures.
