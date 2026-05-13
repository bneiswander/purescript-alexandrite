## Why

The analyzer could report false `NoInstanceFound` diagnostics for compiler-solved constraints produced by non-final instance-chain candidates. In particular, record-folding through `PickMax` could commit to an `Eval (LT == ord)` candidate too early, yielding an impossible residual constraint such as `Compare 2 1 LT` instead of continuing toward the valid branch.

## What Changes

- Instance-chain matching SHALL avoid committing to non-final `else` candidates when the match depends on unresolved input-side unification variables.
- Instance subgoals SHALL be canonicalized after applying candidate match unifications so immediately-apart compiler-solved subgoals reject that candidate.
- Add regression coverage for direct `PickMax` and record-folding shapes that previously produced `Compare 2 1 LT`.

## Capabilities

### New Capabilities

- `type-checker-instance-solving`: Behavior for instance-chain matching, compiler-solved subgoals, and residual constraint selection.

### Modified Capabilities

None.

## Impact

- `compiler-core/checking/src/core/constraint/matching.rs`: instance-chain matching and candidate subgoal handling.
- `tests-integration/fixtures/checking/1778700540_instance_subgoal_apart_compare/`: focused instance-chain candidate regression.
- `tests-integration/fixtures/checking/1778701380_pick_max_compare_eval/`: direct `PickMax` regression.
- `tests-integration/fixtures/checking/1778701500_pick_max_record_fold_order/`: record-folding regression matching the downstream `BuildO`/`PickMax` shape.
