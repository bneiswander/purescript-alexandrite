## ADDED Requirements

### Requirement: Visible type application against Eval-rigid variables
When a function signature declares an explicit constraint with a functional dependency that resolves to an Eval-like class whose result unifies with an explicit forall-bound variable, the type checker SHALL allow visible type applications against that variable in the function body.

#### Scenario: VTA resolves against explicit constraint
- **WHEN** a function is checked with signature `forall x. IsBoolean x => Eval (NotEq False True) x => Boolean` and the body uses `Proxy @x`
- **THEN** the type checker resolves `x` against the explicit `IsBoolean x` constraint and does not emit a `NoVisibleTypeVariable` or `NoInstanceFound` error

#### Scenario: Functional dependency substitution still applies to arguments and result
- **WHEN** the same function is checked
- **THEN** argument and result types are substituted according to the FD improvement, producing concrete types (e.g., `Eval (NotEq False True) True`)

### Requirement: Original givens retained alongside improved givens
When functional-dependency elaboration produces a non-empty substitution, the type checker SHALL retain the original given constraints in the constraint set alongside the improved (substituted) constraints.

#### Scenario: Original constraint available for body resolution
- **WHEN** `analyse_equation_set` applies a FD substitution from `elaborate_given_substitution`
- **THEN** the original constraints (with rigid variable names intact) are appended to the constraint list after the improved constraints

#### Scenario: No regression in existing tests
- **WHEN** the existing test suite is run
- **THEN** all previously passing tests continue to pass, including tests that use Eval with functional dependencies (e.g., `1778701380_pick_max_compare_eval`)