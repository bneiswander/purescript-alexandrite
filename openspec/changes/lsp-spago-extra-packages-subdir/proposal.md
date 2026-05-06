## Why

The LSP currently fails to resolve imports for Spago `workspace.extraPackages` entries that use `subdir` (e.g. monorepos), leading to false diagnostics like `InvalidImportStatement Cannot import module 'Deku.Core'` even though `spago build` succeeds.

## What Changes

- Teach the LSP's `spago.lock` source discovery to account for `subdir` information in the lockfile when constructing `.spago/p/...` source roots.
  - `spago.yaml` uses `workspace.extraPackages.<pkg>.subdir`.
  - `spago.lock` records this under `workspace.extra_packages` and (for git lock entries) may also include a `subdir` field in `packages.<pkg>`.
- Ensure this works for `git`-sourced `extraPackages` (the common case) and does not regress existing package resolution.
- Add regression coverage in `compiler-lsp/spago` for subdir extra packages.

## Capabilities

### New Capabilities
- `spago-extra-packages-subdir-sources`: Include `extraPackages` `subdir` sources in lockfile-based LSP module discovery.

### Modified Capabilities

## Impact

- Affected code: `compiler-lsp/spago` lockfile parsing and source root construction.
- User impact: removes incorrect import/module diagnostics and enables navigation/analysis for monorepo subpackages.
