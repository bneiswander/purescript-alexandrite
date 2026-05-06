---
name: manual-lsp-install
description: Ensure locally built purescript-analyzer is installed for manual editor testing
allowed-tools: Bash(cargo:*), Bash(which:*), Bash(command:*), Bash(ls:*)
---

# Manual LSP Install (For Editor Testing)

When we build `purescript-analyzer` locally, your editor will usually start whatever `purescript-analyzer` it finds on `PATH` (commonly `~/.cargo/bin/purescript-analyzer`). A plain `cargo build --release` does **not** update that binary.

## Checklist

1. Bump version (so you can confirm the editor is using the new build): edit `compiler-bin/Cargo.toml` `version = "..."`.

2. Build the release binary:

```bash
cargo build --release -p purescript-analyzer
./target/release/purescript-analyzer --version
```

3. Check what your shell/editor will actually run:

```bash
which -a purescript-analyzer
purescript-analyzer --version
```

4. Install the locally built binary into `~/.cargo/bin` (overwriting the old one on PATH):

```bash
cargo install --path compiler-bin --force
~/.cargo/bin/purescript-analyzer --version
```

## Notes

- If your editor is configured with an explicit LSP path, point it at `~/.cargo/bin/purescript-analyzer` (or the `target/release` binary if you prefer) and restart the editor/LSP.
- If you still see the old version, the editor may be launching an older PATH environment (restart the editor, or restart your login session if needed).
