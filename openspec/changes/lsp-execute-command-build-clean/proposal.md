## Why

This LSP server currently cannot respond to `workspace/executeCommand`, which prevents editor workflows (e.g. Emacs/Spacemacs leader keys) that rely on `purescript.build` and `purescript.clean` from working.

Adding a small, explicit execute-command surface allows users to trigger a real build/clean on demand, and to clear or refresh diagnostics without changing the default on-open/on-save diagnostics behavior.

## What Changes

- Implement `workspace/executeCommand` handling and advertise an `executeCommandProvider`.
- Implement `purescript.build` to run a real build via `spago` or `purs`, parse compiler JSON errors, and publish build diagnostics without flooding every workspace file.
- Implement `purescript.clean` to delete the workspace `output/` directory and clear stale build diagnostics.
- Add `purescript.reset` as a fast reset that cancels in-flight work, clears published diagnostics for known/open files, and invalidates analyzer caches while keeping loaded file contents.
- Add `purescript.analyzerRefresh` to publish analyzer diagnostics for refreshable workspace source files (manual trigger), excluding dependencies and generated output.
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
