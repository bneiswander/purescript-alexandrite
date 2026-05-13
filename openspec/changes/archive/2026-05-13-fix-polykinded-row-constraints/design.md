## Context

The analyzer had two separate bugs affecting polykinded row type classes used with higher-kinded effect rows (e.g., `Row (Type -> Type)`).

**Bug 1 — Fundep canonical argument indexing:** Functional-dependency positions in class declarations refer to type parameters in declaration order. However, the canonical form of class constraints interleaves kind arguments (from `forall k.`) before type arguments. For a class like `class SafeUnion :: forall k. Row k -> Row k -> Row k -> Constraint`, the canonical arguments are `[k, a, b, c]` but the fundep `a b -> c` indexes positions `0, 1, 2`. Without an offset, the checker would substitute into `k` instead of `c`.

**Bug 2 — Open row tail kind:** The `Prim.Row.Union` compiler solver has a branch for open left rows `( a :: A | t )`. When the right side and output are not both closed, it creates a fresh unification variable as the new tail and emits a residual `Union` constraint. This fresh variable was created with kind `Row Type` (from `context.prim.row_type`), which is correct for `Row Type` rows but wrong for `Row (Type -> Type)` rows.

**Constraint:** Both fixes must work without knowing the specific row kind at solver invocation time, since the kind is determined by which instance matches and is discovered during constraint solving.

## Goals / Non-Goals

**Goals:**
- Fix fundep elaboration to account for kind binders in canonical constraint arguments.
- Fix open-row tail creation to infer the correct row kind from the constraint arguments.
- Preserve all existing behavior for monomorphic row classes (e.g., `Prim.Row.Union` with `Row Type`).

**Non-Goals:**
- Do not change the `Prim.Row.*` class declarations or their functional dependencies.
- Do not add new compiler solver rules — only fix the two existing paths.

## Decisions

### Decision 1: Use checked class metadata for fundep positions in `matching.rs`

**Choice:** Instead of extracting fundep positions from the raw `ClassIr` (which uses declaration-order type-parameter positions), look up the `CheckedClass` via `toolkit::lookup_file_class` and use its metadata directly.

**Rationale:** `CheckedClass` is the resolved representation. In `matching.rs`, the `results` vector only contains type arguments (kind arguments are skipped by the `if let (KindOrType::Type(...))` guard). Since fundep positions refer to type parameters and `results` indices also track type arguments only, no offset is needed here — positions match directly. In `elaborate.rs`, the situation differs: positions index the full `constraint.arguments` which includes kind arguments, so `kind_binders.len()` must be added. This two-location model is what was actually implemented.

### Decision 2: Add offset in `elaborate.rs` only, not in `matching.rs`

**Choice:** Offset by `kind_binders.len()` in `elaborate_given_substitution` (which indexes into the full `constraint.arguments` that includes kind arguments). Leave `matching.rs` with zero offset because it already filters arguments to type-only before computing match positions.

**Rationale:** In `matching.rs`, the `results` vector only contains type arguments (the code explicitly skips kind arguments with the `if let (KindOrType::Type(...))` guard). Since match indices are already over this filtered list, the fundep positions (also over type parameters) match directly without offset. In `elaborate.rs`, positions index the full `constraint.arguments` which includes kind arguments, so the offset is required.

### Decision 3: Infer row kind from constraint arguments in `match_union`

**Choice:** Call `infer_row_constraint_kind` (already defined in the same file) on the three constraint arguments to determine the row element kind, then construct `context.prim.row` applied to that kind as the fresh tail variable kind.

**Rationale:** `infer_row_constraint_kind` already exists and correctly identifies the row element kind by looking at the arguments (preferring any already-known row argument). Reusing it means the logic is centralized and consistent.

**Alternative considered:** Pass the kind explicitly from the caller — rejected because the caller (`match_union`) is called from the compiler solver dispatcher and doesn't have convenient access to the kind.

## Risks / Trade-offs

- **[Risk]** The `infer_row_constraint_kind` approach in `match_union` falls back to `Row Type` if no argument has a detectable row kind. This could mask missing kind information in some cases. → **Mitigation:** The existing behavior was already falling back to `Row Type`, so this is no worse. A future improvement could report an ambiguity error instead.
- **[Trade-off]** The fundep offset in `elaborate.rs` is applied speculatively (before constraint solving). Incorrect offsets could cause wrong substitutions with no immediate error. → **Mitigation:** The test suite covers fundep scenarios with both monomorphic and polykinded classes. The new regression fixture specifically targets the polykinded case.
- **[Risk]** `toolkit::lookup_file_class` in `matching.rs` requires mutating access to `CheckState` (for zonk/normalization inside lookup). This was already the case since `can_determine_stuck` was already doing file lookups. No new state mutation is introduced.