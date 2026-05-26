# Variant #2 - Dependencies Installed in Box

## Architectural status

This is now the **recommended** variant for governed PNPM monorepos in this repository, but only in its validated form:

- dependencies are installed in a dedicated **install box**
- the dependency tree is **materialized to the real host-visible workspace path**
- daily execution happens in a separate **run box**

This is no longer the old "dependencies only inside the box" idea.
That earlier interpretation remains invalid.

## Definition

In this variant, dependency installation runs inside a Sandboxie install box, for example through `pnpm install` in a boxed terminal.

The goal is to prevent dependency install scripts from executing directly on the host system while still keeping the host IDE operational.

## Core problem of the old box-only idea

This variant conflicts with the host-not-isolated editor model if it is used without an additional host-visible materialization architecture.

The reason is simple:

- VS Code or Cursor still runs on the host
- host-side language services still resolve packages from the host-visible workspace
- if `node_modules` exists only inside the box, the host editor cannot reliably resolve the dependency tree

So even if runtime execution inside the box can be made to work for specific commands, the overall development model remains incomplete until the host can see a consistent dependency tree.

## What went wrong in practice

When dependencies existed only inside the box, multiple failure patterns appeared:

- host-side `node.exe` or host-side wrappers were still used in parts of the command chain
- native modules and framework-specific runtime helpers could no longer be resolved cleanly across the host/box boundary
- host VS Code could not reliably resolve packages and types because the dependency tree was not present at the real host path
- monorepo, Vite, Electron-Vite, Angular, and similar toolchains introduced additional path-resolution and wrapper complexity
- `nvm use` inside a privacy box rewired shim/symlink state in a way that was not stable for the boxed workflow
- `pnpm.ps1` and other PowerShell shim surfaces added extra avoidable failure points

This is why the issue was not just one missing allow rule. The architectural mismatch remained:

- execution could be boxed
- but the host IDE still expected host-visible dependencies

## Important consequence

A purely boxed dependency tree is not enough in the host-not-isolated method.

If the editor stays on the host, you still need a host-visible dependency tree for:

- package resolution
- type resolution
- IntelliSense
- editor diagnostics
- host-side tooling that operates outside the box context

## Validated architecture

The validated architecture is:

1. **Host VS Code / Cursor** remains the control plane
2. **Install Box** runs `pnpm install`, `pnpm update`, `pnpm rebuild`, and similar dependency commands
3. **Run Box** runs backend, frontend, Electron, tests, watchers, and other daily execution flows
4. the install box **materializes** the dependency tree directly to the real host-visible workspace path
5. any framework-specific external runtime payloads that are not reliably materialized in-repo (for example an Electron binary tree) are mirrored into a dedicated shared toolchain path and then referenced explicitly during execution

## What "host-visible materialization" means

It does **not** mean:

- a symlink into `C:\Sandbox\...`
- ad-hoc manual copying after each install
- keeping the dependency tree only inside the sandbox and hoping the editor will still work

It means:

- the install box writes the intended dependency-owned surfaces directly to the host-visible project path
- the host editor sees the same resulting `node_modules` / `.pnpm` tree that the boxed runtime expects
- the write boundary is narrow and intentional

## Why manual copy is risky

Manual copying may appear simple, but it creates real operational hazards:

- the copy can become stale without obvious symptoms
- the editor can read old type declarations while the boxed runtime uses newer dependencies
- drift can remain invisible until a breaking change or subtle runtime mismatch appears

Example:

- the box installs Zod version 4
- the host mirror still contains Zod version 3
- the host editor resolves version 3 types
- runtime execution uses version 4 behavior

That is a version-drift failure and must be treated as a first-class architectural risk.

## PNPM-specific complications

This risk becomes worse with `pnpm`, especially in monorepos and workspaces:

- the dependency layout is more complex than a flat `npm` tree
- symlink, hardlink, and workspace resolution can make naive copying unsafe
- the result can differ depending on how the tree was materialized and how the copy was performed
- strict governance surfaces such as `verifyDepsBeforeRun`, `nodeLinker: isolated`, shared lockfiles, catalogs, and explicit build approvals make half-synchronized states fail-closed instead of merely becoming flaky

With `pnpm`, you must test the exact workflow you intend to rely on.

## Operational rules

The validated workflow depends on a few strict rules:

- do **not** use `nvm use` inside the boxes
- use the fixed versioned `node.exe` / `pnpm.cmd` from the real NVM home
- prefer `pnpm.cmd` or `cmd.exe`-driven task surfaces over `pnpm.ps1` inside boxed workflows
- keep the install box and the run box separate
- clear old box contents when testing new architecture states so stale boxed artifacts do not fake success

## From-scratch process summary

1. Prepare the shared toolchain root under `C:\shared\sandbox-toolchains\...`
2. Create the dedicated install and run shell launchers
3. Configure the install box and the run box
4. Delete existing host `node_modules` / `.pnpm` surfaces before validating the materialization flow
5. Run the install command in the install box using the fixed versioned toolchain
6. Validate that the dependency tree now exists on the real host workspace path
7. Start backend/frontend/Electron from the run box
8. For framework-specific exceptions, use the smallest documented overlay necessary

## Config boilerplates

Full generic monorepo boilerplates live here:

- `docs\applications\IDE\vscode\methods\host-not-isolated\templates\node-monorepo-materialized-dependencies.md`

Electron / Electron-Vite overlays live here:

- `docs\applications\programming-languages\node\dependencies\frameworks\electron\general.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\electron-vite\general.md`

## Testing is mandatory

If you use this variant, you must test at least:

- editor package resolution
- editor type resolution
- direct Node binary execution
- package-manager-driven script execution
- monorepo workspace commands
- framework-specific development commands
- native module loading
- debug mode and watch mode
- dependency updates and materialization refresh behavior

Do not assume that a successful install alone proves that the architecture works.

## Bottom line

This variant is now the recommended method for governed PNPM monorepos in this repository, but only in its **install-box + host-visible materialization + run-box** form.
