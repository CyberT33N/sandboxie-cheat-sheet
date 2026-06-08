
# Python

## Scope

This folder is the Python-domain documentation area for this repository.

It owns:

- architecture-specific Python runtime placement
- Python package-boundary guidance
- wrapper-automation guidance
- Python-specific troubleshooting and GPU notes

## Architecture split

### Repository direction

At the repository level, the preferred overall VS Code method remains:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\general.md`

That repository-level preference does **not** by itself mean that a Python-specific boxed-owned-toolchain workflow has already been validated.

### Boxed-owned-toolchain

- `docs\applications\programming-languages\python\architectures\boxed-owned-toolchain\overview.md`

Current status:

- this path is kept to represent the preferred repository direction
- but there is not yet a validated Python-domain boxed-owned-toolchain workflow in this repository
- therefore it must **not** be used to claim detailed boxed Python runtime/package guidance that has not been verified

### Host-sync

- `docs\applications\programming-languages\python\architectures\host-sync\overview.md`
- `docs\applications\programming-languages\python\architectures\host-sync\dev-python-build-helper.md`

Current status:

- the currently documented Python runtime/package/workflow material in this repository belongs to the **host-sync** side
- this includes the preserved `python-general` toolchain-root workflow and the shared `dev\python\current.txt` build-helper contract used by host-sync Node build flows

## Current host-sync reference set

Until a Python-specific boxed-owned-toolchain workflow is actually verified, the currently documented Python operational material should be read as host-sync-oriented reference content:

- `docs\applications\programming-languages\python\cli.md`
- `docs\applications\programming-languages\python\dependencies.md`
- `docs\applications\programming-languages\python\powershell-scripts.md`
- `docs\applications\programming-languages\python\versioning.md`
- `docs\applications\programming-languages\python\gpu\general.md`
- `docs\applications\programming-languages\python\troubleshooting.md`

## Important boundary

Do not treat the existence of optional Python hooks in shared boxed bootstrap scripts as proof that a full Python-domain boxed-owned-toolchain workflow has been verified.

The Python domain should only promote boxed-owned-toolchain reference truth after that workflow has been explicitly tested and documented.


