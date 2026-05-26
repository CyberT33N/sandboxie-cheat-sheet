# pnpm

## Architectural status

For the current recommended Sandboxie architecture in this repository, `pnpm` belongs to the **tooling / package-manager runtime layer**.

That means:

- boxed shells should use the fixed versioned `pnpm.cmd` from the real NVM home
- the dependency store should live in a dedicated shared toolchain cache path
- the install box materializes `node_modules` / `.pnpm` to the host-visible project path

Recommended shared store location:

```text
C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store\
```

## Why `pnpm.cmd` is preferred in boxed workflows

In boxed PowerShell sessions, the `pnpm` command may resolve to `pnpm.ps1`, which introduces avoidable ExecutionPolicy and PowerShell shim issues.

For the documented architecture, prefer:

```powershell
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd" -v
```

instead of:

```powershell
pnpm -v
```

## Why the store is moved out of the repo

During validation of the install-box materialization architecture, `pnpm` needed a dedicated store directory to avoid early boxed write probes at the repository root.

The resulting model is:

- repo-local `.pnpm\` and `node_modules\` remain the host-visible materialized dependency tree
- the content-addressable store itself lives under the shared toolchain cache root

## Typical boxed install command

```powershell
Set-Location "C:\git\test\test-mono"
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd" install --store-dir "C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store"
```

## The same `--store-dir` rule applies to rebuilds

In the install box, keep the dedicated shared store path for `pnpm rebuild` as well. Otherwise `pnpm` may fall back to early boxed write probes at the repository root again.

```powershell
Set-Location "C:\git\test\test-mono"
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd" --store-dir "C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store" rebuild
```

If the workspace uses Nx native bindings during install or rebuild, keep `NX_DAEMON=false` and `NX_NATIVE_FILE_CACHE_DIRECTORY=C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native` active in the same shell. See `docs\applications\software-development\monorepo\nx\general.md`.

## Sandboxie access rules

```ini
# --- pnpm toolchain visibility ---
NormalFilePath=node.exe,C:\Users\yourusername\AppData\Local\pnpm\
NormalFilePath=powershell.exe,C:\Users\yourusername\AppData\Local\pnpm\
NormalFilePath=cmd.exe,C:\Users\yourusername\AppData\Local\pnpm\
NormalFilePath=pnpm.exe,C:\Users\yourusername\AppData\Local\pnpm\

# --- Dedicated pnpm store ---
OpenFilePath=node.exe,C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store\
```

## Related documents

- `docs\applications\IDE\vscode\methods\host-not-isolated\dependencies-installed-in-box.md`
- `docs\applications\IDE\vscode\methods\host-not-isolated\templates\node-monorepo-materialized-dependencies.md`
- `docs\applications\programming-languages\node\nvm\general.md`