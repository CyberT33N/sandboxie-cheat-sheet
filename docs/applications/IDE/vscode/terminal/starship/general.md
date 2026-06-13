# VS Code Terminal Starship

## Status

This page remains only as a thin VS Code terminal reference.

The older content here described a host-shaped setup with paths such as:

- `C:\Tools\DevBoxShell\...`
- `C:\Program Files\starship\bin\starship.exe`

That is no longer the current boxed-owned-toolchain contract.

## Current boxed-owned-toolchain reading

For the current preferred boxed-owned-toolchain method:

- the Starship binary is governed under `C:\shared\sandbox-toolchains\dev\starship\1.25.1\`
- bootstrap mirrors that binary locally into the box execution tree
- boxed PowerShell is the preferred interactive VS Code default shell
- boxed CMD is supported through the separate `Clink` adapter lane
- boxed Git Bash remains an explicit supported shell lane

Starship is therefore not restricted to Git Bash.

Different shell lanes use different bootstrap-owned initialization shapes:

- PowerShell uses the boxed PowerShell init files
- CMD uses `Clink`
- Git Bash uses the boxed Bash RC files






























## Source of truth

Use these documents instead of the older host-shaped examples that previously lived here:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\vscode-runtime-and-catalog.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\starship.md`
- `docs\applications\terminal\starship\architectures\boxed-owned-toolchain\overview.md`
- `docs\cli\shell\clink.md`

## Legacy note

If you specifically need the older host-sync / host-visible Starship model, use:

- `docs\applications\terminal\starship\architectures\host-sync\overview.md`












## Related

- `docs\applications\IDE\vscode\terminal\debug\general.md`