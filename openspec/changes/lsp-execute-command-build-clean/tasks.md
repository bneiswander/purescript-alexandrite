## 1. ExecuteCommand Plumbing

- [x] 1.1 Advertise `executeCommandProvider` with implemented command ids in `initialize`
- [x] 1.2 Add router handler for `workspace/executeCommand` that dispatches to internal events
- [x] 1.3 Implement “unknown command” error response for unrecognized command ids

## 2. Analyzer Commands

- [x] 2.1 Implement `purescript.analyzerRefresh` event to publish analyzer diagnostics for all known workspace files
- [x] 2.2 Implement `purescript.reset` event to clear all diagnostics, reset analyzer state, and reload workspace sources
- [x] 2.3 Add cancellation semantics for refresh/reset (cancel in-flight queries before starting)

## 3. Clean Command

- [x] 3.1 Implement `purescript.clean` event to delete `<workspaceRoot>/output` safely (validate target path)
- [x] 3.2 Report clean success/failure to the client (log message or request error)

## 4. Build Command (Spago/Purs)

- [x] 4.1 Extend CLI config with build tool selection (spago vs purs) and optional extra args
- [x] 4.2 Implement spago build invocation with JSON errors enabled
- [x] 4.3 Implement purs compile invocation with JSON errors enabled and workspace source file list
- [x] 4.4 Implement JSON error parsing into per-file `lsp_types::Diagnostic` values
- [x] 4.5 Implement “replace analyzer diagnostics” behavior: clear diagnostics for known files before publishing build results
- [x] 4.6 Report build success/failure to the client (log/show message)

## 5. Tests

- [x] 5.1 Add integration/unit tests for `workspace/executeCommand` dispatch and unknown command error
- [x] 5.2 Add tests for `purescript.reset` clearing diagnostics and reloading sources
- [x] 5.3 Add tests for `purescript.analyzerRefresh` publishing across multiple files
- [x] 5.4 Add tests for `purescript.clean` deleting `output/` in a fixture workspace
- [x] 5.5 Add unit tests for compiler JSON error parsing (fixtures)
- [x] 5.6 Add tests asserting build clears prior diagnostics before publishing build diagnostics
