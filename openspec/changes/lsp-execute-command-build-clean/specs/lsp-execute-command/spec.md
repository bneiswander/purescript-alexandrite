## ADDED Requirements

### Requirement: ExecuteCommand Provider Is Advertised
The language server SHALL advertise an `executeCommandProvider` capability containing exactly the commands it implements.

#### Scenario: Capability advertisement
- **WHEN** the client initializes the server
- **THEN** the server response includes `capabilities.executeCommandProvider.commands` containing `purescript.build`, `purescript.clean`, `purescript.reset`, and `purescript.analyzerRefresh`

### Requirement: Unknown Commands Are Rejected
The language server SHALL reject `workspace/executeCommand` requests for commands it does not implement.

#### Scenario: Unknown execute-command
- **WHEN** the client sends `workspace/executeCommand` with an unrecognized command id
- **THEN** the server responds with an LSP request error indicating the command is not supported

### Requirement: Clean Deletes Output Directory
The language server SHALL implement `purescript.clean` by deleting the workspace `output/` directory.

#### Scenario: Clean removes compiled output
- **WHEN** the client executes `purescript.clean`
- **THEN** the server deletes `<workspaceRoot>/output` recursively

### Requirement: Reset Clears Diagnostics And Reloads Analyzer State
The language server SHALL implement `purescript.reset` by clearing all published diagnostics for known workspace files and resetting/reloading analyzer state.

#### Scenario: Reset clears diagnostics
- **WHEN** the client executes `purescript.reset`
- **THEN** the server publishes empty diagnostics for every known workspace file

#### Scenario: Reset reloads analyzer
- **WHEN** the client executes `purescript.reset`
- **THEN** the server resets internal analyzer state and reloads workspace source files using the same discovery mechanism as initialization

### Requirement: Analyzer Refresh Publishes Workspace Diagnostics
The language server SHALL implement `purescript.analyzerRefresh` by publishing analyzer diagnostics for all known workspace files.

#### Scenario: Manual analyzer refresh
- **WHEN** the client executes `purescript.analyzerRefresh`
- **THEN** the server computes and publishes analyzer diagnostics for all known workspace files

### Requirement: Build Runs External Tool And Publishes Build Diagnostics
The language server SHALL implement `purescript.build` by invoking an external build tool (`spago` or `purs`) configured for the workspace and publishing diagnostics derived from compiler JSON errors.

#### Scenario: Build publishes diagnostics from compiler output
- **WHEN** the client executes `purescript.build`
- **THEN** the server runs the configured build tool with JSON error output enabled and publishes diagnostics derived from those JSON errors

### Requirement: Build Diagnostics Replace Analyzer Diagnostics
The language server SHALL ensure that build diagnostics replace any previously published diagnostics for known workspace files.

#### Scenario: Replace diagnostics on build
- **WHEN** the client executes `purescript.build`
- **THEN** the server first clears diagnostics for all known workspace files
- **THEN** the server publishes build diagnostics for files reported by the compiler

### Requirement: Automatic Diagnostics Triggers Are Unchanged
The language server SHALL NOT change its existing automatic diagnostics triggers as a result of implementing execute-commands.

#### Scenario: Diagnostics trigger defaults remain
- **WHEN** the server is started with default configuration
- **THEN** diagnostics are published on open and save, and are not published on change unless explicitly enabled
