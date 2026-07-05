# Nested Child-Process Orchestration Hangs Under Sandboxie

## Scope

This is the primary reference document for one specific troubleshooting class in boxed Windows workflows:

- a command works in isolation
- the same lower-level command surface can also work through a shallow manual test
- but the **full canonical orchestration path** still hangs at the final child-process boundary

This document is intentionally:

- not Nx-specific
- not PNPM-specific
- not Electron-specific
- not `electron-vite`-specific
- not tied to one repository layout

At the same time, the fully validated repository example used in this document is:

- `pnpm exec`
- `nx run`
- a project-owned tooling runner
- `electron-vite.CMD`
- direct `node.exe -> electron-vite.js`

That example is included here **as an example only**.

## What is proven and what is still a hypothesis

### Proven in the validated example

The currently validated boxed-owned-toolchain example proved all of the following:

1. the isolated dependency wrapper surface can work
2. the isolated direct JavaScript runtime surface can work
3. the project-owned inner tooling runner can work
4. backend/frontend/readiness sibling surfaces can work
5. the full canonical orchestration lane can still hang later at the final child-process spawn edge
6. the final productive fix can live entirely in the box/bootstrap/control-plane layer without changing business code

### Not yet proven generically

The following broader statements remain a **reasoned hypothesis**, not a universal proof:

1. that every package manager can hit this same issue
2. that every orchestrator can hit this same issue
3. that every `.CMD`-based dependency wrapper behaves the same way
4. that the exact same root cause always appears outside the validated `pnpm exec -> Nx -> tooling runner -> dependency wrapper` example

So the correct generalization is:

- this failure class is **architecturally plausible** across other stacks
- but in the current repository evidence it is only **fully validated** for the concrete example described below

## Problem statement

The core problem is **not**:

- "the dependency command cannot run at all"
- "the framework cannot run at all"
- "Sandboxie blocks every `.CMD` file"
- "Nx is broken"
- "PNPM is broken"

The core problem is:

- a **deeply nested child-process orchestration chain**
- under a **boxed Windows runtime**
- reaches a final process-spawn edge
- where the last Windows shell / wrapper / handle / `stdio` boundary blocks

That means a system can show this combination without contradiction:

1. the inner framework command works directly
2. the wrapper command works directly
3. the project-owned runner works directly
4. but the full outer canonical command still hangs at the final launch edge

## The important architectural split

The most important split in this troubleshooting class is:

1. **isolated execution**
2. **fully orchestrated execution**

Those are not the same surface.

In the validated repository example, all of the following could be true at different stages:

- direct `electron-vite.CMD dev --watch` worked
- direct `node .\node_modules\electron-vite\bin\electron-vite.js dev --watch` worked
- direct `pnpm exec tsx tooling/run/cli.ts serve --profile=desktop-dev` worked
- backend and frontend readiness checks were green
- but the canonical `pnpm exec nx run ...` path still hung at the last process-start edge

That is the architectural signal.

## Why isolated success does not settle the real issue

Isolated success only proves that:

- the lower-level executable exists
- the lower-level wrapper is runnable
- the inner application/runtime is not universally broken

It does **not** prove that the same surface will behave identically when it is launched through:

1. a package-manager execution layer
2. an orchestration framework
3. sibling watch tasks
4. a project-owned runner
5. a final shell-wrapper boundary
6. inherited console handles / inherited `stdio`
7. sandboxed process mediation

That is why the correct diagnostic question is not:

- "Can the inner command run at all?"

It is:

- "Can the inner command still be launched correctly at the end of the real canonical orchestration chain?"

## Typical ingredients of this failure class

This troubleshooting class becomes more likely when several of the following are present together:

1. Windows shell wrappers such as `.CMD`
2. `cmd.exe /d /s /c ...`
3. Node-based `child_process.spawn(...)`
4. deep orchestration through package managers and build tools
5. long-lived sibling watch processes
6. environment projection through bootstrap layers
7. strict boxed/sandboxed process mediation
8. inherited terminal handles or inherited `stdio`

None of these ingredients is automatically wrong on its own.

The issue arises from the **combination** and from the **exact final boundary** where the last process is launched.

## What `.CMD` files are and are not

### What they usually are

Dependency-provided `.CMD` files on Windows are often:

- wrappers
- trampolines
- indirection layers
- entrypoint shims

They often resolve a path and then launch:

- `node.exe`
- a JavaScript file
- or another command interpreter

### What they are not

They are **not** all identical.

Different dependencies can use:

- different argument handling
- different environment handling
- different fallback logic
- different quoting rules
- different child-process depth

So the correct architectural statement is:

- many `.CMD` files play a similar **role**
- but they do **not** all implement the exact same internal behavior

### What the validated example proved about `.CMD`

The validated repository example proved:

- a dependency `.CMD` wrapper can work in isolation under the current Sandboxie architecture
- therefore the root cause was **not** "boxed `.CMD` execution is generally impossible"

The actual problem sat one level higher:

- the **final nested orchestration launch edge**

## Fully validated example chain

The following chain is a **project-specific example**, not the generic definition of the problem:

```text
interactive boxed terminal
  -> pnpm exec nx run <target>
    -> Nx task orchestration
      -> sibling watch/readiness tasks
        -> project-owned tooling runner
          -> child_process.spawn(cmd.exe /d /s /c <dependency>.CMD ...)
            -> dependency wrapper (.CMD)
              -> node.exe
                -> framework JS entrypoint
                  -> final application/runtime boot
```

In the concrete validated case, the last part of the chain became:

```text
tooling runner
  -> spawn(cmd.exe /d /s /c electron-vite.CMD dev --watch)
    -> rewritten to:
       node.exe electron-vite.js dev --watch
```

That example is repository-specific.

The architectural lesson is generic:

- the lower-level framework entrypoint can be healthy
- while the **fully orchestrated path to that entrypoint** is still unhealthy

## What worked in isolation in the validated example

The validated example established these green surfaces:

1. direct dependency-wrapper execution
2. direct JavaScript-entrypoint execution
3. direct inner tooling-runner execution
4. backend readiness
5. frontend readiness
6. direct Electron runtime boot

Architecturally that means:

- the problem was **not** a blanket runtime incompatibility
- the problem was **not** a blanket dependency-wrapper incompatibility
- the problem was **not** a blanket framework incompatibility

## What did not work, or did not explain the problem

The validated example also established that the following were **not** the primary answer:

1. blaming backend readiness
2. blaming frontend readiness
3. blaming the dependency wrapper alone
4. blaming the `.CMD` shell layer alone
5. blaming the project-owned runner as a generic idea
6. rewriting application/business code
7. changing the orchestration graph just to avoid the boundary

There were also important intermediate non-solutions:

1. replay lanes could reproduce parts of the behavior, but they were not the source of truth
2. direct rewrite from `.CMD` to `node.exe -> <tool>.js` was valuable, but alone it was not yet sufficient
3. `stdin=inherit`, `stdout/stderr=pipe` still left the final spawn edge hanging in the canonical lane

This matters because the document must preserve what **did not** solve the issue, not only what eventually did.

## Root-cause interpretation

The current best-supported interpretation is:

- the failure sat at the **last direct spawn-/handle-/lifecycle boundary**
- in the **real canonical orchestration lane**
- after the rest of the orchestration graph had already done its work

More precisely:

- the final process-start edge was still crossing a Windows shell/wrapper/runtime boundary
- that edge inherited a process context created by a much deeper orchestration chain
- and that final edge could block synchronously before the child became healthy and visible as a normal running process

So the problem should be described as:

- a **nested child-process orchestration problem**
- with a **final Windows spawn boundary failure**

not merely as:

- an Nx problem
- a PNPM problem
- an Electron-Vite problem
- or a `.CMD` file problem

## Why the outer command matters

The outermost command matters because it defines the **parent process tree** and therefore the final launch context.

That is why this troubleshooting class can be present only in the canonical command, even if:

- the inner command works
- the shell wrapper works
- the direct framework entrypoint works

In the validated example, the critical difference was exactly this:

- the outer canonical chain created the process tree in which the final launch edge became unstable

That is why the real truth surface had to remain the full canonical orchestration command.

## Solution architecture

The validated solution pattern was:

1. keep the business/application code unchanged
2. keep the real canonical command unchanged
3. fix the problem in the bootstrap/control-plane layer
4. instrument the last spawn edge
5. target the exact problematic final launch surface
6. remove unnecessary shell indirection from that final surface when possible
7. apply an explicit `stdio` / handle strategy to that same final launch
8. mirror output back to the parent console so the developer still sees logs

This keeps the architecture honest:

- the application still runs through its real intended path
- but the boxed toolchain owns the process-boundary adaptation it needs

## Validated concrete solution pattern

The validated concrete example used all of the following:

1. a generated child-process tracer injected through `NODE_OPTIONS`
2. detection of the final problematic spawn:
   - `cmd.exe /d /s /c <dependency>.CMD dev --watch`
3. direct rewrite of that final spawn to:
   - local `node.exe`
   - local JavaScript framework entrypoint
4. explicit stabilized `stdio` / handle treatment on that rewritten spawn
5. mirrored `stdout` / `stderr` back to the parent console
6. preservation of the original working directory and environment
7. retention of a dependency-wrapper shim for isolated/manual/fallback surfaces

## Why the validated solution worked

The current best explanation is:

1. the final shell-wrapper boundary was reduced
2. the last launch no longer depended on the same unstable wrapper path
3. the inherited console/handle surface was narrowed and controlled explicitly
4. the final child could therefore start without blocking the same way it did before

The decisive architectural point is:

- the successful fix was **not** "use Node directly" alone
- and it was **not** "change `stdio`" alone

The successful pattern was:

- **direct rewrite**
- **plus**
- **a matching stable `stdio` / handle policy on the same rewritten spawn**

## What the example does not prove

The validated example does **not** prove all of the following:

1. that every future similar issue must be solved by the same exact rewrite
2. that every wrapper should always be bypassed
3. that every package manager or orchestrator will show the same failure shape
4. that every sandboxed Windows issue is really this failure class

So the generic reusable conclusion is narrower:

- when lower-level surfaces work but the fully orchestrated canonical lane hangs at the last spawn edge, investigate the final nested child-process boundary first

## Correct general reuse rule

Use this troubleshooting model when all of the following are true:

1. direct lower-level execution works
2. the inner framework entrypoint works
3. the outer canonical command still hangs
4. the hang happens at or immediately after a final child-process spawn
5. a Windows shell/wrapper layer is involved
6. the environment is mediated by a strict box/sandbox/control-plane

Do **not** use this document as a shortcut to skip proper lower-level validation first.

## When not to use this pattern

Do not jump directly to wrapper bypass or direct JavaScript entrypoint rewrites when:

1. the inner framework command already fails in isolation
2. the framework/runtime installation itself is broken
3. the failure is obviously a missing file, missing dependency, or wrong runtime version
4. the orchestration layer has not yet been separated from the inner command surface

## Documentation placement

This document is the **primary SSOT** for the generic failure class.

Domain-specific documents should only keep:

1. their domain-local consequences
2. their domain-local example code
3. their domain-local execution contract

They should re-reference this document for the generic architecture explanation instead of duplicating it.

## Related

- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\lifecycle-and-command-surface.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\electron-vite\architectures\boxed-owned-toolchain\overview.md`
