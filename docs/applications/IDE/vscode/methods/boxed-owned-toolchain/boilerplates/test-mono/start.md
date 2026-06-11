# Start a Project Box - `test-mono` Boilerplate

## Scope

This document is the sanitized project-box start boilerplate for a generic example project named `test-mono`.

It documents:

- the concrete host commands
- the runtime contract
- the test commands
- the relationship between the shared extension store and the local runtime copy

Replace the example identifiers with your real project values:

- `test-mono`
- `VS_CODE_TEST_MONO`
- `C:\Users\yourusername\source\test-mono`

## Why this document lives here

From an enterprise and domain-driven perspective, this is a boilerplate project-adapter document.

It is not the generic CLI source of truth, and it is not the shared bootstrap-kernel source of truth.

That is why the correct location is:

```text
docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\start.md
```

## References

Generic command-line semantics:

- `docs\cli\general.md`
- `docs\cli\start\general.md`
- `docs\cli\terminal\general.md`

Shared bootstrap tree and shared script bodies:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`

Project-adapter boilerplate scripts:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`

## Example shared tree

The sanitized example project layout inside the shared toolchain root is:

```text
C:\shared\sandbox-toolchains\
  projects\
    test-mono\
      bootstrap\
        Project.Config.ps1
        Start-TestMonoVSCode.ps1
        Start-TestMonoTerminal.ps1
```

This `projects\` subtree is part of the shared runtime tree, not part of the cheat-sheet documentation tree.

The documentation tree intentionally keeps boilerplates separately from real projects.

## Runtime contract

The current project-box contract is:

- Maintenance Box publishes the canonical shared extension store:
  - `C:\shared\sandbox-toolchains\ide\vscode\extensions`
- Project Box is a consumer
- Shared extensions are mirrored to:
  - `C:\Program Files\SandboxToolchains\VSCodeBoxes\test-mono\state\extensions`
- VS Code `user-data` is local to the project box
- local mirrored VS Code runtime lives under:
  - `C:\Program Files\SandboxToolchains\VSCodeBoxes\test-mono\execution\runtime\...`
- local mirrored toolchain lives under:
  - `C:\Program Files\SandboxToolchains\VSCodeBoxes\test-mono\execution\toolchain\...`
- the canonical VS Code user catalog is copied into local `user-data`
- seed-backed runtime state is initialized if missing

This prevents a multi-writer shared extension directory.

## Default debugging posture

All host-side PowerShell commands below intentionally include:

```powershell
-NoExit
```

That keeps the starter terminal visible for debugging.

## Step 1 - verify the canonical shared extension store

```powershell
Get-ChildItem -LiteralPath "C:\shared\sandbox-toolchains\ide\vscode\extensions" -Directory | Select-Object Name
```

This confirms that the canonical extension store is populated before the project box mirrors it locally.

## Step 2 - initial boxed clone before the repo exists

The project bootstrap requires the target `RepoPath` to already exist.

That means:

- `Start-TestMonoVSCode.ps1 -Action OpenTerminal` is the correct path only **after** the repo exists
- a fresh machine or a fully cleared project box needs one initial boxed clone first

The full Git auth and device-code login flow is the Git-domain source of truth:

- `docs\applications\git\architectures\boxed-owned-toolchain\overview.md`

The preferred one-shot clone pattern is:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_TEST_MONO `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -Command "Set-Location 'C:\'; `$env:HOME = `$env:USERPROFILE; `$env:GIT_CEILING_DIRECTORIES = 'C:/Users/yourusername/source'; New-Item -ItemType Directory -Force -Path 'C:\Users\yourusername\source' | Out-Null; & 'C:\shared\sandbox-toolchains\dev\git\2.54.0\cmd\git.exe' -c credential.helper=manager clone 'https://yourgithubuser@github.com/yourorg/test-mono.git' 'C:\Users\yourusername\source\test-mono'"
```

If Git for Windows shows the helper-selection dialog during this first private access:

- choose `manager`
- enable `Always use this from now on`

If Git Credential Manager then offers the device-code flow, complete that in the normal host browser according to:

- `docs\applications\git\architectures\boxed-owned-toolchain\overview.md`

This keeps the clone inside the project box while still letting the host launch exactly one reproducible command.

## Step 3 - open the project terminal from the host

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_TEST_MONO `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoVSCode.ps1" `
  -Action OpenTerminal `
  -RepoPath "C:\Users\yourusername\source\test-mono"
```

What happens:

- the named project box is entered
- the normal Windows PowerShell binary is started inside the box
- the project config is loaded
- the generic VS Code project bootstrap is called
- local `user-data`, `extensions`, and `bootstrap-bin` paths are prepared
- the selected VS Code runtime is mirrored locally
- the shared extension store is mirrored into the local project runtime copy
- the canonical VS Code user catalog is copied into local `user-data`
- the Node stack is wired into `PATH`
- the explicit local CMD/PowerShell shell lanes are prepared
- optional Python, Starship, and Clink runtime layers can also be initialized by the project config

This intentionally avoids project-specific shell copies and shared terminal binaries.

In the Git Bash terminal path, `pnpm` is expected to resolve through the bootstrap-generated shell wrapper in `bootstrap-bin`, not only through a Windows `.cmd` shim.

## Step 4 - verify the project toolchain in the open boxed terminal

Run these commands inside the open project terminal:

```powershell
git --version
node --version
pnpm --version
node20 --version
python --version
```

Expected interpretation:

- `git` should resolve to the shared Git runtime
- `node` should resolve to the primary shared Node runtime
- `pnpm` should be callable from the prepared environment
- `node20` should resolve only if the example project keeps the additional secondary runtime
- `python` should resolve only if the example project keeps the optional shared Python runtime

If the resolved `pnpm` version does not match the design target, treat that as a follow-up command-resolution item rather than a project-box start failure.

## Step 5 - start the boxed VS Code GUI from the host

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_TEST_MONO `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoVSCode.ps1" `
  -Action LaunchVSCode `
  -RepoPath "C:\Users\yourusername\source\test-mono"
```

What happens:

- the project adapter script loads `Project.Config.ps1`
- the generic project bootstrap mirrors the shared extension store locally
- VS Code launches with:
  - local `--user-data-dir`
  - local `--extensions-dir`
  - the repo path

## Step 6 - verify the VS Code integrated terminal

Inside the integrated terminal in the boxed VS Code window, run:

```powershell
git --version
node --version
pnpm --version
node20 --version
python --version
```

This confirms that the bootstrap-provided environment also reached the integrated terminal.

The current validated integrated-terminal profile set can include:

- `Boxed Git Bash (Starship)`
- `Boxed Git Bash (Minimal)`
- `Boxed PowerShell`
- `Boxed PowerShell (Starship Test)`
- `Boxed CMD`
- `Boxed CMD (Starship Test)`

## Maintenance prerequisite commands

These host-driven maintenance wrapper commands are the preferred operational path.
Do not treat manual `code.cmd` typing inside an already-open maintenance terminal as the normal workflow; keep that as a troubleshooting fallback.

### Install an extension into the canonical shared store

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_MAINTENANCE `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeMaintenance.ps1" `
  -Action InstallExtension `
  -ExtensionId "RooVeterinaryInc.roo-cline"
```

### List extensions from the canonical shared store

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_MAINTENANCE `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeMaintenance.ps1" `
  -Action ListExtensions
```

## Project-box example - run `pnpm install` from the host

The governance-approved boxed-owned-toolchain pattern is:

1. keep the selected PNPM version in the project contract
2. let the host launch one explicit project-owned install PS1
3. let that script enter the project box through the normal project bootstrap
4. let that script set the validated lifecycle `scriptShell`
5. run `pnpm install`

The sanitized boilerplate install script body still lives in:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`

The PNPM-domain source of truth for the install-script contract lives in:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`

The PNPM-domain source of truth for the clean-reinstall script contract lives in:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`

The PNPM-domain source of truth for the version-provisioning step that must happen **before** such an install when the project contract is updated lives in:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\versioning-and-provisioning.md`

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\cli\start\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\cli\start\terminal.md`
