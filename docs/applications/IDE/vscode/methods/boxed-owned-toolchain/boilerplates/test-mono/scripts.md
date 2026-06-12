# `test-mono` Boilerplate Scripts

## Scope

This document contains the complete project-adapter PowerShell boilerplate for a sanitized example project named `test-mono`.

Replace:

- `test-mono`
- `VS_CODE_TEST_MONO`
- `C:\Users\yourusername\source\test-mono`

with your real project slug, box name, and repo path.

## Why this document lives here

This is not a generic CLI document and not a shared bootstrap-kernel document.

It is a sanitized project adapter example, so the correct documentation location is:

```text
docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\
```

That keeps:

- generic CLI semantics in `docs\cli\...`
- shared bootstrap scripts in `bootstrap\...`
- project-adapter examples in `boilerplates\...`

## `Project.Config.ps1`

```powershell
$sharedRoot = 'C:\shared\sandbox-toolchains'
$vsCodeRoot = Join-Path $sharedRoot 'ide\vscode'
$devRoot = Join-Path $sharedRoot 'dev'

return @{
  SharedRoot = $sharedRoot
  ProjectName = 'test-mono'
  BoxName = 'VS_CODE_TEST_MONO'
  DefaultRepoPath = 'C:\Users\yourusername\source\test-mono'
  VSCode = @{
    CodeExe = Join-Path $vsCodeRoot 'runtime\1.121.0\Code.exe'
    CodeCli = Join-Path $vsCodeRoot 'runtime\1.121.0\bin\code.cmd'
    CatalogUserRoot = Join-Path $vsCodeRoot 'catalog\vscode-user'
    SharedExtensionsRoot = Join-Path $vsCodeRoot 'extensions'
    SeedGlobalStorageRoot = Join-Path $vsCodeRoot 'catalog\seed\globalStorage'
    SeedRooRoot = Join-Path $vsCodeRoot 'catalog\seed\roo'
  }
  Toolchain = @{
    GitRoot = Join-Path $devRoot 'git\2.54.0'
    NodeRoot = Join-Path $devRoot 'node\26.2.0\node-v26.2.0-win-x64'
    PnpmCli = Join-Path $devRoot 'pnpm\11.5.0\package\bin\pnpm.cjs'
    PythonRoot = Join-Path $devRoot 'python'
    AdditionalNodeCommands = [ordered]@{
      node20 = Join-Path $devRoot 'node\20.9.0\node-v20.9.0-win-x64\node.exe'
    }
  }
  Shells = @{
    CmdRoot = Join-Path $devRoot 'shells\cmd\10.0.26100.8457'
    PowerShellRoot = Join-Path $devRoot 'shells\powershell\10.0.26100.8457'
    RegRoot = Join-Path $devRoot 'shells\reg\10.0.26100.8457'
    ClinkRoot = Join-Path $devRoot 'shells\clink\1.9.26'
    StarshipRoot = Join-Path $devRoot 'starship\1.25.1'
    StarshipConfigPath = Join-Path $env:USERPROFILE '.config\starship.toml'
  }
}
```

## `Start-TestMonoVSCode.ps1`

```powershell
param(
  [ValidateSet('LaunchVSCode', 'OpenTerminal')]
  [string]$Action = 'LaunchVSCode',

  [string]$RepoPath
)

$ErrorActionPreference = 'Stop'
Set-StrictMode -Version Latest

$config = & (Join-Path $PSScriptRoot 'Project.Config.ps1')

if (-not $config) {
  throw 'Project config did not return a configuration object.'
}

$resolvedRepoPath = if ([string]::IsNullOrWhiteSpace($RepoPath)) {
  $config.DefaultRepoPath
}
else {
  $RepoPath
}

$baseScript = Join-Path $config.SharedRoot 'dev\bootstrap\platforms\vscode\Start-VSCodeProjectBase.ps1'

if (-not (Test-Path -LiteralPath $baseScript)) {
  throw "Base project launcher not found: $baseScript"
}

$parameters = @{
  Action = $Action
  ProjectName = $config.ProjectName
  RepoPath = $resolvedRepoPath
  CodeExe = $config.VSCode.CodeExe
  CodeCli = $config.VSCode.CodeCli
  CatalogUserRoot = $config.VSCode.CatalogUserRoot
  SharedExtensionsRoot = $config.VSCode.SharedExtensionsRoot
  SeedGlobalStorageRoot = $config.VSCode.SeedGlobalStorageRoot
  SeedRooRoot = $config.VSCode.SeedRooRoot
  GitRoot = $config.Toolchain.GitRoot
  NodeRoot = $config.Toolchain.NodeRoot
  PnpmCli = $config.Toolchain.PnpmCli
  PythonRoot = $config.Toolchain.PythonRoot
  CmdRoot = $config.Shells.CmdRoot
  PowerShellRoot = $config.Shells.PowerShellRoot
  RegRoot = $config.Shells.RegRoot
  StarshipRoot = $config.Shells.StarshipRoot
  ClinkRoot = $config.Shells.ClinkRoot
  StarshipConfigPath = $config.Shells.StarshipConfigPath
  AdditionalNodeCommands = $config.Toolchain.AdditionalNodeCommands
}

& $baseScript @parameters

if ($Action -eq 'OpenTerminal') {
  return
}

exit $LASTEXITCODE
```

## `Start-TestMonoTerminal.ps1`

```powershell
param(
  [string]$RepoPath
)

$ErrorActionPreference = 'Stop'
Set-StrictMode -Version Latest

$launcher = Join-Path $PSScriptRoot 'Start-TestMonoVSCode.ps1'

if (-not (Test-Path -LiteralPath $launcher)) {
  throw "Project launcher not found: $launcher"
}

& $launcher -Action OpenTerminal -RepoPath $RepoPath
```

## `Start-TestMonoPnpmInstall.ps1`

This is the sanitized project-box install script.

The PNPM-domain source of truth for the install-script contract now lives here:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`

The PNPM-domain source of truth for the clean-reinstall script contract now lives here:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`

It encodes the governance-approved boxed-owned-toolchain contract:

- the host launches one explicit project-owned PS1
- the script enters the project box through the normal project bootstrap
- the script does **not** choose the PNPM version through host arguments
- the effective PNPM version comes from `Project.Config.ps1`

The current shell-specific requirement is also part of this contract:

- boxed `cmd.exe` is the preferred productive lifecycle `scriptShell`
- boxed PowerShell remains the preferred interactive default shell
- Git Bash remains an alternative shell lane and still needs a shell-native `pnpm` wrapper in `bootstrap-bin`, not only `pnpm.cmd`

```powershell
param(
  [string]$RepoPath = 'C:\Users\yourusername\source\test-mono'
)

$ErrorActionPreference = 'Stop'
Set-StrictMode -Version Latest

$launcher = Join-Path $PSScriptRoot 'Start-TestMonoVSCode.ps1'

if (-not (Test-Path -LiteralPath $launcher)) {
  throw "Project launcher not found: $launcher"
}

& $launcher -Action OpenTerminal -RepoPath $RepoPath

if ([string]::IsNullOrWhiteSpace($env:BOXED_CMD_EXE)) {
  throw 'BOXED_CMD_EXE was not initialized by project bootstrap.'
}

$cmdExe = $env:BOXED_CMD_EXE
if (-not (Test-Path -LiteralPath $cmdExe)) {
  throw 'Local boxed CMD executable not found.'
}

$null = Initialize-NodeGypWindowsBuildEnvironment `
  -CmdExe $env:BOXED_CMD_EXE `
  -RegExe $env:BOXED_REG_EXE `
  -PythonExe $env:BOXED_PYTHON_EXE

pnpm config set --location=project scriptShell "$cmdExe"
pnpm install

exit $LASTEXITCODE
```

### Host command after materializing the boilerplate above into the shared project bootstrap subtree

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_TEST_MONO `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoPnpmInstall.ps1" `
  -RepoPath "C:\Users\yourusername\source\test-mono"
```

After you materialize the boilerplate script above into the sanitized shared project subtree, this host command executes it.

The example assumes the sanitized project keeps its repo path visible inside the project box and wants to run a project-owned dependency refresh with the currently validated boxed PNPM lifecycle-shell setup.

The version-provisioning SSOT for moving the project contract to a newer PNPM version lives here:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\versioning-and-provisioning.md`

## Notes

This boilerplate intentionally keeps `node20` as an additional command because it reflects the validated monorepo example currently used for the architecture.

The optional `PythonRoot`, `StarshipRoot`, `ClinkRoot`, and `StarshipConfigPath` entries reflect the current boxed-owned-toolchain runtime shape where the project bootstrap can initialize:

- Python
- Starship
- `reg.exe` as an explicit boxed helper lane
- the CMD-specific `Clink` adapter
- explicit local PowerShell/CMD shell lanes

in addition to the core Git/Node/pnpm surfaces.

If a project does not need a secondary runtime, remove the `AdditionalNodeCommands` entry entirely.

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\start.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
