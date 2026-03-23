# Variant #1 - Dependencies Installed on Host

## Definition

In this variant, the effective dependency tree used by the editor and by host-visible development flows is installed on the host system.

That means the host-visible `node_modules` tree is the active source of truth for:

- VS Code or Cursor package resolution
- type resolution and IntelliSense
- workspace tooling that resolves packages from the host context
- host-visible wrappers and binaries that ultimately run from the normal project path

## Why it works

This is the operationally simplest variant of the host-not-isolated method.

The host IDE can resolve the workspace without additional synchronization because:

- the dependency tree exists at the real host path
- the host editor sees the same package tree that runtime wrappers expect
- host-visible `node.exe`, package-manager shims, and script wrappers are not forced to cross the host/box dependency boundary

In practice, this is why standard development flows often work again as soon as the dependencies exist on the host-visible project path.

## Security cost

This variant has a real security trade-off:

- package-manager install commands can execute post-install, prepare, install, and similar lifecycle scripts on the host
- the exact behavior depends on the language, package manager, dependency graph, and current project configuration
- those scripts run outside the dedicated box, so the host becomes the execution target

This is the main architectural weakness of the variant.

## When to choose this variant

Choose this variant when:

- development operability is the highest priority
- the IDE must work without additional synchronization layers
- the team wants the lowest-friction workflow
- host-side execution of dependency install scripts is acceptable for the project risk profile

## Workflow

1. Install dependencies so that `node_modules` is materialized on the real host project path.
2. Start the dedicated boxed terminals for execution.
3. Run Node, Angular, Electron-Vite, test, and build commands from the boxed execution plane as documented by the project-specific guides.
4. Keep the editor on the host.

## Advantages

- simplest operational model
- host editor resolves packages and types immediately
- fewer path-resolution edge cases across host and box
- fewer framework-specific failures caused by boxed-only dependency trees

## Risks

- host-side install scripts execute on the host
- native module build steps may run on the host
- the host receives the full dependency tree and all resulting side effects

## Bottom line

This is the most stable variant of the host-not-isolated method, but it accepts the risk that dependency installation can execute uncontrolled third-party scripts on the host system.
