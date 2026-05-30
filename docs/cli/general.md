# CLI

## Purpose

This folder is the single source of truth for general command-line control-plane behavior in this repository.

From an enterprise and domain-driven perspective, this area belongs at `docs\cli\` because the capability is cross-cutting:

- it is not specific to VS Code
- it is not specific to one project box
- it can apply to maintenance, project, runner, or future utility boxes

The capability being documented is:

- select a specific box
- launch a program into that box
- optionally open a terminal in that box
- optionally pass startup commands to the newly launched process

That makes this a control-plane concern, not an application-specific concern.

## Why this lives in `docs\cli`

If the same launch semantics are needed by:

- VS Code maintenance flows
- VS Code project authoring flows
- future runner flows
- generic diagnostics
- non-VS-Code applications

then the documentation must be centralized outside `docs\applications\IDE\vscode\...`.

Putting the single source of truth under `docs\cli\` keeps the model capability-centric instead of consumer-centric.

## Domain split

The recommended high-level split is:

```text
docs\
  cli\
    general.md
    start\
      general.md
    terminal\
      general.md
```

### `start\`

The `start` area documents how a program is launched into a named box.

This is where `Start.exe` belongs.

### `terminal\`

The `terminal` area documents how an interactive shell is opened inside a named box, how commands are executed there, and how stdout/stderr should be observed.

This is a separate subdomain because terminal behavior carries additional operational concerns:

- keeping the shell open
- visible logs
- interactive diagnosis
- command-passing boundaries
- exit-code handling

## Single source of truth rule

Generic CLI semantics must live here.

Application-specific areas such as:

- `docs\applications\IDE\vscode\...`

should keep only the integration-specific overlays and reference the generic CLI documents.

That means:

- box launch semantics stay here
- generic terminal-open semantics stay here
- VS Code-specific extension installation stays in the VS Code area
- VS Code-specific GUI/bootstrap launch behavior stays in the VS Code area

## Current documents

Start here:

- `docs\cli\start\general.md`
- `docs\cli\terminal\general.md`
