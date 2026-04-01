# Python Manager — pyenv

> Legacy reference only.
>
> This document is intentionally kept for archive purposes because the historical commands worked in an earlier host-virtualization scenario.
>
> It is not the recommended architecture for the current Python-in-Sandboxie model in this repository.

## Architectural classification

This method belongs to the host-virtualization approach.

That means:

- Python is managed primarily on the host
- packages may be installed on the host
- the sandbox later relies on host paths, host shims, and host-managed runtime state

This is explicitly not the recommended baseline.

## Why this method is not recommended

- it increases host-side exposure during package installation
- it depends on host-managed `pyenv` shims and version state
- it is less reproducible than the dedicated shared toolchain-root model
- it becomes fragile when native dependencies and more complex CLI behavior are involved

## Archived host-virtualization example

```ini
ReadFilePath=C:\Users\denni\.pyenv\pyenv-win\versions\3.9.13\
ReadFilePath=C:\Users\denni\.pyenv\pyenv-win\shims\
ReadFilePath=C:\Users\denni\.pyenv\pyenv-win\bin\
ReadFilePath=C:\Users\denni\.pyenv\pyenv-win\versions\3.9.13\Scripts\
```

## Archived usage note

The historical method worked when the sandbox consumed a host-installed runtime and the host-side package state happened to align with the boxed execution scenario.

That does not make it the correct long-term architecture.

## Recommended replacement

Use the dedicated shared toolchain-root model documented here:

- docs\applications\programming-languages\python\general.md
- docs\applications\programming-languages\python\cli.md
- docs\applications\programming-languages\python\dependencies.md
- docs\applications\programming-languages\python\versioning.md
