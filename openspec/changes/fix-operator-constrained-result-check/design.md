## Context

Operator chains are checked through `compiler-core/checking/src/source/operator.rs`, which decomposes the operator type, checks operands, and optionally uses an expected result type to guide checking. The `$` operator is just a normal infix function, but result-guided checking can cause its result type to be unified against the expected signature before the left operand's constrained polymorphic result has been elaborated.

The reported false positive occurs when `UseHotRefST.env` has type `forall a m. a -> MonadEffect m => m (Env a)` and is used as the left operand of `$` in a definition checked against `Run (EFFECT r) Env`. Direct application works because the ordinary application path instantiates the constrained result and pushes `MonadEffect (Run (EFFECT r))` as a wanted constraint.

## Goals / Non-Goals

**Goals:**

- Make `$` operator application behave consistently with direct function application for constrained polymorphic results.
- Keep existing early expected-result propagation where it is safe and useful for concrete result types.
- Preserve higher-rank result ordering behavior covered by existing operator regression tests.
- Add a focused checking fixture that reproduces the `MonadEffect (Run (EFFECT r))` case.

**Non-Goals:**

- Redesign operator-chain elaboration or dictionary insertion globally.
- Change PureScript source semantics, `Run`, `MonadEffect`, or `TypeEquals` behavior.
- Change diagnostics formatting or LSP publication behavior.

## Decisions

- Defer expected-result subtyping when the operator result is still an unresolved unification variable.
  - Rationale: a bare unification result for `$` needs operand checking to reveal whether the result is constrained. Early subtyping can solve the variable to the expected type too soon, making the left operand's constrained function type unify in a non-elaborating function position.
  - Alternative considered: always defer expected-result checking for operators. This is broader and risks regressing existing result-guided checks such as `1774448340_operator_result_ordering`.

- Keep the final result check after operand checking in the existing elaborating path.
  - Rationale: once operands have contributed type information, constrained results can be instantiated with wanted constraints and solved by the normal constraint solver.
  - Alternative considered: special-case `$`. This would be narrower but incorrect because any operator with the same type shape can expose the same bug.

- Cover the behavior with a reduced integration fixture instead of relying on the original project modules.
  - Rationale: the fixture can model `Run`, `EFFECT`, `MonadEffect`, and the `TypeEquals`-guarded instance without depending on external package sources.
  - Alternative considered: import real package modules in a fixture. This is brittle and less auditable.

## Risks / Trade-offs

- Risk: Deferring expected-result checking for too many operator results could reduce type information available for operand checking. Mitigation: restrict the deferral to unresolved unification result types and run existing operator ordering fixtures.
- Risk: The reduced fixture may not fully model the source package's row aliases. Mitigation: include the key semantics: a constrained polymorphic function, `Run (EFFECT r)`, `MonadEffect`, and a solvable `TypeEquals` instance.
- Risk: Constraint elaboration may still occur too late if the left operand is checked against a non-elaborating function type. Mitigation: verify both the `$` form and direct-call form pass in the fixture and inspect emitted terms/diagnostics.
