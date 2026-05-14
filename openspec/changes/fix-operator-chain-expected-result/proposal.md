## Why

The type checker can report false-positive `CannotUnify` diagnostics for valid PureScript expressions that use operator chains with polymorphic natural transformations, such as `#` pipelines around `Run` interpreters. Expected result type information is available from the surrounding expression, but it is applied too late to guide operand checking inside the operator chain.

## What Changes

- Ensure operator-chain checking propagates simple expected result types into branch checking before operands are checked.
- Preserve existing inference behavior for operator chains that are not being checked against an expected type.
- Add a regression fixture covering a `#` pipeline over `Interpreter`/natural-transformation-shaped values.
- No breaking changes.

## Capabilities

### New Capabilities
- `type-checker-operator-chain-checking`: Correctness requirements for checking term operator chains against an expected type.

### Modified Capabilities

## Impact

- Affects type checker behavior in `compiler-core/checking/src/source/operator.rs`.
- Adds a type checker integration test fixture under `tests-integration/fixtures/checking/`.
- No public API or dependency changes.
