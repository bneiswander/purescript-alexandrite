## MODIFIED Requirements

### Requirement: Analyzer Refresh Publishes Source File Diagnostics
The language server SHALL implement `purescript.analyzerRefresh` by scheduling progressive analyzer diagnostics for refreshable workspace PureScript source files.

#### Scenario: Manual analyzer refresh
- **WHEN** the client executes `purescript.analyzerRefresh`
- **THEN** the server computes analyzer diagnostics for `file://` `.purs` files under the workspace root
- **AND** excludes dependency, generated, external, and non-file URIs such as `.spago`, `output`, `.git`, `node_modules`, and `prim://` files

#### Scenario: Manual analyzer refresh publishes progressively
- **WHEN** the client executes `purescript.analyzerRefresh`
- **THEN** the server SHALL NOT require all refreshable workspace files to finish diagnostic computation before publishing the first changed diagnostic result
- **AND** the server SHALL avoid publishing diagnostics for files whose merged diagnostics have not changed
