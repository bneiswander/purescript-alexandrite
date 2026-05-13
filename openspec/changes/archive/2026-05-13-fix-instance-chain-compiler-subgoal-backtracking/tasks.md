## 1. Regression Coverage

- [x] 1.1 Add a focused instance-chain regression where a false `Compare 2 1 LT` subgoal would previously be reported instead of trying the next `else` candidate.
- [x] 1.2 Add a direct `PickMax` regression using `Compare`, `Eval (LT == ord)`, and `If`.
- [x] 1.3 Add a record-folding regression matching the `BuildO` shape for `{ includeDocs: Proxy True, attachments: Proxy True }`.

## 2. Solver Implementation

- [x] 2.1 Apply candidate match unifications to instance subgoals before canonicalizing and checking compiler-solved subgoals.
- [x] 2.2 Reject instance-chain candidates whose compiler-solved subgoals become immediately apart after candidate substitutions.
- [x] 2.3 Defer non-final `else` candidates that depend on unresolved unification variables in non-determined input positions.
- [x] 2.4 Ensure fundep determination for candidate deferral uses type-argument positions, not canonical positions including kind arguments.

## 3. Verification

- [x] 3.1 Run focused regressions for instance-chain subgoal apartness, direct `PickMax`, and record fold order.
- [x] 3.2 Run related integer compare and compiler-solver regression fixtures.
- [x] 3.3 Run `cargo check -p checking --tests`.
- [x] 3.4 Install local analyzer and confirm the downstream `Aff.purs` error is fixed.
