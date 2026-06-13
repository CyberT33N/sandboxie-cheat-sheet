# Microsoft Visual Studio Build Tools

## Architectural status

In the current validated host-sync `node-gyp` architecture, Microsoft Visual Studio Build Tools and Windows SDK are **host-provided**.

That means:

- they remain installed on the host system
- the install box consumes them through explicit Sandboxie visibility rules
- the install-box shell must bootstrap the Visual Studio developer environment before `node-gyp` runs

This is the current validated baseline for this repository.

The boxed-owned-toolchain projection alternative is documented separately here:

- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\boxed-owned-toolchain\overview.md`

## Why the host provides these tools in the validated baseline

Compared to the central shared Python binary, Microsoft build tools are a much heavier Windows toolchain boundary:

- `vswhere.exe`
- `VsDevCmd.bat`
- `MSBuild.exe`
- `cl.exe`
- `link.exe`
- `lib.exe`
- Windows SDK include/lib/bin trees
- installer-managed and registry-backed discovery state

For that reason, the currently validated `node-gyp` workflow in this repository uses the host-installed Build Tools instead of a mirrored portable copy under the shared toolchain root.

## Required host checks

Run these checks inside the install box after the relevant host paths are visible:

```powershell
$vswhere = "${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vswhere.exe"

Test-Path $vswhere
& $vswhere -latest -products * -requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64 -property installationPath
Test-Path "C:\Program Files\Microsoft Visual Studio"
Test-Path "C:\Program Files (x86)\Windows Kits\10"
```

Expected result:

- `vswhere.exe` exists
- a valid Build Tools installation path is returned
- the Windows SDK root exists

## Bootstrap the Visual Studio build environment

Run this in every new install-box shell before `node-gyp configure` / `node-gyp rebuild`:

```powershell
$vswhere = "${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vswhere.exe"
$vsRoot  = & $vswhere -latest -products * -requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64 -property installationPath
$vsDevCmd = Join-Path $vsRoot "Common7\Tools\VsDevCmd.bat"

cmd /c "`"$vsDevCmd`" -arch=x64 -host_arch=x64 && set" |
ForEach-Object {
    if ($_ -match '^(.*?)=(.*)$') {
        Set-Item -Path "Env:$($matches[1])" -Value $matches[2]
    }
}

Write-Host "VCINSTALLDIR = $env:VCINSTALLDIR"
Get-Command cl.exe
Get-Command msbuild.exe
```

Expected result:

- `VCINSTALLDIR` is set
- `cl.exe` resolves
- `msbuild.exe` resolves

## Why this bootstrap is required

During validation, `node-gyp` could still fail with:

```text
VCINSTALLDIR not set, not running in VS Command Prompt
Could not find any Visual Studio installation to use
```

even when the host Build Tools were installed and visible in the box.

Importing the `VsDevCmd.bat` environment in the current shell fixed that failure mode.

## Related

- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\host-sync\overview.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\install-box-config.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\commands.md`
