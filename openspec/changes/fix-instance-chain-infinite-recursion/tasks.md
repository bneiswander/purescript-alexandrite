## 1. Solver Implementation

- [x] 1.1 Add `ProbeKey`, `ProbeTypeKey`, and `ProbeArgKey` structs to `constraint.rs` for structural constraint matching.
- [x] 1.2 Add `probe_key` and `probe_type_key` helper functions.
- [x] 1.3 Add `candidate_constraint_probes: Vec<Vec<ProbeKey>>` and `candidate_constraint_probe_cache: FxHashMap<Vec<ProbeKey>, bool>` to `CheckState`.
- [x] 1.4 Add depth guard (`len() >= 2`) to `candidate_constraints_are_unsatisfiable` in `matching.rs`.
- [x] 1.5 Add structural probe key matching to detect equivalent constraint sets across probes.
- [x] 1.6 Add memoization cache lookup and insertion to `candidate_constraints_are_unsatisfiable`.

## 2. Hover Restoration

- [x] 2.1 Add `hover_checked_type` helper that renders a `TypeId` using `Pretty`.
- [x] 2.2 Update `hover_binder` to fall back to checked types for non-constructor binders.
- [x] 2.3 Update `hover_expression` to fall back to checked types for non-literal/non-constructor expressions.
- [x] 2.4 Update `hover_type` to fall back to checked types for non-constructor types.
- [x] 2.5 Update `hover_let` to return checked types via the fallback.
- [x] 2.6 Update `hover_pun` to return checked types via the fallback.
- [x] 2.7 Update `AnnotationSyntaxRange::from_ptr` to not extract syntax for equation annotations (syntax is not available there).

## 3. Verification

- [x] 3.1 Run `cargo check -p purescript-analyzer --tests`.
- [x] 3.2 Run `just t lsp` to verify no regressions in LSP snapshots.
- [x] 3.3 Verify release build completes and installs successfully.
