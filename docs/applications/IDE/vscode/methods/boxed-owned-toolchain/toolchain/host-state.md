# Host State

## Final host target state

The final target state for this method is:

- no host VS Code
- no host Git
- no host Node
- no host pnpm
- no host nvm
- no host Starship binary

The host must not remain the accidental fallback execution surface for the development toolchain.

## Windows shell nuance

This must not be misread as:

- remove Windows `cmd.exe`
- remove Windows PowerShell
- or pretend the operating system has no shell binaries

The current validated method now provisions governed shared shell artifacts for:

- `cmd.exe`
- Windows PowerShell
- `Clink`

under:

- `C:\shared\sandbox-toolchains\dev\shells\...`

Those governed shared shell artifacts are then mirrored locally into the box execution tree.

The architectural rule is:

- do not rely on the host/system shell path as the accidental execution fallback inside boxed workflows
- do rely on governed shared shell artifacts plus explicit box-local mirrors when `cmd.exe` or PowerShell are needed inside integrated or child-process-controlled flows

## Why this matters

If host-side developer toolchains remain installed, the architecture becomes vulnerable to:

- accidental fallback to host binaries
- inconsistent runtime selection
- hidden dependence on host state
- drift between intended boxed behavior and actual execution behavior

## Prompt config location

The final method provisions the `Starship` binary under the shared toolchain root and mirrors it locally into the box execution tree during bootstrap.

The prompt config file may still remain user-owned, for example:

- `C:\Users\denni\.config\starship.toml`

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\starship.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\vscode-runtime-and-catalog.md`
