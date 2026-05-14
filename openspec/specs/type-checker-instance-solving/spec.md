## ADDED Requirements

### Requirement: Non-final instance-chain candidates do not force unresolved inputs
The analyzer SHALL NOT commit to a non-final instance-chain candidate when that candidate would solve or constrain unresolved unification variables in non-determined input positions. The analyzer SHALL only collect blocking unification variables from arguments whose match result is unknown (stuck or skolem); arguments that already matched or were definitively apart are excluded from the blocking check.

#### Scenario: Equality helper must not force compare result before compare solves
- **WHEN** an instance-chain candidate for a helper class such as `Eval (Eq LT ord) isLess` can match by unifying an unresolved input variable `ord`
- **THEN** the analyzer MUST defer that non-final candidate until the input variable is solved by other constraints or givens

#### Scenario: Example function instance with resolved match does not block
- **WHEN** a wanted constraint `Example (arg -> ?m Unit) arg Aff` is matched against an `Example` instance chain where `exampleFunc` matches the first and third arguments (with `m ~ Aff` unified) but the lambda result contains an unresolved variable `?m`
- **THEN** the analyzer MUST NOT block on `?m` because the match result for that argument is already `Apart`, and MUST proceed to unify `m ~ Aff` from the third argument

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

### Requirement: Compiler Reflectable solver only returns Apart for provable incompatibility
The analyzer SHALL NOT return `Apart` from the compiler's built-in `Reflectable` literal solver when the solver simply does not apply to the given arguments. The solver SHALL only return `Apart` when it can conclusively prove incompatibility between a literal value and an incompatible type target.

#### Scenario: Reflectable with non-literal first argument defers to user instances
- **WHEN** a `Reflectable` constraint has a first argument that is a type constructor (not a literal symbol, integer, boolean, ordering, or unification variable)
- **THEN** the analyzer MUST return `None` from the compiler literal solver, allowing normal instance chain search to find user-defined `Reflectable` instances

#### Scenario: Reflectable with literal and mismatched type still reports Apart
- **WHEN** a `Reflectable` constraint has a literal first argument (e.g., a symbol `"hello"`) and a type second argument that is provably incompatible (e.g., `Int`)
- **THEN** the analyzer MUST return `Apart` from the compiler literal solver, correctly rejecting all candidates
