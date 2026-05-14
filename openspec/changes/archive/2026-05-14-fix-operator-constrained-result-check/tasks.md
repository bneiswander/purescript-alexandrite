## 1. Regression Coverage

- [x] 1.1 Add a reduced checking fixture that defines `Run`, `EFFECT`, `MonadEffect`, a solvable `Run` `MonadEffect` instance, and a constrained polymorphic function shaped like `forall a m. a -> MonadEffect m => m (Env a)`.
- [x] 1.2 Include both direct application and `$` application forms checked against `Run (EFFECT r) Env` in the fixture.
- [x] 1.3 Run the targeted checking fixture and confirm the `$` form currently reproduces the false `CannotUnify` diagnostic before fixing the checker.

## 2. Checker Fix

- [x] 2.1 Update operator-chain checking so early expected-result subtyping is skipped when the operator result type is still an unresolved unification variable.
- [x] 2.2 Keep the existing post-operand result check so constrained operator results can elaborate wanted constraints after operands contribute type information.
- [x] 2.3 Avoid special-casing the `$` operator; apply the fix to the general operator result-checking path.

## 3. Verification

- [x] 3.1 Re-run the new checking fixture and verify both direct and `$` forms pass without `CannotUnify`.
- [x] 3.2 Re-run `just t checking 1774448340_operator_result_ordering` to ensure higher-rank operator result ordering remains intact.
- [x] 3.3 Run `cargo check -p checking --tests`.
