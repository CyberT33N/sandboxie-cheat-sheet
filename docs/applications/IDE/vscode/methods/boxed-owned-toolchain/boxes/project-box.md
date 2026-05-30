# Project Box

## Role

The Project Box is the normal boxed authoring environment for exactly one project.

Each project gets its own dedicated VS Code box.

## Responsibilities

A Project Box owns the live runtime state for one project:

- the boxed repo
- box-local VS Code `user-data`
- box-local mirrored extension runtime copy
- box-local `globalStorage`
- box-local `.roo`
- box-local logs, caches, sessions, and workspace state
- project-local build, test, and development artifacts

## Why one project per box

This split exists because:

- Project A and Project B must not share the same live IDE / extension / state context
- a compromised or broken project state must not mutate the runtime state of other projects
- the blast radius must stay project-bounded

## Relationship to shared artifacts

The Project Box consumes shared artifacts but does not become a global writer.

It reads:

- shared VS Code runtime
- shared catalog
- shared canonical extension store
- shared toolchain runtimes

It then materializes the required local runtime state:

- local `user-data`
- local mirrored extension directory
- local seed-backed paths

## Normal operating mode

The Project Box is the place that should normally:

- open the boxed VS Code GUI
- provide the integrated terminal for boxed development
- run project-local dev/build/test commands

## What it must not do

The Project Box must not directly author:

- the canonical catalog
- the canonical shared extension store
- the canonical seed store

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\architecture\governance.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\extensions-seeds-and-local-state.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\sandboxie\project-box.md`
