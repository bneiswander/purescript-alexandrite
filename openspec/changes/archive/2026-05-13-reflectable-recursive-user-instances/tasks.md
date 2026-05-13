## 1. Compiler Fix

- [ ] 1.1 Change `Ok(Some(MatchInstance::Apart))` → `Ok(None)` in `prim_reflectable.rs` for non-literal, non-unification first arguments
- [ ] 1.2 Verify the change compiles cleanly (`cargo check -p checking`)

## 2. Testing

- [ ] 2.1 Add regression fixture `tests-integration/fixtures/checking/1778704560_reflectable_recursive_user_instances/` with recursive `Reflectable` instances over user-defined type-level list
- [ ] 2.2 Run `just t checking 1778704560_reflectable_recursive_user_instances` and accept snapshot
- [ ] 2.3 Run `just t checking 1772442600_prim_reflectable` to confirm existing behavior unchanged
- [ ] 2.4 Run full checking test suite (`just t checking`) to ensure no regressions

## 3. Documentation

- [ ] 3.1 Create OpenSpec proposal at `openspec/changes/reflectable-recursive-user-instances/proposal.md`
- [ ] 3.2 Create OpenSpec design at `openspec/changes/reflectable-recursive-user-instances/design.md`
- [ ] 3.3 Create OpenSpec spec at `openspec/changes/reflectable-recursive-user-instances/specs/reflectable-recursive-user-instances/spec.md`
- [ ] 3.4 Update delta spec at `openspec/changes/reflectable-recursive-user-instances/specs/type-checker-instance-solving/spec.md`
- [ ] 3.5 Create OpenSpec tasks at `openspec/changes/reflectable-recursive-user-instances/tasks.md`