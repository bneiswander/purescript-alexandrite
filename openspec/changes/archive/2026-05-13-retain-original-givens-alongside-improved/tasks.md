## 1. Implementation

- [x] 1.1 Modify `analyse_equation_set` in `equations.rs` to retain original givens alongside improved givens
- [x] 1.2 Add regression test `1778701560_eval_given_visible_type_application` in `tests-integration/fixtures/checking/`

## 2. Verification

- [x] 2.1 Run `just t checking 1778701560` — regression test passes
- [x] 2.2 Run `cargo check -p checking --tests` — no type errors