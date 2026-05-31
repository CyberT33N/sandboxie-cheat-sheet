# Git

## Scope

This folder is the Git-domain documentation area for this repository.

Git-specific concerns belong here, for example:

- authentication
- credential-helper behavior
- private-repo access checks
- boxed clone workflows
- architecture-specific Git path selection

This split is intentional:

- Git authentication and cloning are not VS Code-specific
- the same Git access model can be invoked from VS Code, a boxed terminal, or a future runner
- keeping the source of truth in the Git area avoids duplicating auth logic across multiple IDE documents

## Architecture split

### Preferred

- `docs\applications\git\boxed-owned-toolchain\overview.md`

This is the preferred architecture path.

It documents the Git access model for:

- shared `PortableGit`
- boxed authentication
- host-driven device-code sign-in
- initial boxed clone before project bootstrap

### Secondary / legacy reference

- `docs\applications\git\host-sync\general.md`

This remains only as a host-sync / legacy reference and is **not** the preferred default.

## Architectural recommendation

For the current repository direction:

1. prefer the boxed-owned-toolchain Git flow
2. use host-sync only when the IDE must stay on the host
3. do not treat host-installed Git as the normal path

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\general.md`
- `docs\applications\IDE\vscode\methods\host-sync\general.md`
