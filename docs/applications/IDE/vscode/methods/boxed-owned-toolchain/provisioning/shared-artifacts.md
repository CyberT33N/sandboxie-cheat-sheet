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
  "$SharedIdeRoot\catalog\vscode-user\assets\backgrounds\t33n",
  "$SharedIdeRoot\catalog\seed\globalStorage",
  "$SharedIdeRoot\catalog\seed\roo",
  "$SharedIdeRoot\extensions",
  "$SharedIdeRoot\maintenance\user-data",
  "$SharedRoot\dev\git\2.54.0",
  "$SharedRoot\dev\node\26.2.0",
  "$SharedRoot\dev\node\20.19.6",
  "$SharedRoot\dev\pnpm\11.2.2",
  "$SharedRoot\dev\pnpm\11.5.0",
  "$SharedRoot\dev\python\3.14.5",
  "$SharedRoot\dev\starship\1.25.1",
  "$SharedBootstrapRoot\core",
  "$SharedBootstrapRoot\platforms\vscode",
  "$SharedBootstrapRoot\stacks\node",
  "$SharedBootstrapRoot\stacks\python",
  "$SharedBootstrapRoot\stacks\starship",
  "$SharedProjectRoot\bootstrap",
  "$SharedProjectRoot\export",
  "$SharedProjectRoot\runner-input"
)

$Dirs | ForEach-Object {
  New-Item -ItemType Directory -Force -Path $_ | Out-Null
}
```

The shared `maintenance\user-data` path above remains part of the modeled tree, but it is not the current live maintenance authoring surface.

The current maintenance workflow authors local state under `C:\Program Files\SandboxToolchains\VSCodeBoxes\maintenance\...` and promotes approved changes back into the shared canonical surfaces.

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

## Toolchain-specific provisioning ownership

Binary-specific provisioning is now owned by the application domains and is re-referenced here instead of duplicated in full:

- Git: `docs\applications\git\architectures\boxed-owned-toolchain\overview.md`
- Node runtime: `docs\applications\programming-languages\node\runtime\architectures\boxed-owned-toolchain\overview.md`
- PNPM: `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\versioning-and-provisioning.md`
- Python: `docs\applications\programming-languages\python\general.md`
- Starship: `docs\applications\terminal\starship\architectures\boxed-owned-toolchain\overview.md`

This method document keeps only the shared-tree orchestration and VS Code-specific provisioning surfaces.

## Canonical VS Code settings file

The canonical settings file belongs here:

```text
C:\shared\sandbox-toolchains\ide\vscode\catalog\vscode-user\settings.json
```

Target content:

```json
{
  "terminal.integrated.automationProfile.windows": {
    "path": "${env:BOXED_GIT_ROOT}\\bin\\bash.exe",
    "args": [
      "--noprofile",
      "--rcfile",
      "${env:BOXED_BASH_MINIMAL_RC}",
      "-i"
    ],
    "env": {
      "CHERE_INVOKING": "1"
    }
  },
  "terminal.integrated.defaultProfile.windows": "Boxed Git Bash (Starship)",
  "terminal.integrated.profiles.windows": {
    "Boxed Git Bash (Starship)": {
      "path": "${env:BOXED_GIT_ROOT}\\bin\\bash.exe",
      "args": [
        "--noprofile",
        "--rcfile",
        "${env:BOXED_BASH_STARSHIP_RC}",
        "-i"
      ],
      "env": {
        "CHERE_INVOKING": "1"
      }
    },
    "Boxed Git Bash (Minimal)": {
      "path": "${env:BOXED_GIT_ROOT}\\bin\\bash.exe",
      "args": [
        "--noprofile",
        "--rcfile",
        "${env:BOXED_BASH_MINIMAL_RC}",
        "-i"
      ],
      "env": {
        "CHERE_INVOKING": "1"
      }
    }
  },
  "terminal.integrated.inheritEnv": true,
  "[windows]": {
    "eslint.runtime": "C:\\shared\\sandbox-toolchains\\dev\\node\\20.19.6\\node-v20.19.6-win-x64\\node.exe"
  }
}
```

This current recovery state intentionally:

- keeps the automation profile on a minimal Bash RC
- keeps the default interactive profile on a Starship-enabled Bash RC
- resolves the shell and RC paths through bootstrap-provided `${env:...}` variables

This allows the canonical settings file to remain shared while bootstrap still injects the correct box-local runtime paths at launch time.

The deliberate extension-specific exception is `eslint.runtime`, which is pinned to the governed Node 20 binary instead of a host `nvm` path.

For the architecture rationale, read:

- `docs\applications\IDE\vscode\extensions\eslint\general.md`
- `docs\applications\IDE\vscode\extensions\eslint\architectures\boxed-owned-toolchain\settings-json.md`

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
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\starship.md`
