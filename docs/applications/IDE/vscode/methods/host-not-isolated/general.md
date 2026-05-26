# Method #1 - VS Code on Host, Toolchain in Dedicated Box, Debug via Attach-Inspect

## Core model

This method keeps VS Code or Cursor on the host system while moving the execution plane into dedicated Sandboxie boxes.

The host remains the control plane:

- editor UI
- language services
- workspace indexing
- debugging UI

The box layer remains the execution plane:

- PowerShell and CMD terminals
- Node executables
- Electron and Electron-Vite processes
- build tools, test runners, and package-manager-driven commands

This remains the strongest method currently available when the full IDE cannot be run inside the box with acceptable operability.

## Host-not-isolated variants

This method now has two documented dependency-placement variants:

1. **Dependencies installed in a dedicated install box and materialized on the host-visible workspace path**  
   Recommended for governed PNPM monorepos.  
   `docs\applications\IDE\vscode\methods\host-not-isolated\dependencies-installed-in-box.md`
2. **Dependencies installed directly on the host**  
   Legacy host-mirror / host-installed variant. Keep only as a reference for simpler or exceptional environments.  
   `docs\applications\IDE\vscode\methods\host-not-isolated\dependencies-installed-on-host.md`

## Why the split matters

Dependency placement changes:

- which `node.exe` actually runs scripts
- whether host VS Code can resolve packages and types
- whether host-side tools can read `node_modules`
- whether framework-specific development flows remain operational
- whether dependency install scripts execute on the host or only inside the install box
- whether package-manager governance surfaces such as lockfiles, catalogs, native postinstalls, and package-level runtime intent remain consistent

In practice, this is the main architectural fault line of the host-not-isolated method.

## Current recommendation

For the current documented Sandboxie architecture in this repository:

- use **install-box materialization** for Node/PNPM monorepos with strict governance
- keep the host-installed variant only as a **legacy / not recommended** fallback
- do **not** rely on a purely boxed dependency tree without host-visible materialization

This recommendation became materially stronger after validating the architecture against:

- `pnpm-workspace.yaml` governance such as `verifyDepsBeforeRun: error`
- `nodeLinker: isolated`
- shared lockfile / catalog enforcement
- mixed runtime intent where the package-manager/tooling shell and the delivered application runtime intentionally differ

## Reading order

1. Read the install-box materialization architecture first:  
   `docs\applications\IDE\vscode\methods\host-not-isolated\dependencies-installed-in-box.md`
2. Then read the full generic monorepo boilerplate:  
   `docs\applications\IDE\vscode\methods\host-not-isolated\templates\node-monorepo-materialized-dependencies.md`
3. Read the host-installed variant only as a legacy reference:  
   `docs\applications\IDE\vscode\methods\host-not-isolated\dependencies-installed-on-host.md`

## Decision summary

- If the priority is **operability under modern PNPM governance**, use the install-box materialization variant.
- If the priority is **minimum workflow complexity** and host-side dependency install scripts are acceptable, the host-installed variant still works in some environments, but it is no longer the recommended baseline here.
- A purely boxed dependency tree remains operationally incomplete in this method, because the host IDE still needs host-visible dependencies for package and type resolution.