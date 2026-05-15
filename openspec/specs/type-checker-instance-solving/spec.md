## Purpose

Define instance-chain solving behavior for compiler constraints, candidate rejection, and recursive probes.
## Requirements
### Requirement: Rigid variables in instance row patterns match any row
The analyzer SHALL match an open row pattern with a rigid variable tail (from `freshen_instance_signature`) against any given row type, including closed rows. Rigid variables in instance `matchable` types act as pattern variables and must not be treated as `Apart`.

#### Scenario: Open row pattern matches closed record row
- **WHEN** an instance has head `Class { | r }` and the wanted constraint has a closed row type like `{ a :: Int, b :: String }`
- **THEN** the analyzer MUST return `Match` (not `Apart`) and bind the rigid tail variable to the concrete row

#### Scenario: Open row pattern with rigid tail absorbs additional given fields
- **WHEN** an instance has head `Class { | r }` and the wanted constraint has an open row with rigid tail like `{ a :: Int | ?r }` matching against a given row with additional fields `{ a :: Int, b :: String }`
- **THEN** the analyzer MUST return `Match` and allow the rigid tail to absorb the extra fields

#### Scenario: Unification variable tails still defer via Stuck
- **WHEN** an instance has head `Class { | r }` where `r` is an unification variable (not a rigid variable)
- **THEN** the analyzer MUST return `Stuck` (not `Apart`) to defer the match until the unification variable is solved

### Requirement: Compiler row constraints defer to instance matching for rigid variables
The analyzer SHALL defer `Row.Cons` and `RowToList` compiler constraints to instance matching when their arguments are rigid variables. This allows instance matching to bind the rigid variables to concrete row types first.

#### Scenario: Row.Cons with rigid row argument defers
- **WHEN** a `Row.Cons` constraint has a rigid variable as the row argument
- **THEN** the constraint solver MUST return `None` from the compiler constraint handler, allowing instance matching to bind the rigid variable

#### Scenario: RowToList with rigid row argument defers
- **WHEN** a `RowToList` constraint has a rigid variable as the row argument
- **THEN** the constraint solver MUST return `None` from the compiler constraint handler, allowing instance matching to bind the rigid variable

#### Scenario: RowToList with rigid tail variable defers
- **WHEN** a `RowToList` constraint has a row with a rigid variable tail
- **THEN** the constraint solver MUST return `None` from the compiler constraint handler, allowing instance matching to bind the rigid variable

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

### Requirement: Nested-unsatisfiable instance-chain candidates are rejected
The analyzer SHALL reject an instance-chain candidate when the candidate's generated subgoals become provably unsatisfiable after nested class solving. Rejected candidates MUST NOT leak probe-only unifications, canonical errors, or residual constraints into the committed solver state, and the analyzer MUST continue to later `else` candidates when available.

#### Scenario: Nested comparison failure rejects stale attachment codec branch
- **WHEN** record-option folding processes `includeDocs`, `attachments`, and `binary` options whose type-level state is selected through `PickMax` and attachment codec classes
- **THEN** the analyzer MUST select the maximum `Blob` attachment codec branch without emitting `NoInstanceFound` for stale `Stub` codec constraints or impossible comparisons such as `Compare 3 1 LT`

#### Scenario: Stuck candidate subgoals are not rejected as impossible
- **WHEN** a matched instance-chain candidate produces subgoals that remain blocked on unresolved unification variables but are not provably apart
- **THEN** the analyzer MUST keep the candidate viable or blocked according to existing solver rules instead of rejecting it as unsatisfiable

### Requirement: Record-folding with type-level max selects the valid branch
The analyzer SHALL type check record-folding code where a fold updates type-level state using `PickMax` over integer-ranked constructors.

#### Scenario: Attachments and includeDocs config fold
- **WHEN** a record contains both `attachments: Proxy True` and `includeDocs: Proxy True`, and folding updates `CA NoDoc NoKeys` through `PickMax (Doc Base64) (Doc Stub)`
- **THEN** the analyzer MUST infer the maximum document kind without emitting `Compare 2 1 LT`

### Requirement: Operator applications elaborate constrained monadic results
The analyzer SHALL check operator applications consistently with equivalent direct function applications when the operator result comes from a constrained polymorphic function. Expected result guidance MUST NOT cause a constrained result such as `MonadEffect m => m a` to be unified directly with a concrete monad type before wanted constraints can be elaborated and solved.

#### Scenario: Dollar application solves Run MonadEffect wanted
- **WHEN** a definition with signature `Run (EFFECT r) Env` uses `$` to apply a function of type `forall a m. a -> MonadEffect m => m (Env a)` to an argument
- **THEN** the analyzer MUST solve the wanted `MonadEffect (Run (EFFECT r))` through the available `Run` instance and MUST NOT emit `CannotUnify` between the constrained monadic result and `Run (EFFECT r) Env`

#### Scenario: Direct and operator applications agree
- **WHEN** the same constrained polymorphic function call is written once with direct application and once with `$`
- **THEN** the analyzer MUST accept both forms or reject both forms for the same semantic reason, rather than accepting the direct form while reporting a unification diagnostic only for the operator form

### Requirement: Compiler Reflectable solver only returns Apart for provable incompatibility
The analyzer SHALL NOT return `Apart` from the compiler's built-in `Reflectable` literal solver when the solver simply does not apply to the given arguments. The solver SHALL only return `Apart` when it can conclusively prove incompatibility between a literal value and an incompatible type target.

#### Scenario: Reflectable with non-literal first argument defers to user instances
- **WHEN** a `Reflectable` constraint has a first argument that is a type constructor (not a literal symbol, integer, boolean, ordering, or unification variable)
- **THEN** the analyzer MUST return `None` from the compiler literal solver, allowing normal instance chain search to find user-defined `Reflectable` instances

#### Scenario: Reflectable with literal and mismatched type still reports Apart
- **WHEN** a `Reflectable` constraint has a literal first argument (e.g., a symbol `"hello"`) and a type second argument that is provably incompatible (e.g., `Int`)
- **THEN** the analyzer MUST return `Apart` from the compiler literal solver, correctly rejecting all candidates

### Requirement: Inferred hover returns checked types for uncovered nodes
The analyzer MUST return the checked type from `engine.checked(current_file)` for hover nodes not covered by fast paths (constructors, variables, operators, literals). This applies to local binders, let bindings, inferred expressions, inferred types, and puns.

#### Scenario: Hover on let binding shows inferred type
- **WHEN** the user hovers over a let-bound identifier whose type is inferred
- **THEN** the analyzer MUST render the checked type using `Pretty` (not return `None` or syntax only)

#### Scenario: Hover on inferred expression shows checked type
- **WHEN** the user hovers over an expression whose kind does not match a fast path
- **THEN** the analyzer MUST render the checked type from the checked module

