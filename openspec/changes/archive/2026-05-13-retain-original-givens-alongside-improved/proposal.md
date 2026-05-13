## Why

When the type checker elaborates given constraints via functional-dependency improvement, it substitutes the resulting rigid variable substitutions into the original constraints to form "improved" givens. However, the body of a function also has access to the original explicit constraints. If an improved form unifies a rigid variable with a unification variable, any visible type application (VTA) in the body against that rigid name would fail, because the lookup mechanism expects the name to map to a `Type::Rigid`, not a `Type::Unification`.

## What Changes

- When `analyse_equation_set` applies a functional-dependency substitution to a signature's constraints, arguments, and result, it now retains the original constraints alongside the improved ones.
- The original constraints are pushed after the improved constraints, preserving ordering while ensuring explicit binders remain resolvable via `with_implicit`.
- This fixes `NoInstanceFound` / `NoVisibleTypeVariable` errors when using VTA (e.g., `Proxy @x`) inside a function body where `x` appears in an `Eval`-like constraint that resolves to a unification variable via functional dependency.

## Capabilities

### New Capabilities
- `eval-boolean-given-vta`: Allows visible type applications against variables resolved from explicit given constraints that also participate in functional-dependency chains.

### Modified Capabilities
- (none)

## Impact

- **Affected code**: `compiler-core/checking/src/source/terms/equations.rs` — `analyse_equation_set`
- **New tests**: `tests-integration/fixtures/checking/1778701560_eval_given_visible_type_application/`