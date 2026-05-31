# Clean Reinstall

## Purpose

This document is the source of truth for the validated clean reinstall flow when a governed Node monorepo may auto-trigger `node-gyp` during `pnpm install`.

It documents only the working reinstall sequence:

1. delete the install-box contents
2. delete the host-visible materialized dependency tree
3. bootstrap the install-box shell and run `pnpm install`

## When to use this document

Use this flow when:

- the workspace should be reinstalled from a clean state
- the install may auto-trigger a native dependency build through `node-gyp`
- the install box should rebuild the host-visible dependency tree from scratch

## Step 1 - clear the install-box contents

Delete the install-box contents through the Sandboxie UI before retesting.

## Step 2 - clear the host-visible materialized dependency tree

Run this on the host:

```powershell
Set-Location "C:\git\test\test-mono"

Remove-Item ".\node_modules" -Recurse -Force -ErrorAction SilentlyContinue

Remove-Item ".\apps\desktop-app\node_modules" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item ".\apps\frontend\node_modules" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item ".\apps\backend\node_modules" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item ".\apps\webpages\node_modules" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item ".\tools\installer\node_modules" -Recurse -Force -ErrorAction SilentlyContinue

Remove-Item ".\.pnpm" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item ".\node_modules\.modules.yaml" -Force -ErrorAction SilentlyContinue
```

## Step 3 - bootstrap the install-box shell and run the install

Run this inside the install box exactly as shown:

```powershell
Set-Location "C:\git\test\test-mono"

# -----------------------------
# Shared Python build helper
# -----------------------------
$toolRoot      = "C:\shared\sandbox-toolchains\dev"
$pythonVersion = (Get-Content (Join-Path $toolRoot "python\current.txt") -Raw).Trim()
$pythonExe     = Join-Path $toolRoot "python\$pythonVersion\python.exe"

# -----------------------------
# Fixed Node / pnpm toolchain
# -----------------------------
$node = "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\node.exe"
$pnpm = "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd"

# -----------------------------
# Visual Studio Build Tools bootstrap
# -----------------------------
$vswhere = "${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vswhere.exe"
$vsRoot  = & $vswhere -latest -products * -requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64 -property installationPath
$vsDevCmd = Join-Path $vsRoot "Common7\Tools\VsDevCmd.bat"

cmd /c "`"$vsDevCmd`" -arch=x64 -host_arch=x64 && set" |
ForEach-Object {
    if ($_ -match '^(.*?)=(.*)$') {
        Set-Item -Path "Env:$($matches[1])" -Value $matches[2]
    }
}

# -----------------------------
# node-gyp / native build env
# -----------------------------
$env:PYTHON                = $pythonExe
$env:NODE_GYP_FORCE_PYTHON = $pythonExe
$env:npm_config_python     = $pythonExe

# -----------------------------
# Monorepo install env
# -----------------------------
$env:NX_DAEMON = "false"
$env:NX_NATIVE_FILE_CACHE_DIRECTORY = "C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native"

# -----------------------------
# Sanity checks
# -----------------------------
& $node -v
& $pnpm -v
& $pythonExe --version
Get-Command cl.exe | Out-Host
Get-Command msbuild.exe | Out-Host

# -----------------------------
# Install
# -----------------------------
& $pnpm install --store-dir "C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store"
```

## Why Step 3 is required

On a fresh install, `pnpm install` may automatically trigger `node-gyp` for a dependency with `binding.gyp` / `gypfile: true`.

Without the full shell bootstrap above, validation produced failures such as:

```text
Could not find any Python installation to use
Could not find any Visual Studio installation to use
```

The validated reinstall flow therefore requires:

- the shared Python binary
- the imported `VsDevCmd.bat` environment
- the `PYTHON` / `NODE_GYP_FORCE_PYTHON` / `npm_config_python` environment variables
- the shared PNPM store path

## Interpretation

If the reinstall succeeds, then:

- the install box can materialize the dependency tree to the host-visible workspace path
- the shared PNPM store is wired correctly
- the install-box shell is native-build-ready for dependencies that auto-trigger `node-gyp`

## Related documents

- `docs\applications\programming-languages\node\dependencies\node-gyp\Architectures\host-sync\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\Architectures\host-sync\commands.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\host-sync\troubleshooting\performance.md`
