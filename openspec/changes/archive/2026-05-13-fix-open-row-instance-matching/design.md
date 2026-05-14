## Context

The constraint solver in `compiler-core/checking/src/core/constraint/matching.rs` handles matching wanted constraints against instance chains. The `match_row_type` function specifically handles row type matching during instance resolution.

When an instance has an open row pattern (e.g., `{ | r }`), the `freshen_instance_signature` function replaces quantified type variables with rigid variables. These rigid variables should act as pattern variables that match any type.

However, two separate issues caused valid instances to be rejected:

1. **`match_row_type`**: When the instance's given row has a tail (rigid variable from freshening), the function recursed with `match_instance_type`, which returns `Skolem` for rigid variables. This caused the instance chain to reject the candidate. The `blocking_type` function also returned `Apart` for rigid variables in the `Additional => Open` and `Closed => Open` cases.

2. **Compiler constraints (`Row.Cons`, `RowToList`)**: When these compiler constraints received rigid variables as arguments (from instance freshening), they would either block on unresolvable rigid variables or produce unifications that embedded rigid variables deeper without binding them. This prevented instance matching from binding the rigid variables to concrete row types.

## Goals / Non-Goals

**Goals:**
- Fix `match_row_type` to treat rigid variables in instance patterns as matching any row type (all three row rest cases)
- Fix `Row.Cons` to defer to instance matching when arguments are rigid variables
- Fix `RowToList` to defer to instance matching when arguments or tails are rigid variables

**Non-Goals:**
- Changes to `blocking_type` or `collect_blocking`
- Changes to how unification variables are handled (existing behavior preserved)
- Changes to other compiler constraints

## Decisions

**Decision 1: Check `Type::Rigid` in `match_row_type` before calling `blocking_type` or recursing**

When `wanted_tail` is a rigid variable (in `Additional => Open` and `Closed => Open` cases), return `Match` instead of calling `blocking_type`. When `given_tail` is a rigid variable (in `Open(given_tail)` case), return `Match` instead of recursing. Also check for blocking unification variables in `given_tail` before recursing.

*Rationale:* Rigid variables in instance `matchable` types are created by `freshen_instance_signature` to act as pattern variables. They should match anything.

**Decision 2: Defer to instance matching in `Row.Cons` and `RowToList` for rigid variables**

When `Row.Cons` or `RowToList` receive rigid variables as arguments, return `None` to defer to instance matching. This allows instance matching to bind the rigid variables to concrete row types first, after which the compiler constraints can be solved.

*Rationale:* Compiler constraints cannot make progress on rigid variables. By deferring to instance matching, the rigid variables get bound to concrete types, and the constraints are re-solved with the bound values.

*Alternatives considered:*
1. Modify `blocking_type` to handle rigid variables - rejected: rigid variables are not blocking, they're patterns. The fix belongs in the caller.
2. Convert rigid variables to unification variables before matching - rejected: would change the semantics of rigid variables globally.
3. Add a separate `Pattern` match result type - rejected: over-engineered for this use case.

## Risks / Trade-offs

[Risk] Existing tests may fail if they relied on the buggy behavior where open rows with rigid tails were rejected.
→ [Mitigation] Review failing tests; most should be legitimate cases that now correctly match.

[Risk] Deferring `Row.Cons` and `RowToList` for rigid variables could cause infinite loops if instance matching doesn't bind the rigid variables.
→ [Mitigation] Instance matching binds rigid variables to concrete types via pattern variable binding. After binding, the constraints are re-solved with concrete types and proceed normally.

[Risk] The fix affects three separate files, increasing the risk of unintended interactions.
→ [Mitigation] All integration tests pass with the changes.
