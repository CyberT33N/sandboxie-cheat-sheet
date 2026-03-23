# Variant #2 - Dependencies Installed in Box

## Definition

In this variant, dependency installation runs inside the Sandboxie box, for example through `pnpm install` in a boxed terminal.

The goal is to prevent dependency install scripts from executing directly on the host system.

## Core problem

This variant conflicts with the host-not-isolated editor model if it is used without an additional synchronization architecture.

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

## Required architecture

If this variant is chosen, a host-visible mirror or equivalent synchronization strategy becomes mandatory.

That means:

- the dependency tree installed in the box must be copied, mirrored, or otherwise materialized to the host-visible project path
- the host-visible copy must remain consistent with the boxed source of truth
- the synchronization model must be tested as part of the workflow, not assumed

Without this, the variant is operationally incomplete.

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

## pnpm-specific complications

This risk becomes worse with `pnpm`, especially in monorepos and workspaces:

- the dependency layout is more complex than a flat `npm` tree
- symlink, hardlink, and workspace resolution can make naive copying unsafe
- the result can differ depending on how the tree was materialized and how the copy was performed

In theory, a simple copy may be more straightforward with a plain `npm` layout. With `pnpm`, you must test the exact workflow you intend to rely on.

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
- dependency updates and mirror refresh behavior

Do not assume that a successful install alone proves that the architecture works.

## Preferred design direction

The safest form of this variant is:

1. install dependencies inside the box
2. materialize a host-visible mirror in a controlled way
3. ensure the host editor always reads the mirrored dependency tree
4. ensure the mirror is refreshed whenever dependencies change

Any mirror strategy must minimize drift and make refresh behavior explicit.

## Possible implementation direction

One possible direction is a carefully scoped host-visible materialization path, for example through explicit synchronization or a tightly controlled host-write boundary.

If `OpenFilePath` or another host-visible write mechanism is used to support this, it must be:

- narrowly scoped
- intentional
- tested
- documented as part of the architecture

It must not be treated as an incidental workaround.

## When to choose this variant

Choose this variant only when:

- reducing host-side dependency install-script execution is a higher priority than workflow simplicity
- the team accepts the added complexity of synchronization
- a tested mirror strategy exists or will be implemented

## Bottom line

This variant improves host-side installation risk, but it is not self-sufficient. In the host-not-isolated method, dependencies installed only inside the box still have to become host-visible again if the host IDE is expected to work correctly.
