# Node Runtime

## Scope

This folder is the Node-runtime documentation area for this repository.

It owns:

- architecture-specific runtime placement
- version selection
- shared-runtime versus host-runtime boundaries
- binary-specific provisioning and path truth

## Architecture split

### Preferred

- `docs\applications\programming-languages\node\runtime\architectures\boxed-owned-toolchain\overview.md`

### Secondary / legacy reference

- `docs\applications\programming-languages\node\runtime\architectures\host-sync\overview.md`

## Current boundary

The important split in the current repository state is:

- boxed-owned-toolchain documents the currently validated shared-runtime contract that the boxed bootstrap scripts actually consume
- host-sync continues to own large parts of the older Node operational and dependency-install history

So this Node runtime area must not be read as if every Node-related workflow in the repository were already validated under boxed-owned-toolchain.

## Related

- `docs\applications\programming-languages\node\nvm\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\node.md`
