## Why

Checking an expression through the `$` operator can report a false `CannotUnify` diagnostic when the left operand returns a constrained polymorphic monad, even though the equivalent direct function application type checks. This produces incorrect LSP errors for valid PureScript code that relies on `MonadEffect` instances for `Run (EFFECT r)`.

## What Changes

- Fix operator-chain expression checking so expected result guidance does not prematurely unify constrained polymorphic operator results in a non-elaborating context.
- Preserve existing result-guided checking behavior for concrete operator results, including higher-rank cases covered by existing regression tests.
- Add a reduced type-checker integration fixture for a `MonadEffect`-constrained function used through `$` and checked against `Run (EFFECT r)`.
- Ensure the direct-call and `$` forms produce consistent diagnostics and inferred/checked terms.

## Capabilities

### New Capabilities


### Modified Capabilities

- `type-checker-instance-solving`: Type checker must elaborate constrained results from operator applications so valid wanted instances, such as `MonadEffect (Run (EFFECT r))`, are solved instead of leaking as `CannotUnify`.

## Impact

- Affects expression operator-chain checking in `compiler-core/checking/src/source/operator.rs`.
- Adds or updates checking integration fixtures under `tests-integration/fixtures/checking`.
- No PureScript API, CLI, or dependency changes.
