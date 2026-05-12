## 1. Regression Coverage

- [ ] 1.1 Add a focused checking fixture that models a row-polymorphic result determined by a functional-dependency constraint.
- [ ] 1.2 Confirm the fixture reproduces the current false `CannotUnify` diagnostic before applying the checker fix.

## 2. Checker Implementation

- [ ] 2.1 Trace the explicit-signature checking path to identify where the determined rigid row is unified before given-constraint improvement is available.
- [ ] 2.2 Update the generic checker or constraint-solving path so functional-dependency givens can justify determined row variables before reporting direct row unification failure.
- [ ] 2.3 Ensure unsupported or contradictory row requirements still produce type errors.

## 3. Verification

- [ ] 3.1 Update the regression snapshot to show the valid fixture type checks without `CannotUnify`.
- [ ] 3.2 Run `just t checking <new-fixture-filter>` for the focused regression.
- [ ] 3.3 Run broader relevant checking tests or `cargo check -p checking --tests` to catch solver regressions.
