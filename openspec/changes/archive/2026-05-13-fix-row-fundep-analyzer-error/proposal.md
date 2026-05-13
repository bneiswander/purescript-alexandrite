## Why

The analyzer can report `CannotUnify` diagnostics for valid PureScript programs that `spago build` accepts when explicit signatures use row-polymorphic constraints whose functional dependencies determine result rows. This creates noisy editor errors and undermines trust in analyzer diagnostics.

## What Changes

- Fix analyzer type checking so row variables determined by functional-dependency constraints do not produce false `CannotUnify` diagnostics during definition checking.
- Add a focused checking regression test that reproduces a valid row-polymorphic constrained signature accepted by the reference compiler.
- Preserve existing diagnostics for real row mismatches and unsatisfied constraints.

## Capabilities

### New Capabilities
- `type-checker-row-fundep-diagnostics`: Covers analyzer diagnostic correctness for row-polymorphic functional-dependency constraints.

### Modified Capabilities

## Impact

- Affected code: `compiler-core/checking` constraint solving, unification, and/or signature checking paths.
- Affected tests: type checker integration fixtures and snapshots.
- No expected public API, CLI, or dependency changes.
