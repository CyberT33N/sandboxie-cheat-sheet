# UsePrivacyMode: Host-User-Space vs. Box-Root

## Scope

This document explains a recurring Sandboxie architecture problem:

- a boxed workflow mirrors or stages runtime files under a host-visible user-space path
- `UsePrivacyMode=y` is enabled
- processes inside the box later fail to see or execute those files unless explicit file-access exceptions are added

This is a troubleshooting and architecture note, not a single-application workaround.

## Problem statement

With `UsePrivacyMode=y`, Sandboxie treats user-space as a default-blocked area.
That includes paths such as:

- `C:\Users\<user>\AppData\Roaming\...`
- `C:\Users\<user>\AppData\Local\...`
- other user-profile paths below `C:\Users\...`

If boxed processes later need to execute or discover files staged there, they may fail unless those paths are explicitly exposed through Sandboxie rules such as `NormalFilePath`.

Typical symptoms include:

- executables not being found by `where.exe`
- process initialization failures
- permission-denied behavior that disappears after adding path-specific visibility rules

## Why this happens

The important distinction is not only "host vs sandbox", but also the *path class* that Sandboxie applies under Privacy Mode.

### Host-specific user-space paths

When mirrored artifacts are placed below user-space paths like `AppData`, they inherit the Privacy Mode default-block behavior of that path family.

That means:

- the files may exist as part of the boxed working state
- but later boxed processes do not automatically get executable visibility to them
- the operator is forced to add `NormalFilePath` or stronger exceptions

This is where the architectural smell appears: the system starts depending on host-user-space exceptions just to make an otherwise local boxed runtime usable.

## Why this is an anti-pattern

From an enterprise, domain-driven, and 12-factor perspective, this is the wrong place for mirrored execution surfaces.

Why:

1. The execution plane becomes coupled to host-user-space semantics instead of a stable boxed runtime boundary.
2. Reproducibility becomes worse because path visibility depends on additional Sandboxie exceptions.
3. Operational drift becomes more likely because every new mirrored executable family may require new path rules.
4. The system begins to normalize `NormalFilePath` / `OpenFilePath` workarounds for paths that should have stayed local to the boxed runtime domain.

In short:

- **host-user-space mirror path** = tactical workaround
- **box-root-aligned mirror path** = correct architecture direction

## Why a box-root-aligned path behaves differently

If the mirrored execution surface is placed under a path class that aligns with the boxed system/runtime domain, the Privacy Mode user-space problem does not arise in the same way.

Examples:

- a box-root-aligned system path
- a dedicated sandbox image root
- another boxed path family that does not rely on host-user-space visibility

The key architectural property is:

- the mirror is still box-local working state
- but it is *not modeled as host-user-space content*

That removes the need to keep reopening host-user-space boundaries just to run the mirrored binaries.

## Correct architectural rule

For boxed-owned-toolchain style architectures:

### Golden source

Keep canonical, versioned source artifacts in the shared toolchain root, for example:

- runtime binaries
- canonical extension store
- canonical settings catalog
- canonical seed content

### Local working state

Mirror and execute mutable working copies from a box-root-aligned local path class.

That applies especially to:

- locally executed mirrored runtimes
- local toolchain wrappers
- local native caches
- local extension/runtime working state

### Promotion

If local work should become canonical again, do not write live against the shared source.
Instead use an explicit promotion step.

## Boundary guidance

### Not recommended

- adding `NormalFilePath` to host-user-space paths such as `AppData\...` as the main architecture
- adding `OpenFilePath` to those paths for convenience
- treating those exceptions as a normal baseline

### Recommended

- keep the canonical source in `Shared`
- keep live execution and mutable state box-local
- choose a box-root-aligned mirror destination
- use explicit promotion back to `Shared` only when canonical state should be updated

## Example interpretation

A boxed workflow may mirror runtimes successfully into:

```text
C:\Users\<user>\AppData\Local\...
```

but still fail later when boxed processes try to discover or execute them.

That does **not** prove the mirror itself is wrong.
It proves the chosen mirror destination is in the wrong path class under `UsePrivacyMode=y`.

## Decision

If a mirrored execution surface only works after opening host-user-space path exceptions, that should be treated as an architectural warning signal.

The preferred fix is:

1. move the mirror to a box-root-aligned local path
2. keep execution local
3. keep `Shared` canonical
4. use explicit promotion when publishing canonical updates

## Related

- `docs\box-types\red.md`
- `docs\troubleshooting\error-codes\SBIE1231.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\architecture\governance.md`
