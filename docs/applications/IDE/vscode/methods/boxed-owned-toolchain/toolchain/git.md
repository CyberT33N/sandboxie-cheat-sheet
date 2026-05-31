# Git

## Decision

Git is a canonical shared runtime in this method.

## Selected form

- `PortableGit`
- version `2.54.0`

## Why this form

`PortableGit` is used because:

- it avoids requiring a host Git installation
- it provides a full Git runtime for boxed workflows
- it fits the shared versioned runtime model

## Canonical path

```text
C:\shared\sandbox-toolchains\dev\git\2.54.0\
```

## Verification

The validated verification command is:

```powershell
& "C:\shared\sandbox-toolchains\dev\git\2.54.0\cmd\git.exe" --version
```

Expected result:

```text
git version 2.54.0.windows.1
```

## Related

- `docs\applications\git\boxed-owned-toolchain\overview.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
