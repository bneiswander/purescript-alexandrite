## MODIFIED Requirements

### Requirement: Diagnostics publication triggers SHALL be configurable via CLI flags
The language server SHALL expose CLI flags to enable diagnostic publication on `didChange` (opt-in) and to configure whether diagnostics are published on `didOpen` and `didSave`.

The language server SHALL advertise `textDocument/didSave` support via `ServerCapabilities.textDocumentSync` so that clients which honor capability negotiation will send `textDocument/didSave` notifications.

When diagnostics on save are enabled, saving a workspace PureScript document SHALL trigger dependency-aware analyzer diagnostics for the saved file and affected dependant workspace files.

#### Scenario: Diagnostics on change is opt-in
- **WHEN** the server is started without an explicit `--diagnostics-on-change` flag
- **THEN** the server SHALL NOT publish diagnostics in response to `textDocument/didChange`

#### Scenario: Client opts into diagnostics on change
- **WHEN** the server is started with `--diagnostics-on-change`
- **THEN** the server SHALL publish diagnostics in response to `textDocument/didChange`

#### Scenario: Diagnostics on open can be disabled
- **WHEN** the server is started with `--diagnostics-on-open=false`
- **THEN** the server SHALL NOT publish diagnostics in response to `textDocument/didOpen`

#### Scenario: Clients send didSave when supported
- **GIVEN** the server advertises `textDocumentSync.save`
- **WHEN** the client saves an open document
- **THEN** the client SHOULD send `textDocument/didSave` and the server can publish diagnostics (when `--diagnostics-on-save` is enabled)

#### Scenario: Save triggers dependency-aware diagnostics
- **GIVEN** the server is started with diagnostics on save enabled
- **AND** workspace file `B.purs` depends on workspace file `A.purs`
- **WHEN** the client saves `A.purs`
- **THEN** the server SHALL refresh analyzer diagnostics for `A.purs`
- **AND** the server SHALL refresh analyzer diagnostics for `B.purs`

#### Scenario: Save diagnostics can be disabled
- **GIVEN** the server is started with `--diagnostics-on-save=false`
- **WHEN** the client saves an open document
- **THEN** the server SHALL NOT publish diagnostics in response to `textDocument/didSave`
