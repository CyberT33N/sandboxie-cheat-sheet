# Boxed-Owned-Toolchain Starship

## Status

This is the preferred Starship architecture path in this repository.

## Current contract

For the boxed-owned-toolchain method:

- `Starship` is a versioned shared runtime
- the canonical shared binary lives under `C:\shared\sandbox-toolchains\dev\starship\1.25.1\`
- bootstrap mirrors it locally into the box execution tree
- bootstrap-generated Bash RC files initialize it for the integrated shell

## Provisioning

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

Expected verification:

```text
starship 1.25.1
```

## Bootstrap consumption

The current boxed-owned-toolchain bootstrap contract is:

1. provision the governed shared Starship binary under `dev\starship\1.25.1\`
2. mirror it locally into the box execution tree
3. generate the Bash RC files that initialize it
4. have those Bash RC files prepend the local `bootstrap-bin` directory to the Bash `PATH`
5. keep the prompt config file separate from the binary location

That extra `bootstrap-bin` PATH step matters because the same Git Bash shell must also be able to resolve bootstrap-generated toolchain wrappers such as:

- `pnpm`
- `node20`

The current runtime adapter lives in:

- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\starship\Bootstrap.Starship.psm1`

## Related

- `docs\applications\terminal\starship\general.md`
- `docs\applications\terminal\starship\architectures\host-sync\overview.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\starship.md`
