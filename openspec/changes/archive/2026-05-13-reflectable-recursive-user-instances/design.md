## Context

The compiler provides built-in literal solvers for `Data.Reflectable.Reflectable` in `prim_reflectable.rs`. These solvers handle the common cases where the first argument is a primitive literal (symbol, integer, boolean, ordering). For all other cases, the solver returned `MatchInstance::Apart`, meaning the compiler had definitively proven no instance could exist.

This was wrong for non-literal, non-unification-variable first arguments: a user-defined `Reflectable` instance could legitimately exist for such a type, and returning `Apart` would permanently reject all instance candidates whose subgoals depended on such user-defined instances.

## Goals / Non-Goals

**Goals:**
- Eliminate false-positive `NoInstanceFound` errors for recursive `Reflectable` instances over user-defined type-level lists.
- Keep compiler literal solvers for cases where they provably prove incompatibility.

**Non-Goals:**
- No change to how literal cases are handled.
- No change to how unification-variable cases are handled.
- No change to other compiler solvers (`prim_int`, `prim_symbol`, `prim_row`, `prim_row_list`).

## Decisions

### Decision: Return `None` instead of `Apart` for non-literal, non-unification first arguments

**Choice**: Change `Ok(Some(MatchInstance::Apart))` → `Ok(None)` at `prim_reflectable.rs:50`.

**Rationale**: Returning `None` means "the compiler's literal solver does not apply here", which defers to the normal instance chain search. This allows user-defined `Reflectable` instances to be found. Returning `Apart` meant "no instance exists", which incorrectly rejected valid candidates.

**Alternatives considered**:
- Returning `Stuck` with collected unification variables: Would cause the constraint to be retried, but there are no unification variables in this case, so `Stuck` would be permanent and equivalent to `Apart`.
- Returning `Match` with no constraints: Would incorrectly claim an instance exists without requiring proof.

## Risks / Trade-offs

- **Risk**: Previously, returning `Apart` for a non-literal `Reflectable` constraint with no user instances in scope would cause a faster `NoInstanceFound` error. Now the solver will also search instance chains before reporting the error. **Mitigation**: The instance chain search is efficient and the performance impact is negligible for normal usage.