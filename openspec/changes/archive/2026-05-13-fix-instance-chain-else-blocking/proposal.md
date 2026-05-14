## Why

When matching a wanted constraint against an instance-chain candidate that is not the last in the chain, the solver checks whether any non-functional-dependency-determined arguments have unresolved unification variables. If so, it defers to later `else` candidates. However, `non_determined_unification_ids` collects blocking unification variables from **all** non-determined arguments, regardless of whether the match for those arguments was already known (`Match`) or definitively apart (`Apart`). This causes false `NoInstanceFound` errors when an earlier `else` candidate matches some arguments but has unification variables only in already-resolved argument positions.

A concrete example: `Test.Spec.Example (arg -> ?m Unit) arg Aff` should match the `exampleFunc` instance `Example (arg -> m Unit) arg m`. The solver selects `exampleFunc`, unifies `m ~ Aff` from the third argument, but then the non-final `else` candidate's `non_determined_unification_ids` check collects the unification variable `?m` from the lambda result position (which is not in a functional dependency determinant) and blocks, even though the match for that argument was already `Apart` from the other candidate. This cascades into spurious `NoInstanceFound` and `MonadThrow` errors.

## What Changes

- Fix `non_determined_unification_ids` in `matching.rs` to only collect blocking unification variables from arguments whose match result is actually unknown/stuck (`MatchType::Stuck` or `MatchType::Skolem`), skipping arguments that already matched or were definitively apart.
- Add regression fixture `1778720160_example_function_fundep` covering the `Example` instance-chain pattern.

## Capabilities

### Modified Capabilities
- `type-checker-instance-solving`: The requirement "Non-final instance-chain candidates do not force unresolved inputs" needs refinement — blocking should only apply to arguments whose match is unknown, not all non-determined arguments.

## Impact

- `compiler-core/checking/src/core/constraint/matching.rs` — `non_determined_unification_ids` function
- `tests-integration/fixtures/checking/` — new regression fixture
