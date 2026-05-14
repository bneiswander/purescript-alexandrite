## Why

The type checker can report false-positive `CannotUnify` diagnostics for valid PureScript expressions that use operator chains with polymorphic natural transformations, such as `#` pipelines around `Run` interpreters. Expected result type information is available from the surrounding expression, but it is applied too late to guide operand checking inside the operator chain. The same failure can appear when the operator chain is inside an inline lambda callback whose expected body type is hidden under a `forall`, such as a natural transformation argument to `hoistSpec`.

## What Changes

- Ensure operator-chain checking propagates simple expected result types into branch checking before operands are checked.
- Ensure lambda checking decomposes/skolemises expected signatures before checking lambda bodies, so inline callbacks pass the correct body result type to nested operator chains.
- Preserve existing inference behavior for operator chains that are not being checked against an expected type.
- Add regression fixtures covering a `#` pipeline over `Interpreter`/natural-transformation-shaped values, including the inline `hoistSpec` callback shape.
- No breaking changes.

## Capabilities

### New Capabilities
- `type-checker-operator-chain-checking`: Correctness requirements for checking term operator chains against an expected type.

### Modified Capabilities

## Impact

- Affects type checker behavior in `compiler-core/checking/src/source/operator.rs` and `compiler-core/checking/src/source/terms/forms.rs`.
- Adds a type checker integration test fixture under `tests-integration/fixtures/checking/`.
- No public API or dependency changes.
