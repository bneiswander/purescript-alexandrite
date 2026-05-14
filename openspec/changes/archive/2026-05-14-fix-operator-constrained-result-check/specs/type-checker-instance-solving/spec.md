## ADDED Requirements

### Requirement: Operator applications elaborate constrained monadic results
The analyzer SHALL check operator applications consistently with equivalent direct function applications when the operator result comes from a constrained polymorphic function. Expected result guidance MUST NOT cause a constrained result such as `MonadEffect m => m a` to be unified directly with a concrete monad type before wanted constraints can be elaborated and solved.

#### Scenario: Dollar application solves Run MonadEffect wanted
- **WHEN** a definition with signature `Run (EFFECT r) Env` uses `$` to apply a function of type `forall a m. a -> MonadEffect m => m (Env a)` to an argument
- **THEN** the analyzer MUST solve the wanted `MonadEffect (Run (EFFECT r))` through the available `Run` instance and MUST NOT emit `CannotUnify` between the constrained monadic result and `Run (EFFECT r) Env`

#### Scenario: Direct and operator applications agree
- **WHEN** the same constrained polymorphic function call is written once with direct application and once with `$`
- **THEN** the analyzer MUST accept both forms or reject both forms for the same semantic reason, rather than accepting the direct form while reporting a unification diagnostic only for the operator form
