## Why

The compiler's built-in `Reflectable` literal solver incorrectly returns `Apart` when the first argument is not a primitive literal (symbol, integer, boolean, or ordering). This permanently rejects user-defined `Reflectable` instances whose recursive subgoals are also user-defined, such as instances over type-level lists (e.g., `MkTransitCoreTL` / `MkMatchTL` / `MkReturnTL` chains in `Transit.Core`). Users see a false-positive `NoInstanceFound` error even though valid instances exist in scope.

## What Changes

- **Modified**: `compiler-core/checking/src/core/constraint/compiler/prim_reflectable.rs` — the `match_reflectable` function now returns `None` (compiler solver does not apply) instead of `Apart` for non-literal first arguments that are not unification variables.
- **Added**: Regression fixture at `tests-integration/fixtures/checking/1778704560_reflectable_recursive_user_instances/` demonstrating recursive `Reflectable` instances over a user-defined type-level list.

## Capabilities

### New Capabilities

- `reflectable-recursive-user-instances`: The constraint solver SHALL allow recursive user-defined `Reflectable` instances where subgoals are themselves user-defined `Reflectable` constraints (not only compiler-solvable literals). This is the core capability enabled by the fix.

### Modified Capabilities

- `type-checker-instance-solving`: The requirement "Immediately-apart compiler subgoals reject their candidate" is updated to clarify that the `Apart` verdict from compiler solvers only applies when the compiler solver can conclusively prove incompatibility, not when it simply does not apply.

## Impact

- **Affected**: `compiler-core/checking/src/core/constraint/compiler/prim_reflectable.rs` — one line change.
- **Affected**: `compiler-core/checking/src/core/constraint/matching.rs` — candidate rejection path when compiler subgoals return `Apart`.
- **User-facing**: Eliminates false-positive `NoInstanceFound` errors for `Reflectable` over recursive type-level constructs.