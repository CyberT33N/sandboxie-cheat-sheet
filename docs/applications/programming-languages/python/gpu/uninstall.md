# GPU uninstall

## Goal

This document describes how to remove or reset the GPU-specific `torch` state inside the dedicated `python-general` toolchain.

## Step 1 — bootstrap the session variables

```powershell
$toolRoot = "C:\shared\sandbox-toolchains\python-general"
$pythonExe = "$toolRoot\python\3.12.9\python.exe"
$env:PYTHONUSERBASE = "$toolRoot\userbase"
$env:PIP_CACHE_DIR = "$toolRoot\cache\pip"
$env:HF_HOME = "$toolRoot\cache\huggingface"
$env:TORCH_HOME = "$toolRoot\cache\torch"
$env:TEMP = "$toolRoot\cache\tmp"
$env:TMP = "$toolRoot\cache\tmp"
$env:PATH = "$toolRoot\userbase\Python312\Scripts;" + $env:PATH
```

## Step 2 — uninstall `torch` and `torchvision`

```powershell
& $pythonExe -m pip uninstall -y torch torchvision
```

## Step 3 — purge cached wheels and temp artifacts

```powershell
& $pythonExe -m pip cache purge
```

Optional cleanup of the dedicated temp directory:

```powershell
Get-ChildItem "$toolRoot\cache\tmp" -Force | Remove-Item -Recurse -Force
```

## Step 4 — verify the removal state

```powershell
& $pythonExe -m pip show torch
```

If `pip show torch` no longer returns a package, the toolchain is ready for a clean reinstall.

## Related documents

- docs/applications/programming-languages/python/gpu/general.md
- docs/applications/programming-languages/python/gpu/install.md
- docs/applications/programming-languages/python/gpu/troubleshooting.md
