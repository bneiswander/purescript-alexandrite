## MODIFIED Requirements

### Requirement: Non-final instance-chain candidates do not force unresolved inputs
The analyzer SHALL NOT commit to a non-final instance-chain candidate when that candidate would solve or constrain unresolved unification variables in non-determined input positions. The analyzer SHALL only collect blocking unification variables from arguments whose match result is unknown (stuck or skolem); arguments that already matched or were definitively apart are excluded from the blocking check.

#### Scenario: Equality helper must not force compare result before compare solves
- **WHEN** an instance-chain candidate for a helper class such as `Eval (Eq LT ord) isLess` can match by unifying an unresolved input variable `ord`
- **THEN** the analyzer MUST defer that non-final candidate until the input variable is solved by other constraints or givens

#### Scenario: Example function instance with resolved match does not block
- **WHEN** a wanted constraint `Example (arg -> ?m Unit) arg Aff` is matched against an `Example` instance chain where `exampleFunc` matches the first and third arguments (with `m ~ Aff` unified) but the lambda result contains an unresolved variable `?m`
- **THEN** the analyzer MUST NOT block on `?m` because the match result for that argument is already `Apart`, and MUST proceed to unify `m ~ Aff` from the third argument
