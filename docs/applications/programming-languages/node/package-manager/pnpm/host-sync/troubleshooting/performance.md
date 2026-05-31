# Host-Sync Performance

## Scope

This page documents a **host-sync-specific** performance pattern.

It does **not** describe the preferred boxed-owned-toolchain baseline.

It exists to preserve the technical behavior of the old host-sync `--store-dir` experiments and to explain why they may look faster while still remaining the wrong default architectural answer.

## Important correction

The earlier project intuition around slow `pnpm install` runs was over-attributed to PNPM store placement.

The first diagnostic question must now always be:

- are broad Sandboxie debug traces still enabled?

That cross-cutting repository lesson is the main prerequisite and lives here:

- `docs\performance\filesystem\sandboxie-debug-tracing.md`

Only after tracing is disabled and the box is restarted cleanly should store placement be discussed.

## What is technically true

A host-shared PNPM store can improve throughput in some host-sync flows because:

- fewer package-manager writes happen inside the sandboxed write domain
- cached payloads may be reused across reinstall cycles
- repeated downloads and unpack steps may be reduced

So the performance effect itself can be real.

## Why this is still not the recommended fix

Even if the throughput gain is real, the host-shared PNPM store remains an anti-pattern from a security and governance perspective:

1. compressed package payloads are written into a host-visible shared store
2. multiple boxes may later consume those same payloads
3. trust and blast radius are therefore widened beyond a single box-local install state
4. one compromised or drifted cache/store surface can propagate across boxes

That is why this repository no longer recommends the shared store as the baseline solution.

The preferred baseline is:

- per-box default store
- normal `pnpm install`
- no cross-box shared PNPM content-addressable store

## Legacy host-sync reference

The old host-sync shared-store flow is kept here only as a legacy technical reference.

### Example shared store

```text
C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store\
```

### Example install command

```powershell
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd" install --store-dir "C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store"
```

### Example rebuild command

```powershell
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd" rebuild --store-dir "C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store"
```

These commands remain useful for:

- reading older notes and handoffs
- reproducing the legacy host-sync variant
- understanding the historical performance experiments

They are not the repository's current recommended path.

## Native addon caveat

On a fresh install, `pnpm install` may still trigger `node-gyp` for dependencies with `binding.gyp` / `gypfile: true`.

If the shell has not been bootstrapped with the required Python and Visual Studio context, failures may still occur independently of the store discussion.

For that overlay, read:

- `docs\applications\programming-languages\node\dependencies\node-gyp\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\clean-reinstall.md`

## Interpretation

So the correct interpretation order is:

1. first disable trace wildcards and retest
2. then decide whether the observed throughput issue was mostly debug-state noise
3. only then treat host-shared cache/store placement as a technical performance lever
4. even then, keep it categorized as a host-sync legacy anti-pattern, not as the preferred repository answer

## Related

- `docs\applications\programming-languages\node\package-manager\pnpm\host-sync\overview.md`
- `docs\performance\filesystem\sandboxie-debug-tracing.md`
- `docs\performance\filesystem\high-file-count-workloads.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\clean-reinstall.md`
