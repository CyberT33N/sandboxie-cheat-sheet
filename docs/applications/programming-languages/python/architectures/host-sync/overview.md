# Host-Sync Python

## Status

This is the secondary / legacy Python architecture path in this repository.

It is not the preferred default.

## Why this file now contains more detail

The previous `docs\applications\programming-languages\python\general.md` contained valid legacy Python architecture content that was removed during normalization.

That content is preserved here to avoid a breaking documentation loss while keeping `python/general.md` as the domain index.

## Additional host-sync module

The current shared `dev\python\current.txt` build-helper contract used by host-sync Node build flows is documented separately here:

- `docs\applications\programming-languages\python\architectures\host-sync\dev-python-build-helper.md`

## Architectural problem statement

Python inside Sandboxie is not just about launching `python.exe`.

The effective runtime boundary includes:

- the copied interpreter
- generated CLI launchers
- installed packages
- native `.pyd` modules
- DLL-backed dependencies
- caches
- working directories

This becomes critical for packages such as `docling`, `torch`, OCR stacks, and image libraries, because these workloads often use native extensions and Windows-facing runtime behavior.

## Architecture options preserved from the legacy Python write-up

### Option 1 - install installers directly inside the sandbox

This option is not the recommended baseline for this preserved legacy path.

In the tested repository context, MSI, MSIX, and related installer flows did not operate cleanly enough inside the sandbox without weakening additional protections.

### Option 2 - install on the host and rely on host virtualization

This option is a legacy pattern and is explicitly not recommended.

Why it is an anti-pattern:

- package installation runs unsandboxed on the host
- compromised dependencies would affect the host directly
- the runtime becomes dependent on host-managed Python state
- the model is harder to reproduce and harder to reason about

### Option 3 - dedicated shared toolchain root with copied runtime binaries

Within the preserved host-sync-era Python write-up, this was the preferred operating model for keeping a visible shared toolchain root while avoiding pure host-installed package execution.

The model is:

- copy the real runtime binaries into a dedicated shared root
- install packages from inside the sandbox into a sandbox-owned user base
- keep runtime, packages, caches, temp files, and work directories under one explicit toolchain area
- avoid host-side package installation as the primary operating model

## Preserved folder structure

```text
C:\shared\sandbox-toolchains\python-general\
  python\
    3.12.9\
      python.exe
  userbase\
  tools\
    bin\
    ffmpeg\
      bin\
        ffmpeg.exe
  cache\
    pip\
    huggingface\
    torch\
    tmp\
  work\
    input\
    output\
```

### Folder responsibilities

- `python\` stores the copied runtime binaries for the selected Python version.
- `userbase\` stores packages and generated CLI launchers from `pip install --user`.
- `tools\bin\` stores mirrored external launchers or generic binary entry points that should live inside the toolchain boundary.
- `tools\<tool>\bin\` stores tool-specific external binary payloads when a dedicated subtree is clearer, for example FFmpeg.
- `cache\pip\` stores pip download cache data.
- `cache\huggingface\` stores model and hub cache data.
- `cache\torch\` stores torch-related cache data.
- `cache\tmp\` stores temporary unpack and reinstall artifacts for large wheels.
- `work\input\` stores input data that the sandboxed tools should consume.
- `work\output\` stores generated outputs.

### PowerShell commands to create the structure

```powershell
$toolRoot = "C:\shared\sandbox-toolchains\python-general"

New-Item -ItemType Directory -Force -Path "$toolRoot\python\3.12.9"
New-Item -ItemType Directory -Force -Path "$toolRoot\userbase"
New-Item -ItemType Directory -Force -Path "$toolRoot\tools\bin"
New-Item -ItemType Directory -Force -Path "$toolRoot\tools\ffmpeg\bin"
New-Item -ItemType Directory -Force -Path "$toolRoot\cache\pip"
New-Item -ItemType Directory -Force -Path "$toolRoot\cache\huggingface"
New-Item -ItemType Directory -Force -Path "$toolRoot\cache\torch"
New-Item -ItemType Directory -Force -Path "$toolRoot\cache\tmp"
New-Item -ItemType Directory -Force -Path "$toolRoot\work\input"
New-Item -ItemType Directory -Force -Path "$toolRoot\work\output"
```

## Preserved workflow summary

### Step 1 - copy the runtime into the toolchain root

Copy the real runtime binaries (`C:\Users\denni\.pyenv\pyenv-win\versions`) for the chosen version into:

```text
C:\shared\sandbox-toolchains\python-general\python\3.12.9\
```

### Step 2 - start PowerShell inside the sandbox

Use PowerShell inside the sandbox when you want direct logs and stable boxed execution.

If the session starts in `cmd.exe`, switch into PowerShell first before using `$toolRoot`, `$env:...`, or `New-Item` syntax.

### Step 3 - set the session variables

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

### Step 4 - install dependencies into the sandbox-owned user base

The dependency installation must use the `--user` flag so that packages and generated launchers are written into the dedicated `userbase` area instead of the base interpreter directories.

```powershell
& $pythonExe -m pip install --user <package-name>
```

If `--user` is omitted, `pip` may place packages under the base runtime and place generated launchers under `python\<version>\Scripts`, which breaks the documented toolchain layout.

### Step 5 - execute the CLI from the dedicated toolchain

Generic pattern:

```powershell
& "$toolRoot\userbase\Python312\Scripts\<tool>.exe" <arguments>
```

### Step 6 - use a PowerShell wrapper script when manual bootstrap repetition should be avoided

The core architecture does not change when a PowerShell wrapper script is introduced.

The wrapper only automates the same toolchain-root rules already defined in this legacy flow, for example:

- reading the selected Python version
- setting the documented environment variables
- adding required toolchain paths to `PATH`
- validating mirrored external binaries such as FFmpeg
- performing optional package checks or package installation
- launching the final Python script or CLI entry point

See:

- `docs\applications\programming-languages\python\powershell-scripts.md`

## Preserved GPU note

GPU-capable workloads require their own documentation set because the deciding factor is not Sandboxie alone.

The exact Python version, the exact `torch` build, and the toolchain-specific reinstall flow must all line up.

When the Python box or toolchain state should be rebuilt from scratch, start with the GPU uninstall flow because it documents the complete uninstall/reset path that often helps to set the box up again cleanly.

See:

- `docs\applications\programming-languages\python\gpu\general.md`
- `docs\applications\programming-languages\python\gpu\install.md`
- `docs\applications\programming-languages\python\gpu\uninstall.md`
- `docs\applications\programming-languages\python\gpu\troubleshooting.md`

## Operational notes

- If a package depends on native `.pyd` or DLL-backed modules, validate it against the documented Python toolchain root and Sandboxie box configuration.
- Host installation remains a legacy method and is intentionally not the recommended architecture.

## Additional legacy references

- `docs\applications\programming-languages\python\python-manager\pyenv\general.md`
- `docs\applications\programming-languages\python\box-presets\strict-protected-host.md`

## Related

- `docs\applications\programming-languages\python\general.md`
- `docs\applications\programming-languages\python\architectures\host-sync\dev-python-build-helper.md`
- `docs\applications\programming-languages\python\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\python\cli.md`
- `docs\applications\programming-languages\python\dependencies.md`
- `docs\applications\programming-languages\python\powershell-scripts.md`
- `docs\applications\programming-languages\python\versioning.md`
