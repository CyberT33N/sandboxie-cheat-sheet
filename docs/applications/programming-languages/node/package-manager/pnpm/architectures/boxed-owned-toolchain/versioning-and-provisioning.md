# PNPM Versioning And Provisioning

## Provisioning

```powershell
$Node26Root = "C:\shared\sandbox-toolchains\dev\node\26.2.0\node-v26.2.0-win-x64"
$PnpmVersion = "11.5.0"
$PnpmRoot = "C:\shared\sandbox-toolchains\dev\pnpm\$PnpmVersion"

New-Item -ItemType Directory -Force -Path $PnpmRoot | Out-Null
Remove-Item "$PnpmRoot\*" -Recurse -Force -ErrorAction SilentlyContinue

Push-Location $PnpmRoot
& "$Node26Root\npm.cmd" pack "pnpm@$PnpmVersion"
tar -xf "pnpm-$PnpmVersion.tgz"
Pop-Location
```

## Host-side provisioning of another project-specific version

The shared `dev\pnpm\` area may keep multiple PNPM versions side by side.

If one project contract now requires `11.5.0`, provision it as an additional governed version instead of mutating older PNPM subtrees in place.

Sanitized host-side example:

```powershell
$SharedRoot = "C:\shared\sandbox-toolchains"
$Node26Root = Join-Path $SharedRoot "dev\node\26.2.0\node-v26.2.0-win-x64"
$PnpmVersion = "11.5.0"
$PnpmRoot = Join-Path $SharedRoot "dev\pnpm\$PnpmVersion"

New-Item -ItemType Directory -Force -Path $PnpmRoot | Out-Null
Remove-Item "$PnpmRoot\*" -Recurse -Force -ErrorAction SilentlyContinue

Push-Location $PnpmRoot
& "$Node26Root\npm.cmd" pack "pnpm@$PnpmVersion"
tar -xf "pnpm-$PnpmVersion.tgz"
Pop-Location

& "$Node26Root\node.exe" "$PnpmRoot\package\bin\pnpm.cjs" --version
```

Expected verification:

```text
11.5.0
```

## What to change when a project must move to a newer PNPM version

The current boxed-owned-toolchain sequence is:

1. provision `11.5.0` in `C:\shared\sandbox-toolchains\dev\pnpm\11.5.0\...`
2. update the project adapter so `PnpmCli` points to `dev\pnpm\11.5.0\package\bin\pnpm.cjs`
3. restart the project box through the normal bootstrap launcher

After that, the boxed `pnpm` command resolves to the newly selected shared CLI because the bootstrap-generated wrapper will be rebuilt from the updated project contract.

## Why `pnpm self-update` or `npm install -g pnpm` did not fix the boxed command

In this architecture, the boxed `pnpm` command is not supposed to be the machine-global PNPM installation.

It is the bootstrap-generated command surface for the currently selected shared `PnpmCli`.

So commands such as:

- `pnpm self-update 11.5.0`
- `npm install -g pnpm@11.5.0`

do not change the project-box contract by themselves when the project bootstrap still points at some other governed PNPM version.

## Architectural recommendation

For a one-box-one-repo boxed-owned-toolchain contract, the more correct architecture is:

- provision multiple governed PNPM versions side by side under `dev\pnpm\...`
- declare the selected version in the project adapter / box contract
- let the generic bootstrap materialize that declared contract

This is preferable to making the everyday host launch command choose the PNPM version ad hoc, because:

1. the project contract stays explicit and reviewable
2. one box keeps one deterministic runtime/toolchain contract
3. restart behavior is reproducible
4. the box does not silently drift because of caller-provided launch arguments

## Override stance

An argument-based host override can exist as a **break-glass or migration tool**, but it should not be the normal contract.

If such an override is ever introduced, it should be treated as:

- exceptional
- explicit
- validated against the project contract
- and preferably rejected unless the caller intentionally opts into a temporary override mode

That keeps the default architecture deterministic while still leaving room for controlled maintenance workflows.

## Update workflow summary

When a project wants to move to a newer PNPM version, the repeatable workflow is:

1. provision the new governed PNPM binary in `dev\pnpm\...` using the provisioning script above
2. update the project contract so `PnpmCli` points at the new version
3. restart the project box through the normal bootstrap launcher
4. run the project-owned install script from the PNPM scripts area

This document owns **steps 1-3**.

The project-box scripts area owns **step 4**:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`

## Related

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
