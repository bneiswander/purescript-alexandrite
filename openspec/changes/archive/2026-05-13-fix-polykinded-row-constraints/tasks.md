## 1. Diagnosis

- [x] 1.1 Confirmed the errors (`CannotUnify 'Type -> Type' with 'Type'`, `CannotUnify 'Row (Type -> Type)' with 'Row Type'`) are kind mismatches, not value unification failures.
- [x] 1.2 Identified two root causes: fundep position indexing missing kind-binder offset in `elaborate.rs`, and open-row solver creating tails with hardcoded `Row Type` kind in `prim_row.rs`.

## 2. Fundep Canonical Argument Fix

- [x] 2.1 Fixed `elaborate_given_substitution` in `compiler-core/checking/src/core/constraint/elaborate.rs` to add `type_argument_offset = class.kind_binders.len()` and offset determined positions before indexing `constraint.arguments`.
- [x] 2.2 Fixed `get_functional_dependencies` in `compiler-core/checking/src/core/constraint/matching.rs` to use `CheckedClass` via `toolkit::lookup_file_class` instead of raw `ClassIr` data. No offset needed in matching because `results` already filters to type-only arguments.

## 3. Open Row Solver Fix

- [x] 3.1 Fixed `match_union` in `compiler-core/checking/src/core/constraint/compiler/prim_row.rs` to call `infer_row_constraint_kind` and use the inferred kind (e.g. `Type`) to construct `context.prim.row` applied to that kind for the fresh tail variable.

## 4. Regression Coverage

- [x] 4.1 Added focused fixture `1778685420_polykinded_row_fundep_safe_union` that reproduces the `SafeUnion` pattern using `Eval`/`RowList`/`FromRow`/`ToRow` (mimicking `Type.Eval.Row.Util.SafeUnion`) with `composeFlippedThree = computation1 >+> computation2 >+> computation3`.
- [x] 4.2 Verified the fixture type-checks without `CannotUnify` diagnostics after the fix.
- [x] 4.3 Updated `1772440440_prim_row_open` snapshot (diagnostic variable numbering changed).
- [x] 4.4 Ran broader row-checking regressions: `1778682360_row_fundep_determined_result`, `1777091100_row_union_documentation`, `1777091040_row_union_open_duplicates`, `1772440320_prim_row`, `1772440440_prim_row_open` — all pass.

## 5. Verification

- [x] 5.1 `cargo check -p checking --tests` passes.
- [x] 5.2 LSP confirmed working on real user code: `purescript-analyzer 0.0.15` reports no errors on `composeThree`, `composeFlippedThree`, or `bindThree` with `SafeUnion`-based `Run` composition.
- [x] 5.3 OpenSpec verification: design clarified (Decision 1 updated to reflect two-location offset model), delta specs created for `type-checker-row-fundep-diagnostics`, tasks complete.