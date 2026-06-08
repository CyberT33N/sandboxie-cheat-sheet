# Boxed-Owned-Toolchain Python

## Status

This is the preferred Python architecture path for the current repository direction.

## Current shared runtime contract

The current shared Python runtime surface is:

```text
C:\shared\sandbox-toolchains\dev\python\
  current.txt
  3.14.5\
    python.exe
```

The current selected version is read from:

```text
C:\shared\sandbox-toolchains\dev\python\current.txt
```

Bootstrap then mirrors that selected version into the local box execution tree.

## Provisioning contract

The current shared Python runtime contract is version-pointer based:

1. place the selected runtime under `C:\shared\sandbox-toolchains\dev\python\<version>\`
2. update `current.txt` so bootstrap can resolve the active version

Current selected version:

- `3.14.5`

## Bootstrap consumption

The current Python runtime mirror implementation lives in:

- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\python\Bootstrap.Python.psm1`

That layer:

- reads `current.txt`
- validates the selected shared runtime
- mirrors it into the local box execution tree
- prepends the local mirrored Python root into `PATH`

## Why this is the preferred path

This keeps Python aligned with the same boxed-owned-toolchain rules used for the other governed runtimes:

- shared versioned source artifacts
- local mirrored execution
- host-independent runtime truth
- bootstrap-owned process wiring

## What this owns

This architecture owns the governed Python binary truth for the boxed-owned-toolchain method.

Application-specific package installation, wrapper automation, GPU specifics, and troubleshooting remain documented in the broader Python area.

## Related

- `docs\applications\programming-languages\python\general.md`
- `docs\applications\programming-languages\python\architectures\host-sync\overview.md`
- `docs\applications\programming-languages\python\powershell-scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
