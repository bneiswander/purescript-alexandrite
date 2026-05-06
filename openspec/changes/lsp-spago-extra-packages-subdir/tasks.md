## 1. Lockfile Model Updates

- [ ] 1.1 Extend `compiler-lsp/spago/src/lockfile.rs` workspace model to deserialize `workspace.extra_packages` (serde defaulted)
- [ ] 1.2 Add an `ExtraPackage` model capturing `subdir` (and `path` for local extras)

## 2. Source Root Discovery

- [ ] 2.1 Update `Lockfile::sources()` to look up `extra_packages` by name while iterating `packages`
- [ ] 2.2 Determine a package `subdir` from the git lock entry (`packages.<name>.subdir`) when present, otherwise from `workspace.extra_packages.<name>.subdir`
- [ ] 2.3 When `subdir` is present, add additional candidate roots that insert `<subdir>` before `{src,test}` for git packages
- [ ] 2.4 Ensure existing non-subdir roots are preserved (no behavior change for packages without `subdir`)
- [ ] 2.5 Optionally dedupe roots if needed to avoid redundant scanning (only if duplication is observed to cause issues)

## 3. Tests

- [ ] 3.1 Add a `compiler-lsp/spago` test fixture `spago.lock` that includes a git package with `rev` plus `subdir` recorded via either `workspace.extra_packages.<name>.subdir` or `packages.<name>.subdir`
- [ ] 3.2 Add/extend a unit test asserting `Lockfile::sources()` includes `.spago/p/<name>/<rev>/<subdir>/{src,test}`
- [ ] 3.3 Add/extend a unit test for precedence when both `packages.<name>.subdir` and `workspace.extra_packages.<name>.subdir` are present (prefer `packages`)
- [ ] 3.4 Ensure tests cover absence of `extra_packages` (deserialization default) to avoid regressions

## 4. Validation

- [ ] 4.1 Run LSP against a real project using `extraPackages` + `subdir` (e.g. `deku-core`) and verify `InvalidImportStatement` for those modules disappears
- [ ] 4.2 Confirm module navigation / indexing works for subdir packages (module file mapping present)
