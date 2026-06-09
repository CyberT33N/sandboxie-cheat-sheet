# Electron In The Boxed-Owned-Toolchain

## Status

This is the boxed-owned-toolchain source of truth for Electron-specific runtime troubleshooting in this repository.

Use this document when:

- the project is running under the boxed-owned-toolchain architecture
- `pnpm install` has already been wired through the boxed project bootstrap
- Electron is still not available afterwards
- a runtime probe such as `smoke-electron-runtime` reports that Electron was not materialized correctly

## Core problem

The relevant failure class is:

1. `pnpm install` appears to succeed or mostly succeed
2. the `electron` package resolves
3. but the Electron runtime tree inside the workspace is incomplete
4. therefore the runtime probe cannot actually launch Electron

In practical terms, the problem is **not** that `electron/cli.js` is missing.

The actual problem is that the Electron package is in a **partially materialized state**, for example:

- `path.txt` is missing
- or `dist\electron.exe` is missing
- or only a partial `dist\` tree exists

That is why a project can still show:

```text
...\.pnpm\electron@29.4.6\node_modules\electron\cli.js
```

while the actual runtime is still broken.

## Why a plain `pnpm install` is not always sufficient here

In the current boxed-owned-toolchain setup, there are two separate failure surfaces:

1. **PNPM / lifecycle shell / child-spawn**
2. **Electron package materialization**

The boxed project already had a validated PNPM lifecycle-shell fix through box-local Git Bash.

However, Electron still remained vulnerable to a second failure class:

- the package install step may leave the Electron runtime tree incomplete
- the package can then resolve at the JS layer
- but the native runtime payload is not ready

This can happen even when the overall `pnpm install` run looks mostly healthy.

## What must be checked first

If Electron is not available after `pnpm install`, first check the actual package state.

Run this in the boxed project shell:

```bash
node -e "const fs=require('node:fs'); const path=require('node:path'); const {createRequire}=require('node:module'); const appRoot=path.resolve(process.cwd(),'apps/test'); const req=createRequire(path.resolve(appRoot,'package.json')); const electronCli=req.resolve('electron/cli.js'); const electronDir=path.dirname(electronCli); const pathTxt=path.join(electronDir,'path.txt'); const distDir=path.join(electronDir,'dist'); const exe=path.join(distDir,'electron.exe'); console.log('electronCli=' + electronCli); console.log('electronDir=' + electronDir); console.log('path.txt exists=' + fs.existsSync(pathTxt)); if (fs.existsSync(pathTxt)) { const rel=fs.readFileSync(pathTxt,'utf8').trim(); console.log('path.txt content=' + rel); console.log('binary from path.txt exists=' + fs.existsSync(path.join(distDir, rel))); } console.log('dist dir exists=' + fs.existsSync(distDir)); console.log('electron.exe exists=' + fs.existsSync(exe));"
```

Interpretation:

- `electronCli` exists -> JS package resolves
- `path.txt exists=false` -> install state is incomplete
- `electron.exe exists=false` -> native Electron runtime is not materialized

Important nuance:

- `path.txt` contains a filename such as `electron.exe`
- that filename is interpreted relative to `dist\`
- so the correct existence check is:
  `path.join(distDir, rel)`
- **not**:
  `path.join(electronDir, rel)`

## Current validated troubleshooting sequence

### Step 1 - set a box-local Electron cache

Run this in Git Bash:

```bash
export ELECTRON_CACHE="/c/Program Files/SandboxToolchains/VSCodeBoxes/test/execution/caches/browsers/electron"
mkdir -p "$ELECTRON_CACHE"
echo "ELECTRON_CACHE=$ELECTRON_CACHE"
```

Why:

- Electron should not fall back to an uncontrolled host-user-space cache during repair
- the browser/runtime payload should stay in the box-local execution cache tree

### Step 2 - resolve the current `install.js`

```bash
electronInstall="$(node -e "const path=require('node:path'); const {createRequire}=require('node:module'); const appRoot=path.resolve(process.cwd(),'apps/test'); const req=createRequire(path.resolve(appRoot,'package.json')); const electronCli=req.resolve('electron/cli.js'); process.stdout.write(path.join(path.dirname(electronCli),'install.js'));")"
printf '%s\n' "$electronInstall"
```

Why:

- the repair should target the exact currently resolved `electron` package in the workspace
- not some guessed or hard-coded package location

### Step 3 - run the Electron installer with the secondary Node 20 runtime

```bash
node20 "$electronInstall"
printf 'install_exit=%s\n' "$?"
```

Why `node20`:

- the current boxed-owned-toolchain project uses Node `26.2.0` as the primary control-plane runtime
- but the Electron runtime truth in this project is Node `20.9.0`
- in the validated debugging flow, the direct Electron install path had to be retried with the boxed `node20` command

### Step 4 - verify the package state again

Run the same verification block again:

```bash
node -e "const fs=require('node:fs'); const path=require('node:path'); const {createRequire}=require('node:module'); const appRoot=path.resolve(process.cwd(),'apps/test'); const req=createRequire(path.resolve(appRoot,'package.json')); const electronCli=req.resolve('electron/cli.js'); const electronDir=path.dirname(electronCli); const pathTxt=path.join(electronDir,'path.txt'); const distDir=path.join(electronDir,'dist'); const exe=path.join(distDir,'electron.exe'); console.log('electronCli=' + electronCli); console.log('electronDir=' + electronDir); console.log('path.txt exists=' + fs.existsSync(pathTxt)); if (fs.existsSync(pathTxt)) { const rel=fs.readFileSync(pathTxt,'utf8').trim(); console.log('path.txt content=' + rel); console.log('binary from path.txt exists=' + fs.existsSync(path.join(distDir, rel))); } console.log('dist dir exists=' + fs.existsSync(distDir)); console.log('electron.exe exists=' + fs.existsSync(exe));"
```

The expected healthy state is:

- `path.txt exists=true`
- `path.txt content=electron.exe`
- `binary from path.txt exists=true`
- `dist dir exists=true`
- `electron.exe exists=true`

### Step 5 - re-run the smoke guard directly

```bash
( cd apps/test-tooling && node scripts/smoke-electron-runtime.mjs )
```

The expected healthy result is:

```text
[smoke-electron-runtime] ok electron=29.4.6 node=20.9.0 @types/node=20.19.41
```

## What this proves when it succeeds

If the sequence above succeeds, then:

1. the Electron package was initially only partially materialized
2. the direct repair path through `install.js` was required
3. the current boxed-owned-toolchain architecture cannot blindly assume that a fresh `pnpm install` always leaves Electron in a healthy runtime state

That does **not** mean that every future install will fail.

It means:

- this failure surface is real
- the troubleshooting sequence above is the current validated repair path

## Relationship to the existing PNPM and Puppeteer documents

The boxed-owned-toolchain PNPM documents still own:

- package-manager contract
- lifecycle-shell behavior
- install and clean-reinstall orchestration

But the Electron-specific postinstall/runtime troubleshooting truth now lives here.

That means:

- `pnpm install` documentation should re-reference this file for Electron-specific runtime repair
- `pnpm clean reinstall` documentation should re-reference this file for the Electron-specific follow-up checks

## What not to conclude from this

Do **not** conclude that:

- the whole boxed-owned-toolchain architecture is invalid
- Electron can never be used repo-locally in the boxed path
- every start path should silently mutate the workspace on launch

The current validated conclusion is narrower:

- a repo-local Electron install can land in a broken state
- when it does, the repair flow above is the current validated boxed-owned-toolchain troubleshooting path

## Current architectural recommendation

For the current boxed-owned-toolchain architecture, the most correct posture is:

1. keep normal `pnpm install` as the default path
2. if Electron is still unavailable afterward, verify `path.txt` and `dist\electron.exe`
3. if Electron is not materialized, run the explicit repair flow above
4. if the repo-local package still cannot be repaired reliably, move to a stronger explicit runtime strategy

That stronger strategy can include a mirrored explicit runtime path similar to the older host-sync Electron fallback, but that is **not** the first boxed-owned-toolchain troubleshooting step.

## Related

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`
- `docs\applications\programming-languages\node\runtime\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\host-sync\general.md`
