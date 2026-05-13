## ADDED Requirements

### Requirement: Non-literal Reflectable arguments defer to user-defined instances
The constraint solver SHALL NOT return `Apart` for a `Reflectable` constraint when the first argument is a non-literal type constructor that is not a unification variable. Instead, the solver SHALL return `None` (no compiler solver applies), allowing normal instance chain search to find user-defined `Reflectable` instances.

#### Scenario: Recursive Reflectable over type-level list
- **WHEN** a `Reflectable (MkTransitCoreTL transitions) TransitCore` constraint is solved, where `transitions` is a type-level list built from `MkMatchTL` / `MkReturnTL` type constructors
- **THEN** the constraint solver SHALL find user-defined `Reflectable` instances for `MkMatchTL` and `MkReturnTL` and their recursive subgoals, resolving the constraint successfully

#### Scenario: Recursive Reflectable instance chain with multiple levels
- **WHEN** a `Reflectable` instance has subgoals that are also `Reflectable` constraints over user-defined type constructors
- **THEN** the constraint solver SHALL recursively search instance chains for each subgoal, and SHALL NOT prematurely reject the candidate as `Apart`

#### Scenario: Reflectable with no applicable user instance still reports NoInstanceFound
- **WHEN** a `Reflectable v t` constraint has a non-literal `v` and no user-defined `Reflectable` instance exists for `v`
- **THEN** the constraint solver SHALL eventually report `NoInstanceFound` after exhausting all instance chain searches