## Context

The PureScript compiler uses an equality-driven constraint solver for type class instance resolution. Instance chains (declared with `instance ... else instance ...`) are matched left-to-right. When a non-final candidate is evaluated, the solver must decide whether to:
1. Commit to it (if all inputs are determined),
2. Block on unresolved unification variables (deferring to later candidates), or
3. Reject it (if it's apart).

The blocking decision is made by `non_determined_unification_ids` in `compiler-core/checking/src/core/constraint/matching.rs`. It collects unification variables from arguments that are not determined by functional dependencies. The current implementation iterates over **all** non-determined arguments and collects their blocking unification variables, regardless of whether the match result for that argument was already known.

## Goals / Non-Goals

**Goals:**
- Fix the false-positive `NoInstanceFound` errors caused by over-aggressive blocking in non-final instance-chain candidates.
- Ensure that only arguments with unknown/stuck match results contribute blocking unification variables.

**Non-Goals:**
- Refactoring the broader instance-chain matching algorithm.
- Changing functional dependency resolution logic.
- Modifying the `else` instance chain semantics.

## Decisions

### Decision: Filter by match result in `non_determined_unification_ids`

**Choice:** Only collect blocking unification IDs from results where `result.is_unknown()` is true.

**Rationale:** The purpose of blocking is to wait for unification variables to be solved before the solver can make progress on matching. If a match result is already `Match` or `Apart`, there are no unresolved unification variables in that argument's contribution to the match — the solver has already made full progress on it. Collecting unification variables from such arguments is spurious and causes unnecessary deferral.

**Alternatives considered:**
- **Remove blocking entirely for non-final candidates:** Too aggressive; would commit to a candidate before all inputs are determined, potentially leading to incorrect instance selection when functional dependencies are involved.
- **Track blocking per-argument separately:** More complex data structure; the current approach of collecting into a single `Vec<u32>` is simpler and sufficient once filtered.

## Risks / Trade-offs

[Risk] This change could allow earlier candidates to be selected when they should still be deferred.
→ [Mitigation] The `can_determine_stuck` check still blocks on arguments with unknown match results. Only arguments that already have a definitive match result (`Match`/`Apart`) skip the blocking check, which is correct because those arguments are no longer unresolved.

[Risk] Existing fixtures may show new errors or different behavior.
→ [Mitigation] Tested against all related instance-chain/fundep fixtures with no regressions.
