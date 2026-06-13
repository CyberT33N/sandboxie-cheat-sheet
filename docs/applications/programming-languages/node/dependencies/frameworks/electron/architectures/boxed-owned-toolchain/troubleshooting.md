# Electron Troubleshooting In The Boxed-Owned-Toolchain

## Scope

This document records the **validated troubleshooting trail** for Electron runtime materialization failures in the boxed-owned-toolchain architecture.

It focuses on the concrete failure class where:

- `pnpm install` appears to finish
- Electron's JavaScript package resolves
- but the native Electron runtime is not actually usable afterward

Use the architecture overview for the boxed-owned-toolchain contract and the expected end state:

- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\overview.md`

## Problem shape

The observed failure shape was:

1. `pnpm install` completed in the boxed project shell
2. `electron/cli.js` was resolvable
3. the runtime probe still failed
4. inspection then showed:
   - `path.txt` missing
   - `dist\electron.exe` missing
   - only a partial `dist\` tree such as `locales\` existed

So the problem was **not** that the `electron` package was absent.

The problem was:

- Electron's native runtime payload was **partially materialized**

## What we tried

### 1. Normal boxed `pnpm install`

This remained the default baseline.

Observed result in the failing case:

- `pnpm install` did not report a hard Electron failure
- but Electron was still not correctly materialized afterward

### 2. Box-local cache variables + normal boxed `pnpm install`

We explicitly set:

```bash
export ELECTRON_CACHE="/c/Program Files/SandboxToolchains/VSCodeBoxes/test/execution/caches/browsers/electron"
export electron_config_cache="$ELECTRON_CACHE"
mkdir -p "$ELECTRON_CACHE"
```

Then we ran a normal boxed:

```bash
pnpm install
```

Observed result:

- the Electron package still remained in the broken partial state
- `path.txt` still did not exist
- `dist\electron.exe` still did not exist

### 3. Direct `install.js` with the primary Node 26 surface

We then targeted the current resolved package directly:

```bash
electronInstall="$(node -e "const path=require('node:path'); const {createRequire}=require('node:module'); const appRoot=path.resolve(process.cwd(),'apps/test'); const req=createRequire(path.resolve(appRoot,'package.json')); const electronCli=req.resolve('electron/cli.js'); process.stdout.write(path.join(path.dirname(electronCli),'install.js'));")"
node "$electronInstall"
printf 'install_exit=%s\n' "$?"
```

Observed result:

- the command could exit cleanly
- but the runtime state could still remain incomplete afterward

### 4. Direct `install.js` with the secondary boxed `node20` surface

Then we retried the same `install.js` with:

```bash
node20 "$electronInstall"
printf 'install_exit=%s\n' "$?"
```

Observed result:

- Electron runtime materialization completed successfully
- `path.txt` appeared
- `path.txt` contained `electron.exe`
- `dist\electron.exe` existed
- the direct smoke guard succeeded

## What this proves

### It was not a pure cache problem

The evidence in this context does **not** support the claim that cache placement alone was the root cause.

Why:

- with box-local cache variables set, a plain boxed `pnpm install` still left Electron broken

So the cache export remains useful and recommended for deterministic box-local behavior, but it was **not sufficient by itself**.

### The stronger signal was runtime selection

The stronger validated signal in this context was:

- Electron runtime materialization did **not** become healthy through the primary Node 26 install path
- the same package repair flow **did** become healthy through the secondary boxed `node20` path

That means the more likely primary cause in this repository state is:

- **the Node runtime used for Electron materialization**

not:

- cache placement alone

## Why host behavior can still differ

A host-side install can still work even when the boxed path does not, because the boxed path introduces additional constraints:

- different filesystem mediation
- different execution boundary
- different cache boundary
- different child-process behavior

However, in the validated flow here, the decisive differentiator was **not merely "box vs host"**.

The sharper differentiator was:

- the successful direct repair with the boxed `node20` runtime

So the current interpretation should be:

- Sandboxie/boxed execution is part of the environment
- but the most important proven corrective factor was the **Node 20 runtime selection for Electron install**

## Correct current troubleshooting sequence

1. run normal boxed `pnpm install`
2. verify:
   - `path.txt`
   - `dist\electron.exe`
3. if Electron is broken:
   - set `ELECTRON_CACHE`
   - set `electron_config_cache`
4. resolve the current `install.js`
5. run `node20 "$electronInstall"`
6. verify:
   - `path.txt exists=true`
   - `path.txt content=electron.exe`
   - `binary from path.txt exists=true`
   - `electron.exe exists=true`
7. run the direct smoke guard again

If this failure class is recurring for the project, the same validated sequence should be wrapped into the project-owned Electron post-install script instead of being repeated manually each time:

- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\scripts\post-install.md`

## Current conclusion

The current validated conclusion is:

- boxed `pnpm install` remains the default baseline
- if Electron is still unavailable afterward, the next troubleshooting step is **not** to blame cache first
- instead, treat the issue primarily as an **Electron materialization path under the wrong runtime selection**
- the current proven repair step is the direct `install.js` execution through boxed `node20`

## Related

- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\scripts\post-install.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`
- `docs\applications\programming-languages\node\runtime\architectures\boxed-owned-toolchain\overview.md`
