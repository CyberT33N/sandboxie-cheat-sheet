# Boxed-Owned Toolchain / Boxed Authoring

## Status

This is the preferred VS Code method for this repository.

It is the target architecture for fully boxed VS Code authoring with Sandboxie.

## Why this file exists

This file is the method-level landing page.

It intentionally stays small and points to the domain documents that hold the actual architecture details.

## Method summary

Core model:

- no regular VS Code installation remains on the host as part of the normal workflow
- every normal VS Code authoring instance runs inside Sandboxie
- each project gets its own dedicated project box
- a dedicated Maintenance Box owns the canonical shared IDE assets
- shared paths hold only canonical runtime, catalog, extension, seed, and toolchain artifacts
- live writable project state remains box-local
- edge-case projects are still developed boxed and execute later through dedicated runner contexts if necessary

## Reading order

### 1. Architecture

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\architecture\target-state.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\architecture\governance.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\architecture\edge-cases-and-runners.md`

### 2. Box roles

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\maintenance-box.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\project-box.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\runner-context.md`

### 3. State and seeds

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\vscode-runtime-and-catalog.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\extensions-seeds-and-local-state.md`

### 4. Toolchain

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\git.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\node.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\pnpm.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\python.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\host-state.md`

### 5. Bootstrap

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`

### 6. Sandboxie-specific rules

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\sandboxie\maintenance-box.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\sandboxie\project-box.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\sandboxie\observations-and-signals.md`
- `docs\performance\filesystem\sandboxie-debug-tracing.md`

### 7. Provisioning

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`

### 8. CLI overlays

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\cli\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\cli\start\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\cli\start\terminal.md`

### 9. Sanitized boilerplates

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\start.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`

### 10. Git auth and initial clone

- `docs\applications\git\architectures\boxed-owned-toolchain\overview.md`

### 11. Extension-specific settings

- `docs\applications\IDE\vscode\extensions\eslint\general.md`

### 12. Application-domain toolchain sources of truth

- `docs\applications\git\general.md`
- `docs\applications\programming-languages\node\runtime\general.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\general.md`
- `docs\applications\programming-languages\python\general.md`
- `docs\applications\terminal\starship\general.md`
- `docs\applications\version-control\monorepo\nx\general.md`

## Full shared tree snapshot

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

For the bootstrap-specific explanation of this tree, read:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
