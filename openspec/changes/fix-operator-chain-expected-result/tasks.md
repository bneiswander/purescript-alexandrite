## 1. Regression Fixture

- [x] 1.1 Create a checking fixture with local stubs for `Run`, natural transformations, `Interpreter`, `unInterpreter`, flipped application, and composition.
- [x] 1.2 Model the reported `hoistSpec` pipeline shape using chained `#` applications over composed interpreters.
- [x] 1.3 Confirm the fixture reproduces the current false-positive diagnostic before applying the checker fix.
- [x] 1.4 Add an inline lambda callback fixture that keeps the operator chain inside the `hoistSpec` argument rather than hiding it behind a named pipeline.

## 2. Checker Implementation

- [x] 2.1 Update `traverse_operator_branch` so `OperatorKindMode::Check` constrains the operator branch `result_type` against simple expected types before operand traversal.
- [x] 2.2 Preserve the existing `OperatorKindMode::Infer` path and avoid operator-specific special cases.
- [x] 2.3 Review whether any duplicated expected-result subtype call remains after moving the check.
- [x] 2.4 Update checked lambda expressions to decompose/skolemise expected signatures before checking binders and body expressions.

## 3. Verification

- [x] 3.1 Run the new checking fixture and accept the snapshot only after verifying it has no unexpected errors.
- [x] 3.2 Run relevant existing checking fixtures for operator chains, inline lambdas, flipped application, and natural-transformation-shaped types.
- [x] 3.3 Run `cargo check -p checking --tests`.
