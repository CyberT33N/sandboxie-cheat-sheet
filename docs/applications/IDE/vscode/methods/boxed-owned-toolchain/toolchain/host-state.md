# Host State

## Final host target state

The final target state for this method is:

- no host VS Code
- no host Git
- no host Node
- no host pnpm
- no host nvm

The host must not remain the accidental fallback execution surface for the development toolchain.

## Why this matters

If host-side developer toolchains remain installed, the architecture becomes vulnerable to:

- accidental fallback to host binaries
- inconsistent runtime selection
- hidden dependence on host state
- drift between intended boxed behavior and actual execution behavior

## Host exception

`Starship` may remain on the host because it is shell/prompt infrastructure rather than project toolchain governance.

Consumed host paths include:

- `C:\Program Files\starship\bin\starship.exe`
- `C:\Users\denni\.config\starship.toml`

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\vscode-runtime-and-catalog.md`
