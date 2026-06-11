# VS Code Runtime And Catalog

## Scope

This document describes the canonical shared VS Code runtime and catalog surfaces used by the boxed-owned-toolchain method.

## Canonical shared VS Code surfaces

The following remain canonical under:

```text
C:\shared\sandbox-toolchains\ide\vscode\
```

### Runtime

```text
ide\vscode\runtime\1.121.0\
```

This is the canonical VS Code runtime source.

It is read-only from the point of view of normal project consumption.

### User catalog

```text
ide\vscode\catalog\vscode-user\
```

This contains the canonical:

- `settings.json`
- `keybindings.json`
- `snippets\`

These files are copied or mirrored into the box-local `user-data` runtime paths during bootstrap.

### Maintenance user-data

```text
ide\vscode\maintenance\user-data\
```

This path remains part of the shared tree model, but it is not the current live maintenance authoring surface.

The current maintenance workflow authors local state under:

- `C:\Program Files\SandboxToolchains\VSCodeBoxes\maintenance\state\user-data`

and then promotes approved catalog or seed changes back into the canonical shared surfaces.

## Canonical settings role

The canonical `settings.json` contains:

- editor and terminal UX defaults
- Starship integration
- shell profile defaults
- extension-specific governed runtime bindings such as `eslint.runtime`

It does **not** contain project-specific absolute toolchain paths as a general toolchain-selection mechanism.

Those belong in bootstrap and project-adapter logic.

The deliberate exception is an extension-specific runtime binding that must point at one explicit governed binary to remain stable and reviewable.

In the current boxed-owned-toolchain contract, `eslint.runtime` is such an exception and is documented in the VS Code extension domain instead of being treated as generic terminal/toolchain selection.

The current recovery state keeps the terminal profile wiring explicit and lets bootstrap provide the dynamic shell/runtime details.

In particular:

- `Boxed PowerShell (Starship)` is the preferred interactive default profile
- `Boxed PowerShell` is the preferred minimal automation profile
- the explicit CMD and PowerShell profiles resolve their shell paths via `${env:BOXED_CMD_EXE}` and `${env:BOXED_POWERSHELL_EXE}`
- the CMD + Starship lane resolves its Clink profile via `${env:BOXED_CMD_STARSHIP_PROFILE}`
- the explicit Git Bash profiles resolve their shell path via `${env:BOXED_GIT_ROOT}`
- the explicit Git Bash profiles resolve their Bash RC files via `${env:BOXED_BASH_MINIMAL_RC}` / `${env:BOXED_BASH_STARSHIP_RC}`
- the Bash RC files prepend the local `bootstrap-bin` directory into the Bash `PATH`
- the Bash RC files perform the Git-Bash-specific Starship initialization

Those explicit CMD/PowerShell shell paths are now expected to come from governed shared shell artifacts under `dev\shells\...`, mirrored locally by bootstrap into the box execution tree.

That `bootstrap-bin` export still matters because the available Git Bash profile must be able to resolve bootstrap-generated shell wrappers such as:

- `pnpm`
- `node20`

This keeps the canonical settings file generic while still letting each box receive the correct local mirrored shell/runtime surfaces from bootstrap.

For the CMD + Starship lane, `Clink` is the CMD-specific runtime adapter. If `Clink` is not provisioned, the profile intentionally falls back to plain CMD instead of trying to fake Starship support.

## Canonical settings content

```json
{
  "terminal.integrated.automationProfile.windows": {
    "path": "${env:BOXED_POWERSHELL_EXE}",
    "args": [
      "-NoLogo",
      "-NoExit",
      "-NoProfile",
      "-ExecutionPolicy",
      "Bypass",
      "-File",
      "${env:BOXED_POWERSHELL_MINIMAL_INIT}"
    ]
  },
  "terminal.integrated.defaultProfile.windows": "Boxed PowerShell (Starship)",
  "terminal.integrated.profiles.windows": {
    "Boxed PowerShell (Starship)": {
      "path": "${env:BOXED_POWERSHELL_EXE}",
      "args": [
        "-NoLogo",
        "-NoExit",
        "-NoProfile",
        "-ExecutionPolicy",
        "Bypass",
        "-File",
        "${env:BOXED_POWERSHELL_STARSHIP_INIT}"
      ]
    },
    "Boxed PowerShell": {
      "path": "${env:BOXED_POWERSHELL_EXE}",
      "args": [
        "-NoLogo",
        "-NoExit",
        "-NoProfile",
        "-ExecutionPolicy",
        "Bypass",
        "-File",
        "${env:BOXED_POWERSHELL_MINIMAL_INIT}"
      ]
    },
    "Boxed CMD (Starship)": {
      "path": "${env:BOXED_CMD_EXE}",
      "args": [
        "/d",
        "/k",
        "call",
        "${env:BOXED_CMD_STARSHIP_INIT}"
      ]
    },
    "Boxed CMD": {
      "path": "${env:BOXED_CMD_EXE}",
      "args": [
        "/d",
        "/k",
        "call",
        "${env:BOXED_CMD_MINIMAL_INIT}"
      ]
    },
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
    "eslint.runtime": "C:\\shared\\sandbox-toolchains\\dev\\node\\20.9.0\\node-v20.9.0-win-x64\\node.exe"
  }
}
```

This is the corrected declaration for the current preferred profile order:

- PowerShell is the preferred VS Code default
- CMD remains an explicit supported Windows shell lane
- Git Bash remains an explicit supported shell lane instead of being treated as the only valid option

## ESLint runtime exception

`eslint.runtime` is intentionally bound to an explicit Node binary:

- `C:\shared\sandbox-toolchains\dev\node\20.9.0\node-v20.9.0-win-x64\node.exe`

Why:

- the ESLint extension needs a deterministic runtime surface
- the old host `nvm` path is not part of the boxed-owned-toolchain contract
- this binding is extension-specific, not a generic replacement for bootstrap-selected terminal/runtime wiring

The extension-specific explanation lives here:

- `docs\applications\IDE\vscode\extensions\eslint\general.md`
- `docs\applications\IDE\vscode\extensions\eslint\architectures\boxed-owned-toolchain\settings-json.md`

## Why not workspace settings for toolchain selection

Workspace settings are not the canonical place for:

- machine-local absolute binary paths
- shared runtime paths
- bootstrap-selected command surfaces

Those belong in the bootstrap layer.

## Why not box-specific shell copies

The final method does not rely on project-specific `cmd.exe` / `powershell.exe` copies as an architecture principle.

The final design prefers:

- a locally mirrored shell runtime selected by bootstrap
- canonical VS Code terminal settings
- explicit bootstrap-provided environment

## Starship location

`Starship` is provisioned under:

- `C:\shared\sandbox-toolchains\dev\starship\...`

and mirrored locally into the box execution tree during bootstrap.

The prompt config file may still remain user-owned, for example:

- `C:\Users\denni\.config\starship.toml`

## Related

- `docs\cli\shell\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\starship.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\extensions-seeds-and-local-state.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
- `docs\applications\IDE\vscode\extensions\eslint\general.md`
