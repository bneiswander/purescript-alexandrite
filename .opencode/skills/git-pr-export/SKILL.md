---
name: git-pr-export
description: Export clean upstream PR branches from interleaved dev commits (skipping openspec/opencode)
allowed-tools: Bash(git:*), Bash(rg:*), Bash(cargo:*)
---

# Git PR Export Workflow (Fork + Upstream, Interleaved Commits)

Use this workflow when you want to keep committing `openspec/**` and `.opencode/**` (and other local tooling) to your fork, but create upstream PR branches that contain only functional changes.

## Remotes / Branch Roles

- `origin`: your fork (default push)
- `upstream`: upstream repo (source of truth)

- `dev/<topic>`: working branch on your fork; may include `openspec/**`, `.opencode/**`, and other local tooling commits
- `pr/<topic>`: clean export branch created from `upstream/main`; contains only functional commits (new SHAs)

## Hard Rule (Makes Everything Easy)

**Never mix `openspec/**` or `.opencode/**` (or other opencode-related files) with functional changes in the same commit.**

Interleaving commits is fine. Mixing concerns inside a commit is what forces manual surgery during export.

## Commit Tagging Convention

Prefix commit subjects so we can auto-select functional commits.

Functional (PR-eligible):
- `feat:` `fix:` `refactor:` `test:` `docs:`

Tooling-only (must NOT go upstream):
- `openspec:`
- `opencode:`

Examples:
- `feat: add subdir roots for git packages`
- `openspec: archive change artifacts`
- `opencode: add local skill`

## Daily Work (Dev Branch)

1. Start from upstream:

```bash
git fetch upstream
git switch -c dev/<topic> upstream/main
```

2. Make incremental commits, but stage by path:

```bash
git add <functional paths>
git commit -m "feat: ..."

git add openspec .opencode
git commit -m "openspec: ..."   # or "opencode: ..."
```

3. Push dev branch to your fork:

```bash
git push -u origin dev/<topic>
```

## Export: Create a Clean PR Branch From Interleaved Commits

1. Create PR branch from upstream:

```bash
git fetch upstream
git switch -c pr/<topic> upstream/main
```

2. Cherry-pick all non-tooling commits from `dev/<topic>`:

```bash
git cherry-pick $(
  git rev-list --reverse --no-merges \
    --invert-grep --grep='^(openspec|opencode):' \
    upstream/main..dev/<topic>
)
```

Notes:
- This intentionally creates new commit SHAs on the PR branch.
- This relies on commit-message tagging; mixed commits still need manual handling.

3. Verify the PR diff contains no tooling files:

```bash
git diff --name-only upstream/main...HEAD | rg '^(openspec/|\.opencode/|\.opencode$|opencode)'
```

Expect: no output.

4. Run tests (pick relevant crate/test suite) and push:

```bash
git push -u origin pr/<topic>
```

## Updating an Existing PR

When you add more functional commits to `dev/<topic>`:

1. Switch to `pr/<topic>`
2. Cherry-pick the new functional commits (same command as export, or cherry-pick specific SHAs)
3. Push to update the PR

## If You Accidentally Mixed Functional + Tooling in One Commit

You cannot automatically “filter” a mixed commit by message. Export it manually:

```bash
git cherry-pick -n <mixed_sha>   # apply without committing
git reset                        # unstage everything
git add <functional paths only>
git commit -m "<functional message>"

# discard tooling changes from the PR branch
git restore --staged --worktree openspec .opencode
```

Then continue exporting the remaining commits.

## Troubleshooting

- Cherry-pick conflicts: resolve, then `git cherry-pick --continue`.
- Wrong base: ensure PR branches start at `upstream/main` (not `origin/main`).
- Sanity check ahead/behind:

```bash
git status
git rev-list --count upstream/main..HEAD
```
