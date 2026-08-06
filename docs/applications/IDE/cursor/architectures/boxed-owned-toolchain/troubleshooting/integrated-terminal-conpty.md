# Cursor Integrated Terminal / ConPTY In The Boxed-Owned-Toolchain

## Scope

This document owns one specific troubleshooting class:

- `Cursor` launches successfully in the boxed-owned-toolchain architecture
- the shared VSCode-family `settings.json` and shared extensions are already mirrored correctly
- but `Terminal -> New Terminal` in the Cursor GUI fails with:
  - `The terminal process failed to launch`
  - `A native exception occurred during launch (Cannot launch conpty)`
  - or a visually equivalent "opens and closes immediately" behavior

This document records the validated fix for that exact failure class.

It does **not** own:

- generic Cursor window-mode semantics such as `Agent Window` vs. `Editor Window`
- generic Windows console / `conhost.exe` / integrity-boundary guidance
- generic boxed shell-spawn guidance
- Chromium / `SpecialImage` / `PrintCompositorLPAC` CLI-maintenance failures

## Why this document lives here

The correct location is:

```text
docs\applications\IDE\cursor\architectures\boxed-owned-toolchain\troubleshooting\
```

and not a broader generic folder such as:

- `docs\applications\terminal\...`
- `docs\applications\operating-systems\windows\terminal\...`
- or the product-root `docs\applications\IDE\cursor\general.md`

### Why this is not a generic terminal-domain document

From an enterprise-grade domain-driven perspective, the failing surface here is **not** "Windows terminals in general".

Why:

- the boxed shell runtimes themselves were already provisioned correctly
- the shared VSCode-family terminal profiles were already present in the canonical shared `settings.json`
- the box-local Cursor `User\settings.json` already contained the mirrored terminal profile configuration
- the failure only appeared when the **Cursor integrated terminal backend** tried to create its PTY/ConPTY path

So the failing bounded context is:

- **Cursor as an IDE runtime**
- under the **boxed-owned-toolchain architecture**
- in the **integrated terminal troubleshooting lane**

### Why this is not a product-root `Cursor` document

The product-root Cursor area should own:

- product-wide window-mode semantics
- generic Cursor-surface behavior across multiple architectures

This issue is narrower than that.

The current fix depends on:

- the shared VSCode-family settings catalog
- the boxed-owned-toolchain runtime model
- the validated boxed terminal environment

Therefore the problem is architecture-specific enough to live under:

- `architectures\boxed-owned-toolchain\troubleshooting`

### Why this is a troubleshooting document

This document records:

- a concrete failure shape
- a concrete tested mitigation
- the interpretation of what **was not** broken

That is troubleshooting ownership, not target-state architecture ownership.

## Observed failure shape

The validated failure shape in this repository context was:

1. `Cursor` could be launched successfully
2. the shared VSCode-family `settings.json` was already mirrored into the box-local Cursor `User\settings.json`
3. the shared VSCode-family extension store was already mirrored locally
4. the boxed shell/toolchain environment was already healthy
5. clicking `Terminal -> New Terminal` in the Cursor GUI still failed

The visible symptom could appear as either:

- a terminal-launch toast mentioning:
  - `Cannot launch conpty`
- or a short-lived terminal creation attempt that immediately disappears

## What was already proven healthy

Before applying the fix, all of the following were already established:

- shared `settings.json` transfer was healthy
- shared extension transfer was healthy
- the boxed shell/toolchain environment was healthy
- `BOXED_*` environment surfaces existed
- maintenance and project bootstrap both published the expected terminal/runtime environment

That means the problem should **not** be described primarily as:

- "the terminal profiles were not copied"
- "the extensions were not copied"
- "Git Bash was missing"
- "PowerShell was missing"
- "the maintenance box was not wired"

## Important negative finding

The repository also hit a **separate** failure class in the Cursor maintenance CLI lane:

```text
Cursor.exe: bad option: --disable-features=PrintCompositorLPAC
```

That belongs to the Chromium / `SpecialImage` / `code.cmd` maintenance-command surface and must **not** be conflated with the integrated-terminal `conpty` failure.

So the current repository state distinguishes between:

1. **Cursor maintenance CLI issue**
   - `code.cmd` / `PrintCompositorLPAC`
2. **Cursor integrated terminal issue**
   - `Cannot launch conpty`

This document owns the second issue only.

## Validated fix

The validated fix in this repository context was to add the following setting to the canonical shared VSCode-family `settings.json`:

```json
"terminal.integrated.windowsUseConptyDll": true
```

## Canonical write surface

The setting belongs in:

```text
C:\shared\sandbox-toolchains\ide\vscode\catalog\vscode-user\settings.json
```

Why this is the correct write surface:

- the boxed-owned-toolchain method already treats that path as the canonical VSCode-family user catalog
- `Cursor` and `VS Code` both consume that shared catalog through the family bootstrap
- the problem was observed in `Cursor`, but the setting itself is part of the shared integrated-terminal engine/configuration surface

So the correct enterprise interpretation is:

- the symptom is Cursor-specific in current evidence
- the write surface is still the shared VSCode-family catalog

## Why this shared setting is acceptable

This setting was added only after the repository had already proven:

- the normal terminal profile wiring was otherwise correct
- the failure sat at the integrated-terminal backend boundary
- and the explicit `windowsUseConptyDll` option restored working behavior

That means this is not random settings churn.

It is a repository-validated control-plane adjustment to the shared IDE-family terminal engine contract.

## Verification steps

After adding the setting, validate the following:

1. launch `Cursor` in the boxed-owned-toolchain path
2. open `Terminal -> New Terminal`
3. confirm that a boxed terminal now stays alive
4. confirm that the expected shared terminal profiles remain available, for example:
   - `Boxed PowerShell (Starship)`
   - `Boxed PowerShell`
   - `Boxed CMD (Starship)`
   - `Boxed CMD`
   - `Boxed Git Bash (Starship)`
   - `Boxed Git Bash (Minimal)`

## Interpretation of success

If the terminal opens successfully after this setting change, the correct conclusion is:

- the primary break sat in the Cursor-integrated ConPTY/PTY launch path
- not in shared settings distribution
- not in extension distribution
- not in maintenance-vs-project box ownership

## Architectural guidance

Do **not** use this document to claim that every future terminal failure in Cursor or VS Code should be solved by the same setting.

The reusable conclusion is narrower:

- when the boxed shell environment is already healthy
- and the shared VSCode-family settings/extensions are already mirrored correctly
- but the **Cursor integrated terminal** still fails specifically at `conpty`
- then `terminal.integrated.windowsUseConptyDll` is a valid boxed-owned-toolchain troubleshooting step

## Related

- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\general.md`
- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\cursor\architectures\boxed-owned-toolchain\troubleshooting\agent-shell-host-powershell-projection.md`
- `docs\applications\IDE\cursor\general.md`
- `docs\applications\IDE\cursor\architectures\boxed-owned-toolchain\sign-in.md`
- `docs\applications\operating-systems\windows\terminal\powershell\general.md`
- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\vscode-runtime-and-catalog.md`
