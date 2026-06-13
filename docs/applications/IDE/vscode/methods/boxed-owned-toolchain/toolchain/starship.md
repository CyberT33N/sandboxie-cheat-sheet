# Starship

## Decision

`Starship` is shell/prompt infrastructure, not project dependency governance.

The boxed-owned-toolchain method provisions it as a versioned shared runtime and mirrors it locally into the box execution tree during bootstrap.

Starship is not limited to Git Bash in this method.

Current lane split:

- PowerShell is the preferred default interactive lane
- CMD is supported through the separate `Clink` adapter lane
- Git Bash remains an explicit supported shell lane

## Selected form

- official GitHub release archive
- Windows asset: `starship-x86_64-pc-windows-msvc.zip`
- version: `1.25.1`

## Canonical shared path

```text
C:\shared\sandbox-toolchains\dev\starship\1.25.1\
```

Expected executable:

```text
C:\shared\sandbox-toolchains\dev\starship\1.25.1\starship.exe
```

## Provisioning code

```powershell
$StarshipVersion = '1.25.1'
$StarshipZip = Join-Path $env:TEMP "starship-x86_64-pc-windows-msvc-$StarshipVersion.zip"
$StarshipDest = "C:\shared\sandbox-toolchains\dev\starship\$StarshipVersion"

Invoke-WebRequest `
  -Uri "https://github.com/starship/starship/releases/download/v$StarshipVersion/starship-x86_64-pc-windows-msvc.zip" `
  -OutFile $StarshipZip

New-Item -ItemType Directory -Force -Path $StarshipDest | Out-Null
Remove-Item "$StarshipDest\*" -Recurse -Force -ErrorAction SilentlyContinue

Expand-Archive -LiteralPath $StarshipZip -DestinationPath $StarshipDest -Force

& "$StarshipDest\starship.exe" --version
```

## Expected verification

```text
starship 1.25.1
```

## Bootstrap consumption model

The preferred bootstrap consumption shape is:

1. mirror `C:\shared\sandbox-toolchains\dev\starship\1.25.1\` into the box-local toolchain tree
2. prepend the local `bootstrap-bin` directory to the Bash `PATH` through bootstrap-generated shell startup files
3. expose the local mirrored Starship binary through the same shell startup files
4. let VS Code terminal profiles choose the shell and its startup file explicitly, with PowerShell as the preferred default profile

Current lane split:

- PowerShell uses bootstrap-generated PowerShell init files and is the preferred default interactive lane
- CMD uses `Clink` as the CMD-specific Starship adapter
- Git Bash uses bootstrap-generated Bash RC files and remains an explicit supported shell lane

This keeps the roles separated correctly:

- shared provisioning decides which Starship binary version is canonical
- bootstrap decides which local mirrored executable the shell should use and which helper command directory Bash should see
- VS Code terminal settings decide which shell/profile should start by default

## Why the Bash RC files matter beyond Starship

The current Bash RC files are not only prompt-initialization files.

They also have to export the local `bootstrap-bin` path so Git Bash can resolve bootstrap-generated shell wrappers such as:

- `pnpm`
- `node20`

Without that PATH step, the integrated Git Bash shell can start successfully and still fail to find toolchain commands that are available in PowerShell/CMD.

## Config file note

The Starship binary and the Starship config are separate concerns.

The binary lives in the shared toolchain model while the config file may still remain user-owned, for example:

```text
C:\Users\denni\.config\starship.toml
```

This keeps prompt configuration separate from binary/runtime placement.

## Related

- `docs\cli\shell\general.md`
- `docs\applications\terminal\starship\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\host-state.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
- `docs\applications\terminal\starship\general.md`
