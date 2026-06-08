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

The current recovery state keeps the terminal profile wiring minimal and lets bootstrap provide the dynamic shell/runtime details.

In particular:

- the terminal profile resolves its shell path via `${env:BOXED_GIT_ROOT}`
- the terminal profile resolves its Bash RC file via `${env:BOXED_BASH_MINIMAL_RC}` / `${env:BOXED_BASH_STARSHIP_RC}`
- the RC file performs the shell-specific Starship initialization

This keeps the canonical settings file generic while still letting each box receive the correct local mirrored shell/runtime surfaces from bootstrap.

## Canonical settings content

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

## ESLint runtime exception

`eslint.runtime` is intentionally bound to an explicit Node binary:

- `C:\shared\sandbox-toolchains\dev\node\20.19.6\node-v20.19.6-win-x64\node.exe`

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

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\starship.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\extensions-seeds-and-local-state.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
- `docs\applications\IDE\vscode\extensions\eslint\general.md`
