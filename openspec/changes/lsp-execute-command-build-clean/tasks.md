## 1. ExecuteCommand Plumbing

- [x] 1.1 Advertise `executeCommandProvider` with implemented command ids in `initialize`
- [x] 1.2 Add router handler for `workspace/executeCommand` that dispatches to internal events
- [x] 1.3 Implement “unknown command” error response for unrecognized command ids

## 2. Analyzer Commands

- [x] 2.1 Implement `purescript.analyzerRefresh` event to publish analyzer diagnostics for refreshable workspace source files
- [x] 2.2 Implement fast `purescript.reset` to clear diagnostics, cancel stale work, and invalidate caches without reloading workspace sources
- [x] 2.3 Add cancellation semantics for refresh/reset (cancel in-flight queries before starting)

## 3. Clean Command

- [x] 3.1 Implement `purescript.clean` event to delete `<workspaceRoot>/output` safely (validate target path)
- [x] 3.2 Report clean success/failure to the client (log message or request error)
- [x] 3.3 Clear stale build diagnostics after clean and republish affected files

## 4. Build Command (Spago/Purs)

- [x] 4.1 Extend CLI config with build tool selection (spago vs purs) and optional extra args
- [x] 4.2 Implement spago build invocation with JSON errors enabled
- [x] 4.3 Implement purs compile invocation with JSON errors enabled and workspace source file list
- [x] 4.4 Implement JSON error parsing into per-file `lsp_types::Diagnostic` values
- [x] 4.5 Implement separate build/analyzer diagnostic storage with merged publication and duplicate suppression
- [x] 4.6 Report build success/failure to the client (log/show message)
- [x] 4.7 Avoid diagnostic notification floods by publishing build results only for affected files

## 5. Tests

- [x] 5.1 Add integration/unit tests for `workspace/executeCommand` dispatch and unknown command error
- [x] 5.2 Add tests for `purescript.reset` clearing diagnostics while keeping loaded files
- [x] 5.3 Add tests for `purescript.analyzerRefresh` publishing across multiple files and scoping refreshable source files
- [x] 5.4 Add tests for `purescript.clean` deleting `output/` in a fixture workspace
- [x] 5.5 Add unit tests for compiler JSON error parsing (fixtures)
- [x] 5.6 Add tests asserting build/analyzer diagnostics merge behavior and stale diagnostic suppression
