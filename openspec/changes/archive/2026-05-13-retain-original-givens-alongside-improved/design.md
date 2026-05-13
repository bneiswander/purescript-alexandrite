## Context

When a function signature declares explicit constraints that participate in functional dependencies, the type checker elaborates those constraints to produce a substitution mapping rigid variable names to concrete or unification types. This elaboration is used to substitute the rigid names in the signature's arguments and result type.

For example, given `test :: forall x. IsBoolean x => Eval (NotEq False True) x => Boolean`, the constraint `Eval (NotEq False True) x` resolves via FD improvement to `IsBoolean True`, producing a substitution `x ↦ True`. The checker then substitutes `x` in the result type.

The problem occurs when the body of the function also references the rigid variable `x` — via a visible type application like `Proxy @x`. The substitution pipeline that maps `x` to `True` replaces the original rigid bindings with unification types. When `with_implicit` later resolves the name `x` for the VTA, it finds a `Type::Unification` rather than the expected `Type::Rigid`, causing the invariant violation in `bind_forall_substitution` and `bind_implicit_substitution`.

## Goals / Non-Goals

**Goals:**
- Fix visible type application (VTA) failures against rigid variables that appear in both explicit given constraints and Eval-like class constraints with functional dependencies.
- Preserve correct argument and result substitution for signature processing.
- Ensure existing tests continue to pass.

**Non-Goals:**
- This is not a general overhaul of the constraint solver or FD improvement mechanism.
- It does not change how unification variables are created or managed.
- It does not affect constraint solving outside of signature processing.

## Decisions

**1. Extend original givens rather than replace them**

When `elaborate_given_substitution` produces a non-empty substitution, the improved constraints (with rigid names replaced) are followed by the original constraints. This means both the improved form (used for argument/result substitution) and the original form (with rigid names intact) are available as given constraints.

This is the minimal fix: it adds two lines (`let original_constraints = constraints.clone();` and `constraints.extend(original_constraints);`) to the existing pipeline without changing any solver logic.

**Alternatives considered:**
- Skipping substitution entirely would lose the benefit of FD improvement for arguments and results.
- Creating separate "original" and "improved" constraint lists with careful ordering is equivalent to the chosen approach but more complex.
- Modifying `with_implicit` to accept unification types in bindings was rejected because it would weaken an important invariant.

**2. Original constraints follow improved constraints**

The improved constraints are processed first so that argument/result substitution uses the concrete type (e.g., `True`). The original constraints are then appended, ensuring they are present for body-level resolution via `with_implicit`.

## Risks / Trade-offs

- [Risk] Duplicate constraints in the given set could theoretically cause solver performance degradation. → Mitigation: The solver deduplicates canonical constraints internally, so this is not a practical concern.
- [Risk] The fix is narrowly scoped and may not address similar issues in other constraint-solver entrypoints. → Mitigation: Only the signature-processing path in `analyse_equation_set` was affected. Other entrypoints (e.g., class instance checking) have separate constraint sets and do not share this code path.