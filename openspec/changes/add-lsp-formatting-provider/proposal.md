## Why

The language server currently lacks `textDocument/formatting`, so editors and automation (including LLM-driven tooling) cannot request a canonical, server-approved reformat of a document. Providing formatting improves consistency and enables safe, deterministic code transformations.

## What Changes

- Add `textDocument/formatting` support for PureScript documents.
- Advertise `documentFormattingProvider` only when a formatter command is configured.
- Format by running an external formatter command and returning a single full-document `TextEdit`.
- Expose a new CLI configuration for the formatter command.

## Capabilities

### New Capabilities
- `lsp-document-formatting`: Provide LSP document formatting via `textDocument/formatting` with command-configured formatting.

### Modified Capabilities

<!-- none -->

## Impact

- `compiler-bin` LSP server capabilities and request routing.
- Adds a new CLI flag/config option for specifying the formatter command.
- Introduces a subprocess execution path (stdin/stdout) for formatting requests.
