# Method #1 - VS Code on Host, Toolchain in Dedicated Box, Debug via Attach-Inspect

## Core model

This method keeps VS Code or Cursor on the host system while moving the execution plane into a dedicated Sandboxie box.

The host remains the control plane:

- editor UI
- language services
- workspace indexing
- debugging UI

The box remains the execution plane:

- PowerShell and CMD terminals
- Node executables
- Electron and Electron-Vite processes
- build tools, test runners, and package-manager-driven commands

This is the strongest method currently available when the full IDE cannot be run inside the box with acceptable operability.

## Host-not-isolated variants

This method has two dependency-placement variants:

1. **Dependencies installed on the host**
   `docs\applications\IDE\vscode\methods\host-not-isolated\dependencies-installed-on-host.md`
2. **Dependencies installed in the box**
   `docs\applications\IDE\vscode\methods\host-not-isolated\dependencies-installed-in-box.md`

## Why the split matters

Dependency placement changes:

- which `node.exe` actually runs scripts
- whether host VS Code can resolve packages and types
- whether host-side tools can read `node_modules`
- whether framework-specific development flows remain operational
- whether dependency install scripts execute on the host or only inside the box

In practice, this is the main architectural fault line of the host-not-isolated method.

## Reading order

1. Read the host-installed variant first if the priority is maximum editor compatibility and minimum workflow friction.
2. Read the box-installed variant second if the priority is reducing host-side execution of dependency install scripts.
3. Treat the box-installed variant as incomplete until a tested host-visible mirror or equivalent synchronization architecture exists.

## Decision summary

- If the priority is **operability**, use the host-installed dependency variant.
- If the priority is **reducing host-side install-script execution**, use the box-installed dependency variant together with a tested synchronization or mirror strategy.
- A purely boxed dependency tree is not operationally complete in this method, because the host IDE still needs host-visible dependencies for package and type resolution.