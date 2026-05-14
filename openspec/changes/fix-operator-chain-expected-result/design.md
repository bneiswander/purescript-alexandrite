## Context

Term operator chains are lowered as `ExpressionKind::OperatorChain`, bracketed into binary operator trees, then checked by `compiler-core/checking/src/source/operator.rs`. In checking mode, the surrounding expected type is known, but `traverse_operator_branch` currently checks both operands against the operator argument types before constraining the branch result against that expected type.

This ordering is too weak for operators whose operand types contain result variables that should be informed by the expected type. A concrete example is `# :: a -> (a -> b) -> b` applied to polymorphic natural transformations such as `Run r1 ~> Run r2`. If `b` remains unconstrained while the right operand is checked, the checker can attempt to unify a `Run ...` value with a `Function ...` shape and emit a false `CannotUnify` diagnostic.

## Goals / Non-Goals

**Goals:**
- In checking mode, use simple surrounding expected result types to constrain an operator branch result before checking its operands.
- Preserve the original late result check for higher-rank, constrained, or function-shaped expected results.
- Keep the existing operator bracketing, associativity, and inference behavior unchanged.
- Cover the regression with a minimal type checker integration fixture.

**Non-Goals:**
- Do not change parser precedence, lowering representation, or operator bracketing.
- Do not add special cases for `#`, `>>>`, `Run`, or `Interpreter`.
- Do not change diagnostic formatting or LSP publishing behavior.

## Decisions

- Move expected-result subtyping earlier in `traverse_operator_branch` for `OperatorKindMode::Check` when the expected result has no binders, constraints, or function arguments.

  Rationale: the expected type is already part of checking mode and applies to the whole operator branch. Applying `result_type <: expected_type` before operand traversal lets unification variables in simple operator signatures be solved before operand checks depend on them. Higher-rank and constrained expected results must remain late because early skolemisation can make valid programs such as lens construction appear invalid.

  Alternative considered: add a dedicated special case for flipped application. Rejected because the failure is an ordering issue in generic operator checking, not a property of one operator.

- Leave `OperatorKindMode::Infer` unchanged.

  Rationale: inference mode has no external expected result to propagate. Changing it would risk altering inferred types for unrelated operator chains.

- Use a local regression fixture with stubbed PureScript definitions.

  Rationale: the bug depends on type shapes, not runtime implementations or full package-set dependencies. A focused fixture is easier to audit and less brittle than importing the original project modules.

## Risks / Trade-offs

- Earlier subtyping can produce a diagnostic before operand diagnostics in malformed programs -> Mitigation: the same constraint already exists today, only later; limit early subtyping to simple expected results and review snapshots for diagnostic order changes in affected tests.
- Solving result variables earlier may affect ambiguous invalid operator chains -> Mitigation: run the new focused fixture plus the checking suite subset, and inspect any snapshot diffs before accepting.
- The minimal fixture may under-approximate the original project case -> Mitigation: model both `#` and composed `Interpreter` natural transformations, and keep the implementation generic.
