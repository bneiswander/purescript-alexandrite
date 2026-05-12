## 1. Configuration

- [ ] 1.1 Add `--format-command` CLI option to configure external formatter command
- [ ] 1.2 Ensure formatting capability is advertised only when `--format-command` is set

## 2. LSP Formatting Implementation

- [ ] 2.1 Implement `textDocument/formatting` request handler that runs the configured formatter via stdin/stdout
- [ ] 2.2 Return a single full-document `TextEdit` on successful formatting
- [ ] 2.3 Return an empty edit list when formatter output matches input
- [ ] 2.4 Surface formatter failures (spawn/exit/stdout utf-8) as LSP request failures with useful messages

## 3. Tests

- [ ] 3.1 Add an integration test that asserts `documentFormattingProvider` is not advertised without `--format-command`
- [ ] 3.2 Add an integration test that asserts `documentFormattingProvider` is advertised when `--format-command` is set
- [ ] 3.3 Add an integration test for `textDocument/formatting` that formats a fixture document using a stable formatter command
