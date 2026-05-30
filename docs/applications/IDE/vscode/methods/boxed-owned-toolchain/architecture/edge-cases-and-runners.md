# Edge Cases And Runners

## Scope

This document defines how the boxed-owned-toolchain method treats edge-case projects and runner contexts.

## Standard projects

For normal projects, the target state is:

- repo boxed
- VS Code boxed
- development boxed
- tests and builds boxed
- no host-visible active working tree required

## Edge-case projects

Examples include:

- recursive host filesystem scanners
- registry-near tools
- host-near system manipulation utilities
- later host-near security or audit tools

## Core decision

Even for these edge-case projects:

- development remains boxed
- the project is still opened boxed in VS Code

The method does **not** fall back to a host IDE just because execution later needs a different context.

## Runner concept

When execution later cannot remain in the normal project box, the method uses a dedicated runner context.

Planned variants:

1. a runner box with deliberately expanded rights
2. a host runner after explicit export / sync

## Why runners exist

The runner layer separates:

- authoring
- normal project-box execution
- edge-case execution

This preserves the integrity of the authoring model while still allowing exceptional execution paths when truly needed.

## What runners are not

Runners are not:

- a replacement for boxed authoring
- a reason to keep VS Code on the host
- a justification for collapsing the Maintenance Box / Project Box split

## Export and runner-input surfaces

The modeled shared tree keeps explicit project-level surfaces for later runner use:

```text
C:\shared\sandbox-toolchains\
  projects\
    test-mono\
      export\
      runner-input\
```

These surfaces exist to support explicit handoff into later runner contexts rather than accidental live mutation of the normal boxed project runtime.

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\architecture\target-state.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\runner-context.md`
