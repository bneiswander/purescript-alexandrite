## ADDED Requirements

### Requirement: Rigid variables in instance row patterns match any row
The analyzer SHALL match an open row pattern with a rigid variable tail (from `freshen_instance_signature`) against any given row type, including closed rows. Rigid variables in instance `matchable` types act as pattern variables and must not be treated as `Apart`.

#### Scenario: Open row pattern matches closed record row
- **WHEN** an instance has head `Class { | r }` and the wanted constraint has a closed row type like `{ a :: Int, b :: String }`
- **THEN** the analyzer MUST return `Match` (not `Apart`) and bind the rigid tail variable to the concrete row

#### Scenario: Open row pattern with rigid tail absorbs additional given fields
- **WHEN** an instance has head `Class { | r }` and the wanted constraint has an open row with rigid tail like `{ a :: Int | ?r }` matching against a given row with additional fields `{ a :: Int, b :: String }`
- **THEN** the analyzer MUST return `Match` and allow the rigid tail to absorb the extra fields

#### Scenario: Unification variable tails still defer via Stuck
- **WHEN** an instance has head `Class { | r }` where `r` is an unification variable (not a rigid variable)
- **THEN** the analyzer MUST return `Stuck` (not `Apart`) to defer the match until the unification variable is solved
