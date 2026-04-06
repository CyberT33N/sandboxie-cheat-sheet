# PowerShell script wrappers

## Single source of truth

This document explains how a PowerShell wrapper script can automate the existing Python toolchain workflow without changing the underlying architecture.

The architectural core remains owned by these documents:

- docs/applications/programming-languages/python/general.md
- docs/applications/programming-languages/python/cli.md
- docs/applications/programming-languages/python/dependencies.md
- docs/applications/programming-languages/python/versioning.md
- docs/applications/tools/media/ffmpeg/general.md

This document is therefore an automation guide, not a replacement architecture.

## Why wrapper scripts are useful

The documented toolchain model often requires the same repeated bootstrap steps every time a new PowerShell session starts inside the sandbox.

A wrapper script is useful when the workload should avoid manually repeating:

- reading the selected Python version
- resolving the dedicated interpreter path
- setting `PYTHONUSERBASE`
- setting cache-related environment variables
- adding toolchain script directories to `PATH`
- adding mirrored external binary directories such as FFmpeg to `PATH`
- validating the runtime and optional external binaries before launch

## What does not change when a wrapper script is introduced

The wrapper script does not change the core rules:

- the Python runtime still lives under `python\<version>\`
- packages still install into `userbase\` with `--user`
- version boundaries still remain explicit
- external binaries still need to be mirrored into the toolchain `tools` area
- host-side `ReadFilePath` execution remains a legacy pattern rather than the recommended baseline

## Recommended responsibilities of a wrapper script

A PowerShell wrapper script may automate the following responsibilities in order:

1. detect the selected Python version
2. resolve the dedicated interpreter path under the toolchain root
3. set toolchain environment variables
4. verify that required runtime files exist
5. verify that required mirrored external binaries exist
6. optionally install or reinstall packages into the toolchain `userbase`
7. start the final CLI or Python script

## Example scenario — application wrapper with mirrored FFmpeg

The following example demonstrates a workload-specific wrapper that keeps the core toolchain rules unchanged while automating them for a real application startup flow.

```powershell
$toolRoot = "C:\shared\sandbox-toolchains\python-general"
$pythonVersion = "3.12.7"
$pythonExe = "$toolRoot\python\$pythonVersion\python.exe"
$toolScripts = "$toolRoot\userbase\Python312\Scripts"
$ffmpegExe = "$toolRoot\tools\ffmpeg\bin\ffmpeg.exe"
$ffmpegBin = "$toolRoot\tools\ffmpeg\bin"

$env:PYTHONUSERBASE = "$toolRoot\userbase"
$env:PIP_CACHE_DIR = "$toolRoot\cache\pip"
$env:HF_HOME = "$toolRoot\cache\huggingface"
$env:TORCH_HOME = "$toolRoot\cache\torch"
$env:TEMP = "$toolRoot\cache\tmp"
$env:TMP = "$toolRoot\cache\tmp"
$env:PATH = "$toolScripts;$ffmpegBin;" + $env:PATH

if (-not (Test-Path $pythonExe)) {
    throw "Toolchain Python not found."
}

if (-not (Test-Path $ffmpegExe)) {
    throw "Mirrored FFmpeg binary not found."
}

& $pythonExe -m pip install --user -r requirements.txt
& $pythonExe main.py
```

## Why the FFmpeg example matters

Some Python applications call external binaries through `PATH` instead of importing a Python package.

In that case, the wrapper script is the correct place to bridge the toolchain runtime with the mirrored external binary location.

For FFmpeg specifically:

- mirror the binary into the toolchain
- add the mirrored `bin` directory to `PATH`
- keep the final application code unchanged when it already expects `ffmpeg` on `PATH`

See:

- docs/applications/tools/media/ffmpeg/general.md
- docs/applications/operating-systems/windows/dependency-manager/chocolatey/general.md

## When to prefer a wrapper script

Prefer a wrapper script when:

- the bootstrap variables are repetitive
- the application must validate both Python and mirrored external binaries before launch
- the project should be easier to start from a boxed PowerShell session
- package checks or package installation should happen as part of a controlled startup path

## When not to treat the wrapper as architecture authority

Do not treat the wrapper script as the primary architecture source.

The wrapper only materializes rules that remain owned by the core Python toolchain documents.

If the architectural rule and the wrapper ever drift apart, the core documents win and the wrapper must be updated.

## Related documents

- docs/applications/programming-languages/python/general.md
- docs/applications/programming-languages/python/cli.md
- docs/applications/programming-languages/python/dependencies.md
- docs/applications/programming-languages/python/versioning.md
- docs/applications/tools/media/ffmpeg/general.md
