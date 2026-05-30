# VS Code Methods

## Method #1 - Boxed-Owned Toolchain / Boxed Authoring (Preferred)

This is now the preferred method for this repository.

Core principles:

- no regular VS Code installation remains on the host as part of the normal workflow
- each project gets its own dedicated boxed VS Code instance
- a dedicated Maintenance Box owns the canonical shared IDE assets
- shared paths hold runtime, catalog, extensions, seeds, and toolchains
- live writable project state remains box-local
- edge-case projects are still developed boxed and can later execute through dedicated runner contexts

Read here first:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\start.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`

## Method #2 - Host-Sync / Materialization (Second Recommendation)

This remains the secondary recommended method when the IDE must stay on the host.

Core model:

- VS Code stays on the host
- dependencies are installed in a dedicated install box
- the dependency tree is materialized onto the host-visible workspace path
- daily execution happens in a separate run box

Read here:

- `docs\applications\IDE\vscode\methods\host-sync\general.md`
- `docs\applications\IDE\vscode\methods\host-sync\dependencies-installed-in-box.md`
- `docs\applications\IDE\vscode\methods\host-sync\templates\node-monorepo-materialized-dependencies.md`

## Method #3 - Host-Installed Toolchain / Host Dependency Installation (Not Recommended)

This is the anti-pattern for the current repository direction.

Core problem:

- package-manager install scripts execute on the host
- the host becomes the dependency execution target
- the host becomes the active source of truth for dependency side effects

Keep this only as a legacy reference:

- `docs\applications\IDE\vscode\methods\host-sync\dependencies-installed-on-host.md`


## Method ranking summary

1. **Boxed-Owned Toolchain / Boxed Authoring**  
   Preferred default architecture.
2. **Host-Sync / Materialization**  
   Secondary recommendation when the IDE must remain on the host.
3. **Host-Installed Toolchain / Host Dependency Installation**  
   Legacy anti-pattern / not recommended.

## Related terminal note

The classic IDE `Debug Script` behavior in `package.json` remains method-specific and is documented separately here:

- `docs\applications\IDE\vscode\terminal\debug\package-json\debug-scripts-not-working.md`

## Related

- `docs\applications\IDE\vscode\terminal\debug\general.md`
- `docs\applications\IDE\vscode\terminal\package.json vs new terminal.md`
- `docs\applications\operating-systems\windows\terminal\powershell\general.md`
