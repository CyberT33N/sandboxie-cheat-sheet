# Provision Shared Artifacts

## Scope

This document contains the host-side provisioning commands that create and populate the shared artifact tree for the boxed-owned-toolchain method.

These commands are method-specific because they provision:

- the shared VS Code runtime tree
- the shared catalog and seed tree
- the shared extension store
- the shared Git / Node / pnpm toolchain tree
- the shared bootstrap tree
- the shared project / runner subtrees

## Why this document lives here

These commands do not belong in generic Git, Node, pnpm, or VS Code application reference folders.

They are not "how to use Git" or "how to install Node" in general.

They are the host-side provisioning code for the shared artifact tree required by this method.

That is why the correct documentation location is:

```text
docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\
```

## Shared tree creation

Sanitized example using `test-mono`:

```powershell
$VsCodeVersion = "1.121.0"

$SharedRoot = "C:\shared\sandbox-toolchains"
$SharedIdeRoot = "$SharedRoot\ide\vscode"
$SharedProjectRoot = "$SharedRoot\projects\test-mono"
$SharedBootstrapRoot = "$SharedRoot\dev\bootstrap"

$Dirs = @(
  "$SharedIdeRoot\runtime\$VsCodeVersion",
  "$SharedIdeRoot\catalog\vscode-user",
  "$SharedIdeRoot\catalog\vscode-user\snippets",
  "$SharedIdeRoot\catalog\seed\globalStorage",
  "$SharedIdeRoot\catalog\seed\roo",
  "$SharedIdeRoot\extensions",
  "$SharedIdeRoot\maintenance\user-data",
  "$SharedRoot\dev\git\2.54.0",
  "$SharedRoot\dev\node\26.2.0",
  "$SharedRoot\dev\node\20.19.6",
  "$SharedRoot\dev\pnpm\11.2.2",
  "$SharedBootstrapRoot\core",
  "$SharedBootstrapRoot\platforms\vscode",
  "$SharedBootstrapRoot\stacks\node",
  "$SharedProjectRoot\bootstrap",
  "$SharedProjectRoot\export",
  "$SharedProjectRoot\runner-input"
)

$Dirs | ForEach-Object {
  New-Item -ItemType Directory -Force -Path $_ | Out-Null
}
```

## Download VS Code runtime

```powershell
$VsCodeVersion = "1.121.0"
$SharedIdeRoot = "C:\shared\sandbox-toolchains\ide\vscode"
$ZipPath = Join-Path $env:TEMP "vscode-$VsCodeVersion-win32-x64.zip"
$RuntimePath = "$SharedIdeRoot\runtime\$VsCodeVersion"

Invoke-WebRequest `
  -Uri "https://update.code.visualstudio.com/$VsCodeVersion/win32-x64-archive/stable" `
  -OutFile $ZipPath

New-Item -ItemType Directory -Force -Path $RuntimePath | Out-Null
Remove-Item "$RuntimePath\*" -Recurse -Force -ErrorAction SilentlyContinue

Expand-Archive -LiteralPath $ZipPath -DestinationPath $RuntimePath -Force

Test-Path "$RuntimePath\Code.exe"
```

## Download Node `26.2.0`

```powershell
$Node26Version = "26.2.0"
$Node26Zip = Join-Path $env:TEMP "node-v$Node26Version-win-x64.zip"
$Node26Dest = "C:\shared\sandbox-toolchains\dev\node\$Node26Version"

Invoke-WebRequest `
  -Uri "https://nodejs.org/dist/v$Node26Version/node-v$Node26Version-win-x64.zip" `
  -OutFile $Node26Zip

Remove-Item "$Node26Dest\*" -Recurse -Force -ErrorAction SilentlyContinue
Expand-Archive -LiteralPath $Node26Zip -DestinationPath $Node26Dest -Force
```

## Download Node `20.19.6`

```powershell
$Node20Version = "20.19.6"
$Node20Zip = Join-Path $env:TEMP "node-v$Node20Version-win-x64.zip"
$Node20Dest = "C:\shared\sandbox-toolchains\dev\node\$Node20Version"

Invoke-WebRequest `
  -Uri "https://nodejs.org/dist/v$Node20Version/node-v$Node20Version-win-x64.zip" `
  -OutFile $Node20Zip

Remove-Item "$Node20Dest\*" -Recurse -Force -ErrorAction SilentlyContinue
Expand-Archive -LiteralPath $Node20Zip -DestinationPath $Node20Dest -Force
```

## Prepare shared pnpm CLI content

```powershell
$Node26Root = "C:\shared\sandbox-toolchains\dev\node\26.2.0\node-v26.2.0-win-x64"
$PnpmVersion = "11.2.2"
$PnpmRoot = "C:\shared\sandbox-toolchains\dev\pnpm\$PnpmVersion"

New-Item -ItemType Directory -Force -Path $PnpmRoot | Out-Null
Remove-Item "$PnpmRoot\*" -Recurse -Force -ErrorAction SilentlyContinue

Push-Location $PnpmRoot
& "$Node26Root\npm.cmd" pack "pnpm@$PnpmVersion"
tar -xf "pnpm-$PnpmVersion.tgz"
Pop-Location
```

Resulting entrypoint:

```text
C:\shared\sandbox-toolchains\dev\pnpm\11.2.2\package\bin\pnpm.cjs
```

## 7-Zip helper download for PortableGit extraction

```powershell
$SevenZipInstaller = Join-Path $env:TEMP '7z2601-x64.exe'
$SevenZipDir = Join-Path $env:TEMP '7zip2601'

Invoke-WebRequest `
  -Uri 'https://github.com/ip7z/7zip/releases/download/26.01/7z2601-x64.exe' `
  -OutFile $SevenZipInstaller

Remove-Item $SevenZipDir -Recurse -Force -ErrorAction SilentlyContinue

Start-Process -FilePath $SevenZipInstaller -ArgumentList '/S',('/D=' + $SevenZipDir) -Wait
```

## Extract PortableGit `2.54.0`

```powershell
$SevenZipDir = Join-Path $env:TEMP '7zip2601'
$SevenZipExe = Join-Path $SevenZipDir '7z.exe'
$GitSfx = Join-Path $env:TEMP 'PortableGit-2.54.0-64-bit.7z.exe'
$GitDest = 'C:\shared\sandbox-toolchains\dev\git\2.54.0'

Remove-Item "$GitDest\*" -Recurse -Force -ErrorAction SilentlyContinue

& $SevenZipExe x $GitSfx ('-o' + $GitDest) -y

& "$GitDest\cmd\git.exe" --version
```

Expected verification:

```text
git version 2.54.0.windows.1
```

## Canonical VS Code settings file

The canonical settings file belongs here:

```text
C:\shared\sandbox-toolchains\ide\vscode\catalog\vscode-user\settings.json
```

Target content:

```json
{
  "terminal.integrated.automationProfile.windows": {
    "path": "C:\\Windows\\System32\\cmd.exe"
  },
  "terminal.integrated.defaultProfile.windows": "Boxed PowerShell",
  "terminal.integrated.profiles.windows": {
    "Boxed PowerShell": {
      "path": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
      "args": [
        "-NoExit",
        "-ExecutionPolicy",
        "Bypass",
        "-Command",
        "& 'C:\\Program Files\\starship\\bin\\starship.exe' init powershell --print-full-init | Out-String | Invoke-Expression"
      ],
      "env": {
        "PATH": "C:\\Program Files\\starship\\bin;${env:PATH}",
        "STARSHIP_CONFIG": "C:\\Users\\denni\\.config\\starship.toml"
      }
    },
    "Boxed CMD": {
      "path": "C:\\Windows\\System32\\cmd.exe"
    }
  },
  "terminal.integrated.inheritEnv": true
}
```

## Optional migration examples

These are optional migration helpers, not required as part of the scratch-first default:

### Copy host `settings.json`

```powershell
Copy-Item "$env:APPDATA\Code\User\settings.json" `
  "C:\shared\sandbox-toolchains\ide\vscode\catalog\vscode-user\settings.json" `
  -Force
```

### Copy host `globalStorage`

```powershell
Copy-Item "$env:APPDATA\Code\User\globalStorage\zgsm-ai.zgsm" `
  "C:\shared\sandbox-toolchains\ide\vscode\catalog\seed\globalStorage\zgsm-ai.zgsm" `
  -Recurse -Force
```

### Copy host `.roo`

```powershell
Copy-Item "$env:USERPROFILE\.roo\*" `
  "C:\shared\sandbox-toolchains\ide\vscode\catalog\seed\roo" `
  -Recurse -Force
```

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
