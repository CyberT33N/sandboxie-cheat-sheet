# Host-Sync PNPM Overview

## Architectural status

This document belongs to the **host-sync** architecture only.

It is **not** the preferred PNPM method in this repository.

The preferred method now lives here:

- `docs\applications\programming-languages\node\package-manager\pnpm\boxed-owned-toolchain\overview.md`

This host-sync document remains as a legacy / reference write-up for cases where:

- the IDE stays on the host
- dependency installation happens in a dedicated install box
- the resulting dependency tree is materialized to the host-visible project path

## What still remains true

For governed Node monorepos in this repository, `pnpm` remains the preferred package manager because the governance model is expressed through PNPM-specific surfaces such as:

- the shared workspace lockfile
- `allowBuilds`
- strict engine enforcement
- explicit workspace ownership
- anti-hoisting and reproducibility controls

In this legacy host-sync variant, boxed shells should still prefer the explicit `.cmd` surface instead of PowerShell shims:

```powershell
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd" -v
```

## Shared PNPM store: technical, but anti-pattern

The old host-sync write-up treated a host-shared PNPM store as the validated baseline.

That is no longer the recommended position.

Why it is now treated as an anti-pattern from a security and governance perspective:

1. compressed package payloads are written into a host-visible shared cache/store surface
2. multiple boxes may later reuse those same cached payloads
3. one compromised or drifted cache surface can therefore propagate across box boundaries
4. the blast radius becomes larger than a single box-local install state

So for the current recommended governance posture:

- **preferred**: each box uses its own default PNPM store through a normal `pnpm install`
- **not preferred**: multiple boxes share one host-visible PNPM content-addressable store

## Why the old shared-store commands are kept here

The old `--store-dir` commands are still kept in this document because they remain technically useful as a reference for:

- reproducing the old host-sync architecture
- understanding historical install-box flows
- reading earlier troubleshooting output and handoffs

They should be read as **legacy technical reference**, not as the current recommendation.

## Performance: fact versus recommendation

A host-shared PNPM store can still improve throughput in some host-sync scenarios.

That part is technically true because:

- fewer high-churn writes happen inside the sandboxed write domain
- repeated installs may reuse already downloaded payloads across boxes

But this must **not** be treated as the primary explanation for earlier slow installs in this repository.

The first diagnostic question is always:

- are broad Sandboxie debug traces still enabled?

That cross-cutting performance confounder is documented here:

- `docs\performance\filesystem\sandboxie-debug-tracing.md`

Only after tracing has been removed from the runtime baseline should shared-cache/store placement be discussed at all.

## Why the install box may still open the repo root

The legacy host-sync install-box write-up remains technically relevant in one place:

Workspace commands such as `pnpm add` or `pnpm update` may rewrite the shared root lockfile by creating a temporary file such as `pnpm-lock.yaml.<random>` and then renaming/replacing it into `pnpm-lock.yaml`.

So even if a host-shared `--store-dir` is used, the install box may still need repo-root write access for `node.exe`.

Example historical failure:

```text
[EPERM] EPERM: operation not permitted, rename
'C:\git\test\test-mono\pnpm-lock.yaml.<random>' -> 'C:\git\test\test-mono\pnpm-lock.yaml'
```

## Legacy reference commands

These commands are kept as legacy host-sync reference only.

### Install with shared store

```powershell
Set-Location "C:\git\test\test-mono"
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd" install --store-dir "C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store"
```

### Rebuild with shared store

```powershell
Set-Location "C:\git\test\test-mono"
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd" rebuild --store-dir "C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store"
```

If the workspace uses Nx native bindings during this legacy flow, the old shell also kept:

```powershell
$env:NX_DAEMON = 'false'
$env:NX_NATIVE_FILE_CACHE_DIRECTORY = 'C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native'
```

That is likewise historical reference, not the preferred baseline.

## Native addon / `node-gyp` overlay

If a Windows dependency rebuild in the host-sync install box needs `node-gyp`, use the host-sync overlay here:

- `docs\applications\programming-languages\node\dependencies\node-gyp\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\clean-reinstall.md`

## Sandboxie access rules

The old host-sync write-up also used path rules such as:

```ini
# --- Install-box repo materialization surface ---
OpenFilePath=node.exe,C:\git\test\test-mono\

# --- Legacy shared pnpm store ---
OpenFilePath=node.exe,C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store\
```

Again:

- technically possible
- historically validated
- not the current recommended governance baseline

## Related documents

- `docs\applications\IDE\vscode\methods\host-sync\dependencies-installed-in-box.md`
- `docs\applications\IDE\vscode\methods\host-sync\templates\node-monorepo-materialized-dependencies.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\nvm\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\clean-reinstall.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\host-sync\troubleshooting\performance.md`
- `docs\performance\filesystem\sandboxie-debug-tracing.md`
- `docs\performance\filesystem\high-file-count-workloads.md`