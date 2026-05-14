## Context

Instance-chain matching currently commits to the first candidate whose head matches and whose immediate compiler-solved subgoals are not apart. For candidates with ordinary class subgoals, impossibility may only become visible after those subgoals solve into compiler constraints such as `Prim.Int.Compare`. When that happens after commitment, the invalid branch leaves residual `NoInstanceFound` diagnostics instead of allowing the solver to continue to later `else` candidates.

This appears in record-option folds that build a type-level maximum attachment kind. The fold can temporarily consider a lower-ranked document kind, then later discover a `Compare` constraint that contradicts the requested ordering.

## Goals / Non-Goals

**Goals:**

- Reject an instance-chain candidate when its generated subgoals are provably unsatisfiable after nested solving.
- Preserve normal residual behavior for genuinely unsolved constraints and incomplete candidate searches.
- Avoid committing probe unifications, canonical errors, or diagnostics from rejected candidates.
- Cover the failure with a minimized checking fixture based on attachment option folding.

**Non-Goals:**

- Redesign the full constraint solver or introduce comprehensive Prolog-style backtracking.
- Change PureScript source semantics, public compiler APIs, or diagnostic formatting.
- Treat stuck constraints as failures; only provably unsatisfiable subgoals should reject candidates.

## Decisions

- Add candidate viability probing rather than changing final residual reporting.
  - Rationale: the bug is candidate commitment, not how top-level unsatisfied constraints are displayed.
  - Alternative considered: suppress `NoInstanceFound` for impossible compiler constraints. That would hide symptoms while still selecting the wrong instance branch.

- Run probing in isolated solver state.
  - Rationale: checking candidate subgoals may allocate unification variables, produce canonical errors, or solve local variables. Rejected candidates must not mutate the real solver state.
  - Alternative considered: solve directly and roll back only selected fields. That is more fragile because solver state spans unifications, canonicals, errors, and checked module side effects.

- Treat probe success with residuals as viable unless a residual has an attached canonical error or a compiler solver proves it apart.
  - Rationale: stuck constraints can still become solvable from surrounding work, while attached errors and apart compiler constraints are evidence that the candidate is impossible.
  - Alternative considered: reject any residual from the probe. That would be too aggressive and could reject valid candidates requiring external givens or later unifications.

## Risks / Trade-offs

- Probe solving may add cost to instance-chain matching. Mitigation: only probe after a candidate head matches and produces subgoals, and keep the probe limited to candidate subgoals.
- Isolating state may require cloning or snapshotting solver data structures that are not currently clone-friendly. Mitigation: implement the smallest snapshot/restore mechanism needed for solver-local state, or introduce a dedicated probe helper that preserves existing external state.
- Over-rejecting stuck candidates would regress valid code. Mitigation: regression tests should include existing instance-chain cases and only reject provably impossible nested subgoals.
