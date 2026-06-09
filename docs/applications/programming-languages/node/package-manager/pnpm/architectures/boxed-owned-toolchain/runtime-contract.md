# PNPM Runtime Contract

## Runtime surface

The governed PNPM runtime surface shape is:

```text
C:\shared\sandbox-toolchains\dev\pnpm\<version>\package\bin\pnpm.cjs
```

It is hosted by the governed shared Node runtime rather than by `@pnpm/exe`.

The current project-specific example contract points at `11.5.0`.

## Why not `@pnpm/exe`

`@pnpm/exe` embeds its own Node runtime.

That is incompatible with this method because the runtime truth must remain explicit in the shared versioned Node layer rather than being hidden inside PNPM itself.

## Preferred operational posture

For the current boxed-owned-toolchain architecture:

- use the locally mirrored `pnpm` command surface prepared by bootstrap
- keep the default per-box PNPM store
- do **not** introduce a host-shared PNPM content-addressable store as the baseline

Why:

1. a host-shared store widens the blast radius across boxes
2. cached package payloads become reusable across trust boundaries
3. one compromised or drifted shared store surface can spread to multiple boxes

So the preferred baseline is:

- normal `pnpm install`
- no shared `--store-dir`
- one store per box

## Current reference truth

The current boxed-owned-toolchain PNPM reference truth is:

1. governed shared `pnpm.cjs`
2. local mirrored command surface from bootstrap
3. default per-box store
4. box-local Bash as the validated lifecycle shell path
5. no shared PNPM store as the default baseline

## How the effective PNPM version is selected

In the current boxed-owned-toolchain contract, the generic bootstrap does **not** choose a PNPM version by itself.

Instead:

1. the shared `dev\pnpm\<version>\package\bin\pnpm.cjs` inventory provides the available governed versions
2. the project adapter selects one exact `PnpmCli` path
3. the reusable bootstrap mirrors that selected CLI locally and generates the boxed command surface from it

That means the effective project-box PNPM version is currently selected through the **project bootstrap contract**, not by running `pnpm self-update` or `npm install -g pnpm` inside the box.

Architecturally:

- shared `dev\pnpm\...` = provisioned binary inventory
- project adapter = project-specific version contract
- generic bootstrap = contract materialization and local wrapper generation

## Sandboxie visibility range

When the PNPM version is changed in the project contract, the relevant box would also have to be updated **if** the Sandboxie rule were pinned to one exact PNPM version directory.

That is why the recommended Sandboxie visibility rule is **not**:

```ini
ReadFilePath=C:\shared\sandbox-toolchains\dev\pnpm\11.5.0\
```

but:

```ini
ReadFilePath=C:\shared\sandbox-toolchains\dev\pnpm\
```

Why this is the better governance rule:

1. PNPM often needs security updates
2. version-pinned box rules would otherwise require repeated manual config churn
3. the project contract already selects the exact `PnpmCli` version
4. the broader PNPM subtree rule keeps visibility narrow enough while avoiding needless box-rule edits

So the intended split is:

- **exact version selection** = bootstrap/project contract
- **broad PNPM visibility range** = Sandboxie box rule

## Related

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\versioning-and-provisioning.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\lifecycle-and-command-surface.md`
