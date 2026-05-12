## Context

The checker validates explicit value signatures by skolemising their quantified variables, collecting givens from signature constraints, and then checking each equation against the skolemised type. Constraint solving already supports functional dependencies and given-constraint improvement, but the analyzer can still emit a direct `CannotUnify` when a rigid row from an explicit signature is compared with a concrete row before the relevant functional-dependency evidence has been applied.

The motivating case is a valid helper whose result row `c` is determined by `StripColumns Columns c`; downstream query expressions require fields such as `account_guid`, and the analyzer reports that rigid `c` cannot unify with a row containing those fields even though the reference compiler accepts the module.

## Goals / Non-Goals

**Goals:**

- Accept valid programs where a given functional-dependency constraint determines a row-polymorphic signature variable used in expression checking.
- Keep row mismatch diagnostics for genuinely invalid programs.
- Add a compact regression fixture that fails before the checker fix and passes after it.

**Non-Goals:**

- Rework the full constraint solver architecture.
- Change PureScript source syntax, emitted interfaces, or public CLI behavior.
- Add project-specific knowledge of `Yoga.SQLite.Schema.StripColumns`.

## Decisions

- Use a reduced fixture rather than the full downstream module.
  Rationale: the bug is in generic checker behavior around functional dependencies and rows, so a local class and small value definition will make the regression stable and auditable.
  Alternative considered: import the real package APIs. That would add dependency noise and obscure the checker rule being fixed.

- Fix the generic checker path that handles determined row variables from given constraints.
  Rationale: the behavior should apply to any class with a functional dependency that determines a row, not only to `StripColumns`-like classes.
  Alternative considered: suppress `CannotUnify` for rigid rows during checking. That would hide real type errors and weaken diagnostics.

- Preserve existing constraint-solving residual behavior.
  Rationale: unsolved or ambiguous constraints should still surface as diagnostics rather than being accepted by default.
  Alternative considered: eagerly force all stuck constraints before row unification. That risks changing unrelated inference behavior.

## Risks / Trade-offs

- Over-broad improvement of skolem variables could accept invalid programs -> Mitigate by limiting changes to constraints whose functional dependencies actually determine the row and by retaining existing mismatch tests.
- A minimal fixture might miss the exact `Run`/query pipeline interaction -> Mitigate by modeling the same shape: explicit constrained signature, row result, and record-field use requiring a determined row.
- Solver ordering changes can affect snapshots -> Mitigate by running focused and broader checking tests after implementation.
