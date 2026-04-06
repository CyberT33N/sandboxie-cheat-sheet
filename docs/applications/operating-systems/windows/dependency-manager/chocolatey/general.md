# Chocolatey

## Architectural status

This document separates the recommended toolchain-based binary mirroring model from the legacy host-virtualization model.

Chocolatey is not the recommended active runtime boundary for the current Sandboxie toolchain architecture in this repository.

## Recommended pattern — mirror Chocolatey-provided binaries into the toolchain

Use Chocolatey only as a discovery or source location when a required Windows binary originally comes from a host-side Chocolatey installation.

The recommended model is:

- keep the runtime boundary inside the dedicated shared toolchain root
- copy the required binary or binary tree into the toolchain `tools` area
- execute the mirrored binary from the toolchain instead of from a host `ReadFilePath`

Recommended destination patterns:

```text
C:\shared\sandbox-toolchains\python-general\tools\bin\<tool>.exe
C:\shared\sandbox-toolchains\python-general\tools\<tool>\bin\<tool>.exe
```

Example posture for Sandboxie:

```ini
OpenFilePath=C:\shared\sandbox-toolchains\python-general\
```

With this model, host-side Chocolatey paths do not remain part of the active runtime boundary.

## Why the recommended pattern exists

- the runtime stays explicit and reproducible
- the box no longer depends on a host package-manager boundary during execution
- external binaries can be versioned and mirrored together with the Python toolchain layout
- the host-installed package remains optional instead of becoming an always-on execution dependency

## Legacy host-virtualization reference (not recommended)

This legacy variant remains documented only when an older host-shaped setup must be mirrored intentionally.

It is an anti-pattern for the current recommended architecture.

```ini
ReadFilePath=C:\ProgramData\chocolatey\bin\
ReadFilePath=C:\ProgramData\chocolatey\lib\ffmpeg\tools\ffmpeg\bin\
```

## Related documents

- docs/applications/programming-languages/python/general.md
- docs/applications/programming-languages/python/powershell-scripts.md
- docs/applications/tools/media/ffmpeg/general.md
