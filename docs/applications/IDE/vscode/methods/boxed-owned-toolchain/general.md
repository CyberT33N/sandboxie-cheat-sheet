# Boxed-Owned Toolchain / Boxed Authoring

## Status

This is the preferred VS Code method for this repository.

It is the target architecture for fully boxed VS Code authoring with Sandboxie.

## Ranking against the other methods

This method is ranked as:

1. preferred default architecture
2. above the host-sync / materialization method
3. far above the host-installed toolchain anti-pattern

Use the host-sync method only when the IDE must remain on the host.

Keep host-installed dependency installation only as a legacy anti-pattern reference.

## What this method means

The target model is:

- no regular VS Code installation on the host as part of the development workflow
- every normal VS Code authoring instance runs inside Sandboxie
- each project gets its own dedicated project box
- a dedicated Maintenance Box owns the canonical shared IDE assets
- shared paths hold only canonical runtime, catalog, extension, seed, and toolchain artifacts
- live writable project state remains box-local
- edge-case projects are still developed boxed and execute later through dedicated runner contexts if necessary

This method is therefore:

- fully boxed for regular authoring
- not host-not-isolated
- not host-materialized as the primary model
- not based on host-side `nvm use`, host Git, host pnpm, or host VS Code

## Why this is Method #1

From an enterprise and domain-driven perspective, this method now scores highest because it creates the cleanest control-plane / execution-plane split:

- shared canonical artifacts are centrally governed
- project boxes consume those artifacts without becoming global writers
- project state stays isolated per box
- edge-case execution can be handled later by runner contexts without breaking the boxed authoring contract

The previously documented obsolete framing is no longer a supported method category.

## Method principles

### 1. No host IDE as the normal path

The host is not the primary VS Code control plane anymore.

### 2. One project, one box

Each project gets its own boxed VS Code authoring environment.

### 3. Shared is canonical, not live multi-writer state

Shared paths are used for:

- runtime
- catalog
- extensions
- seeds
- toolchains
- bootstrap and runner helpers

Shared paths are not used as a live writable cross-project IDE state surface.

### 4. Maintenance is the single writer

The Maintenance Box is the only global author for shared IDE artifacts.

### 5. Project boxes are consumers

Project boxes consume runtime, extensions, settings, and seeds. They do not directly author the shared canonical state.

### 6. Development stays boxed even for edge cases

Projects that later need host-near execution are still opened and developed boxed first. Their execution can later move to:

- a runner box with deliberately expanded rights
- or a host runner after explicit export / sync

### 7. No hidden runtime switching

The architecture uses fixed versioned binaries and bootstrap-selected runtimes. It does not depend on `nvm use` or similar mutable shim switching.

## Full shared tree

The fully modeled target tree for this method is:

```text
C:\shared\sandbox-toolchains\
  ide\
    vscode\
      runtime\
        1.121.0\
      catalog\
        vscode-user\
          settings.json
          keybindings.json
          snippets\
        seed\
          globalStorage\
          roo\
      extensions\
      maintenance\
        user-data\
  dev\
    git\
      2.54.0\
    node\
      26.2.0\
        node-v26.2.0-win-x64\
      20.19.6\
        node-v20.19.6-win-x64\
    pnpm\
      11.2.2\
        package\
          bin\
            pnpm.cjs
    bootstrap\
      core\
        Bootstrap.Common.psm1
      platforms\
        vscode\
          Bootstrap.VSCode.psm1
          Start-VSCodeMaintenance.ps1
          Start-VSCodeProjectBase.ps1
      stacks\
        node\
          Bootstrap.Node.psm1
  projects\
    test-mono\
      bootstrap\
        Project.Config.ps1
        Start-TestMonoVSCode.ps1
        Start-TestMonoTerminal.ps1
      export\
      runner-input\
```

## Why the download / provisioning docs belong here

The host-side commands that:

- create the shared tree
- download the VS Code runtime
- download versioned Node binaries
- prepare shared pnpm
- extract PortableGit

are not generic Git, Node, or VS Code tutorials.

They provision the shared artifact tree that exists specifically for this method.

That is why the correct documentation location is inside this method area, under:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\...`

and not under generic application reference folders.

Generic Git/Node/pnpm conceptual docs can still live in their own application areas, but the concrete shared-artifact provisioning code for this architecture belongs here.

## Box roles

### Maintenance Box

The Maintenance Box is the global control-plane box for shared IDE assets.

Its responsibilities are:

- install, update, remove, and list shared extensions
- maintain the canonical `settings.json`
- maintain the canonical `keybindings.json`
- maintain snippets
- maintain seed material for `globalStorage`
- maintain seed material for `.roo`
- validate the shared extension store and the shared runtime

It is CLI-first. GUI launch may still be useful for diagnosis or curation, but it is not the normal daily authoring plane.

### Project Box

The Project Box is the normal authoring environment for one project.

It owns:

- the boxed repo
- box-local VS Code `user-data`
- box-local extension runtime copy
- box-local `globalStorage`
- box-local `.roo`
- box-local caches, logs, sessions, and workspace state

### Runner Box / Runner Context

The runner layer exists for edge-case execution that cannot remain inside the normal project box.

It is not a replacement for boxed authoring. It is a separate execution plane used later when needed.

## VS Code runtime, catalog, seeds, and local state

### Canonical shared surfaces

The following stay canonical in `C:\shared\sandbox-toolchains\ide\vscode\...`:

- VS Code runtime
- `settings.json`
- `keybindings.json`
- snippets
- canonical extension store
- seed material for extension data that cannot be freely relocated

### Box-local runtime state

The following stay box-local:

- `user-data`
- extension runtime copy used by the project box
- `globalStorage`
- `workspaceStorage`
- logs
- cache
- sessions
- `.roo`

### Hardcoded path handling

For extension data that cannot be redirected cleanly, the architecture uses:

- a canonical seed in shared
- a local copy in the box
- explicit promotion later if needed

Important examples:

- `.roo`  
  canonical seed: `ide\vscode\catalog\seed\roo\`  
  live runtime path: `C:\Users\denni\.roo`
- global storage payloads such as extension-managed state  
  canonical seed: `ide\vscode\catalog\seed\globalStorage\...`  
  live runtime path: box-local `user-data\User\globalStorage\...`

## Why extension state is synchronized locally

This is a critical architectural decision.

The project box does **not** point `--extensions-dir` directly at the shared extension store during normal runtime.

Why:

1. VS Code attempted to write under the shared extension path when that directory was used directly as the project runtime `--extensions-dir`.
2. Granting project boxes write rights there would create a multi-writer canonical extension store.
3. That would violate the Maintenance Box contract.
4. Project boxes are consumers, not global authors.

Therefore the final runtime contract is:

- Maintenance Box writes to the canonical shared extension store
- project bootstrap mirrors that shared store into a box-local extension directory
- the project VS Code instance runs with the local mirrored `--extensions-dir`

This is why the sync architecture exists.

It preserves:

- single-writer governance
- auditability
- reproducibility
- disposable project boxes

## Toolchain architecture

### Git

Git is a canonical shared runtime:

- form: `PortableGit`
- version: `2.54.0`

### Node

Node is provided as versioned shared runtimes.

The validated example architecture uses:

- `26.2.0` as the primary control-plane / tooling runtime
- `20.19.6` as an additional secondary runtime domain

### pnpm

pnpm is stored centrally as unpacked CLI content:

- version: `11.2.2`
- entrypoint: `package\bin\pnpm.cjs`

It is intentionally **not** modeled as `@pnpm/exe`, because the runtime must not be hidden inside pnpm itself.

### nvm

`nvm` is not part of the final architecture contract.

Why:

- `nvm use` mutates shims and runtime selection state
- it creates implicit runtime switching
- it conflicts with the explicit shared-runtime model

The method uses fixed versioned binaries selected by bootstrap.

### Host state

Final target state:

- no host VS Code
- no host Git
- no host Node
- no host pnpm
- no host nvm

Host exception:

- `Starship` can remain on the host because it is shell/prompt infrastructure rather than project toolchain governance

## Monorepo multi-runtime model

For sanitized monorepo examples such as `test-mono`, the method allows a mixed runtime contract:

- primary tooling and package-manager shell on `Node 26.2.0`
- a secondary project-visible command such as `node20` bound to `Node 20.19.6`

This supports architectures where:

- the outer monorepo control plane uses a newer Node runtime
- one inner application/runtime domain still targets an older fixed runtime

Bootstrap exposes this deliberately and explicitly rather than relying on mutable version switching.

## Bootstrap architecture

### Why bootstrap is mandatory

Bootstrap is the place where the method:

- selects the correct shared artifacts
- initializes the local runtime state
- copies canonical catalog files
- initializes seed-backed paths
- mirrors the shared extension store locally
- wires the toolchain environment
- launches `Code.exe` or `code.cmd`

### Why not `$PROFILE`

`$PROFILE` is not the architecture center because it is:

- implicit
- user-specific
- drift-prone
- hard to govern

### Why not workspace settings for toolchain selection

Workspace settings are not the canonical place for:

- machine-local absolute binary paths
- shared runtime paths
- bootstrap-selected command surfaces

Those belong in the bootstrap layer.

### Why no box-specific shell copies

The final method does not rely on project-specific `cmd.exe` / `powershell.exe` copies as an architecture principle.

That was an older workaround pattern.

The final design prefers:

- normal Windows shell binaries
- canonical VS Code terminal settings
- explicit bootstrap-provided environment

## Canonical VS Code settings

The canonical `settings.json` role is:

- editor and terminal UX defaults
- Starship integration
- shell profile defaults

It is **not** the place for project-specific absolute toolchain paths.

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

## Sandboxie-specific validated rules

### Maintenance Box

For CLI-driven extension operations:

- do **not** keep `Template=Chrome_KB5027231_fix`
- do **not** keep `SpecialImage=chrome,Code.exe`

This is necessary so `code.cmd`-based extension commands do not fail with `PrintCompositorLPAC` flag injection.

### Project Boxes

For project-authoring boxes:

- do **not** keep `Template=Chrome_KB5027231_fix`
- do **not** keep `SpecialImage=chrome,Code.exe`
- use `UseWin32kHooks=y` as the validated compatibility choice

Rationale:

- `Code.exe` is used for both GUI and internal server processes
- classifying it as a Chromium special image injects flags that break internal Node/server modes
- removing that special handling avoids the fatal CLI/server mismatch

### Expected observations

- `SBIE2190` can appear after removing `SpecialImage` and is not the primary blocker
- `SBIE2205` / `ConsoleInit` should be treated as a lower-priority warning unless it causes a concrete function break

## Current validated implementation areas

The current method documents:

- the shared bootstrap tree
- the shared script bodies
- a sanitized project-adapter boilerplate
- host-side commands for:
  - maintenance extension installation
  - maintenance extension listing
  - project terminal launch
  - boxed VS Code GUI launch
- terminal verification commands for:
  - `git`
  - `node`
  - `pnpm`
  - `node20`

## Documentation map

### Method overview

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\general.md`

### Shared bootstrap layout and code

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`

### Boxed CLI overlays

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\cli\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\cli\start\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\cli\start\terminal.md`

### Sanitized project boilerplate

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\start.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`

### Method-specific provisioning

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
