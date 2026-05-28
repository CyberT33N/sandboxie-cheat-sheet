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

## Why the install box also opens the repo root

The dedicated `--store-dir` is necessary, but in PNPM monorepos it is not sufficient on its own.

Workspace commands such as `pnpm add` or `pnpm update` may rewrite the shared root lockfile by creating a temporary file such as `pnpm-lock.yaml.<random>` and then renaming/replacing it into `pnpm-lock.yaml`.

If the install box keeps the repo root read-only and opens only selected manifests, the install can still fail with:

```text
[EPERM] EPERM: operation not permitted, rename
'C:\git\test\test-mono\pnpm-lock.yaml.<random>' -> 'C:\git\test\test-mono\pnpm-lock.yaml'
```

For that reason, the validated install-box boilerplate opens the full example project root for `node.exe` in addition to the dedicated shared store path.

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

# --- Install-box repo materialization surface ---
# PNPM monorepos may rewrite the shared root lockfile through a temp-file +
# rename/replace flow at the repo root.
OpenFilePath=node.exe,C:\git\test\test-mono\

# --- Dedicated pnpm store ---
OpenFilePath=node.exe,C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store\
```

## Related documents

- `docs\applications\IDE\vscode\methods\host-not-isolated\dependencies-installed-in-box.md`
- `docs\applications\IDE\vscode\methods\host-not-isolated\templates\node-monorepo-materialized-dependencies.md`
- `docs\applications\programming-languages\node\nvm\general.md`