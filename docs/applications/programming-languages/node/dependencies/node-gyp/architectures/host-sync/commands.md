# Commands

## Purpose

This page documents the validated command flow for `node-gyp` in the current host-sync architecture.

The commands below assume:

- the generic install-box monorepo baseline is already in place
- the shared Python binary exists under `C:\shared\sandbox-toolchains\dev\python\...`
- the install-box config additions from
  `docs\applications\programming-languages\node\dependencies\node-gyp\Architectures\host-sync\install-box-config.md`
  are already applied

## Step 1 - bootstrap the install-box shell

Run this in every new install-box shell before `node-gyp configure`, `node-gyp rebuild`, or `pnpm rebuild` of a native addon:

```powershell
$toolRoot      = "C:\shared\sandbox-toolchains\dev"
$pythonVersion = (Get-Content (Join-Path $toolRoot "python\current.txt") -Raw).Trim()
$pythonExe     = Join-Path $toolRoot "python\$pythonVersion\python.exe"

$vswhere = "${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vswhere.exe"
$vsRoot  = & $vswhere -latest -products * -requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64 -property installationPath
$vsDevCmd = Join-Path $vsRoot "Common7\Tools\VsDevCmd.bat"

cmd /c "`"$vsDevCmd`" -arch=x64 -host_arch=x64 && set" |
ForEach-Object {
    if ($_ -match '^(.*?)=(.*)$') {
        Set-Item -Path "Env:$($matches[1])" -Value $matches[2]
    }
}

$env:PYTHON                = $pythonExe
$env:NODE_GYP_FORCE_PYTHON = $pythonExe
$env:npm_config_python     = $pythonExe

Write-Host "VCINSTALLDIR = $env:VCINSTALLDIR"
Get-Command cl.exe
Get-Command msbuild.exe

& $pythonExe --version
& $pythonExe -c "import sys; print(sys.executable)"
```

## Step 2 - direct `node-gyp` smoke test

The following is the validated direct smoke test pattern.

The package name below is only an example of a native addon package. Replace the versioned `.pnpm\...` path when the real package version changes.

```powershell
$repoRoot = "C:\git\test\test-mono"
$node = "C:\Users\yourusername\AppData\Local\nvm\v20.19.6\node.exe"
$gyp  = "$repoRoot\.pnpm\node-gyp@12.3.0\node_modules\node-gyp\bin\node-gyp.js"
$pkg  = "$repoRoot\.pnpm\node-firebird-native-api@3.2.0\node_modules\node-firebird-native-api"

Set-Location $pkg

& "$node" "$gyp" clean
& "$node" "$gyp" configure --verbose --python "$pythonExe"
& "$node" "$gyp" rebuild   --verbose --python "$pythonExe"
```

Expected result:

- Python is found at the shared `dev` root
- Visual Studio Build Tools are found through the imported `VsDevCmd.bat` environment
- `binding.sln`, `.vcxproj`, and `build\Release\addon.node` are materialized to the real host-visible repo path

## Step 3 - validate the default `pnpm` route

After the direct `node-gyp` smoke test succeeds, validate the default package-manager route from the install box.

### Important note about `pnpm rebuild`

In validated monorepo tests, `pnpm rebuild <package>` can be a no-op when:

- the package is only transitive and not selected the way the caller expects
- the workspace install state is already considered clean
- `node_modules\.modules.yaml` no longer reports any pending builds

So `pnpm rebuild` is useful, but it is **not** the strongest proof that the default route works.

### Deterministic default-route validation

For a deterministic validation of the automatic `pnpm install` path:

1. clear the box contents
2. delete the host-visible `node_modules` / `.pnpm` tree
3. bootstrap the install-box shell as documented in Step 1
4. run a fresh `pnpm install` with the shared store path

Validated sanitized pattern:

```powershell
$repoRoot = "C:\git\test\test-mono"
$appDir   = Join-Path $repoRoot "apps\desktop-app"
$pkg      = Join-Path $repoRoot ".pnpm\node-firebird-native-api@3.2.0\node_modules\node-firebird-native-api"

# host-visible dependency cleanup
Set-Location $repoRoot

Remove-Item ".\node_modules" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item ".\apps\desktop-app\node_modules" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item ".\apps\frontend\node_modules" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item ".\apps\backend\node_modules" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item ".\apps\webpages\node_modules" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item ".\tools\installer\node_modules" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item ".\.pnpm" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item ".\node_modules\.modules.yaml" -Force -ErrorAction SilentlyContinue

# Step 1 bootstrap from this document must already have been run in this shell.
$env:NX_DAEMON = "false"
$env:NX_NATIVE_FILE_CACHE_DIRECTORY = "C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native"
$pnpm = "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd"

Remove-Item -Recurse -Force "$pkg\build" -ErrorAction SilentlyContinue

Set-Location $appDir

& "$pnpm" install --store-dir "C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store" --reporter ndjson
```

### What success looks like

If the default route is working, a fresh install should:

- enter the real install path rather than reporting "Already up to date"
- run the native dependency install/build lifecycle
- recreate repo-local build outputs such as:
  - `build\binding.sln`
  - `.vcxproj`
  - `build\Release\addon.node`

## What a successful validation means

If both the direct `node-gyp` flow and the default `pnpm` flow succeed in the install box, then the architectural problem has been solved at the platform/toolchain level.

That means:

- the shared Python binary is valid
- the host-provided Visual Studio Build Tools are visible and initialized correctly
- the relevant spawned processes can materialize their outputs to the real repo path

Any later failure is then more likely to be package-specific, addon-code-specific, or ABI-specific rather than a missing Sandboxie toolchain surface.

## Related documents

- `docs\applications\programming-languages\node\dependencies\node-gyp\Architectures\host-sync\python-binary.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\Architectures\host-sync\visual-studio-build-tools.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\Architectures\host-sync\install-box-config.md`
