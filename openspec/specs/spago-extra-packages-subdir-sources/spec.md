## Purpose

Define source discovery for Spago lockfile extra packages that are stored in repository subdirectories.

## Requirements

### Requirement: Lockfile extra package subdir sources are discoverable
When a Spago project uses an extra package with `subdir` (configured via `spago.yaml` `workspace.extraPackages.<package>.subdir`), the LSP lockfile integration SHALL use `spago.lock` to discover sources under that subdir.

If `subdir` is present in both places, the LSP SHALL prefer `packages.<name>.subdir` over `workspace.extra_packages.<name>.subdir`.

#### Scenario: Git extra package with subdir
- **WHEN** `spago.lock` contains a `deku-core` git package entry with some `rev` and `subdir` information available via either `packages.deku-core.subdir` or `workspace.extra_packages.deku-core.subdir` (or both)
- **THEN** the source discovery includes candidates under `.spago/p/deku-core/<rev>/<subdir>/src` and `.spago/p/deku-core/<rev>/<subdir>/test`

#### Scenario: No subdir extra package
- **WHEN** `spago.lock` contains an entry in `packages` for `foo` but there is no `workspace.extra_packages.foo.subdir`
- **THEN** source discovery behaves as before (e.g. `.spago/p/foo/<rev>/src` for git packages)

#### Scenario: Lockfile without extra_packages
- **WHEN** `spago.lock` omits `workspace.extra_packages` (e.g. older format or no extra packages)
- **THEN** lockfile parsing and source discovery continues to work (no crash, no hard error)
