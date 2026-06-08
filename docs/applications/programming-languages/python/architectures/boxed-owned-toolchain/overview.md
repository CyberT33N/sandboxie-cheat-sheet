# Boxed-Owned-Toolchain Python

## Status

This path exists because the repository-wide preferred direction is the boxed-owned-toolchain method.

However, for the Python domain specifically, there is **not yet** a validated boxed-owned-toolchain workflow in this repository.

## What this means

At the current state:

- do **not** treat the Python domain as having a fully verified boxed-owned-toolchain runtime/package workflow
- do **not** copy host-sync Python guidance into this path and relabel it as boxed-owned-toolchain truth
- do **not** infer a validated boxed Python application workflow merely from optional Python hooks in shared bootstrap scripts

## Current documentation boundary

The current detailed Python runtime/package/install guidance in this repository remains on the host-sync side:

- `docs\applications\programming-languages\python\architectures\host-sync\overview.md`
- `docs\applications\programming-languages\python\architectures\host-sync\dev-python-build-helper.md`

That host-sync material covers the currently documented Python operational truth.

## Repository-level preference versus Python-domain validation

The important distinction is:

- **repository-level preference**: boxed-owned-toolchain remains the preferred overall architecture direction
- **Python-domain validation state**: a Python-specific boxed-owned-toolchain workflow has not yet been verified enough to become detailed reference truth

## Method-level note

The boxed-owned VS Code bootstrap currently contains optional Python runtime hooks.

That is a method-level implementation detail, not a Python-domain proof that package installation, dependency behavior, GPU flows, or Python application execution are already validated under boxed-owned-toolchain.

## Related

- `docs\applications\programming-languages\python\general.md`
- `docs\applications\programming-languages\python\architectures\host-sync\overview.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\python.md`
