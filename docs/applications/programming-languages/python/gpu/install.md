# GPU install

## Goal

This document describes the validated install and reinstall workflow for a CUDA-capable `torch` setup inside the dedicated `python-general` toolchain.

## Preconditions

- the copied runtime exists under `C:\shared\sandbox-toolchains\python-general\python\3.12.9\`
- the documented Python toolchain box configuration is in place
- PowerShell is used for the commands below

## Step 1 — ensure the temp directory exists

```powershell
$toolRoot = "C:\shared\sandbox-toolchains\python-general"
New-Item -ItemType Directory -Force -Path "$toolRoot\cache\tmp"
```

## Step 2 — bootstrap the session variables

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

## Step 3 — upgrade `pip` inside the toolchain

```powershell
& $pythonExe -m pip install --user --upgrade pip
```

## Step 4 — clean old `torch` state before a GPU reinstall

```powershell
& $pythonExe -m pip uninstall -y torch torchvision
& $pythonExe -m pip cache purge
```

## Step 5 — install the CUDA-enabled build

Validated example for the tested case:

```powershell
& $pythonExe -m pip install --user --no-cache-dir torch torchvision --index-url https://download.pytorch.org/whl/cu130
```

The `--user` flag is mandatory here as well.

Without `--user`, `pip` may place the package and generated launchers into the base runtime under `python\3.12.9\...` instead of the documented `userbase\Python312\...` area.

If a different CUDA channel is intentionally selected later, the index URL must be adjusted accordingly.

## Step 6 — verify the result in the exact toolchain interpreter

```powershell
& $pythonExe -c "import sys, torch; print(sys.executable); print(torch.__version__); print(torch.__file__); print(torch.version.cuda); print(torch.cuda.is_available())"
```

Expected success characteristics:

- the interpreter path points to the shared toolchain runtime
- `torch.__version__` includes a CUDA suffix such as `+cu130`
- `torch.version.cuda` is not `None`
- `torch.cuda.is_available()` returns `True`

## Step 7 — use GPU-dependent workloads only after the direct test passes

Only after the exact toolchain interpreter reports `True` should GPU-backed workloads such as `docling` be evaluated.

## Related documents

- docs/applications/programming-languages/python/gpu/general.md
- docs/applications/programming-languages/python/gpu/uninstall.md
- docs/applications/programming-languages/python/gpu/troubleshooting.md
