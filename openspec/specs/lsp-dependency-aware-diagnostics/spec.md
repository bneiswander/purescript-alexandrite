# lsp-dependency-aware-diagnostics Specification

## Purpose
TBD - created by archiving change dependency-aware-lsp-diagnostics. Update Purpose after archive.
## Requirements
### Requirement: Dependency-Aware Diagnostic Refresh
The language server SHALL refresh analyzer diagnostics for a saved workspace PureScript file and refresh analyzer diagnostics for workspace PureScript files that depend on the saved file.

#### Scenario: Saved module refreshes dependant diagnostics
- **GIVEN** workspace file `B.purs` imports workspace file `A.purs`
- **WHEN** the client saves `A.purs`
- **THEN** the server computes analyzer diagnostics for `A.purs`
- **AND** computes analyzer diagnostics for `B.purs`

#### Scenario: Transitive dependants are refreshed
- **GIVEN** workspace file `C.purs` imports `B.purs` and `B.purs` imports `A.purs`
- **WHEN** the client saves `A.purs`
- **THEN** the server computes analyzer diagnostics for `B.purs`
- **AND** computes analyzer diagnostics for `C.purs`

#### Scenario: Unrelated modules are not refreshed on save
- **GIVEN** workspace file `C.purs` does not depend on saved workspace file `A.purs`
- **WHEN** the client saves `A.purs`
- **THEN** the server SHALL NOT compute analyzer diagnostics for `C.purs` as part of the save-triggered dependant refresh

### Requirement: Diagnostic Refresh Prioritization
The language server SHALL prioritize diagnostics for the saved file and open dependant files before diagnostics for closed dependant files.

#### Scenario: Open dependant files are prioritized
- **GIVEN** workspace files `B.purs` and `C.purs` both depend on saved workspace file `A.purs`
- **AND** `B.purs` is open in the client and `C.purs` is not open
- **WHEN** the client saves `A.purs`
- **THEN** the server computes analyzer diagnostics for `A.purs` before dependant files
- **AND** computes analyzer diagnostics for open dependant `B.purs` before closed dependant `C.purs`

### Requirement: Diagnostic Publication Deduplication
The language server SHALL avoid publishing diagnostics for a file when the merged diagnostic set for that file has not changed.

#### Scenario: Unchanged diagnostics are not republished
- **GIVEN** the server has already published diagnostics for workspace file `A.purs`
- **WHEN** a diagnostic refresh computes the same merged diagnostics for `A.purs`
- **THEN** the server SHALL NOT publish a new `textDocument/publishDiagnostics` notification for `A.purs`

#### Scenario: Cleared diagnostics are published
- **GIVEN** the server has already published non-empty diagnostics for workspace file `A.purs`
- **WHEN** a diagnostic refresh computes no merged diagnostics for `A.purs`
- **THEN** the server publishes an empty `textDocument/publishDiagnostics` notification for `A.purs`

### Requirement: Diagnostic Work Cancellation
The language server SHALL suppress stale diagnostic publications from older diagnostic batches after newer file changes or reset operations supersede them.

#### Scenario: Stale dependant diagnostics are suppressed
- **GIVEN** a dependant diagnostic refresh is running for a saved file
- **WHEN** the client changes or saves another workspace file before the refresh completes
- **THEN** stale diagnostics from the older refresh SHALL NOT overwrite diagnostics from the newer workspace state

### Requirement: Progressive Workspace Diagnostics
The language server SHALL support full-workspace analyzer diagnostics as progressive background work over refreshable workspace PureScript source files.

#### Scenario: Full workspace diagnostics use refreshable source scope
- **WHEN** a full-workspace analyzer diagnostic refresh is requested
- **THEN** the server computes analyzer diagnostics for `file://` `.purs` files under the workspace root
- **AND** excludes dependency, generated, external, and non-file URIs such as `.spago`, `output`, `.git`, `node_modules`, and `prim://` files

#### Scenario: Full workspace diagnostics avoid notification flood
- **WHEN** a full-workspace analyzer diagnostic refresh is requested
- **THEN** the server publishes diagnostics progressively
- **AND** avoids publishing diagnostics for files whose merged diagnostics have not changed

