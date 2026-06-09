# Puppeteer In The Boxed-Owned-Toolchain

## Scope

This document is the **single source of truth** for Puppeteer-specific boxed-owned-toolchain behavior in this repository.

It explains:

- what the actual install problem is when `pnpm install` runs in the boxed-owned-toolchain architecture
- why PNPM itself is not the root problem
- what must be set in the project bootstrap so Puppeteer postinstall runs against the correct cache boundary
- what the clean-reinstall script must additionally remove
- why the solution is architecturally correct

Other documents should re-reference this file instead of duplicating Puppeteer-specific cache and postinstall rationale inline.

## Problem statement

When `pnpm install` runs in the boxed-owned-toolchain architecture, Puppeteer's postinstall does not only resolve packages from the workspace dependency tree.

It also downloads and materializes browser artifacts.

If no explicit boxed cache contract is set, Puppeteer falls back to its default cache surface under host-user-space, for example:

```text
C:\Users\denni\.cache\puppeteer\chrome-headless-shell\...
```

In the boxed-owned-toolchain architecture, that default is the wrong execution boundary because:

1. it is not the box-local execution cache tree
2. it can contain stale or partially materialized browser payloads from earlier runs
3. it is not the deterministic cache surface the project bootstrap owns

The visible result during `pnpm install` is a postinstall failure like:

```text
Failed to set up chrome-headless-shell ...
The browser folder (...) exists but the executable is missing
```

So the practical install failure is:

- `pnpm install` starts correctly
- package extraction succeeds
- Puppeteer postinstall reaches browser setup
- browser setup reuses a stale / wrong cache location
- postinstall aborts

## What the problem is not

The primary problem is **not**:

- generic PNPM package resolution
- the governed shared `pnpm.cjs` contract
- the project-specific `PnpmCli` selection
- the Git-Bash `pnpm` command-surface fix

Those are separate PNPM-domain concerns.

The specific Puppeteer problem is:

> browser-download postinstall artifacts are landing on the wrong cache boundary unless the project bootstrap forces them into the box-local execution cache.

## Correct solution

The correct boxed-owned-toolchain solution is:

1. create a box-local browser cache subtree inside the project execution cache
2. set `PUPPETEER_CACHE_DIR` to that box-local subtree before running install
3. set `PLAYWRIGHT_BROWSERS_PATH` to the sibling box-local browser subtree for the same browser-artifact class
4. expose those resolved paths again through boxed environment variables so later project scripts can observe and clean them
5. make the clean-reinstall script delete those box-local browser caches in addition to repo-local `node_modules` / `.pnpm`

This preserves the intended architecture:

- package-manager contract stays explicit
- browser artifacts remain box-local
- reinstall behavior stays deterministic
- host-user-space browser caches do not silently become the truth

## Mandatory project-base code

The following block is the current required Puppeteer-related code that must exist in the project bootstrap base script.

Current real file:

```text
C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeProjectBase.ps1
```

Required block:

```powershell
$localNxCacheRoot = Join-Path $projectPaths.LocalCacheRoot 'nx-native'

if (-not (Test-Path -LiteralPath $localNxCacheRoot)) {
  New-Item -ItemType Directory -Force -Path $localNxCacheRoot | Out-Null
}

$localTempRoot = Join-Path $projectPaths.LocalCacheRoot 'temp'
if (-not (Test-Path -LiteralPath $localTempRoot)) {
  New-Item -ItemType Directory -Force -Path $localTempRoot | Out-Null
}

$localBrowserCacheRoot = Join-Path $projectPaths.LocalCacheRoot 'browsers'
if (-not (Test-Path -LiteralPath $localBrowserCacheRoot)) {
  New-Item -ItemType Directory -Force -Path $localBrowserCacheRoot | Out-Null
}

$localPuppeteerCacheRoot = Join-Path $localBrowserCacheRoot 'puppeteer'
if (-not (Test-Path -LiteralPath $localPuppeteerCacheRoot)) {
  New-Item -ItemType Directory -Force -Path $localPuppeteerCacheRoot | Out-Null
}

$localPlaywrightBrowsersRoot = Join-Path $localBrowserCacheRoot 'playwright'
if (-not (Test-Path -LiteralPath $localPlaywrightBrowsersRoot)) {
  New-Item -ItemType Directory -Force -Path $localPlaywrightBrowsersRoot | Out-Null
}

$env:TEMP = $localTempRoot
$env:TMP = $localTempRoot
$env:TMPDIR = $localTempRoot
$env:PUPPETEER_CACHE_DIR = $localPuppeteerCacheRoot
$env:PLAYWRIGHT_BROWSERS_PATH = $localPlaywrightBrowsersRoot

$env:BOXED_LOCAL_TEMP_ROOT = $localTempRoot
$env:BOXED_PUPPETEER_CACHE_DIR = $localPuppeteerCacheRoot
$env:BOXED_PLAYWRIGHT_BROWSERS_PATH = $localPlaywrightBrowsersRoot

Write-Host "LocalTemp: $localTempRoot"
Write-Host "PuppeteerCacheDir: $localPuppeteerCacheRoot"
Write-Host "PlaywrightBrowsersPath: $localPlaywrightBrowsersRoot"
```

## Meaning of the project-base code

### `localBrowserCacheRoot`

Creates one explicit box-local root for browser-download artifacts.

This keeps browser payloads in the same execution-cache class as the rest of the box-local runtime surfaces.

### `localPuppeteerCacheRoot`

Defines the exact Puppeteer cache path inside the box-local browser cache tree.

This path becomes the forced replacement for `%USERPROFILE%\.cache\puppeteer`.

### `localPlaywrightBrowsersRoot`

Defines the sibling browser cache path for Playwright.

It is included here because the same browser-artifact class should stay on the same box-local boundary.

### `PUPPETEER_CACHE_DIR`

This is the actual runtime variable Puppeteer postinstall consumes.

Without this variable, Puppeteer falls back to host-user-space defaults.

### `PLAYWRIGHT_BROWSERS_PATH`

This is the corresponding browser-download location for Playwright.

It is part of the same boxed browser-cache contract even when the immediate failing package is Puppeteer.

### `BOXED_PUPPETEER_CACHE_DIR` and `BOXED_PLAYWRIGHT_BROWSERS_PATH`

These are not the package-manager-consumed variables themselves.

They are the **boxed observability and reuse surface** for later project-owned scripts, especially clean-reinstall flows.

### terminal `Write-Host` lines

These lines are operationally important because they let you confirm in the install logs that the current bootstrap really exported the correct box-local cache paths.

## Mandatory clean-reinstall code

The following block is the current required Puppeteer-related code that must exist in the project clean-reinstall script.

Current real file:

```text
C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoPnpmCleanReinstall.ps1
```

Required block:

```powershell
$modulesYaml = Join-Path $resolvedRepoPath 'node_modules\.modules.yaml'
if (Test-Path -LiteralPath $modulesYaml) {
  Write-Host "Removing $modulesYaml"
  Remove-Item -LiteralPath $modulesYaml -Force -ErrorAction SilentlyContinue
}

@(
  $env:BOXED_PUPPETEER_CACHE_DIR,
  $env:BOXED_PLAYWRIGHT_BROWSERS_PATH
) |
Where-Object { -not [string]::IsNullOrWhiteSpace($_) } |
Sort-Object -Unique |
Where-Object { Test-Path -LiteralPath $_ } |
ForEach-Object {
  Write-Host "Removing $_"
  Remove-Item -LiteralPath $_ -Recurse -Force -ErrorAction SilentlyContinue
}
```

## Meaning of the clean-reinstall code

Deleting only:

- repo-root `.pnpm`
- repo-root `node_modules`
- package-level `node_modules`

is **not sufficient** for the Puppeteer failure class documented here.

Why:

- the stale or partially materialized browser payload may live outside the workspace dependency tree
- the reinstall would then rebuild packages while still reusing the broken browser cache
- the same Puppeteer postinstall error can repeat even after the repo dependency tree was fully deleted

The browser-cache cleanup block therefore makes the reinstall complete for this failure class.

## What not to do

Do **not** treat these as the primary boxed-owned-toolchain fix:

- relying on `C:\Users\<user>\.cache\puppeteer` as the browser cache truth
- deleting only repo-local `node_modules`
- setting `PUPPETEER_SKIP_DOWNLOAD=true` as a generic workaround

Why not:

- the first keeps the wrong cache boundary
- the second leaves stale browser payloads behind
- the third can silence the symptom while leaving the install contract incomplete

## Current operational path

The current real host-triggered clean-reinstall command is:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_test_MONO `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoPnpmCleanReinstall.ps1" `
  -RepoPath "C:\Users\denni\source\test-mono"
```

## Verification

When the fixed project bootstrap is active, the terminal startup output must include:

```text
PuppeteerCacheDir: C:\Program Files\SandboxToolchains\VSCodeBoxes\test-mono\execution\caches\browsers\puppeteer
PlaywrightBrowsersPath: C:\Program Files\SandboxToolchains\VSCodeBoxes\test-mono\execution\caches\browsers\playwright
```

If those lines are missing, the wrong bootstrap version is still running.

## Related

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`
