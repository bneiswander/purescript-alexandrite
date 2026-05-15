## Why

Instance-chain matching can enter infinite recursion when a candidate's generated subgoals include constraints that recursively re-enter the same instance chain. For example, `instance recursiveResolve :: Resolve a => Resolve a` generates a subgoal `Resolve a` that re-enters `match_instance_chain` with the same constraint. This causes the LSP hover and diagnostic system to hang indefinitely on files containing such patterns.

## What Changes

- Add structural probe key matching (`ProbeKey`, `ProbeTypeKey`, `ProbeArgKey`) to detect equivalent constraint sets across recursive instance-chain probes.
- Add a depth guard to `candidate_constraints_are_unsatisfiable` that terminates recursion after two nested probes.
- Add a memoization cache (`candidate_constraint_probe_cache`) to avoid re-solving identical constraint sets across probes.
- Restore inferred hover for local binders, let bindings, expressions, types, and puns by falling back to `engine.checked(current_file)` with the `Pretty` renderer when the fast path does not match.
- Add a minimized regression fixture for recursive `Resolve a => Resolve a` instance chains.

## Capabilities

### Modified Capabilities

- `type-checker-instance-solving`: The type checker MUST NOT enter infinite recursion when matching instance-chain candidates whose subgoals recursively re-enter the same chain. The analyzer MUST detect recursive candidate-probe calls and terminate them after a bounded depth, while preserving correct diagnostic output for non-recursive cases.

## Impact

- Affects `compiler-core/checking/src/core/constraint.rs` (probe key structs and helpers).
- Affects `compiler-core/checking/src/core/constraint/matching.rs` (`candidate_constraints_are_unsatisfiable` depth guard and memoization).
- Affects `compiler-core/checking/src/state.rs` (new probe tracking and cache fields on `CheckState`).
- Affects `compiler-lsp/analyzer/src/hover.rs` (restored inferred hover fallback).
- Adds checking integration fixture under `tests-integration/fixtures/checking/1778805900_recursive_instance_chain_candidate_constraint/`.
