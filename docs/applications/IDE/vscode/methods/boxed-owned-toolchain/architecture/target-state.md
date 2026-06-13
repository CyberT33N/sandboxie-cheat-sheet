# Target State

## Scope

This document describes the final target state of the boxed-owned-toolchain method.

It is the architecture-level source of truth for what this method is trying to achieve.

## Final target model

The final target state is:

- no regular VS Code installation remains on the host as part of the development workflow
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

## Architectural layers

### Shared runtime / catalog / toolchain layer

This layer contains:

- the VS Code runtime
- the canonical VS Code user catalog
- the canonical extension store
- seed material for path-hardcoded extension state
- shared Git / Node / pnpm runtimes
- shared shell runtimes under `dev\shells\...`
- shared Starship runtime
- bootstrap and runner helpers

### Project box layer

This layer contains per project:

- a dedicated boxed VS Code instance
- the boxed repo
- box-local `user-data`
- a box-local mirrored extension runtime copy
- box-local mirrored `cmd.exe` and PowerShell shell lanes
- box-local `globalStorage`
- box-local `.roo`
- box-local caches, logs, sessions, and workspace state

### Runner layer

This layer exists for execution cases that must later move beyond the normal project box.

It contains:

- runner boxes with intentionally expanded rights
- or host runner flows after explicit export / sync

## Core architecture principles

### No host IDE as the normal path

The host is not the primary VS Code control plane anymore.

### One project, one box

Each project gets its own boxed VS Code authoring environment.

### Shared is canonical, not live multi-writer state

Shared paths are used for:

- runtime
- catalog
- extensions
- seeds
- toolchains
- bootstrap and runner helpers

Shared paths are not used as a live writable cross-project IDE state surface.

### Maintenance is the single writer

The Maintenance Box is the only global author for shared IDE artifacts.

### Project boxes are consumers

Project boxes consume runtime, extensions, settings, and seeds. They do not directly author the shared canonical state.

### Development stays boxed even for edge cases

Projects that later need host-near execution are still opened and developed boxed first.

### No hidden runtime switching

The architecture uses fixed versioned binaries and bootstrap-selected runtimes. It does not depend on `nvm use` or similar mutable shim switching.

## Full shared tree snapshot

For quick orientation, the current full modeled tree is:

This is a **sanitized modeled snapshot**.

The project subtree uses the sanitized example name `test-mono`.

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
      20.9.0\
        node-v20.9.0-win-x64\
    pnpm\
      11.5.0\
        package\
          bin\
            pnpm.cjs
    shells\
      cmd\
        10.0.26100.8457\
          cmd.exe
      powershell\
        10.0.26100.8457\
          powershell.exe
      clink\
        1.9.26\
          clink_x64.exe
    python\
    starship\
      1.25.1\
        starship.exe
    bootstrap\
      core\
        Bootstrap.Common.psm1
      platforms\
        vscode\
          Bootstrap.VSCode.psm1
          Start-VSCodeMaintenance.ps1
          Start-VSCodeProjectBase.ps1
          Publish-VSCodeMaintenance.ps1
      stacks\
        node\
          Bootstrap.Node.psm1
        shells\
          Bootstrap.WindowsShells.psm1
        python\
          Bootstrap.Python.psm1
        starship\
          Bootstrap.Starship.psm1
  projects\
    test-mono\
      bootstrap\
        Project.Config.ps1
        Start-TestMonoVSCode.ps1
        Start-TestMonoTerminal.ps1
      export\
      runner-input\
```

For the dedicated bootstrap-oriented explanation of the same tree, read:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`

## Method ranking

This method is ranked as:

1. preferred default architecture
2. above the host-sync / materialization method
3. far above the host-installed toolchain anti-pattern

Use the host-sync method only when the IDE must remain on the host.

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\architecture\governance.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\architecture\edge-cases-and-runners.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\general.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
