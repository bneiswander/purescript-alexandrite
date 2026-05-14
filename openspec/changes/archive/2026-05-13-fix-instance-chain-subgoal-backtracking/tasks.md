## 1. Regression Coverage

- [x] 1.1 Create a minimized checking fixture for `includeDocs`, `attachments`, and `binary` record-option folding with `PickMax` and `IsJAttachmentCodec`.
- [x] 1.2 Confirm the new fixture fails before the solver fix with stale `Stub` codec and impossible `Compare` residual diagnostics.

## 2. Solver Implementation

- [x] 2.1 Add a probe path for matched instance-chain candidate subgoals that can detect provably unsatisfiable nested constraints before candidate commitment.
- [x] 2.2 Ensure rejected candidate probes do not commit unifications, canonical errors, diagnostics, or residual constraints to the real solver state.
- [x] 2.3 Preserve existing stuck/blocking behavior for candidate subgoals that are unresolved but not provably apart.

## 3. Verification

- [x] 3.1 Run the new checking fixture plus existing related fixtures `1778700540_instance_subgoal_apart_compare` and `1778701500_pick_max_record_fold_order`.
- [x] 3.2 Run `cargo check -p checking --tests`.
- [x] 3.3 Review snapshots to ensure the valid `Blob` attachment branch is selected and no stale `Stub` codec diagnostic remains.
