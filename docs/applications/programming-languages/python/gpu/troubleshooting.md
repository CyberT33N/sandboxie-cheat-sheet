# GPU troubleshooting

## Scope

This document covers the most important failure patterns that appeared while enabling GPU-backed `torch` inside the dedicated `python-general` toolchain.

## Symptom — global host `python` returns `True`, toolchain Python returns `False`

### Meaning

The global host interpreter and the copied toolchain interpreter are different runtime boundaries.

### What to compare

Run this with the global host interpreter:

```powershell
python -c "import sys, torch; print(sys.executable); print(torch.__version__); print(torch.__file__); print(torch.version.cuda); print(torch.cuda.is_available())"
```

Run this with the toolchain interpreter:

```powershell
$toolRoot = "C:\shared\sandbox-toolchains\python-general"
$pythonExe = "$toolRoot\python\3.12.9\python.exe"
& $pythonExe -c "import sys, torch; print(sys.executable); print(torch.__version__); print(torch.__file__); print(torch.version.cuda); print(torch.cuda.is_available())"
```

### Root cause pattern

If the toolchain shows:

- `torch.version.cuda == None`
- `torch.cuda.is_available() == False`

then the toolchain most likely has a CPU-only `torch` build.

## Symptom — `Wheel 'torch' ... is invalid`

### Meaning

The CUDA wheel was found, but the wheel handling or unpack flow failed.

### Validated fix pattern

1. create a dedicated temp directory under the toolchain cache
2. set `TEMP` and `TMP` to that directory
3. upgrade `pip`
4. purge the pip cache
5. reinstall with `--no-cache-dir`

### Validated command flow

```powershell
$toolRoot = "C:\shared\sandbox-toolchains\python-general"
$pythonExe = "$toolRoot\python\3.12.9\python.exe"
New-Item -ItemType Directory -Force -Path "$toolRoot\cache\tmp"
$env:PYTHONUSERBASE = "$toolRoot\userbase"
$env:PIP_CACHE_DIR = "$toolRoot\cache\pip"
$env:HF_HOME = "$toolRoot\cache\huggingface"
$env:TORCH_HOME = "$toolRoot\cache\torch"
$env:TEMP = "$toolRoot\cache\tmp"
$env:TMP = "$toolRoot\cache\tmp"
$env:PATH = "$toolRoot\userbase\Python312\Scripts;" + $env:PATH

& $pythonExe -m pip install --user --upgrade pip
& $pythonExe -m pip cache purge
& $pythonExe -m pip install --user --no-cache-dir torch torchvision --index-url https://download.pytorch.org/whl/cu130
```

## Symptom — `ModuleNotFoundError: No module named 'torch'`

### Meaning

`torch` was uninstalled or never installed in the current toolchain user base.

### Fix

Reinstall the correct build using the validated install flow from:

- docs/applications/programming-languages/python/gpu/install.md

## Symptom — `$toolRoot` or `New-Item` is not recognized

### Meaning

The session is still in `cmd.exe` and not yet in PowerShell.

### Fix

Start PowerShell first:

```powershell
powershell
```

Then rerun the PowerShell commands.

## Symptom — toolchain Python still reports `False` outside the sandbox

### Meaning

The issue is not primarily Sandboxie. The toolchain runtime itself is still not correctly aligned with the GPU-capable host state.

### Fix

Do not continue tweaking sandbox settings first.

Instead:

1. make the exact toolchain interpreter return `True` on the host
2. only then retest inside the sandbox

## Symptom — toolchain Python returns `True` on the host but `False` in the sandbox

### Meaning

At that point the remaining variable is the sandbox boundary, not the runtime build.

### Next action

Retest with the compatibility sandbox profile documented in:

- docs/applications/programming-languages/python/general.md

## Related documents

- docs/applications/programming-languages/python/general.md
- docs/applications/programming-languages/python/gpu/general.md
- docs/applications/programming-languages/python/gpu/install.md
- docs/applications/programming-languages/python/gpu/uninstall.md
