## Why

The analyzer reported false `CannotUnify` errors when polykinded row type classes (classes polymorphic over `forall k. Row k -> ...`) were used with higher-kinded effect rows like `Row (Type -> Type)`. The root causes were:

1. Functional-dependency elaboration indexing into canonical class arguments without accounting for leading kind binders.
2. The `Prim.Row.Union` compiler solver creating open row tails with an incorrectly hardcoded kind.

## What Changes

- Fixed `elaborate_given_substitution` in `compiler-core/checking/src/core/constraint/elaborate.rs` to offset functional-dependency determined positions by `class.kind_binders.len()` when indexing canonical constraint arguments.
- Fixed `get_functional_dependencies` and `can_determine_stuck` in `compiler-core/checking/src/core/constraint/matching.rs` to use the checked class metadata (which already has all canonical positions correct) instead of raw `ClassIr` data.
- Fixed `match_union` in `compiler-core/checking/src/core/constraint/compiler/prim_row.rs` to infer the row element kind from constraint arguments and use it when constructing residual open-row tails.
- Added a regression fixture reproducing the `SafeUnion` pattern through `Eval`/`RowList`/`FromRow`/`ToRow` (mimicking `Type.Eval.Row.Util.SafeUnion`).
- Updated the existing `prim_row_open` snapshot to reflect new diagnostic variable numbering.

## Capabilities

### New Capabilities

- `type-checker-row-fundep-diagnostics`: Already exists from the previous `fix-row-fundep-analyzer-error` change. The existing requirements already describe the expected behavior. This change extends the capability to cover polykinded classes (classes with `forall k.` kind binders) which was the remaining gap.

### Modified Capabilities

- `type-checker-row-fundep-diagnostics`: Extends the existing requirement to cover polykinded row classes, not just monomorphic ones. The canonical form of polykinded class constraints includes kind arguments before type arguments, and the analyzer must account for this when computing functional dependencies.

## Impact

- `compiler-core/checking/src/core/constraint/elaborate.rs`: Fundep position indexing fix.
- `compiler-core/checking/src/core/constraint/matching.rs`: Class metadata lookup and position calculation fix.
- `compiler-core/checking/src/core/constraint/compiler/prim_row.rs`: Open-row tail kind inference fix.
- Affects any user code using polykinded row type classes with higher-kinded rows (e.g., `SafeUnion`, `RowApply`-style row composition, `Run`-based effect systems).