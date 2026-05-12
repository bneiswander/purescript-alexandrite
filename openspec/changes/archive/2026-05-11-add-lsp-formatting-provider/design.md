## Context

The PureScript language server in this repository (`compiler-bin/src/lsp.rs`) already implements core navigation and diagnostics features but does not currently support `textDocument/formatting`. Editors typically invoke formatting through this request and expect the server to return a `WorkspaceEdit` (or `TextEdit`s) to apply.

PureScript formatting is typically provided by external tools (for example `purs-tidy`). This repository does not currently embed a PureScript formatter library.

Constraints:

- LSP server is written in Rust using `async-lsp`.
- Current text sync is full-document (`TextDocumentSyncKind::FULL`), so server has full content available.
- We want deterministic output and simple failure modes.

## Goals / Non-Goals

**Goals:**

- Provide `textDocument/formatting` for PureScript documents.
- Make formatting opt-in and configurable via CLI (`--format-command`).
- Keep the implementation minimal: one request handler, one subprocess invocation, one full-document `TextEdit`.
- Return no edits when the formatter output matches the current content.

**Non-Goals:**

- Implement a formatter in Rust.
- Implement range formatting (`textDocument/rangeFormatting`) or on-type formatting.
- Provide formatting for non-PureScript files.
- Complex incremental diff edits; full-document replacement is acceptable.

## Decisions

- External formatter via command string.

Rationale: avoids shipping/maintaining a formatter library; aligns with common PureScript tooling (e.g. `purs-tidy`).

- Opt-in capability advertisement.

Rationale: LSP clients often auto-run formatting; advertising support only when configured avoids surprising behavior and avoids errors when no formatter is present.

- Full-document edit.

Rationale: simplest correct `TextEdit` representation; avoids fragile range mapping and diff complexity.

- Communication contract with formatter.

We treat the formatter as a filter: write document text to stdin, read formatted document text from stdout, and treat non-zero exit status as failure.

## Risks / Trade-offs

- Subprocess cost (spawn per request) → Mitigation: keep it opt-in; formatting is user-invoked and low frequency.
- Formatter availability and PATH issues → Mitigation: fail with a clear error message; only advertise capability when configured.
- Command parsing for `--format-command` (whitespace splitting) is limited (no shell quoting) → Mitigation: document expectation; if this becomes an issue, switch to a structured config (program + args array).
