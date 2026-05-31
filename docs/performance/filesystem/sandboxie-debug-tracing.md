# Sandboxie Debug Tracing And Filesystem Performance

## Architectural status

This document is the cross-cutting performance reference for Sandboxie debug tracing overhead in this repository.

It intentionally lives in:

```text
docs\performance\filesystem\
```

because the issue is primarily:

- cross-cutting
- sandbox-level
- file-operation-heavy
- not limited to one IDE, one package manager, or one single error code

It is therefore not just a VS Code note and not only a troubleshooting snippet.

## Problem statement

Sandboxie resource tracing can massively reduce performance when it remains enabled during normal operation.

The impact becomes especially visible for workloads that touch very many files or repeatedly traverse the filesystem, for example:

- `pnpm install`
- `pnpm rebuild`
- `git status`
- large repo scans
- prompt rendering that internally calls Git in big repositories

## Why this matters

In this repository, a large part of the earlier performance intuition was shaped by slow boxed file-heavy workloads.

A later validation showed that active tracing can be a major confounder:

- the box may appear "generally slow"
- but the slowdown may be strongly amplified by active trace settings
- especially when the workload performs large numbers of file and IPC operations

This means performance conclusions must not be drawn from a box that still runs with broad trace rules enabled.

## The problematic settings

The following settings are the primary debug-only tracing surface:

```ini
FileTrace=*
PipeTrace=*
KeyTrace=*
IpcTrace=*
GuiTrace=*
ClsidTrace=*
```

These settings are useful for diagnosis, but they are not part of a normal day-to-day box configuration.

## Governance rule

These tracing settings must be treated as **debug-only**.

That means:

- enable them only when you intentionally need trace evidence
- terminate the box afterwards
- remove or comment them out again before judging normal runtime performance

They must not remain enabled as a default in project boxes, maintenance boxes, install boxes, or run boxes that are used for normal daily work.

## Why performance drops

### 1. High file count workloads already stress the sandboxed write/read path

Sandboxie mediates and virtualizes filesystem operations.

When a workload touches many files, this already adds overhead.

### 2. Tracing adds observation overhead on top of normal mediation

When broad trace rules are active, Sandboxie also has to record and surface those events.

That extra observation cost compounds with:

- repo scans
- package-manager store activity
- repeated status checks
- shell prompt integration that internally runs Git

### 3. The slowdown can look like a product problem when it is actually a debug-state problem

A box with tracing left on can make:

- `git status`
- `pnpm install`
- prompt rendering
- IDE startup

look much slower than the same box in its real normal operating state.

## Verified repository-specific lesson

In the validated VS Code boxed-authoring work:

- broad trace rules were left active in everyday box configs
- `git status` in a large repo was slow enough to block prompt rendering
- this made `Starship` appear to hang
- after removing tracing and retesting, the actual runtime behavior could be evaluated much more fairly

This is why trace hygiene is now treated as an architectural rule.

## Correct operating sequence

When a boxed workload appears too slow:

1. first verify whether broad trace settings are currently enabled
2. if yes, remove/comment them out
3. terminate all processes in that box
4. restart the box in a clean non-tracing state
5. only then perform performance measurements

This sequence must happen before broader architectural conclusions are drawn.

## What this does *not* prove

Removing trace settings does **not** prove that Sandboxie has zero overhead.

Sandboxie still has real mediation overhead for file-heavy workloads.

But removing debug tracing gives you a fairer baseline for deciding:

- whether the box is acceptable as-is
- whether Git/tooling needs local optimization
- whether caches should be externalized
- whether a workflow should move to another method

## Relationship to host-shared caches and stores

Even if tracing is removed, host-shared caches and stores can still improve throughput for some high-file-count infrastructural surfaces.

That technical effect is real, but in this repository it is **not** the default recommendation because host-shared cache/store placement also widens the trust boundary across boxes.

So the decision order is:

1. remove tracing
2. re-measure
3. confirm that the slowdown is still real without debug overhead
4. only then decide whether a host-shared cache/store trade-off is acceptable

## What should *not* be externalized first

A repo working tree is not the same as a package-manager cache.

Do not treat:

- the Git working tree
- the normal project repo itself

as equivalent to:

- pnpm content-addressable store
- Nx native cache
- temp unpack areas

Those are different architectural surfaces with different trust and governance implications.

## UseRamDisk

`UseRamDisk` can improve throughput, but it is not the preferred default for persistent project boxes because it is:

- volatile
- RAM-bound
- better suited to temporary or throwaway scenarios

For persistent development boxes, it should not be the first answer to this class of slowdown.

## Recommended interpretation model

When performance is poor, the right order is:

1. remove debug tracing
2. re-measure
3. evaluate workload-specific optimizations
4. only then consider architectural changes such as externalized cache/store paths

## Related documents

- `docs\performance\filesystem\high-file-count-workloads.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\host-sync\troubleshooting\performance.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\sandboxie\maintenance-box.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\sandboxie\project-box.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\sandboxie\observations-and-signals.md`
