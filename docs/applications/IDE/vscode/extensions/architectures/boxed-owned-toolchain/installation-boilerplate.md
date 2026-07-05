# Extension Installation Boilerplate

## Scope

This document provides the complete boxed-owned-toolchain boilerplate for installing the required VS Code family extensions.

It covers:

- marketplace-backed extension installation
- local VSIX artifact staging in the shared `dev` area
- local forked extension installation from staged VSIX artifacts
- promotion of the local maintenance authoring state into the canonical shared extension store
- verification of the final shared-store result

## Why this document lives here

This is not:

- a single-extension settings document
- a generic VS Code usage document
- a generic bootstrap overview

It is the cross-extension installation contract for the boxed-owned-toolchain method.

That makes the correct location:

```text
docs\applications\IDE\vscode\extensions\architectures\boxed-owned-toolchain\
```

## Architecture rules

The current architecture requires:

1. install only through the Maintenance Box
2. stage local forked VSIX artifacts under the shared `dev` area first
3. publish approved maintenance extension state back into the canonical shared extension store afterward

This preserves the single-writer model.

## Canonical install and publish surfaces

### Maintenance install wrapper

```text
C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeMaintenance.ps1
```

### Publish wrapper

```text
C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Publish-VSCodeMaintenance.ps1
```

### Canonical shared extension store

```text
C:\shared\sandbox-toolchains\ide\vscode\extensions\
```

### Recommended shared VSIX staging root

```text
C:\shared\sandbox-toolchains\dev\vscode-extensions\vsix\CyberT33N\
```

## Important contract detail

The current maintenance wrapper parameter is named:

- `-ExtensionId`

but the inner CLI call forwards that value directly to:

- `--install-extension <value>`

That means the same maintenance wrapper can install either:

- a gallery identifier such as `eamodio.gitlens`
- or an absolute VSIX path such as `C:\shared\sandbox-toolchains\dev\vscode-extensions\vsix\CyberT33N\pretty-ts-errors-1.5.1.vsix`

## Step-by-step install sequence

### Step 1 - verify the local VSIX artifacts exist

Confirm that these source artifacts are present:

- `C:\Projects\development-platform\vs-code\extensions\code-navigation\artifacts\vsix\code-navigation-1.0.0.vsix`
- `C:\Projects\development-platform\vs-code\extensions\pretty-ts-errors\artifacts\vsix\pretty-ts-errors-1.5.1.vsix`
- `C:\Projects\development-platform\vs-code\extensions\vscode-error-lens\artifacts\vsix\errorlens-4.1.0.vsix`
- `C:\Projects\development-platform\vs-code\extensions\vscode-symbols\artifacts\vsix\symbols-secured-0.0.27.vsix`
- `C:\Projects\development-platform\vs-code\extensions\vscode-dotenv\artifacts\vsix\dotenv-1.0.1.vsix`
- `C:\Projects\development-platform\vs-code\extensions\vscode-background\artifacts\vsix\background-3.0.0.vsix`
- `C:\Projects\development-platform\vs-code\extensions\vscode_rainbow_csv\artifacts\vsix\rainbow-csv-4.0.0.vsix`

### Step 2 - copy the local VSIX artifacts into the shared `dev` staging area

Create and populate:

```text
C:\shared\sandbox-toolchains\dev\vscode-extensions\vsix\CyberT33N\
```

### Step 2.1 - ensure the Maintenance Box can read the staged VSIX root

The `VS_CODE_MAINTENANCE` Sandboxie configuration must contain:

```ini
ReadFilePath=C:\shared\sandbox-toolchains\dev\vscode-extensions\
```

Why:

- the staged VSIX files are shared **installation inputs**
- the Maintenance Box must be able to see them before `--install-extension <absolute-vsix-path>` can work
- the canonical shared extension store still remains:
  - `C:\shared\sandbox-toolchains\ide\vscode\extensions\`

### Step 3 - install the marketplace extensions into the local maintenance authoring state

Use the canonical VS Code maintenance wrapper with `-Action InstallExtension`.

### Step 4 - install the staged local VSIX artifacts into the local maintenance authoring state

Use the **same** maintenance wrapper, but pass the staged absolute VSIX path through `-ExtensionId`.

This is the same installation surface that was already documented for `dbaeumer.vscode-eslint`.

### Step 5 - publish the local maintenance extension state into the canonical shared store

Run:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_MAINTENANCE `
  /wait `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Publish-VSCodeMaintenance.ps1" `
  -PromoteExtensions
```

### Step 6 - validate the resulting canonical extension set

Run the maintenance wrapper with:

- `-Action ListExtensions`

## Canonical ESLint-derived install pattern

The previously documented installation shape for `dbaeumer.vscode-eslint` was:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_MAINTENANCE `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeMaintenance.ps1" `
  -Action InstallExtension `
  -ExtensionId "dbaeumer.vscode-eslint"
```

For additional marketplace extensions, the same contract is reused by changing only `-ExtensionId`.

For staged local forked VSIX files, the same contract is reused again, but `-ExtensionId` carries the absolute staged VSIX path.

## Complete host-side PowerShell boilerplate

This is the full host-side flow that follows that same documented install surface for **all** extensions.

```powershell
$ErrorActionPreference = 'Stop'
Set-StrictMode -Version Latest

$SharedRoot = 'C:\shared\sandbox-toolchains'
$StartExe = 'C:\Program Files\Sandboxie-Plus\Start.exe'
$PowerShellExe = 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe'
$VSCodeMaintenanceScript = Join-Path $SharedRoot 'dev\bootstrap\platforms\vscode\Start-VSCodeMaintenance.ps1'
$VSCodePublishScript = Join-Path $SharedRoot 'dev\bootstrap\platforms\vscode\Publish-VSCodeMaintenance.ps1'
$SharedVsixRoot = Join-Path $SharedRoot 'dev\vscode-extensions\vsix\CyberT33N'

$MarketplaceExtensions = @(
  'dbaeumer.vscode-eslint',
  'eamodio.gitlens',
  'golang.Go',
  'Nuxt.mdc',
  'ms-azuretools.vscode-containers',
  'ms-vscode-remote.remote-containers',
  'GitHub.vscode-github-actions',
  'ms-playwright.playwright',
  'ms-python.vscode-python-envs',
  'vitest.explorer',
  'ms-azuretools.vscode-docker',
  'redhat.vscode-yaml',
  'docker.docker',
  'ms-vscode.PowerShell',
  'ms-python.vscode-pylance',
  'ms-python.python',
  'ms-python.debugpy',
  'bradlc.vscode-tailwindcss',
  'ms-vscode-remote.remote-wsl'
)

$LocalVsixArtifacts = @(
  @{
    Id = 'CyberT33N.code-navigation'
    SourcePath = 'C:\Projects\development-platform\vs-code\extensions\code-navigation\artifacts\vsix\code-navigation-1.0.0.vsix'
    SharedFileName = 'code-navigation-1.0.0.vsix'
  },
  @{
    Id = 'CyberT33N.pretty-ts-errors'
    SourcePath = 'C:\Projects\development-platform\vs-code\extensions\pretty-ts-errors\artifacts\vsix\pretty-ts-errors-1.5.1.vsix'
    SharedFileName = 'pretty-ts-errors-1.5.1.vsix'
  },
  @{
    Id = 'CyberT33N.errorlens'
    SourcePath = 'C:\Projects\development-platform\vs-code\extensions\vscode-error-lens\artifacts\vsix\errorlens-4.1.0.vsix'
    SharedFileName = 'errorlens-4.1.0.vsix'
  },
  @{
    Id = 'CyberT33N.symbols-secured'
    SourcePath = 'C:\Projects\development-platform\vs-code\extensions\vscode-symbols\artifacts\vsix\symbols-secured-0.0.27.vsix'
    SharedFileName = 'symbols-secured-0.0.27.vsix'
  },
  @{
    Id = 'CyberT33N.dotenv'
    SourcePath = 'C:\Projects\development-platform\vs-code\extensions\vscode-dotenv\artifacts\vsix\dotenv-1.0.1.vsix'
    SharedFileName = 'dotenv-1.0.1.vsix'
  },
  @{
    Id = 'CyberT33N.background'
    SourcePath = 'C:\Projects\development-platform\vs-code\extensions\vscode-background\artifacts\vsix\background-3.0.0.vsix'
    SharedFileName = 'background-3.0.0.vsix'
  },
  @{
    Id = 'CyberT33N.rainbow-csv'
    SourcePath = 'C:\Projects\development-platform\vs-code\extensions\vscode_rainbow_csv\artifacts\vsix\rainbow-csv-4.0.0.vsix'
    SharedFileName = 'rainbow-csv-4.0.0.vsix'
  }
)

function Invoke-HostMaintenanceInstall {
  param(
    [Parameter(Mandatory = $true)]
    [string]$InstallTarget
  )

  Write-Host "Installing via VS_CODE_MAINTENANCE: $InstallTarget"

  & $StartExe `
    /box:VS_CODE_MAINTENANCE `
    /wait `
    $PowerShellExe `
    -NoLogo `
    -ExecutionPolicy Bypass `
    -File $VSCodeMaintenanceScript `
    -Action InstallExtension `
    -ExtensionId $InstallTarget

  if ($LASTEXITCODE -ne 0) {
    throw "Extension installation failed for '$InstallTarget'. ExitCode: $LASTEXITCODE"
  }
}

function Invoke-HostMaintenanceList {
  Write-Host 'Listing extensions via VS_CODE_MAINTENANCE...'

  & $StartExe `
    /box:VS_CODE_MAINTENANCE `
    /wait `
    $PowerShellExe `
    -NoLogo `
    -ExecutionPolicy Bypass `
    -File $VSCodeMaintenanceScript `
    -Action ListExtensions

  if ($LASTEXITCODE -ne 0) {
    throw "Extension listing failed. ExitCode: $LASTEXITCODE"
  }
}

function Invoke-HostMaintenancePublishExtensions {
  Write-Host 'Promoting local maintenance extension state into the canonical shared store...'

  & $StartExe `
    /box:VS_CODE_MAINTENANCE `
    /wait `
    $PowerShellExe `
    -NoLogo `
    -ExecutionPolicy Bypass `
    -File $VSCodePublishScript `
    -PromoteExtensions

  if ($LASTEXITCODE -ne 0) {
    throw "Extension promotion failed. ExitCode: $LASTEXITCODE"
  }
}

if (-not (Test-Path -LiteralPath $VSCodeMaintenanceScript)) {
  throw "VS Code maintenance script not found: $VSCodeMaintenanceScript"
}

if (-not (Test-Path -LiteralPath $VSCodePublishScript)) {
  throw "VS Code publish script not found: $VSCodePublishScript"
}

New-Item -ItemType Directory -Force -Path $SharedVsixRoot | Out-Null

foreach ($artifact in $LocalVsixArtifacts) {
  if (-not (Test-Path -LiteralPath $artifact.SourcePath)) {
    throw "Local VSIX artifact not found: $($artifact.SourcePath)"
  }

  $sharedTarget = Join-Path $SharedVsixRoot $artifact.SharedFileName
  Copy-Item -LiteralPath $artifact.SourcePath -Destination $sharedTarget -Force
  $artifact['SharedPath'] = $sharedTarget

  Write-Host "Staged VSIX: $sharedTarget"
}

foreach ($extensionId in $MarketplaceExtensions) {
  Invoke-HostMaintenanceInstall -InstallTarget $extensionId
}

foreach ($artifact in $LocalVsixArtifacts) {
  Invoke-HostMaintenanceInstall -InstallTarget $artifact.SharedPath
}

Invoke-HostMaintenancePublishExtensions

Invoke-HostMaintenanceList
```

## Alternative: open the Maintenance Box terminal first

If you prefer to work **inside** the already opened `VS_CODE_MAINTENANCE` terminal, use the same install surface from there instead of inventing a second architecture.

### Host step - open the Maintenance terminal

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_MAINTENANCE `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeMaintenance.ps1" `
  -Action OpenTerminal
```

### Inside the already opened Maintenance terminal

```powershell
$ErrorActionPreference = 'Stop'
Set-StrictMode -Version Latest

$MaintenanceScript = 'C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeMaintenance.ps1'
$PublishScript = 'C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Publish-VSCodeMaintenance.ps1'

$MarketplaceExtensions = @(
  'dbaeumer.vscode-eslint',
  'eamodio.gitlens',
  'golang.Go',
  'Nuxt.mdc',
  'ms-azuretools.vscode-containers',
  'ms-vscode-remote.remote-containers',
  'GitHub.vscode-github-actions',
  'ms-playwright.playwright',
  'ms-python.vscode-python-envs',
  'vitest.explorer',
  'ms-azuretools.vscode-docker',
  'redhat.vscode-yaml',
  'docker.docker',
  'ms-vscode.PowerShell',
  'ms-python.vscode-pylance',
  'ms-python.python',
  'ms-python.debugpy',
  'bradlc.vscode-tailwindcss',
  'ms-vscode-remote.remote-wsl'
)

$LocalVsixPaths = @(
  'C:\shared\sandbox-toolchains\dev\vscode-extensions\vsix\CyberT33N\code-navigation-1.0.0.vsix',
  'C:\shared\sandbox-toolchains\dev\vscode-extensions\vsix\CyberT33N\pretty-ts-errors-1.5.1.vsix',
  'C:\shared\sandbox-toolchains\dev\vscode-extensions\vsix\CyberT33N\errorlens-4.1.0.vsix',
  'C:\shared\sandbox-toolchains\dev\vscode-extensions\vsix\CyberT33N\symbols-secured-0.0.27.vsix',
  'C:\shared\sandbox-toolchains\dev\vscode-extensions\vsix\CyberT33N\dotenv-1.0.1.vsix',
  'C:\shared\sandbox-toolchains\dev\vscode-extensions\vsix\CyberT33N\background-3.0.0.vsix',
  'C:\shared\sandbox-toolchains\dev\vscode-extensions\vsix\CyberT33N\rainbow-csv-4.0.0.vsix'
)

foreach ($extensionId in $MarketplaceExtensions) {
  & $MaintenanceScript `
    -Action InstallExtension `
    -ExtensionId $extensionId

  if ($LASTEXITCODE -ne 0) {
    throw "Gallery-Installation fehlgeschlagen: $extensionId (ExitCode: $LASTEXITCODE)"
  }
}

foreach ($vsixPath in $LocalVsixPaths) {
  if (-not (Test-Path -LiteralPath $vsixPath)) {
    throw "VSIX nicht gefunden: $vsixPath"
  }

  & $MaintenanceScript `
    -Action InstallExtension `
    -ExtensionId $vsixPath

  if ($LASTEXITCODE -ne 0) {
    throw "VSIX-Installation fehlgeschlagen: $vsixPath (ExitCode: $LASTEXITCODE)"
  }
}

& $PublishScript -PromoteExtensions

if ($LASTEXITCODE -ne 0) {
  throw "Promotion fehlgeschlagen. ExitCode: $LASTEXITCODE"
}
```

## Minimal command sequence without the larger boilerplate

If you want the high-level manual sequence instead of the combined script:

1. copy the local VSIX artifacts into:
   - `C:\shared\sandbox-toolchains\dev\vscode-extensions\vsix\CyberT33N\`
2. ensure the `VS_CODE_MAINTENANCE` box config contains:
   - `ReadFilePath=C:\shared\sandbox-toolchains\dev\vscode-extensions\`
3. run the maintenance wrapper once per marketplace identifier
4. run the maintenance wrapper once per staged VSIX path
5. run:
   - `Start.exe /box:VS_CODE_MAINTENANCE ... -File Publish-VSCodeMaintenance.ps1 -PromoteExtensions`
6. validate with `-Action ListExtensions`

## Related

- `docs\applications\IDE\vscode\extensions\architectures\boxed-owned-toolchain\inventory.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\extensions-seeds-and-local-state.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\cli\start\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\maintenance-box.md`
