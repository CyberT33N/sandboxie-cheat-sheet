# Monorepo - Install Box + Run Box + Host Materialization (Electron-Vite)

## Architectural status

This document is the Electron-Vite-specific overlay for the current recommended host-not-isolated IDE model in this repository.

Single source of truth:

- the generic Node / PNPM monorepo materialization baseline lives in `docs\applications\IDE\vscode\methods\host-not-isolated\templates\node-monorepo-materialized-dependencies.md`
- this document keeps only the Electron-Vite-specific deltas on top of that baseline

Assumptions:

- VS Code or Cursor stays on the host
- dependencies are installed in a dedicated install box
- `.pnpm` / `node_modules` are materialized onto the host-visible workspace path
- Electron runtime payload may be mirrored into the shared toolchain root when the repo-local `electron` postinstall result is not stable enough

This replaces the older host-installed / host-mirror Electron-Vite templates as the recommended baseline.

## Baseline reference

Before applying anything below, complete the generic monorepo baseline here:

- `docs\applications\IDE\vscode\methods\host-not-isolated\templates\node-monorepo-materialized-dependencies.md`

That document is the source of truth for:

- shared toolchain root creation
- clean-start removal of old dependency trees
- install box base config
- run box base config
- root `.vscode\settings.json`
- host-visible dependency materialization workflow

## Electron-specific shared addition

Add the Electron mirror subtree under the same shared root used by the generic baseline:

```text
C:\shared\sandbox-toolchains\node-monorepo-general\
  tools\
    electron\
      29.4.6\
        electron.exe
```

## Electron-specific config additions

Start from the generic configs and add only the Electron-Vite-specific lines below.

### Install box additions

`esbuild.exe` validation during postinstall needs one additional rule:

```ini
# --- esbuild postinstall validation ---
NormalFilePath=esbuild.exe,C:\git\test\test-mono\
```

### Run box additions

Add the following lines to the generic run-box config:

```ini
# --- Shared toolchain ancestor traversal ---
# When the Electron runtime moved from a flat shared path into the deeper
# node-monorepo-general tree, these read rules became necessary.
# The narrower toolchain-scoped surface is sufficient; do not broadly open
# the entire shared root when only the toolchain subtree is needed.
ReadFilePath=node.exe,C:\shared\sandbox-toolchains\
ReadFilePath=electron.exe,C:\shared\sandbox-toolchains\
ReadFilePath=powershell.exe,C:\shared\sandbox-toolchains\
ReadFilePath=cmd.exe,C:\shared\sandbox-toolchains\

# --- Cursor / VS Code debugger bootloader visibility ---
ReadFilePath=node.exe,C:\Users\yourusername\AppData\Local\Programs\cursor\resources\app\
ReadFilePath=electron.exe,C:\Users\yourusername\AppData\Local\Programs\cursor\resources\app\

# --- Runtime dependency surfaces also need esbuild/electron read access ---
ReadFilePath=esbuild.exe,C:\git\test\test-mono\.pnpm\
ReadFilePath=electron.exe,C:\git\test\test-mono\.pnpm\

ReadFilePath=esbuild.exe,C:\git\test\test-mono\node_modules\
ReadFilePath=electron.exe,C:\git\test\test-mono\node_modules\

ReadFilePath=esbuild.exe,C:\git\test\test-mono\apps\*\node_modules
ReadFilePath=esbuild.exe,C:\git\test\test-mono\apps\*\node_modules\*
ReadFilePath=esbuild.exe,C:\git\test\test-mono\libs\*\node_modules
ReadFilePath=esbuild.exe,C:\git\test\test-mono\libs\*\node_modules\*
ReadFilePath=esbuild.exe,C:\git\test\test-mono\tools\*\node_modules
ReadFilePath=esbuild.exe,C:\git\test\test-mono\tools\*\node_modules\*

# App-local dependency surface needed by electron.exe
ReadFilePath=electron.exe,C:\git\test\test-mono\apps\desktop-app\node_modules
ReadFilePath=electron.exe,C:\git\test\test-mono\apps\desktop-app\node_modules\*

# Optional: if the built packaged app is launched inside the monorepo
ReadFilePath=desktop-app.exe,C:\git\test\test-mono\apps\desktop-app\node_modules
ReadFilePath=desktop-app.exe,C:\git\test\test-mono\apps\desktop-app\node_modules\*

# --- Shared Electron runtime mirror ---
NormalFilePath=electron.exe,C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6\
ReadFilePath=node.exe,C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6\
ReadFilePath=electron.exe,C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6\
ReadFilePath=powershell.exe,C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6\
ReadFilePath=cmd.exe,C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6\

# --- Workspace Electron / Vite command surfaces ---
NormalFilePath=esbuild.exe,C:\git\test\test-mono\
NormalFilePath=electron.exe,C:\git\test\test-mono\

# --- Desktop app package execution surface ---
NormalFilePath=esbuild.exe,C:\git\test\test-mono\apps\desktop-app\
NormalFilePath=node.exe,C:\git\test\test-mono\apps\desktop-app\
NormalFilePath=powershell.exe,C:\git\test\test-mono\apps\desktop-app\
NormalFilePath=cmd.exe,C:\git\test\test-mono\apps\desktop-app\
NormalFilePath=git.exe,C:\git\test\test-mono\apps\desktop-app\
NormalFilePath=electron.exe,C:\git\test\test-mono\apps\desktop-app\
NormalFilePath=electron.exe,C:\git\test\test-mono\

# Optional packaged app surface inside the monorepo
NormalFilePath=desktop-app.exe,C:\git\test\test-mono\apps\desktop-app\dist\

# Vite temporary dependency optimization cache
OpenFilePath=node.exe,C:\git\test\test-mono\apps\desktop-app\node_modules\.vite\

# Built artifacts must stay host-visible for host IDE debugging
OpenFilePath=node.exe,C:\git\test\test-mono\apps\desktop-app\out\
OpenFilePath=electron.exe,C:\git\test\test-mono\apps\desktop-app\out\
```

## App-local `.vscode\settings.json` overlay (optional)

The root workspace `.vscode\settings.json` stays in the generic baseline.

If the Electron-Vite package should use hover-run / Run Script directly with the mirrored Electron runtime, add this package-local overlay:

```json
{
  "terminal.integrated.automationProfile.windows": {
    "path": "C:\\Tools\\TestRunBoxShell\\cmd.exe",
    "env": {
      "PATH": "C:\\Users\\yourusername\\AppData\\Local\\nvm\\v26.2.0;${env:PATH}",
      "NX_DAEMON": "false",
      "NX_NATIVE_FILE_CACHE_DIRECTORY": "C:\\shared\\sandbox-toolchains\\node-monorepo-general\\cache\\nx-native",
      "ELECTRON_EXEC_PATH": "C:\\shared\\sandbox-toolchains\\node-monorepo-general\\tools\\electron\\29.4.6\\electron.exe"
    }
  },
  "terminal.integrated.defaultProfile.windows": "RunBox CMD",
  "terminal.integrated.profiles.windows": {
    "RunBox CMD": {
      "path": "C:\\Tools\\TestRunBoxShell\\cmd.exe",
      "env": {
        "PATH": "C:\\Users\\yourusername\\AppData\\Local\\nvm\\v26.2.0;${env:PATH}",
        "NX_DAEMON": "false",
        "NX_NATIVE_FILE_CACHE_DIRECTORY": "C:\\shared\\sandbox-toolchains\\node-monorepo-general\\cache\\nx-native",
        "ELECTRON_EXEC_PATH": "C:\\shared\\sandbox-toolchains\\node-monorepo-general\\tools\\electron\\29.4.6\\electron.exe"
      }
    }
  }
}
```

## What this overlay intentionally does not add

The following settings were removed from the recommended reduced configuration because they are unnecessary, too broad, or belong to a different responsibility boundary:

- no `ForceProcess=electron.exe` or `ForceProcess=<product>.exe`
  - too broad; this can capture unrelated Electron executables instead of only the intended shared runtime path
- no repo-local `ForceFolder=...\.pnpm\electron@...\node_modules\electron\dist\`
  - the shared Electron mirror is now the runtime source of truth
- no broad `ReadFilePath=...\C:\shared\`
  - the narrower `C:\shared\sandbox-toolchains\` ancestor surface is sufficient
- no `nvm.exe` visibility
  - boxed workflows use fixed versioned `node.exe` / `pnpm.cmd` directly and must not rely on `nvm use`
- no Angular CLI config lookup in the Electron baseline
  - only add framework-specific config lookups when that framework actually runs in the same box
- no install-box-owned shared Electron mirror write surface in the baseline
  - the install box owns dependency materialization and the PNPM store; refresh the shared Electron mirror on the host when the fallback download path is needed

## Electron-specific workflow

Prerequisite:

- complete the generic monorepo workflow from `docs\applications\IDE\vscode\methods\host-not-isolated\templates\node-monorepo-materialized-dependencies.md` first

### Step 1 - mirror the Electron runtime payload into the shared toolchain root

If the repo-local `electron` postinstall does not leave a stable `path.txt` / `dist\electron.exe` state, mirror a complete Electron runtime payload into the shared toolchain root.

One common host-side source is the cached Electron ZIP:

```powershell
$zip = "C:\Users\yourusername\AppData\Local\electron\Cache\electron-v29.4.6-win32-x64.zip"
$dest = "C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6"

New-Item -ItemType Directory -Force -Path $dest | Out-Null
Remove-Item -Recurse -Force "$dest\*" -ErrorAction SilentlyContinue
Expand-Archive -LiteralPath $zip -DestinationPath $dest -Force

Test-Path "$dest\electron.exe"
```

The mirrored payload must be the full extracted Electron runtime tree, not only a few loose files.

If that cache path does not exist on the host, use another complete host-side Electron payload source, but keep the same destination and the same `Test-Path` verification.

### Step 2 - run Electron-Vite from the run box

If the package scripts use `cross-env` and that wrapper is unstable in the boxed Windows flow, use the raw call:

```powershell
Set-Location "C:\git\test\test-mono\apps\desktop-app"
$env:PVS = "z1"
$env:ELECTRON_EXEC_PATH = "C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6\electron.exe"
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd" run _dev:cmd -- --watch
```

This raw call remains the most reliable boxed Windows fallback even if the project later keeps the higher-level wrapper scripts for unboxed or other environments.

## Troubleshooting

### `esbuild.exe` fails in postinstall

The install box needs:

```ini
NormalFilePath=esbuild.exe,C:\git\test\test-mono\
```

### `electron-vite` cannot find Electron

Prefer the mirrored shared Electron runtime plus:

```powershell
$env:ELECTRON_EXEC_PATH = "C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6\electron.exe"
```

Then verify the mirror itself:

```powershell
Test-Path "C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6\electron.exe"
```

If that returns `False`, or if the directory contains only a few loose files and no `electron.exe`, the mirror is incomplete and `spawn ... ENOENT` is expected.

### Shared Electron mirror is missing or corrupt

Try a rebuild in the install box first:

```powershell
Set-Location "C:\git\test\test-mono\apps\desktop-app"
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd" --store-dir "C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store" rebuild electron
```

Then verify the mirrored executable in the shared toolchain root:

```powershell
$dest = "C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6"

Test-Path "$dest\electron.exe"
Get-ChildItem $dest
```

A corrupt state is possible: `pnpm` may report a successful Electron postinstall while the shared mirror still lacks `electron.exe` or contains only a partial payload.

The rebuild is still the correct first validation step, but in the reduced baseline the install box does **not** own the shared Electron mirror directly. If the shared mirror still lacks `electron.exe` after the rebuild, refresh the mirror from the host.

If `electron.exe` is still missing, use the host-side fallback and refresh the shared mirror from the official Electron ZIP directly:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

$dest = "C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6"
$zip = Join-Path $env:TEMP "electron-v29.4.6-win32-x64.zip"

Invoke-WebRequest `
  -Uri "https://github.com/electron/electron/releases/download/v29.4.6/electron-v29.4.6-win32-x64.zip" `
  -OutFile $zip

New-Item -ItemType Directory -Force -Path $dest | Out-Null
Remove-Item -Recurse -Force "$dest\*" -ErrorAction SilentlyContinue
Expand-Archive -LiteralPath $zip -DestinationPath $dest -Force

Test-Path "$dest\electron.exe"
```

### Electron starts but exits immediately after `start electron app...`

If `electron.exe` now exists but the process still exits with `[ELIFECYCLE]` and no longer shows `ENOENT`, the failure moved past path resolution and into the sandboxed Electron startup itself.

Before tracing, confirm that the run box still keeps the shared ancestor read surface:

```ini
ReadFilePath=node.exe,C:\shared\sandbox-toolchains\
ReadFilePath=electron.exe,C:\shared\sandbox-toolchains\
ReadFilePath=powershell.exe,C:\shared\sandbox-toolchains\
ReadFilePath=cmd.exe,C:\shared\sandbox-toolchains\
```

When the runtime path was refactored from a flat shared location into `C:\shared\sandbox-toolchains\node-monorepo-general\tools\electron\29.4.6\`, removing those toolchain-scoped `ReadFilePath` lines can let `electron.exe` exist and still exit immediately inside the run box.

In that case, capture a fresh run-box deny trace for both:

- `node.exe`
- `electron.exe`

### Vite cannot create `node_modules\.vite`

The run box needs:

```ini
OpenFilePath=node.exe,C:\git\test\test-mono\apps\desktop-app\node_modules\.vite\
```

## Related documents

- `docs\applications\IDE\vscode\methods\host-not-isolated\templates\node-monorepo-materialized-dependencies.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\general.md`
- `docs\applications\programming-languages\node\dependencies\esbuild\general.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\general.md`
- `docs\applications\programming-languages\node\nvm\general.md`
