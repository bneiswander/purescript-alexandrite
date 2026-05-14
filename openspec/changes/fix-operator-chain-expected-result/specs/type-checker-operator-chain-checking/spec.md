## ADDED Requirements

### Requirement: Checked Operator Chains Use Simple Expected Result Type
When checking a term operator chain against a simple expected result type, the type checker SHALL use the expected type to constrain each operator branch result before checking the branch operands. Expected result types with binders, constraints, or function arguments MUST be checked after operands to avoid premature skolemisation or dictionary placement.

#### Scenario: Flipped application pipeline with polymorphic natural transformation
- **WHEN** a term operator chain using flipped application is checked against an expected type and the flipped function operand contains polymorphic natural-transformation-shaped types whose result is determined by the chain result
- **THEN** the checker MUST propagate the expected result type before checking the flipped function operand and MUST NOT emit a false `CannotUnify` between `Run` and `Function` shapes.

#### Scenario: Invalid operator chain remains rejected
- **WHEN** a term operator chain is checked against an expected type that is incompatible with the operator branch result
- **THEN** the checker MUST still report the type mismatch.

#### Scenario: Higher-rank operator result remains accepted
- **WHEN** a term operator chain is checked against a higher-rank or constrained expected result type such as a lens type synonym
- **THEN** the checker MUST defer the operator branch result check until after operands have been checked and MUST NOT introduce a false higher-rank unification diagnostic.

### Requirement: Operator Chain Inference Remains Unchanged
When inferring a term operator chain without an expected type, the type checker SHALL preserve existing inference behavior and MUST NOT require an expected result type.

#### Scenario: Inferred operator chain
- **WHEN** a term operator chain is inferred in a context that does not provide an expected type
- **THEN** the checker MUST infer the result using the operator signature and operand types as before.
