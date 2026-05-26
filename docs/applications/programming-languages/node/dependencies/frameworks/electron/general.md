# Electron

## Architectural status

For the current recommended Sandboxie architecture in this repository:

- the **dependency tree** is materialized by a dedicated install box
- the **Electron runtime binary tree** may be mirrored into the shared toolchain root when the repo-local postinstall result is not reliable enough
- the **run box** launches the application by referencing that mirrored runtime explicitly

Recommended shared destination pattern:

```text
C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6\
```

This keeps the Electron runtime payload in the same explicit shared toolchain domain as the other Node monorepo support assets, instead of leaving it at the loose root level of `C:\shared\`.

## Recommended runtime pattern

If the framework supports it, point the runtime to the mirrored Electron executable explicitly:

```powershell
$env:ELECTRON_EXEC_PATH = "C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6\electron.exe"
```

This avoids relying on a fragile repo-local `electron/path.txt` state when the package postinstall does not materialize the full binary tree cleanly onto the host-visible workspace path.

## Shared Electron visibility rules

```ini
# --- Shared toolchain ancestor traversal ---
ReadFilePath=node.exe,C:\shared\sandbox-toolchains\
ReadFilePath=electron.exe,C:\shared\sandbox-toolchains\
ReadFilePath=powershell.exe,C:\shared\sandbox-toolchains\
ReadFilePath=cmd.exe,C:\shared\sandbox-toolchains\

# --- Mirrored Electron runtime ---
NormalFilePath=electron.exe,C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6\
ReadFilePath=node.exe,C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6\
ReadFilePath=electron.exe,C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6\
ReadFilePath=powershell.exe,C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6\
ReadFilePath=cmd.exe,C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6\
```

When the runtime path is refactored from a flat `C:\shared\electron-<version>\` layout into the deeper `node-monorepo-general\tools\electron\<version>\` tree, do not remove the toolchain-scoped ancestor `ReadFilePath` surface. The narrower `C:\shared\sandbox-toolchains\` scope is sufficient; opening the entire shared root is not required. Otherwise `electron.exe` may exist and still fail during startup because traversal to the deeper shared path is blocked in the run box.

## Legacy repo-local / host-mirror references

Older Electron documents that assume a host-shaped dependency tree remain in the repository, but they should now be read as **legacy / not recommended** references.

The new primary Electron-Vite overlay is:

- `docs\applications\programming-languages\node\dependencies\frameworks\electron\electron-vite\templates\monorepo-install-run-boxes.md`

## Related documents

- `docs\applications\IDE\vscode\methods\host-not-isolated\dependencies-installed-in-box.md`
- `docs\applications\IDE\vscode\methods\host-not-isolated\templates\node-monorepo-materialized-dependencies.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\electron-vite\general.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\troubleshooting.md`
