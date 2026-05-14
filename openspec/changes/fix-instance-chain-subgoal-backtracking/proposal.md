## Why

The type checker can emit false-positive `NoInstanceFound` diagnostics when an instance-chain candidate produces subgoals that only become provably impossible after nested class solving. This affects real PureScript code that folds record options through type-level `PickMax`, where an invalid intermediate branch leaks stale codec constraints into diagnostics.

## What Changes

- Reject instance-chain candidates whose generated subgoals become unsatisfiable after local candidate solving, not only when a compiler-solved subgoal is immediately apart.
- Preserve valid backtracking through later `else` candidates when an earlier candidate's subgoals imply impossible type-level integer comparisons.
- Add a regression fixture covering `includeDocs`, `attachments`, and `binary` record-option folding with attachment codec selection.
- No breaking changes.

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `type-checker-instance-solving`: Instance-chain candidate selection must reject candidates whose nested subgoals are provably unsatisfiable before committing residual constraints.

## Impact

- Affected code: `compiler-core/checking/src/core/constraint/matching.rs` and related constraint solving helpers if needed.
- Affected tests: checking integration fixtures for instance solving and record-folding option inference.
- Public APIs and dependencies are unchanged.
