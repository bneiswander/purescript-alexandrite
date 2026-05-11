## Context

The LSP server already supports symbol location and navigation features (`textDocument/references`, `definition`, `hover`, `completion`) via `compiler-lsp/analyzer` and wires handlers in `compiler-bin/src/lsp.rs`.

`textDocument/documentHighlight` can be implemented by reusing the existing reference-finding logic and converting the resulting locations into `DocumentHighlight` values for the current document.

## Goals / Non-Goals

**Goals:**

- Implement `textDocument/documentHighlight` end-to-end.
- Return highlights only for the active document URI.
- Keep behavior conservative: if we cannot confidently locate a symbol, return `null`.

**Non-Goals:**

- Cross-file highlighting (that is `references`).
- Distinguishing read vs write highlights (initial implementation can return `kind: None`).
- Full semantic classification of operators/puns/locals beyond what `references` already supports.

## Decisions

- Reuse `analyzer::references::implementation(...)` as the source of candidate ranges.
  - Rationale: it already maps the cursor to a symbol (`locate`) and resolves all references.
- Filter to `Location`s whose `uri` matches the current document.
  - Rationale: `documentHighlight` is a single-document feature; cross-file results are noise.
- Convert `Location.range` to `DocumentHighlight.range` and set `kind: None`.
  - Rationale: avoids incorrect read/write signaling; can be improved later.

## Risks / Trade-offs

- Some cursor targets intentionally return no references today (locals/let binders/puns). This will surface as no highlights.
- Using `references` means we may include the definition site if the reference implementation returns it; this is acceptable and consistent with many servers.
