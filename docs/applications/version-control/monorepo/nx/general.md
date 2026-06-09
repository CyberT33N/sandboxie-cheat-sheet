# Nx

## Scope

This folder is the Nx-domain documentation area for this repository.

It owns:

- architecture-specific Nx runtime behavior
- cache placement guidance
- daemon/plugin topology guidance

## Architecture split

### Preferred

- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`

The boxed-owned-toolchain Nx source of truth is now split by concern under that TOC:

- architecture rationale
- cache boundary
- execution surfaces
- runtime contract
- bootstrap integration

### Secondary / legacy reference

- `docs\applications\version-control\monorepo\nx\architectures\host-sync\overview.md`

## Related

- `docs\applications\version-control\monorepo\nx\debugging.md`
