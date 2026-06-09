# Boxed-Owned-Toolchain Node Runtime

## Status

This is the preferred Node-runtime architecture path in this repository.

## Why this file exists

The Node runtime needs its own domain entrypoint instead of living only as a VS Code method detail.

## Current target

The boxed-owned-toolchain method uses governed shared Node binaries under:

- `C:\shared\sandbox-toolchains\dev\node\26.2.0\...`
- `C:\shared\sandbox-toolchains\dev\node\20.9.0\...`

Bootstrap selects and mirrors the required runtime into the local box execution tree.

## Documentation boundary

This document records the currently validated shared Node runtime contract used by the boxed bootstrap scripts.

It does **not** mean that every Node-domain workflow in this repository is already validated under boxed-owned-toolchain.

In particular:

- dependency-install overlays such as `node-gyp` still have host-sync-owned reference material
- host-managed runtime history remains on the host-sync side
- this page should be read as runtime-placement truth, not as blanket proof for every Node use case

## Selected versions

- `26.2.0` = primary control-plane / tooling runtime
- `20.9.0` = secondary runtime domain used where the architecture intentionally requires Node 20

## Canonical shared paths

Primary:

```text
C:\shared\sandbox-toolchains\dev\node\26.2.0\node-v26.2.0-win-x64\
```

Secondary:

```text
C:\shared\sandbox-toolchains\dev\node\20.9.0\node-v20.9.0-win-x64\
```

## Provisioning

### Node `26.2.0`

```powershell
$Node26Version = "26.2.0"
$Node26Zip = Join-Path $env:TEMP "node-v$Node26Version-win-x64.zip"
$Node26Dest = "C:\shared\sandbox-toolchains\dev\node\$Node26Version"

Invoke-WebRequest `
  -Uri "https://nodejs.org/dist/v$Node26Version/node-v$Node26Version-win-x64.zip" `
  -OutFile $Node26Zip

Remove-Item "$Node26Dest\*" -Recurse -Force -ErrorAction SilentlyContinue
Expand-Archive -LiteralPath $Node26Zip -DestinationPath $Node26Dest -Force
```

### Node `20.9.0`

```powershell
$Node20Version = "20.9.0"
$Node20Zip = Join-Path $env:TEMP "node-v$Node20Version-win-x64.zip"
$Node20Dest = "C:\shared\sandbox-toolchains\dev\node\$Node20Version"

Invoke-WebRequest `
  -Uri "https://nodejs.org/dist/v$Node20Version/node-v$Node20Version-win-x64.zip" `
  -OutFile $Node20Zip

Remove-Item "$Node20Dest\*" -Recurse -Force -ErrorAction SilentlyContinue
Expand-Archive -LiteralPath $Node20Zip -DestinationPath $Node20Dest -Force
```

## Bootstrap consumption

The current Node runtime mirror and command-surface generation live in:

- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\node\Bootstrap.Node.psm1`

That layer:

- validates the shared Node roots
- mirrors the selected runtime locally
- creates wrapper commands such as `pnpm.cmd`
- creates shell-native wrappers such as `pnpm`
- exposes secondary commands such as `node20`
- creates shell-native secondary wrappers such as `node20`

This matters because the current boxed-owned-toolchain VS Code path uses integrated Git Bash.

So a Windows `.cmd` wrapper alone is not sufficient for every shell surface. The runtime layer now needs to expose both:

- Windows-shell command surfaces
- Git-Bash-native command surfaces

## Why `nvm` is not the final contract

The boxed-owned-toolchain method does not treat `nvm use` or host-managed shims as the runtime source of truth.

The runtime truth is the explicit governed shared binary inventory above.

## Related

- `docs\applications\programming-languages\node\runtime\general.md`
- `docs\applications\programming-languages\node\runtime\architectures\host-sync\overview.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\node.md`
