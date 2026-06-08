# ESLint In `settings.json`

## Scope

This document explains the ESLint-extension-specific `settings.json` contract for the `boxed-owned-toolchain` architecture.

It documents only the durable ESLint runtime binding.

It does **not** document user-specific background/image personalization.

## Current setting

The current canonical Windows setting is:

```json
{
  "[windows]": {
    "eslint.runtime": "C:\\shared\\sandbox-toolchains\\dev\\node\\20.19.6\\node-v20.19.6-win-x64\\node.exe"
  }
}
```

## Why this setting exists

`eslint.runtime` is an extension-specific runtime binding.

That makes it different from:

- the integrated terminal shell
- bootstrap-selected `PATH` composition
- general project-toolchain selection

Those broader concerns remain owned by the boxed-owned-toolchain bootstrap.

The ESLint extension still needs one explicit runtime value that it can launch deterministically.

## Why the binary is explicit

The boxed-owned-toolchain method must not point ESLint back to host-managed paths such as:

- `C:\Users\<user>\AppData\Local\nvm\...`
- a host-global `node.exe`
- other accidental host fallbacks

Instead, the extension is pinned to the governed shared Node binary:

```text
C:\shared\sandbox-toolchains\dev\node\20.19.6\node-v20.19.6-win-x64\node.exe
```

This keeps the setting:

- explicit
- reviewable
- host-independent
- aligned with the repository's governed boxed-owned-toolchain runtime inventory

## Why this is not a generic exception for all settings

The boxed-owned-toolchain method still treats bootstrap as the architecture center for:

- terminal runtime wiring
- local mirrored runtime preparation
- command-surface initialization

`eslint.runtime` is documented here as a narrow extension-specific exception, not as a license to move generic toolchain selection into `settings.json`.

## What this document does not claim

This document does **not** claim that every VS Code extension should pin its own runtime this way.

It documents the current ESLint contract only.

## Related

- `docs\applications\IDE\vscode\extensions\eslint\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\vscode-runtime-and-catalog.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
