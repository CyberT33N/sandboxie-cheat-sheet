# High-File-Count Workloads

## Architectural status

This document is the cross-cutting performance reference for high-file-count workloads inside Sandboxie in this repository.

It is intentionally **not** application-specific and **not** limited to a single package manager.

## Why this lives under `docs\performance\...`

The issue documented here is a general filesystem and cache-placement concern:

- it affects multiple technology stacks
- it is not tied to one specific error code
- it is not limited to one package or one framework
- it is primarily about architecture and throughput, not a one-off defect

For that reason, this belongs in a dedicated top-level performance area rather than only in troubleshooting.

## Problem statement

Inside Sandboxie, workloads that create, rewrite, link, or unpack **very many files** can become significantly slower when those files are written inside the sandboxed path domain.

This typically affects:

- package-manager stores
- dependency trees
- native build outputs
- caches
- temporary extraction trees

The slowdown is especially visible for operations such as:

- `pnpm install`
- `pnpm rebuild`
- native addon compilation
- large lockfile / dependency-tree materialization

## Why it becomes slow

### 1. Sandboxie file virtualization

When file-heavy workloads remain inside the sandboxed write domain, Sandboxie must mediate and virtualize a large number of filesystem operations.

This cost grows quickly when:

- thousands of files are created
- many small files are unpacked
- files are rewritten repeatedly
- native build tools generate deep output trees

### 2. Microsoft Defender real-time scanning

On standard NTFS host paths, Microsoft Defender real-time protection scans file activity synchronously.

That means:

- many file operations are scanned while they are happening
- high-file-count install/build flows can slow down further

This repository treats that as an architectural fact that must be accounted for during sandbox design.

## Recommended pattern

For high-file-count, performance-sensitive, non-business-data surfaces, the recommended pattern is:

- keep the business workspace / source tree governed
- move large caches and stores into an explicit shared host path
- expose only those specific paths through narrow Sandboxie rules

Typical examples:

- PNPM content-addressable store
- Nx native cache
- shared Python build helper root
- package-manager caches
- other high-churn temp / cache paths

## Why host-shared cache paths help

When a heavy cache/store path is explicitly placed on the host and opened through a narrow rule such as `OpenFilePath`, the expensive sandboxed file-virtualization layer is avoided for that specific path.

That usually leads to much better throughput for:

- unpacking
- linking
- cache reuse
- repeated reinstall / rebuild cycles

## What should be moved to the host-shared area

Good candidates:

- package-manager stores
- native caches
- rebuild caches
- temp unpack areas

Bad candidates:

- sensitive business data
- general user-content roots
- broad uncontrolled application work surfaces

The goal is **not** "open the whole host for speed".

The goal is:

- keep the write surface narrow
- move only clearly infrastructural, high-churn, high-file-count paths out of the sandbox write domain

## Dev Drives

Microsoft documents Dev Drives as a performance-oriented volume type that uses Microsoft Defender performance mode, which scans file opens **asynchronously** ("open now, scan later") on trusted Dev Drives.

In this repository's governed Sandboxie posture, Dev Drives are **not** the recommended baseline for these workloads.

Why:

- they intentionally trade synchronous scan timing for speed
- that is a looser protection posture than the standard synchronous scan path
- this repository prefers explicit shared cache/store placement over changing the scan semantics of a whole volume

So for the current documented architecture here:

- **recommended**: explicit shared host cache/store paths with narrow Sandboxie access rules
- **not recommended as the default baseline**: using Dev Drive performance mode as the main answer to install/build slowness

## Current rule of thumb

If a workload is:

- high-file-count
- cache-like
- reproducible
- infrastructural rather than business data

then prefer:

- a dedicated shared host path
- a narrow explicit Sandboxie rule

instead of:

- keeping the whole workload inside sandboxed storage
- or relaxing protection more broadly at the volume level

## Related documents

- `docs\applications\programming-languages\node\package-manager\pnpm\troubleshooting\performance.md`
- `docs\applications\IDE\vscode\methods\host-sync\templates\node-monorepo-materialized-dependencies.md`
