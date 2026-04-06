# FFmpeg

## Architectural status

This document separates the recommended toolchain placement of FFmpeg from the legacy host-virtualization pattern.

FFmpeg should not remain an active host-side `ReadFilePath` dependency when the Python workload already executes from the dedicated toolchain root.

## Recommended pattern — mirror FFmpeg into the toolchain

When FFmpeg is needed by a toolchain-based Python workload, copy the real FFmpeg binary payload into the toolchain `tools` area and execute it from there.

Recommended toolchain destination:

```text
C:\shared\sandbox-toolchains\python-general\tools\ffmpeg\bin\ffmpeg.exe
```

### How to find the source binary in a host Chocolatey installation

The Chocolatey launcher or shim is typically visible here:

```text
C:\ProgramData\chocolatey\bin\ffmpeg.exe
```

The real FFmpeg package binary is typically here:

```text
C:\ProgramData\chocolatey\lib\ffmpeg\tools\ffmpeg\bin\ffmpeg.exe
```

Copy the real binary payload into the toolchain destination rather than keeping the host package path as a live runtime dependency.

### Runtime expectation

The wrapper script or session bootstrap should add the mirrored FFmpeg directory to `PATH`:

```powershell
$ffmpegBin = "$toolRoot\tools\ffmpeg\bin"
$env:PATH = "$toolScripts;$ffmpegBin;" + $env:PATH
```

## Why the recommended pattern exists

- the Python runtime and FFmpeg remain inside the same explicit toolchain boundary
- the box no longer depends on host Chocolatey during execution
- the copied external binary becomes part of the same reproducible toolchain story as Python, packages, caches, and work directories

## Legacy host-virtualization reference (not recommended)

This legacy variant remains documented only when an older host-shaped setup must be mirrored intentionally.

It is not the recommended architecture.

```ini
ReadFilePath=C:\ProgramData\chocolatey\lib\ffmpeg\tools\ffmpeg\bin\
```

## Related documents

- docs/applications/operating-systems/windows/dependency-manager/chocolatey/general.md
- docs/applications/programming-languages/python/general.md
- docs/applications/programming-languages/python/powershell-scripts.md
