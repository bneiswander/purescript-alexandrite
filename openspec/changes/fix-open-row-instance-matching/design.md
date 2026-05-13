## Context

The constraint solver in `compiler-core/checking/src/core/constraint/matching.rs` handles matching wanted constraints against instance chains. The `match_row_type` function specifically handles row type matching during instance resolution.

When an instance has an open row pattern (e.g., `{ | r }`), the `freshen_instance_signature` function replaces quantified type variables with rigid variables. These rigid variables should act as pattern variables that match any type.

However, `match_row_type` calls `blocking_type` on the wanted row tail when matching against a closed given row. `blocking_type` only collects unification variables and returns `Apart` when no unification variables are found - including when the tail is a rigid variable. This causes valid instances to be rejected.

## Goals / Non-Goals

**Goals:**
- Fix `match_row_type` to treat rigid variables in instance patterns as matching any row type
- Apply the fix in both affected match arms: `Additional => Open(wanted_tail)` and `Closed => Open(wanted_tail)`

**Non-Goals:**
- Changes to `blocking_type` or `collect_blocking`
- Changes to how unification variables are handled (existing behavior preserved)
- Changes to other aspect of instance resolution

## Decisions

**Decision: Check `Type::Rigid` before calling `blocking_type`**

When `wanted_tail` is a rigid variable, return `Match` (with accumulated `result`) instead of calling `blocking_type`.

*Rationale:* Rigid variables in instance `matchable` types are created by `freshen_instance_signature` to act as pattern variables. They should match anything, similar to how a pattern variable in a function match is bound rather than rejected.

*Alternatives considered:*
1. Modify `blocking_type` to handle rigid variables - rejected: rigid variables are not blocking, they're patterns. The fix belongs in the caller.
2. Convert rigid variables to unification variables before matching - rejected: would change the semantics of rigid variables globally.
3. Add a separate `Pattern` match result type - rejected: over-engineered for this single use case.

## Risks / Trade-offs

[Risk] Existing tests may fail if they relied on the buggy behavior where open rows with rigid tails were rejected.
→ [Mitigation] Review failing tests; most should be legitimate cases that now correctly match.

[Risk] The fix could affect performance if rigid variable checks are expensive.
→ [Mitigation] `context.lookup_type` is a simple arena lookup; the `Type::Rigid` match is O(1).

[Risk] The fix only covers two match arms; other arms might have the same issue.
→ [Mitigation] The `Open(given_tail)` case handles rigid tails via the recursion in `match_instance_type`, which already has rigid variable pattern matching logic at lines 289-304.
