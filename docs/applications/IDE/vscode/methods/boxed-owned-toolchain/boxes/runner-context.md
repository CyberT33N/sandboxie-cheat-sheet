# Runner Context

## Role

The Runner Context is the execution layer used when an edge-case workflow cannot remain inside the normal project box.

It is an execution-plane construct, not an authoring-plane replacement.

## Supported forms

Planned runner forms are:

1. a runner box with deliberately expanded rights
2. a host runner after explicit export / sync

## Why this exists

Some projects or commands may later require:

- host-near file access
- registry-near behavior
- system-level operations
- other execution surfaces that should not expand the normal project-box trust envelope

The Runner Context exists to preserve the boxed authoring contract while still enabling those exceptional execution paths.

## What remains boxed

Even when a runner is used later:

- project authoring remains boxed
- the project still opens boxed in VS Code
- the project box remains the default development context

## Shared surfaces used for runners

The modeled shared tree includes runner-oriented project surfaces:

```text
C:\shared\sandbox-toolchains\
  projects\
    test-mono\
      export\
      runner-input\
```

These are explicit handoff surfaces, not a reason to turn the normal project box into a runner substitute.

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\architecture\edge-cases-and-runners.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\project-box.md`
