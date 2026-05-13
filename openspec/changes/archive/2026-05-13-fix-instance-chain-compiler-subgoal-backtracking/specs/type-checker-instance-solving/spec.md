## ADDED Requirements

### Requirement: Non-final instance-chain candidates do not force unresolved inputs
The analyzer SHALL NOT commit to a non-final instance-chain candidate when that candidate would solve or constrain unresolved unification variables in non-determined input positions.

#### Scenario: Equality helper must not force compare result before compare solves
- **WHEN** an instance-chain candidate for a helper class such as `Eval (Eq LT ord) isLess` can match by unifying an unresolved input variable `ord`
- **THEN** the analyzer MUST defer that non-final candidate until the input variable is solved by other constraints or givens

### Requirement: Immediately-apart compiler subgoals reject their candidate
The analyzer SHALL reject an instance-chain candidate when applying the candidate's own match unifications makes one of its compiler-solved subgoals immediately apart.

#### Scenario: False type-level integer comparison from candidate subgoal
- **WHEN** a candidate match produces a compiler-solved subgoal equivalent to `Compare 2 1 LT`
- **THEN** the analyzer MUST reject that candidate and continue to later `else` candidates instead of reporting `NoInstanceFound` for the false subgoal

### Requirement: Record-folding with type-level max selects the valid branch
The analyzer SHALL type check record-folding code where a fold updates type-level state using `PickMax` over integer-ranked constructors.

#### Scenario: Attachments and includeDocs config fold
- **WHEN** a record contains both `attachments: Proxy True` and `includeDocs: Proxy True`, and folding updates `CA NoDoc NoKeys` through `PickMax (Doc Base64) (Doc Stub)`
- **THEN** the analyzer MUST infer the maximum document kind without emitting `Compare 2 1 LT`
