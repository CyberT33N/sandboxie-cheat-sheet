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

### Rebuild an already-installed native package

```powershell
Set-Location "C:\git\test\test-mono"
$env:NX_DAEMON = "false"
$env:NX_NATIVE_FILE_CACHE_DIRECTORY = "C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native"

& "C:\Users\yourusername\AppData\Local\nvm\v20.19.6\pnpm.cmd" rebuild node-firebird-native-api --store-dir "C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store"
```

### Re-run the full install flow

```powershell
Set-Location "C:\git\test\test-mono"
$env:NX_DAEMON = "false"
$env:NX_NATIVE_FILE_CACHE_DIRECTORY = "C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native"

& "C:\Users\yourusername\AppData\Local\nvm\v20.19.6\pnpm.cmd" install --store-dir "C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store"
```

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
