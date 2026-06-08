# Host-Sync

## Architectural status

This document is the `node-gyp`-specific overlay for the current recommended host-sync IDE model in this repository.

Single source of truth:

- the generic Node / PNPM monorepo materialization baseline lives in
  `docs\applications\IDE\vscode\methods\host-sync\templates\node-monorepo-materialized-dependencies.md`
- the install-box / run-box architecture explanation lives in
  `docs\applications\IDE\vscode\methods\host-sync\dependencies-installed-in-box.md`
- this folder keeps only the `node-gyp`-specific host-sync deltas on top of that baseline

## Assumptions

- VS Code or Cursor stays on the host
- dependencies are installed in a dedicated install box
- `.pnpm` / `node_modules` are materialized onto the host-visible workspace path
- the repo root is already opened for `node.exe` in the install box according to the validated PNPM monorepo baseline
- the install box consumes a central shared Python build helper binary from
  `C:\shared\sandbox-toolchains\dev\python\...`
- Microsoft Visual Studio Build Tools and Windows SDK remain host-installed and are exposed into the install box through explicit Sandboxie rules

## Why this overlay exists

For native Node addons on Windows, `node-gyp` needs more than the generic `pnpm` / install-box baseline:

- Python must be discoverable and executable inside the install box
- `python.exe` must be able to read repo-local GYP inputs and write generated build files
- `MSBuild.exe`, `cl.exe`, `link.exe`, and related build helpers must be usable from the host system inside the install box
- the Visual Studio developer environment must be initialized in the shell before the build runs

During validation, the following failure classes appeared until the host-sync overlay was added:

- `Could not find any Python installation to use`
- `Could not find any Visual Studio installation to use`
- missing repo-local `build\binding.sln` / `.vcxproj` outputs because the spawned build processes did not yet have the required repo write surface

## Current recommended posture

- keep the generic Node monorepo install-box / run-box baseline unchanged
- add only the `node-gyp`-specific host-sync additions documented in this folder
- keep Python centralized under the shared `dev` toolchain root
- keep Microsoft Visual Studio Build Tools host-provided in the current validated baseline
- do not use `nvm use` inside the boxes; call the fixed versioned `node.exe` directly

This is the **preferred** `node-gyp` architecture track in the current repository state.

The explored alternative where Microsoft Build Tools should be installed from inside the sandbox is documented here:

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\overview.md`

That alternative is currently not validated and should not replace the host-sync method.

## Governance

From a pure security perspective, there is currently **no documented security advantage** in taking the host-provided Microsoft Visual Studio Build Tools and reinstalling them into a shared toolchain directory such as `C:\shared\sandbox-toolchains\...`.

The security-relevant property of the current documented host-sync baseline is:

- the Microsoft Build Tools remain host-provided
- the install box consumes them through explicit Sandboxie visibility rules
- the same hardened install-box architecture can stay focused on controlled build execution instead of becoming a Microsoft installer box

If the Build Tools were instead installed host-side into a shared toolchain path, that would currently change the architecture mainly in these ways:

- clearer version pinning / side-by-side version management
- clearer organizational ownership of the toolchain location
- potentially cleaner operational documentation

It would **not**, by itself, produce a meaningful security improvement over the current documented host-sync baseline.

Therefore:

- if the question is primarily about security, the current host-provided host-sync baseline is already sufficient from the documented `node-gyp` perspective
- if a future change moves the Build Tools into a shared host-side install path, that should be justified by versioning or organizational reasons, not by an assumed security benefit

## Documentation map

- architecture overview:
  `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\general.md`
- Python build helper provisioning:
  `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\python-binary.md`
- host-provided Microsoft build tools:
  `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\visual-studio-build-tools.md`
- install-box config additions:
  `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\install-box-config.md`
- full clean reinstall flow:
  `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\clean-reinstall.md`
- command snippets and validation flow:
  `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\commands.md`

## Related documents

- `docs\applications\programming-languages\node\dependencies\node-gyp\general.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\host-sync\overview.md`
- `docs\applications\programming-languages\node\nvm\general.md`
- `docs\applications\programming-languages\python\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\clean-reinstall.md`
