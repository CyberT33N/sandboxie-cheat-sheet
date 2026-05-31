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

## Important repository correction

In this repository, high-file-count performance analysis must start with trace hygiene first.

Before treating host-shared caches or stores as the explanation for slowness, first verify that broad Sandboxie debug tracing is disabled and the box has been restarted cleanly.

That cross-cutting prerequisite is documented here:

- `docs\performance\filesystem\sandboxie-debug-tracing.md`

## Performance fact versus governance choice

For high-file-count, performance-sensitive, non-business-data surfaces, moving cache-like paths onto the host can improve throughput.

That part is technically true.

However, in this repository it is **not** the preferred default baseline because a host-shared cache/store also changes the trust boundary and can widen cross-box blast radius.

Typical examples of such high-churn infrastructural surfaces are:

- PNPM content-addressable stores
- Nx native caches
- package-manager caches
- temporary unpack trees
- other reproducible cache-like surfaces

## Why host-shared cache paths can help

When a heavy cache/store path is explicitly placed on the host and opened through a narrow rule such as `OpenFilePath`, the expensive sandboxed file-virtualization layer is avoided for that specific path.

That usually leads to much better throughput for:

- unpacking
- linking
- cache reuse
- repeated reinstall / rebuild cycles

## What may be considered for host-shared placement

Technically plausible candidates:

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

then first ask:

1. are broad trace wildcards already disabled?
2. is this really an infrastructural cache/store surface instead of business data?
3. does the architecture explicitly accept the security trade-off of a host-shared path?

Only if all three answers are satisfactory should a host-shared path even be considered.

For the current preferred governance posture in this repository, box-local caches/stores remain the default baseline and host-shared cache/store placement is a selective legacy optimization, not the main recommendation.

## Related documents

- `docs\performance\filesystem\sandboxie-debug-tracing.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\host-sync\troubleshooting\performance.md`
- `docs\applications\IDE\vscode\methods\host-sync\templates\node-monorepo-materialized-dependencies.md`
