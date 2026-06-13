# Boxed-Owned-Toolchain Git

## Architectural status

This is the **preferred Git architecture path** in this repository.

The host-sync Git path remains only as a secondary / legacy reference here:

- `docs\applications\git\architectures\host-sync\overview.md`

## Role of this file

This file is now the **TOC / entrypoint** for the boxed-owned-toolchain Git source of truth.

The previous high-density overview has been split by concern so the Git domain can stay:

- single-source-of-truth oriented
- easier to re-reference from VS Code method documents
- easier to evolve without mixing provisioning, runtime, auth, and troubleshooting concerns

## Current reference snapshot

The current boxed-owned-toolchain Git reference truth is:

1. governed shared PortableGit under `dev\git\...`
2. local mirrored Git runtime inside the box
3. bootstrap-published boxed `git` wrapper surfaces:
   - `git`
   - `git.cmd`
   - `git.ps1`
4. bootstrap-published boxed Git Credential Manager helper surfaces:
   - `git-credential-manager-boxed`
   - `git-credential-manager-boxed.cmd`
   - `git-credential-manager-boxed.ps1`
5. automatic Git-side long-path participation through:
   - `core.longpaths=true`
6. Git-independent Windows long-path policy documented centrally in:
   - `docs\troubleshooting\filesystem\windows-long-paths.md`
7. authentication, helper selection, and first clone remain Git-domain concerns
8. the preferred sign-in path remains GitHub device flow through the normal host browser

## Domain map

### Provisioning

- `docs\applications\git\architectures\boxed-owned-toolchain\provisioning.md`

Owns:

- governed shared Git root
- PortableGit provisioning
- expected shared executable surfaces

### Runtime contract

- `docs\applications\git\architectures\boxed-owned-toolchain\runtime-contract.md`

Owns:

- local boxed Git projection
- bootstrap-published `git` wrapper surfaces
- bootstrap-published `git-credential-manager-boxed` helper surfaces
- Git-side `core.longpaths=true`
- the reason a boxed helper command is preferred over a whitespace-sensitive absolute helper path

### Authentication and clone

- `docs\applications\git\architectures\boxed-owned-toolchain\authentication-and-clone.md`

Owns:

- public reachability smoke tests
- private-repo access probe
- helper-selection behavior
- GitHub device flow
- first clone workflow
- pre-bootstrap one-shot clone special case

### Git-specific troubleshooting

- `docs\applications\git\architectures\boxed-owned-toolchain\troubleshooting\long-paths.md`

Owns:

- Git-specific `Filename too long` interpretation
- the Git-side long-path option
- re-reference to the Git-independent long-path SSOT

## Cross-domain references

- central shell-selection contract:
  `docs\cli\shell\general.md`
- Git-independent long-path troubleshooting:
  `docs\troubleshooting\filesystem\windows-long-paths.md`
- VS Code method entry:
  `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\general.md`
- VS Code method Git view:
  `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\git.md`
- sanitized project-adapter example:
  `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\start.md`

## Related

- `docs\applications\git\general.md`
- `docs\applications\git\architectures\host-sync\overview.md`
