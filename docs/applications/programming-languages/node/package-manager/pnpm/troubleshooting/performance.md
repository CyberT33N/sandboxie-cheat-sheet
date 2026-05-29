# Performance

## Symptom

`pnpm install` can become extremely slow in large Windows monorepos inside Sandboxie, especially when:

- many packages are unpacked
- thousands of files are linked or materialized
- native dependencies run install/build steps
- the PNPM store is kept inside the sandboxed write domain

## Why this happens

### Sandboxie write virtualization

If the high-churn package-manager surfaces stay inside the sandbox write domain, Sandboxie has to mediate a very large number of file operations.

This becomes especially expensive for:

- `.pnpm`
- `node_modules`
- temp unpack trees
- package-manager stores

### Microsoft Defender real-time scanning

On normal host NTFS paths, Microsoft Defender scans file opens synchronously. Large dependency installs therefore pay both:

- the package-manager file cost
- the antivirus scan cost

## Recommended fix

The validated architecture in this repository is:

- keep the dependency tree materialized to the host-visible workspace path
- move the PNPM content-addressable store into a dedicated shared host path
- open only that dedicated store path for `node.exe`

Recommended shared store:

```text
C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store\
```

## Required install-box config values

At minimum:

```ini
# --- Install-box repo materialization surface ---
OpenFilePath=node.exe,C:\git\test\test-mono\

# --- Dedicated shared PNPM store ---
OpenFilePath=node.exe,C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store\

# --- Nx shared native cache ---
OpenFilePath=node.exe,C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native\
```

If the install may trigger `node-gyp`, also apply the dedicated overlay here:

- `docs\applications\programming-languages\node\dependencies\node-gyp\general.md`

That overlay adds:

- shared Python binary visibility
- host-provided Microsoft Build Tools visibility
- process-specific repo `OpenFilePath` lines for `python.exe`, `MSBuild.exe`, `cl.exe`, and related helpers

## The store path must be passed explicitly

The validated install-box command always passes the host-shared store path:

```powershell
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd" install --store-dir "C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store"
```

The same rule applies to rebuilds:

```powershell
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd" rebuild --store-dir "C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store"
```

## Native addon caveat

On a fresh install, `pnpm install` may automatically trigger `node-gyp` for a dependency with `binding.gyp` / `gypfile: true`.

If the install-box shell has **not** been bootstrapped with the shared Python binary and the Visual Studio developer environment, that automatic lifecycle step can fail with messages such as:

```text
Could not find any Python installation to use
Could not find any Visual Studio installation to use
```

## Full clean reinstall workflow

The full source-of-truth reinstall flow lives here:

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\clean-reinstall.md`

That document contains the exact three-step validated flow:

1. clear the box contents
2. clear the host-visible materialized dependency tree
3. bootstrap the install-box shell and run `pnpm install`

## Interpretation

If this clean reinstall path succeeds, then:

- the host-shared PNPM store solves the major install-performance bottleneck
- the install box is correctly materializing the workspace dependency tree
- the shell bootstrap provides the required Python / Visual Studio context for auto-triggered `node-gyp` installs

If the install still fails, the next troubleshooting step is no longer the general store-path performance issue, but the native addon overlay documented here:

- `docs\applications\programming-languages\node\dependencies\node-gyp\general.md`

## Related documents

- `docs\applications\programming-languages\node\package-manager\pnpm\general.md`
- `docs\performance\filesystem\high-file-count-workloads.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\clean-reinstall.md`
