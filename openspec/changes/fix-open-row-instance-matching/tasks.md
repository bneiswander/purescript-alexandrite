## 1. Implement Fix

- [x] 1.1 Add rigid variable check in `match_row_type` for `Additional => Open(wanted_tail)` case (line ~695)
- [x] 1.2 Add rigid variable check in `match_row_type` for `Closed => Open(wanted_tail)` case (line ~709)
- [x] 1.3 Add rigid variable check in `match_row_type` for `Open(given_tail)` case (line ~705)
- [x] 1.4 Add rigid variable check in `match_cons` to defer to instance matching (prim_row.rs)
- [x] 1.5 Add rigid variable check in `match_row_to_list` to defer to instance matching (prim_row_list.rs)

## 2. Verify

- [x] 2.1 Run `cargo check -p compiler-core --tests` to verify no regressions
- [x] 2.2 Run integration tests: `just t checking`
- [x] 2.3 Create test fixture reproducing the user's `HFoldlWithIndex` scenario with `else` instances and open row patterns
