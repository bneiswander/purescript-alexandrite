## Why

This LSP server currently cannot respond to `workspace/executeCommand`, which prevents editor workflows (e.g. Emacs/Spacemacs leader keys) that rely on `purescript.build` and `purescript.clean` from working.

Adding a small, explicit execute-command surface allows users to trigger a real build/clean on demand, and to reset or refresh analyzer diagnostics without changing the default on-open/on-save diagnostics behavior.

## What Changes

- Implement `workspace/executeCommand` handling and advertise an `executeCommandProvider`.
- Implement `purescript.build` to run a real build via `spago` or `purs`, parse compiler JSON errors, and publish diagnostics.
- Implement `purescript.clean` to delete the workspace `output/` directory.
- Add `purescript.reset` to clear all published diagnostics and reset/reload analyzer state.
- Add `purescript.analyzerRefresh` to publish analyzer diagnostics for all known workspace files (manual trigger).
- Do not change existing automatic diagnostics triggers (open/save/change flags remain as-is).

## Capabilities

### New Capabilities
- `lsp-execute-command`: Provide an advertised, compatible `workspace/executeCommand` surface including `purescript.build` and `purescript.clean`, plus analyzer-specific commands for reset/refresh.

### Modified Capabilities


## Impact

- `compiler-bin/src/lsp.rs`: advertise and route execute-command requests; integrate with existing event model.
- `compiler-bin/src/cli.rs`: add configuration for build tool selection/arguments.
- New build/clean/reset/analyzer-refresh logic in `compiler-bin/src/lsp/` (likely new module(s)).
- Requires invoking external tools (`spago`, `purs`) for `purescript.build` and performing filesystem deletion for `purescript.clean`.
