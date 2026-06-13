# Windows Long Paths In Sandboxed Workflows

## Scope

This document is the **single source of truth** for the Git-independent long-path problem in this repository.

It covers:

- classic Windows path-length limits
- sandbox path-length amplification
- the boxed bootstrap-owned Windows long-path setting
- the meaning of the diagnostic bootstrap metadata

It intentionally does **not** own Git-specific command options.

The Git-specific slice lives here:

- `docs\applications\git\architectures\boxed-owned-toolchain\troubleshooting\long-paths.md`

## Why this lives under `docs\troubleshooting`

From a domain-driven and 12-factor perspective, this is the correct location because the problem is:

- wider than Git
- wider than VS Code
- wider than PNPM
- wider than one single application

It is fundamentally a:

- Windows filesystem / path-length concern
- plus Sandboxie path-redirection concern

That makes it a cross-domain troubleshooting capability, not an application-domain truth owned only by Git.

## Current validated repository finding

The current repository validated a failure class where:

- the **logical project path** was already at or above the classic `MAX_PATH` threshold
- the **sandboxed physical path** became even longer because Sandboxie redirects writes under the sandbox file root

So the real problem is not only:

- "Git has a long file name"

The real problem is the combined path shape:

1. logical repo path on Windows
2. plus sandbox redirection prefix

## Why Sandboxie matters here

Sandboxie redirects writes into the sandbox root.

Upstream Sandboxie documents this through:

- `FileRootPath`
- the sandbox hierarchy

Relevant upstream references in this workspace:

- `upstream\sandboxie-docs\docs\Content\FileRootPath.md`
- `upstream\sandboxie-docs\docs\Content\SandboxHierarchy.md`

The important consequence is:

- the physical redirected path inside the sandbox is longer than the logical path the application thinks it is writing

## What `FileRootPath` means

`FileRootPath` is the root path of the sandbox filesystem container.

Representative upstream meaning:

- if the sandboxed program writes `C:\NEW.TXT`
- Sandboxie redirects that into the sandbox file tree under the configured sandbox root

Why that matters:

- a shorter sandbox root reduces the redirection prefix
- but if the logical path is already too long, shortening the sandbox root alone is **not** a full solution

That is why the repository did **not** treat `FileRootPath` shortening as the only architectural fix.

## Current architecture answer

The current bootstrap-owned answer has two layers:

### Layer 1 - Git-independent Windows long-path policy

The bootstrap now sets the boxed registry view to:

```text
HKLM\SYSTEM\CurrentControlSet\Control\FileSystem\LongPathsEnabled = 1
```

Representative code shape from the boxed bootstrap:

```powershell
if (-not $windowsShellRuntime.RegAvailable -or [string]::IsNullOrWhiteSpace($windowsShellRuntime.RegExe) -or -not (Test-Path -LiteralPath $windowsShellRuntime.RegExe)) {
  throw 'Local boxed REG executable not found for LongPathsEnabled bootstrap step.'
}

& $windowsShellRuntime.RegExe add 'HKLM\SYSTEM\CurrentControlSet\Control\FileSystem' /v LongPathsEnabled /t REG_DWORD /d 1 /f | Out-Null
if ($LASTEXITCODE -ne 0) {
  throw 'Failed to set LongPathsEnabled=1 in the boxed registry view.'
}

$env:BOXED_LONG_PATHS_ENABLED = 'true'
```

Important nuance:

- `BOXED_LONG_PATHS_ENABLED=true` is **diagnostic bootstrap metadata**
- the actual Windows-side switch is the boxed registry value

### Layer 2 - application-specific long-path participation

Applications can still need their own path-length support behavior.

For Git, that Git-specific layer is:

- `core.longpaths=true`

That Git-specific part lives in the Git domain:

- `docs\applications\git\architectures\boxed-owned-toolchain\troubleshooting\long-paths.md`

## What the current repository proved

The current repository state proved:

- the long-path problem is not purely "Sandboxie is broken"
- the long-path problem is not purely "Git is broken"
- the correct architecture is to combine:
  - boxed Windows long-path enablement
  - app-specific participation where required

For Git, the second piece is `core.longpaths=true`.

## Relevant upstream signals

The following upstream signals were relevant to the repository interpretation:

### Sandboxie path amplification issue class

- `sandboxie-plus/Sandboxie#2526`
  - example of path-length trouble becoming worse after Sandboxie path conversion

### Sandbox name length sensitivity

- `sandboxie-plus/Sandboxie#4064`
  - shows that overly long sandbox names are a separate length risk

### Internal long-path bugfix in Sandboxie

- `sandboxie-plus/Sandboxie#2769`
  - fixed a specific internal long-path handling issue

Important interpretation:

- those upstream items are relevant
- but they do **not** replace the need for a correct runtime contract in this repository

## Practical architecture interpretation

For this repository, the right current reading is:

1. treat the Windows long-path setting as a bootstrap-owned OS/filesystem concern
2. treat sandbox path amplification as a Sandboxie/filesystem concern
3. treat Git `core.longpaths=true` as a Git-domain concern
4. do not collapse those three things into one single application-specific workaround

## Related

- `docs\applications\git\architectures\boxed-owned-toolchain\troubleshooting\long-paths.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `upstream\sandboxie-docs\docs\Content\FileRootPath.md`
- `upstream\sandboxie-docs\docs\Content\SandboxHierarchy.md`
