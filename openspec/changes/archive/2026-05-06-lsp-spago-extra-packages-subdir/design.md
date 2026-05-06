## Context

The LSP integrates with Spago projects by reading `spago.lock` and using it to discover source roots to load and index. Today, the lockfile integration assumes that package sources live directly under the package root paths it constructs (e.g. `.spago/p/<name>/<rev>/src`).

Spago supports `workspace.extraPackages` entries that can point at monorepo sub-packages via a `subdir` field. In that layout, sources live under an extra segment (e.g. `.spago/p/deku-core/<rev>/<subdir>/src`).

In `spago.lock`, extra packages are recorded under `workspace.extra_packages` (snake_case). Additionally, git entries in the top-level `packages` map can themselves carry an optional `subdir` field.

Because the LSP ignores `workspace.extra_packages` (and therefore may miss `subdir`) when building its list of source roots, it never loads those modules in cases where the `subdir` information is only available from the workspace extra package entry. Downstream, the resolver reports `InvalidImportStatement` because `queries.module_file(name)` is missing.

Constraints:
- Prefer minimal, additive changes in `compiler-lsp/spago`.
- Maintain existing behavior for projects that do not use `extraPackages`/`subdir`.
- Avoid hard failures for lockfiles that omit `extra_packages`.

## Goals / Non-Goals

**Goals:**
- Parse `workspace.extra_packages` from `spago.lock` and use `subdir` (when present) to construct additional candidate source roots.
- Support the common case: `extraPackages` with `git` source and `subdir`.
- Add regression tests in `compiler-lsp/spago` covering the subdir behavior.

**Non-Goals:**
- Changing resolver semantics or the error emitted when a module file is genuinely missing.
- Implementing `spago.yaml` parsing in the LSP (lockfile remains the source of truth).
- Perfectly modeling every possible Spago lockfile variant beyond what the LSP needs to find sources.

## Decisions

- **Model only the fields we need from `workspace.extra_packages`.**
  - Add an `extra_packages: HashMap<name, ExtraPackage>` to the lockfile `Workspace` model with `subdir: Option<PathBuf>` and `path: Option<PathBuf>`.
  - Use `#[serde(default)]` so older lockfiles continue to deserialize.

- **Additive source discovery: keep existing roots, plus subdir roots when available.**
  - In `Lockfile::sources()`, determine a package's `subdir` from:
    - the lock entry itself (git `packages.<name>.subdir`) when present, otherwise
    - the workspace extra package entry (`workspace.extra_packages.<name>.subdir`) when present.
  - If a `subdir` exists, append additional candidate roots that insert the subdir segment before `{src,test}`.
  - Keep existing non-subdir roots so packages without `subdir` are unaffected.

- **Let filesystem filtering handle non-existent candidates.**
  - The existing walker already filters paths by `canonicalize().ok()`. We can safely include extra candidates without needing repo-specific heuristics.

- **Testing at the `compiler-lsp/spago` level.**
  - Add a fixture lockfile with `workspace.extra_packages.<name>.subdir` and a matching `packages.<name>` git entry (`rev`).
  - Assert that computed roots include the subdir form.

## Risks / Trade-offs

- [Risk] Lockfile field naming differences (`extraPackages` vs `extra_packages`).
  → Mitigation: Deserialize `workspace.extra_packages` using the lockfile's actual JSON field name (`extra_packages`). Add `#[serde(default)]`.

- [Risk] Duplicate roots (subdir and non-subdir) for a package.
  → Mitigation: Either accept duplication (harmless) or dedupe in `sources()` using a set; prefer minimal change unless duplication causes measurable issues.

- [Risk] Local `extraPackages` may specify `path` semantics that differ from package entries.
  → Mitigation: Capture `path` in `ExtraPackage` and prefer it for local-with-subdir cases, but keep behavior best-effort.
