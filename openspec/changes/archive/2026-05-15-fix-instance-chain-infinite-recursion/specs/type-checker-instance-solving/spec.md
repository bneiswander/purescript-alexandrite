## ADDED Requirements

### Requirement: Recursive candidate probes terminate after bounded depth
The analyzer MUST terminate recursive candidate-probe calls in `candidate_constraints_are_unsatisfiable` when the probe depth reaches two or more nested calls. When the depth guard triggers, the function MUST return `false` (do not reject the candidate) rather than continuing to probe.

#### Scenario: Self-referential recursive instance chain
- **WHEN** an instance chain contains `instance recursiveResolve :: Resolve a => Resolve a` and the solver probes whether the generated subgoal `Resolve a` is unsatisfiable
- **THEN** the analyzer MUST detect the recursive probe via the depth guard and return `false`, allowing the candidate to proceed to later `else` branches or normal residual reporting

#### Scenario: Non-recursive instance chain is not affected
- **WHEN** an instance chain has non-recursive candidates whose subgoals do not re-enter the same chain
- **THEN** the analyzer MUST fully probe all candidates without triggering the depth guard

## ADDED Requirements

### Requirement: Candidate-unsatisfiability probes use structural key matching
The analyzer MUST use structural `ProbeKey` matching (independent of canonical IDs) to detect equivalent constraint sets across recursive candidate probes. This allows the depth guard to identify recursive probes even when canonical IDs differ between probe instances.

#### Scenario: Recursive probe with different canonical IDs is detected
- **WHEN** `candidate_constraints_are_unsatisfiable` is called with constraints that structurally match a previously probed constraint set (but have different canonical IDs)
- **THEN** the analyzer MUST detect the recursive probe via structural key matching and return `false`

### Requirement: Candidate-unsatisfiability probe results are memoized
The analyzer MUST cache successful probe results keyed by structural `ProbeKey` to avoid re-solving identical constraint sets across probes within a single checked query.

#### Scenario: Identical probe constraints are not re-solved
- **WHEN** `candidate_constraints_are_unsatisfiable` is called with constraints that structurally match a previously cached probe result
- **THEN** the analyzer MUST return the cached result without re-solving the constraints

## ADDED Requirements

### Requirement: Inferred hover returns checked types for uncovered nodes
The analyzer MUST return the checked type from `engine.checked(current_file)` for hover nodes not covered by fast paths (constructors, variables, operators, literals). This applies to local binders, let bindings, inferred expressions, inferred types, and puns.

#### Scenario: Hover on let binding shows inferred type
- **WHEN** the user hovers over a let-bound identifier whose type is inferred
- **THEN** the analyzer MUST render the checked type using `Pretty` (not return `None` or syntax only)

#### Scenario: Hover on inferred expression shows checked type
- **WHEN** the user hovers over an expression whose kind does not match a fast path
- **THEN** the analyzer MUST render the checked type from the checked module
