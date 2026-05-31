# Host Sync

## Status

This document preserves the older host-sync / host-visible Git baseline.

It is **not** the preferred architecture path anymore.

The recommended Git architecture for this repository is:

- `docs\applications\git\boxed-owned-toolchain\overview.md`

## Why this document still exists

This file is kept as a legacy host-sync reference because older flows in this repository still depended on host-visible Git for prompt integration and host-synced workspaces.

That is no longer the preferred default, but the reference remains useful while old material is being retired.

## INI settings

```ini
# --- Git (needed by Starship prompt) ---
NormalFilePath=starship.exe,C:\Program Files\Git\
NormalFilePath=git.exe,C:\Program Files\Git\
```

## Interpretation

This snippet belongs to the host-sync / legacy family:

- host-visible Git
- host prompt integration
- non-preferred architecture direction

Do not reuse this as the default baseline for the boxed-owned-toolchain method.

## Related

- `docs\applications\git\general.md`
- `docs\applications\git\boxed-owned-toolchain\overview.md`