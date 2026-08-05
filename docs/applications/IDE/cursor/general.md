# Cursor

## Scope

This folder is the Cursor-product documentation area for this repository.

This document owns one **product-level** concern that is intentionally **not** architecture-specific:

- the difference between the `Agent Window` surface
- and the classic `Editor Window` surface
- plus the correct CLI-facing interpretation of both

It does **not** own:

- boxed-owned-toolchain bootstrap implementation details
- boxed-owned-toolchain runtime-installation details
- host-sync materialization details
- login troubleshooting details
- architecture-specific Sandboxie policy

## Why this document lives here

From an enterprise-grade domain-driven perspective, this document belongs directly under:

```text
docs\applications\IDE\cursor\
```

and **not** under one specific architecture subtree.

Why:

- `Agent Window` vs. `Editor Window` is a **Cursor product-surface concern**
- the same product split can appear under:
  - boxed-owned-toolchain
  - host-sync
  - or a future method
- architecture documents may decide **which** window surface to force
- but the meaning of those window surfaces belongs to the Cursor domain itself

So the correct split is:

- **Cursor root docs** own the product/window-mode semantics
- **architecture docs** own how one concrete method invokes Cursor in that environment

This keeps the source of truth stable and avoids duplicating window-mode explanations across multiple architecture trees.

## Current product-level window split

For the current Cursor 3.x surface used in this repository, the important user-visible split is:

1. **Agent Window**
2. **Editor Window**

Those two surfaces must not be treated as interchangeable.

## Agent Window

The `Agent Window` is the agent-oriented / chat-oriented Cursor surface.

Architecturally, it should be interpreted as:

- a product-level agent workspace
- not the normal deterministic repo-opening surface
- not the canonical monorepo IDE entrypoint

Important rule:

- the `Agent Window` should **not** be standardized as the repo-specific project-open contract

Why:

- the agent-oriented surface is not the same thing as the classic repo/file/folder IDE surface
- it may be selected by current Cursor product state, layout memory, or startup policy
- it does not provide the same deterministic "open this monorepo as the active IDE workspace" guarantee

That means:

- do **not** model normal monorepo launch around the `Agent Window`
- do **not** assume that "Cursor started" means "the repo opened in the correct IDE surface"

## Editor Window

The `Editor Window` is the classic IDE/file/folder/workspace surface.

Architecturally, this is the correct surface for:

- opening a repo
- opening a monorepo
- opening a folder/workspace deterministically
- treating Cursor as the repo-scoped IDE

Important rule:

- if the goal is "open this project/monorepo in Cursor", the launch contract should force the `Editor Window`

In current Cursor CLI terms, the relevant product flag is:

- `--classic`

That flag disables the Glass/agent-style window routing and forces the classic editor-style window.

## Correct CLI interpretation

The current Cursor CLI exposes several relevant surfaces:

- `--classic`
- `--new-window`
- `--reuse-window`
- `--chat`
- `--glass`

However, they are **not** equal from a contract/governance perspective.

### Stable repo-opening contract

For deterministic repo opening, the normalized contract is:

1. pass the target folder/repo path
2. force classic editor mode with `--classic`
3. choose either:
   - `--new-window` when a fresh editor window is required
   - `--reuse-window` when the caller explicitly wants an existing editor window reused

### Agent/chat contract

If the goal is an agent/chat-oriented surface rather than a repo-scoped IDE window, the safe explicit surface is:

- `--chat`

This must be interpreted correctly:

- it opens a standalone chat-style Cursor surface
- it is **not** the canonical repo-opening contract
- it should not be documented as "open the monorepo in Cursor"

### `--glass` nuance

The current CLI also exposes:

- `--glass`

But the CLI help marks it as a dev-only-style product surface.

Therefore the enterprise-grade interpretation here is:

- acknowledge that the Glass/agent family exists
- but do **not** normalize `--glass` as the repository's primary deterministic project-open contract

## Deterministic rules

### If the caller wants the agent/chat surface

Use an agent/chat-oriented launch and do **not** describe it as repo-specific IDE open.

### If the caller wants the repo/monorepo opened as an IDE

Use the classic editor contract:

- `--classic`
- plus a folder path
- plus `--new-window` or `--reuse-window`

That is the key architectural distinction.

## Sanitized generic examples

The following examples use the sanitized example project name:

- `test-mono`

and the sanitized user path:

- `C:\Users\yourusername\source\test-mono`

### Open the sanitized monorepo in the classic Editor Window

```powershell
& "C:\Path\To\Cursor.exe" `
  --new-window `
  --classic `
  "C:\Users\yourusername\source\test-mono"
```

### Reuse an existing classic Editor Window for the sanitized monorepo

```powershell
& "C:\Path\To\Cursor.exe" `
  --reuse-window `
  --classic `
  "C:\Users\yourusername\source\test-mono"
```

### Open an agent/chat-oriented Cursor surface

```powershell
& "C:\Path\To\Cursor.exe" `
  --chat
```

Important interpretation:

- the first two commands are repo-/folder-scoped IDE opens
- the third command is **not** the monorepo IDE-open contract

## Current repository example

The current real project name in this repository context is:

- `test-mono`

In the current boxed project launcher shape, the host-side entrypoint is:

```powershell
& "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoCursorBoxed.ps1" `
  -Action LaunchCursor `
  -RepoPath "C:\Users\denni\source\test-mono"
```

The architecture-specific wrapper may remain responsible for:

- entering the correct box/runtime
- preparing the correct `user-data` and extension paths
- selecting the governed shared toolchain

But the Cursor product-level expectation remains:

- that project launch must end in the classic `Editor Window`
- and that the repo path must be opened there as the active workspace/folder

## Normalization rule for future wrappers

Any future project/bootstrap/launcher wrapper that claims to "open a repo in Cursor" should be interpreted as needing this semantic target:

- **Editor Window**
- **not Agent Window**

That wrapper may differ by architecture, but the product-level contract should remain the same.

## What this document does not claim

This document does **not** claim that:

- all Cursor startup bugs are solved by `--classic`
- the `Agent Window` is useless
- Glass/agent-oriented product surfaces should never be used

It only defines the correct architectural split:

- `Agent Window` = agent/chat-oriented surface, not the canonical repo-open contract
- `Editor Window` = deterministic repo/folder/workspace IDE surface

## Related

- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\general.md`
- `docs\applications\IDE\cursor\architectures\boxed-owned-toolchain\troubleshooting\integrated-terminal-conpty.md`
- `docs\applications\IDE\cursor\architectures\boxed-owned-toolchain\sign-in.md`
- `docs\applications\IDE\cursor\architectures\host-sync\debugging.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\start.md`
- `docs\applications\IDE\vscode\methods\general.md`
