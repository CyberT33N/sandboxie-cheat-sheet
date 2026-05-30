# Extensions Seeds And Local State

## Scope

This document explains how the boxed-owned-toolchain method separates:

- canonical shared state
- seed material
- box-local runtime state

for extensions and path-hardcoded data.

## Canonical shared extension store

The canonical extension store lives here:

```text
C:\shared\sandbox-toolchains\ide\vscode\extensions\
```

This store is authored by the Maintenance Box.

## Why project boxes do not use it directly as live runtime

The project box does **not** point `--extensions-dir` directly at the shared extension store during normal runtime.

Why:

1. VS Code attempted to write under the shared extension path when that directory was used directly as the project runtime `--extensions-dir`.
2. Granting project boxes write rights there would create a multi-writer canonical extension store.
3. That would violate the Maintenance Box contract.
4. Project boxes are consumers, not global authors.

## Final extension runtime contract

Therefore the final runtime contract is:

- Maintenance Box writes to the canonical shared extension store
- project bootstrap mirrors that shared store into a box-local extension directory
- the project VS Code instance runs with the local mirrored `--extensions-dir`

This is why the sync architecture exists.

It preserves:

- single-writer governance
- auditability
- reproducibility
- disposable project boxes

## Box-local extension runtime copy

The project box uses a local mirrored extension runtime copy, for example:

```text
%APPDATA%\VSCodeBoxes\test-mono\extensions
```

That local directory is the runtime `--extensions-dir` for the project box.

## What remains box-local

The following remain box-local:

- `user-data`
- mirrored extension runtime copy
- `globalStorage`
- `workspaceStorage`
- logs
- cache
- sessions
- `.roo`

## Seed model for hardcoded or difficult paths

For extension data that cannot be redirected cleanly, the method uses:

- a canonical seed in shared
- a local copy in the box
- explicit promotion later if needed

## Canonical seed paths

### `.roo`

Canonical seed:

```text
C:\shared\sandbox-toolchains\ide\vscode\catalog\seed\roo\
```

Live runtime path:

```text
C:\Users\denni\.roo
```

### Extension-managed `globalStorage`

Canonical seed root:

```text
C:\shared\sandbox-toolchains\ide\vscode\catalog\seed\globalStorage\
```

Live runtime path:

```text
<box-local user-data>\User\globalStorage\
```

## Seed initialization behavior

The current method initializes seed-backed paths when missing.

That means:

- the canonical seed remains the source of truth
- the box receives a local copy
- the box-local runtime path remains the live working state

## Examples of canonical assets

Canonical shared surfaces in this area include:

- extension binaries
- seed-backed `globalStorage` payloads
- seed-backed `.roo` content

The box-local runtime state then contains the live mutable copy used by the project box.

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\architecture\governance.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\maintenance-box.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\project-box.md`
