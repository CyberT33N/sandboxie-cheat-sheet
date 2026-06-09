# Host-Sync

## Architectural status

This document is the `vitest`-specific overlay for the current recommended host-sync IDE model in this repository.

Single source of truth:

- the generic Node / PNPM monorepo materialization baseline lives in
  `docs\applications\IDE\vscode\methods\host-sync\templates\node-monorepo-materialized-dependencies.md`
- the install-box / run-box architecture explanation lives in
  `docs\applications\IDE\vscode\methods\host-sync\dependencies-installed-in-box.md`
- this folder keeps only the `vitest`-specific host-sync deltas on top of that baseline

## Assumptions

- VS Code or Cursor stays on the host
- dependencies are installed in a dedicated install box
- `.pnpm` / `node_modules` are materialized onto the host-visible workspace path
- tests are executed from the run box
- the tested package may use Vite-backed config loading through `vitest`

## Why this overlay exists

`vitest` runs through the Vite toolchain.
When Vitest loads `vitest.config.ts`, Vite may bundle the config and create transient working state under:

- `node_modules\.vite\`
- `node_modules\.vite-temp\`

In the host-sync run-box architecture, that transient write surface is legitimate and must be explicitly allowed.
Without it, a direct `vitest` invocation can fail before any test starts with an error such as:

```text
EPERM: operation not permitted, mkdir 'C:\git\test\test-mono\apps\desktop-app\node_modules\.vite-temp'
```

## Required Sandboxie write surface

In the current validated host-sync architecture, the run box must grant `node.exe` an explicit `OpenFilePath` for the Vitest/Vite temp surface.

Sanitized example:

```ini
OpenFilePath=node.exe,C:\git\test\test-mono\apps\desktop-app\node_modules\.vite-temp\
```

If the same package also uses the normal Vite cache directory during boxed development, keep the existing `.vite` rule as well:

```ini
OpenFilePath=node.exe,C:\git\test\test-mono\apps\desktop-app\node_modules\.vite\
```

The names `test-mono` and `desktop-app` above are sanitized placeholders.
Replace them with the real monorepo root and tested package path in the target project.

## Execution posture

For the current host-sync architecture, the preferred test execution posture is:

- dependency governance stays in the install box
- the run box executes already-materialized local tool surfaces directly
- test run scripts should **not** need to call `pnpm` again from inside other run scripts

That means a nested `pnpm` hop in run-box test flows is architecturally the wrong default.
It adds an extra execution layer even though the local test runner entrypoint already exists under the materialized package path.

## Direct command fallback when test scripts still use `pnpm`

If a project still keeps `pnpm` inside test run scripts, do **not** treat that nested `pnpm` call as the required host-sync baseline.
In the run box, expand the script one level further and invoke Vitest directly through the package-appropriate `node.exe`.

If the tested package models a package-specific runtime through `package.json -> devEngines.runtime`, use that runtime line for the direct call.
The sanitized example below shows a desktop package that remains pinned to the Node `20.9.0` line at the application/runtime boundary.

### Direct replacement for `test`

```powershell
Set-Location "C:\git\test\test-mono\apps\desktop-app"

$node = "C:\Users\yourusername\AppData\Local\nvm\v20.9.0node.exe"

$env:NX_DAEMON = "false"
$env:NX_NATIVE_FILE_CACHE_DIRECTORY = "C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native"

& $node ".\node_modules\vitest\vitest.mjs" run
```

### Direct replacement for project-scoped Vitest runs

```powershell
Set-Location "C:\git\test\test-mono\apps\desktop-app"

$node = "C:\Users\yourusername\AppData\Local\nvm\v20.9.0node.exe"

$env:NX_DAEMON = "false"
$env:NX_NATIVE_FILE_CACHE_DIRECTORY = "C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native"

& $node ".\node_modules\vitest\vitest.mjs" run --project unit
```

Other direct variants follow the same rule:

- `run --project integration`
- `run --project regression`
- `run --project types`
- `--watch`
- `--update`

## Interpretation

Use the direct Vitest route when:

- the package already has a stable local `node_modules\vitest\vitest.mjs` surface
- the nested `pnpm` call is the unstable or unnecessary layer
- the run box should stay focused on execution instead of package-manager recursion

Do **not** misread this as permission to move dependency installation into the run box.
The install box remains the only materialization authority.
This document changes only the inner execution surface for tests.

## Related documents

- `docs\applications\IDE\vscode\methods\host-sync\dependencies-installed-in-box.md`
- `docs\applications\IDE\vscode\methods\host-sync\templates\node-monorepo-materialized-dependencies.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\electron-vite\templates\monorepo-install-run-boxes.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\host-sync\overview.md`
