## Why

opencode’s `debug lsp diagnostics` flow currently fails against `purescript-analyzer`, returning `{}` even when the file contains errors. This happens because the server panics on an unhandled notification and, even when running, does not publish diagnostics for the events opencode triggers.

## What Changes

- Handle `workspace/didChangeWatchedFiles` to avoid server panics when clients send file-watcher notifications.
- Publish diagnostics on `textDocument/didOpen` and optionally on `textDocument/didChange`, so opencode can retrieve diagnostics without requiring a save.
- Add CLI flags to configure when diagnostics are published.
  - Default `--diagnostics-on-change` to off (opt-in), while keeping diagnostics on open enabled by default.

## Capabilities

### New Capabilities
- `lsp-diagnostics-publish-triggers`: Configurable diagnostic publication triggers for the analyzer LSP (open/change/save) suitable for non-editor clients like opencode.

### Modified Capabilities
- `spago-extra-packages-subdir-sources`: None.

## Impact

- `compiler-bin/src/lsp.rs`: LSP notification routing, didOpen/didChange behavior, and diagnostics triggering.
- `compiler-bin/src/cli.rs`: New CLI flags for diagnostics publication triggers.
- opencode integration: projects can opt into on-change diagnostics by adding a flag in their per-project opencode config.
