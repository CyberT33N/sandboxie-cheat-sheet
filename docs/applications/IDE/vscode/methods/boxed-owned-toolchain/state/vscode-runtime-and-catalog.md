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

This is the maintenance control-plane user-data context used for shared IDE maintenance operations.

## Canonical settings role

The canonical `settings.json` contains:

- editor and terminal UX defaults
- Starship integration
- shell profile defaults

It does **not** contain project-specific absolute toolchain paths.

Those belong in bootstrap and project-adapter logic.

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
  "terminal.integrated.inheritEnv": true
}
```

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
