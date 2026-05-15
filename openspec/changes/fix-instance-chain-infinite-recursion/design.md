## Context

`candidate_constraints_are_unsatisfiable` in `match_instance_chain` checks whether an instance-chain candidate's generated subgoals are provably unsatisfiable by recursively calling `solve_constraints`. When a candidate's subgoals include constraints that recursively re-enter the same instance chain (e.g., `instance recursiveResolve :: Resolve a => Resolve a`), this creates an infinite call chain: `match_instance_chain -> candidate_constraints_are_unsatisfiable -> solve_constraints -> match_instance_chain -> candidate_constraints_are_unsatisfiable -> ...`.

The existing `nestedUnsatisfiableCandidates` feature (from `2026-05-13-fix-instance-chain-subgoal-backtracking`) already runs probe solving in isolated state, but it had no recursion guard. When recursive instance chains exist, the probe solver never terminates.

## Goals / Non-Goals

**Goals:**

- Terminate recursive candidate-probe calls to prevent infinite recursion.
- Preserve correct diagnostic output for non-recursive instance chains (existing fixtures must not regress).
- Avoid re-solving identical constraint sets across probes via structural memoization.
- Restore inferred hover for nodes not covered by fast paths (local binders, let bindings, expressions, types, puns).

**Non-Goals:**

- Change the instance-chain matching algorithm or its diagnostic output for valid code.
- Remove the candidate-unsatisfiability probe; only add recursion protection.
- Redesign the hover system; only restore the inferred-type fallback path.

## Decisions

1. **Add a depth guard to `candidate_constraints_are_unsatisfiable`**

   Decision: Return `false` (do not reject the candidate) when `candidate_constraint_probes.len() >= 2`.

   Rationale: A depth of 2 allows the solver to detect self-referential recursive candidates (the first probe calls itself once) while preventing unbounded recursion. The `false` return preserves the candidate for later `else` branches or normal residual reporting rather than incorrectly rejecting it.

   Alternative: Remove the probe entirely. Rejected because the probe is needed for non-recursive cases (e.g., the attachment codec fold).

2. **Add structural probe key matching**

   Decision: Introduce `ProbeKey`, `ProbeTypeKey`, and `ProbeArgKey` structs that represent a canonical constraint's type structure independently of its canonical ID. Use these to detect equivalent constraint sets across probes.

   Rationale: Recursive instance chains produce the same constraint structure with different canonical IDs across probes. Structural matching allows the depth guard to detect equivalent probes even when canonical IDs differ.

   Alternative: Track canonical IDs only. Rejected because recursive probes produce fresh canonical IDs each time, so ID-based deduplication would not detect the recursion.

3. **Add memoization cache for probe results**

   Decision: Add `candidate_constraint_probe_cache: FxHashMap<Vec<ProbeKey>, bool>` to `CheckState`. Cache successful probe results keyed by structural probe keys.

   Rationale: Even with the depth guard, the first few probes may do significant redundant work solving the same constraint sets. Memoization avoids re-solving identical probes within a single checkpoint scope.

   Alternative: No memoization. Rejected because profiling showed the first two probes alone take ~17s on large files with recursive instance chains.

4. **Restore inferred hover via `engine.checked` fallback**

   Decision: Add `hover_checked_type` helper that renders a `TypeId` from the checked module using `Pretty`. Update `hover_binder`, `hover_expression`, `hover_type`, `hover_let`, and `hover_pun` to fall back to this when the fast path does not match.

   Rationale: Hover was intentionally disabled for inferred nodes because the type-checker hang made checked queries too slow. With the recursion guard and memoization in place, inferred hover is viable again for most nodes (the guard only triggers on pathological recursive instance chains).

   Alternative: Keep inferred hover disabled. Rejected because users expect hover on local binders, let bindings, and inferred types.

## Risks / Trade-offs

- **Depth guard may suppress valid candidate rejection** -> A depth of 2 is conservative; it only triggers on recursive candidates. Non-recursive chains can still be fully probed. If false positives are observed, the depth can be increased.
- **Memoization cache grows with probe diversity** -> The cache is scoped to a single `checked` query (cleared on checkpoint restore). It does not persist across files.
- **Restored inferred hover may still be slow on pathological files** -> The depth guard terminates recursion, so even recursive instance chains complete in bounded time. The memoization cache reduces redundant work on large files.
