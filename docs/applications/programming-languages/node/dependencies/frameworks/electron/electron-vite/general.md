# Electron-Vite

## Architectural status

This framework now has two documented architecture tracks in this repository:

1. **Legacy host-installed / host-mirror references**  
   Keep only as historical / not recommended documentation:
   - `docs\applications\programming-languages\node\dependencies\frameworks\electron\electron-vite\templates\single-repo.md`
   - `docs\applications\programming-languages\node\dependencies\frameworks\electron\electron-vite\templates\mono-repo.md`

2. **Recommended install-box materialization + run-box execution**
   - generic monorepo baseline:  
     `docs\applications\IDE\vscode\methods\host-not-isolated\templates\node-monorepo-materialized-dependencies.md`
   - Electron-Vite-specific overlay:  
     `docs\applications\programming-languages\node\dependencies\frameworks\electron\electron-vite\templates\monorepo-install-run-boxes.md`

## Why Electron-Vite needs an overlay

Compared to a generic Node monorepo, Electron-Vite adds a few extra moving parts:

- repo-local build output that must stay host-visible for host IDE debugging
- Electron runtime lookup and startup behavior
- Vite cache writes under `node_modules\.vite\`
- run-script wrappers such as `cross-env` that may need a raw-call workaround in boxed Windows shells

That is why the generic monorepo template is necessary but not sufficient on its own.

## Current recommended posture

- keep the host IDE on the host
- install dependencies in the install box
- materialize `.pnpm` / `node_modules` to the host-visible workspace path
- run Electron-Vite from the run box
- keep Electron build output (`out\`) host-visible
- if the repo-local Electron postinstall does not materialize a stable runtime tree, mirror the Electron binary payload into the shared toolchain root and launch with `ELECTRON_EXEC_PATH`

## Related documents

- `docs\applications\programming-languages\node\dependencies\frameworks\electron\general.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\electron-vite\debug.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\troubleshooting.md`
- `docs\applications\programming-languages\node\dependencies\esbuild\general.md`