## MODIFIED Requirements

### Requirement: Immediately-apart compiler subgoals reject their candidate
The analyzer SHALL reject an instance-chain candidate when applying the candidate's own match unifications makes one of its compiler-solved subgoals immediately apart.

#### Scenario: False type-level integer comparison from candidate subgoal
- **WHEN** a candidate match produces a compiler-solved subgoal equivalent to `Compare 2 1 LT`
- **THEN** the analyzer MUST reject that candidate and continue to later `else` candidates instead of reporting `NoInstanceFound` for the false subgoal

### Requirement: Compiler Reflectable solver only returns Apart for provable incompatibility
The analyzer SHALL NOT return `Apart` from the compiler's built-in `Reflectable` literal solver when the solver simply does not apply to the given arguments. The solver SHALL only return `Apart` when it can conclusively prove incompatibility between a literal value and an incompatible type target.

#### Scenario: Reflectable with non-literal first argument defers to user instances
- **WHEN** a `Reflectable` constraint has a first argument that is a type constructor (not a literal symbol, integer, boolean, ordering, or unification variable)
- **THEN** the analyzer MUST return `None` from the compiler literal solver, allowing normal instance chain search to find user-defined `Reflectable` instances

#### Scenario: Reflectable with literal and mismatched type still reports Apart
- **WHEN** a `Reflectable` constraint has a literal first argument (e.g., a symbol `"hello"`) and a type second argument that is provably incompatible (e.g., `Int`)
- **THEN** the analyzer MUST return `Apart` from the compiler literal solver, correctly rejecting all candidates