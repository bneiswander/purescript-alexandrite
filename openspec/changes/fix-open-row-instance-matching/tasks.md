## 1. Implement Fix

- [ ] 1.1 Add rigid variable check in `match_row_type` for `Additional => Open(wanted_tail)` case (line ~695)
- [ ] 1.2 Add rigid variable check in `match_row_type` for `Closed => Open(wanted_tail)` case (line ~709)

## 2. Verify

- [ ] 2.1 Run `cargo check -p compiler-core --tests` to verify no regressions
- [ ] 2.2 Run integration tests: `just t checking`
- [ ] 2.3 Create test fixture reproducing the user's `HFoldlWithIndex` scenario with `else` instances and open row patterns
