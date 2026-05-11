## ADDED Requirements

### Requirement: LSP server SHALL not crash on watched files notifications
The language server SHALL accept and ignore `workspace/didChangeWatchedFiles` notifications.

#### Scenario: Client sends watched files notification
- **WHEN** the client sends `workspace/didChangeWatchedFiles`
- **THEN** the server SHALL continue running and SHALL not terminate the process

### Requirement: LSP server SHALL publish diagnostics on open by default
The language server SHALL collect and publish diagnostics for a document upon `textDocument/didOpen` by default.

#### Scenario: opencode requests diagnostics without save
- **WHEN** the client opens a document using `textDocument/didOpen`
- **THEN** the server SHALL publish diagnostics via `textDocument/publishDiagnostics` for that document

### Requirement: Diagnostics publication triggers SHALL be configurable via CLI flags
The language server SHALL expose CLI flags to enable diagnostic publication on `didChange` (opt-in) and to configure whether diagnostics are published on `didOpen` and `didSave`.

#### Scenario: Diagnostics on change is opt-in
- **WHEN** the server is started without an explicit `--diagnostics-on-change` flag
- **THEN** the server SHALL NOT publish diagnostics in response to `textDocument/didChange`

#### Scenario: Client opts into diagnostics on change
- **WHEN** the server is started with `--diagnostics-on-change`
- **THEN** the server SHALL publish diagnostics in response to `textDocument/didChange`

#### Scenario: Diagnostics on open can be disabled
- **WHEN** the server is started with `--diagnostics-on-open=false`
- **THEN** the server SHALL NOT publish diagnostics in response to `textDocument/didOpen`
