## ADDED Requirements

### Requirement: Determined row variables do not cause false unification errors
The analyzer SHALL accept programs where an explicit signature quantifies a row variable that is determined by an in-scope functional-dependency constraint, and the checked expression requires a concrete row shape consistent with that constraint.

#### Scenario: Explicit constrained row result is used through a required field
- **WHEN** a value has an explicit signature with a row-polymorphic result and a given constraint whose functional dependency determines that row
- **THEN** checking the value body MUST NOT emit `CannotUnify` solely because the rigid row variable is compared with the determined concrete row shape

### Requirement: Invalid row mismatches are still reported
The analyzer SHALL continue to report row unification or subtype diagnostics when a concrete row requirement is not justified by a determining constraint or contradicts the determined row.

#### Scenario: Required field is not supported by determining constraint
- **WHEN** a value body requires a field that is not part of the row determined by its functional-dependency givens
- **THEN** checking the value MUST report an appropriate type error rather than accepting the program
