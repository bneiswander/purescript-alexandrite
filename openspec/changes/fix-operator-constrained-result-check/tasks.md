## 1. Regression Coverage

- [ ] 1.1 Add a reduced checking fixture that defines `Run`, `EFFECT`, `MonadEffect`, a solvable `Run` `MonadEffect` instance, and a constrained polymorphic function shaped like `forall a m. a -> MonadEffect m => m (Env a)`.
- [ ] 1.2 Include both direct application and `$` application forms checked against `Run (EFFECT r) Env` in the fixture.
- [ ] 1.3 Run the targeted checking fixture and confirm the `$` form currently reproduces the false `CannotUnify` diagnostic before fixing the checker.

## 2. Checker Fix

- [ ] 2.1 Update operator-chain checking so early expected-result subtyping is skipped when the operator result type is still an unresolved unification variable.
- [ ] 2.2 Keep the existing post-operand result check so constrained operator results can elaborate wanted constraints after operands contribute type information.
- [ ] 2.3 Avoid special-casing the `$` operator; apply the fix to the general operator result-checking path.

## 3. Verification

- [ ] 3.1 Re-run the new checking fixture and verify both direct and `$` forms pass without `CannotUnify`.
- [ ] 3.2 Re-run `just t checking 1774448340_operator_result_ordering` to ensure higher-rank operator result ordering remains intact.
- [ ] 3.3 Run `cargo check -p checking --tests`.
