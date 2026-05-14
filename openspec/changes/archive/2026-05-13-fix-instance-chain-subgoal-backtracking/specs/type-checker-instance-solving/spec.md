## ADDED Requirements

### Requirement: Nested-unsatisfiable instance-chain candidates are rejected
The analyzer SHALL reject an instance-chain candidate when the candidate's generated subgoals become provably unsatisfiable after nested class solving. Rejected candidates MUST NOT leak probe-only unifications, canonical errors, or residual constraints into the committed solver state, and the analyzer MUST continue to later `else` candidates when available.

#### Scenario: Nested comparison failure rejects stale attachment codec branch
- **WHEN** record-option folding processes `includeDocs`, `attachments`, and `binary` options whose type-level state is selected through `PickMax` and attachment codec classes
- **THEN** the analyzer MUST select the maximum `Blob` attachment codec branch without emitting `NoInstanceFound` for stale `Stub` codec constraints or impossible comparisons such as `Compare 3 1 LT`

#### Scenario: Stuck candidate subgoals are not rejected as impossible
- **WHEN** a matched instance-chain candidate produces subgoals that remain blocked on unresolved unification variables but are not provably apart
- **THEN** the analyzer MUST keep the candidate viable or blocked according to existing solver rules instead of rejecting it as unsatisfiable
