## 1. Crash Fixes

- [ ] 1.1 Add a no-op handler for `workspace/didChangeWatchedFiles` in the LSP router
- [ ] 1.2 Replace `run_buffered(...).unwrap()` with error reporting and graceful shutdown

## 2. Diagnostics On Open/Change/Save

- [ ] 2.1 Add CLI flags to configure diagnostics publication triggers (`on_open`, `on_change`, `on_save`) with `on_change` defaulting to false and `on_open` defaulting to true
- [ ] 2.2 Implement `textDocument/didOpen` handler to ingest document text via `on_change` and (when enabled) publish diagnostics
- [ ] 2.3 Update `textDocument/didChange` handler to (when enabled) publish diagnostics after applying content changes
- [ ] 2.4 Guard `textDocument/didSave` diagnostics publication behind the `on_save` flag

## 3. Verification

- [ ] 3.1 Run `cargo check -p purescript-analyzer --tests`
- [ ] 3.2 Verify `opencode debug lsp diagnostics <file>` returns non-empty diagnostics for a known-bad `.purs` file when `--diagnostics-on-change` is enabled
