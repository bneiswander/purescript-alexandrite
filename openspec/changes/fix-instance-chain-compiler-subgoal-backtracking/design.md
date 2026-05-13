## Context

Instance chains are tried in source order. A non-final instance can match structurally while still relying on wanted-side unification variables in input positions. If the solver commits to that candidate too early, subgoals can force those input variables to an invalid value and produce false residual diagnostics.

The motivating downstream case folds a record with `attachments: Proxy True` and `includeDocs: Proxy True`. Folding calls `PickMax (Doc Base64) (Doc Stub) c`, which depends on `Compare 2 1 ord` and `Eval (LT == ord) isLess`. The first `Eval` instance for equality can match by unifying `ord` with `LT`; this later creates the false subgoal `Compare 2 1 LT`.

## Goals / Non-Goals

**Goals:**
- Preserve normal instance-chain behavior for final candidates and determined output positions.
- Prevent non-final `else` candidates from prematurely solving input-side unification variables.
- Reject candidates whose subgoals become immediately apart after applying the candidate's own match unifications.

**Non-Goals:**
- Do not introduce general-purpose backtracking for the entire constraint solver.
- Do not change `Prim.Int.Compare`, `Eval`, or user-library instance declarations.

## Decisions

### Decision 1: Defer non-final candidates with unresolved input positions

For non-final instance-chain candidates, matching now checks whether the match would leave unification variables in non-determined type arguments. If so, the constraint is marked stuck instead of committing to the candidate. Determined positions are exempt because functional dependencies permit those outputs to be solved by the candidate.

### Decision 2: Use type-argument indices for fundep determination

The candidate match result list contains only type arguments. Therefore, functional-dependency determination is computed against type-argument positions, not full canonical argument positions that may include kind arguments.

### Decision 3: Apply candidate unifications to subgoals before apart checks

Before returning matched subgoals, candidate match unifications are applied to those subgoals. Compiler-solved subgoals are then checked for immediate `Apart`; if any are apart, the candidate is rejected and the next `else` candidate is tried.

## Risks / Trade-offs

- **[Risk]** Deferring non-final candidates can delay solving constraints that would otherwise make progress. → **Mitigation:** Deferral is limited to non-determined input-side unification variables; determined output positions still solve normally.
- **[Risk]** Applying candidate unifications to subgoals could make diagnostics less direct if a candidate is rejected. → **Mitigation:** Rejected candidates were invalid for the current wanted constraint; reporting their subgoals was the bug.
