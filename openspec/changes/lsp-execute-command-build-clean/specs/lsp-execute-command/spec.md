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

### Requirement: Clean Deletes Output Directory And Clears Build Diagnostics
The language server SHALL implement `purescript.clean` by deleting the workspace `output/` directory and clearing stale build diagnostics.

#### Scenario: Clean removes compiled output
- **WHEN** the client executes `purescript.clean`
- **THEN** the server deletes `<workspaceRoot>/output` recursively

#### Scenario: Clean clears stale build diagnostics
- **WHEN** the client executes `purescript.clean`
- **THEN** the server clears stored build diagnostics
- **AND** republishes diagnostics for files that previously had build diagnostics

### Requirement: Reset Clears Diagnostics And Invalidates Analyzer Caches
The language server SHALL implement `purescript.reset` as a fast reset that clears published diagnostics for known/open files, cancels in-flight analyzer work, suppresses stale diagnostic publications, and invalidates analyzer caches without reloading workspace sources.

#### Scenario: Reset clears diagnostics
- **WHEN** the client executes `purescript.reset`
- **THEN** the server publishes empty diagnostics for file URIs that previously had diagnostics and currently open file URIs

#### Scenario: Reset invalidates analyzer state without reloading files
- **WHEN** the client executes `purescript.reset`
- **THEN** the server clears stored build and analyzer diagnostics
- **AND** invalidates workspace-symbol and suggestion caches
- **AND** keeps currently loaded file contents instead of rediscovering workspace source files

### Requirement: Analyzer Refresh Publishes Source File Diagnostics
The language server SHALL implement `purescript.analyzerRefresh` by publishing analyzer diagnostics for refreshable workspace PureScript source files.

#### Scenario: Manual analyzer refresh
- **WHEN** the client executes `purescript.analyzerRefresh`
- **THEN** the server computes and publishes analyzer diagnostics for `file://` `.purs` files under the workspace root
- **AND** excludes dependency, generated, external, and non-file URIs such as `.spago`, `output`, `.git`, `node_modules`, and `prim://` files

### Requirement: Build Runs External Tool And Publishes Build Diagnostics
The language server SHALL implement `purescript.build` by invoking an external build tool (`spago` or `purs`) configured for the workspace and publishing diagnostics derived from compiler JSON errors.

#### Scenario: Build publishes diagnostics from compiler output
- **WHEN** the client executes `purescript.build`
- **THEN** the server runs the configured build tool with JSON error output enabled and publishes diagnostics derived from those JSON errors

### Requirement: Build Diagnostics Merge With Analyzer Diagnostics
The language server SHALL track build diagnostics separately from analyzer diagnostics and publish a merged diagnostic set that prefers build diagnostics for duplicate/stale diagnostics at the same range.

#### Scenario: Merge diagnostics on build
- **WHEN** the client executes `purescript.build`
- **THEN** the server clears previously stored build diagnostics
- **AND** publishes merged diagnostics for files that previously had build diagnostics and files reported by the compiler
- **AND** suppresses analyzer diagnostics whose range is the same as a build diagnostic range
- **AND** keeps analyzer diagnostics for distinct ranges

#### Scenario: Build avoids diagnostic notification flood
- **WHEN** the client executes `purescript.build`
- **THEN** the server does not publish empty diagnostics for every known workspace file

### Requirement: Automatic Diagnostics Triggers Are Unchanged
The language server SHALL NOT change its existing automatic diagnostics triggers as a result of implementing execute-commands.

#### Scenario: Diagnostics trigger defaults remain
- **WHEN** the server is started with default configuration
- **THEN** diagnostics are published on open and save, and are not published on change unless explicitly enabled
