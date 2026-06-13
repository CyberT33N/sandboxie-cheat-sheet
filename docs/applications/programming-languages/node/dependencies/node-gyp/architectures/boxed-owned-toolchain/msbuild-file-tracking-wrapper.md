## Scope

This document explains the **boxed-owned-toolchain wrapper solution** for Windows native-addon builds that eventually reach:

- `node-gyp`
- generated Visual Studio / MSBuild projects
- `CL.exe`
- MSBuild file-tracking surfaces such as `.tlog` and response-file state

It documents:

- the problem class
- why the problem is not primarily a C++ compiler problem
- why editing dependency source under `node_modules` is rejected
- which Sandboxie options could influence the problem
- why those options are not the preferred architecture
- why the bootstrap-owned wrapper is the preferred solution
- the complete sanitized wrapper code

## Problem statement

In this architecture, native package installation can trigger `node-gyp` even when the project itself does not call it directly.

Typical internal call surfaces include:

1. package lifecycle scripts such as:
   - `install`
   - `preinstall`
   - `postinstall`
2. implicit `gypfile`-driven native package installs
3. transitive dependency scripts that eventually run:
   - `node-gyp rebuild`
4. reinstall / rebuild flows where the package manager executes native build steps on behalf of the dependency tree

That means the architecture must not rely on hand-editing one specific dependency copy.

## File-tracking versus file-watching

The relevant problem class here is **not** application-level source-file watching such as:

- dev-server file watching
- editor file watching
- workspace watch mode

It is specifically **MSBuild file tracking** for native compilation.

That means:

- the problematic files are build-tracking artifacts such as `.tlog`
- and compiler response / command-state artifacts around `.rsp`

So when this document says "file-tracking" or "tracking-files", it means the native-build bookkeeping layer, not a normal application watch-mode feature.

## What actually failed

The validated failure was not:

- missing Python
- missing Visual Studio
- missing Windows SDK
- missing `.NET Framework`
- or invalid C++ source

The validated failure sat in the **MSBuild file-tracking layer** around the Windows build phase.

Observed symptoms included:

- `TRK0002`
- `MSB6006`
- access-denied behavior around `.rsp`
- access-denied behavior around `.tlog`

The important refinement is:

- the compile/link pipeline itself can be healthy
- but the additional tracking layer can still fail in a strict boxed environment

## Simple explanation of `TrackFileAccess`

`TrackFileAccess` is an MSBuild / Visual C++ feature for **incremental build tracking**.

Very simply:

1. the toolchain records which files the compiler reads and writes
2. it stores that tracking state in extra files
3. later builds use that information to decide what needs rebuilding

That is why this document refers to a **file-tracking** or **tracking-files** problem.

When:

- `TrackFileAccess=true` or default tracking is active

MSBuild uses additional file-tracker behavior.

When:

- `TrackFileAccess=false`

the compiler and linker still run, but the problematic tracking layer is disabled.

## Why editing `node_modules` is rejected

Patching:

- `node-gyp`
- or another downloaded dependency

inside `node_modules` is rejected for this architecture because:

1. versions change
2. internal file layout can change
3. the patch target can move between releases
4. the patch would need to be rediscovered and re-applied repeatedly
5. the project would start depending on mutable vendor internals instead of a stable outer contract

From a domain-driven and enterprise-governed perspective, that is the wrong ownership layer.

## Sandboxie options that could influence this class of problem

The upstream Sandboxie documentation does contain options that could influence this problem class.

Important examples:

### Broad compatibility / isolation-reducing options

- `NoSecurityIsolation`
- `NoSecurityFiltering`
- `DisableFileFilter`

These are large compatibility hammers and weaken security isolation.

### Narrower hole-punching / compatibility levers

- `DropChildProcessToken`
- `OpenPipePath`
- `OpenFilePath`
- `OpenIpcPath`

These are narrower, but they still loosen the isolation boundary or reopen resource access.

## Why Sandboxie options are not the preferred solution

Even when one of those settings could theoretically make the problem disappear, they are not the preferred architecture because they weaken or bypass the box boundary.

That means they trade:

- compatibility

for:

- weaker isolation
- wider file or IPC access
- less reviewable security posture

So the current preferred answer is **not**:

- "open the box more"

The preferred answer is:

- keep the box strict
- keep dependency source untouched
- wrap the command surface from outside

## Preferred architecture: bootstrap-owned wrapper

The preferred solution is a **bootstrap-owned `node-gyp` wrapper** that sits outside the delivered dependency code.

The ownership split is:

1. delivered dependency code remains unchanged
2. bootstrap publishes a controlled `node-gyp` command surface
3. the wrapper intercepts the Windows build phase
4. the wrapper disables the problematic file-tracking layer only there

This keeps the architecture:

- reviewable
- reusable
- version-agnostic at the dependency-content level
- and consistent with the boxed-owned-toolchain bootstrap model

## Current wrapper contract

The current boxed-owned-toolchain bootstrap publishes:

- `node-gyp-wrapper.cjs`
- `node-gyp.cmd`
- `node-gyp.ps1`
- `node-gyp`

and exports:

- `BOXED_NODE_GYP_JS`
- `BOXED_NODE_GYP_REAL_JS`
- `npm_config_node_gyp`

That gives both:

- direct command resolution through `PATH`
- and package-manager / script-layer awareness through `npm_config_node_gyp`

## Complete sanitized wrapper example

The following is the current complete sanitized wrapper example.

```javascript
'use strict';

const fs = require('node:fs/promises');
const path = require('node:path');
const { createRequire } = require('node:module');

const nodeGypCli = 'C:\\Program Files\\SandboxToolchains\\VSCodeBoxes\\test-mono\\execution\\toolchain\\node\\26.2.0\\node-v26.2.0-win-x64\\node_modules\\npm\\node_modules\\node-gyp\\bin\\node-gyp.js';
const requireFromNodeGyp = createRequire(nodeGypCli);
const { glob } = requireFromNodeGyp('tinyglobby');
const log = requireFromNodeGyp('../lib/log');
const originalBuild = requireFromNodeGyp('../lib/build.js');
const buildModulePath = requireFromNodeGyp.resolve('../lib/build.js');

async function boxedBuild(gyp, argv) {
  if (process.platform !== 'win32') {
    return originalBuild(gyp, argv);
  }

  let msbuildArgs = Array.from(argv);
  if (msbuildArgs.length > 0) {
    msbuildArgs = msbuildArgs.map((target) => '/t:' + target);
  }

  const configPath = path.resolve('build', 'config.gypi');
  let configRaw;
  try {
    configRaw = await fs.readFile(configPath, 'utf8');
  } catch (error) {
    if (error && error.code === 'ENOENT') {
      throw new Error('You must run node-gyp configure first!');
    }
    throw error;
  }

  const normalizedConfig = configRaw.replace(/^(?:#.*\r?\n)+/, '');
  const config = JSON.parse(normalizedConfig);
  let buildType = config.target_defaults.default_configuration || 'Release';
  if ('debug' in gyp.opts) {
    buildType = gyp.opts.debug ? 'Debug' : 'Release';
  }

  const arch = config.variables.target_arch || process.arch;
  const files = await glob('build/*.sln', { expandDirectories: false });
  if (files.length === 0 || !config.variables.msbuild_path) {
    return originalBuild(gyp, argv);
  }

  log.verbose('build type', buildType);
  log.verbose('architecture', arch);
  log.verbose('using MSBuild:', config.variables.msbuild_path);

  if (!log.logger.isVisible('verbose')) {
    msbuildArgs.push('/clp:Verbosity=minimal');
  }
  msbuildArgs.push('/nologo');
  msbuildArgs.push('/nodeReuse:false');
  msbuildArgs.push('/p:TrackFileAccess=false');

  const archLower = String(arch).toLowerCase();
  const platform = archLower === 'x64'
    ? 'x64'
    : (archLower === 'arm'
        ? 'ARM'
        : (archLower === 'arm64' ? 'ARM64' : 'Win32'));

  msbuildArgs.push('/p:Configuration=' + buildType + ';Platform=' + platform);

  const jobs = gyp.opts.jobs || process.env.JOBS;
  if (jobs) {
    const parsedJobs = parseInt(jobs, 10);
    if (!Number.isNaN(parsedJobs) && parsedJobs > 0) {
      msbuildArgs.push('/m:' + parsedJobs);
    } else if (String(jobs).toUpperCase() === 'MAX') {
      msbuildArgs.push('/m:' + require('node:os').availableParallelism());
    }
  }

  const hasSln = msbuildArgs.some((arg) => path.extname(arg) === '.sln');
  if (!hasSln) {
    msbuildArgs.unshift(gyp.opts.solution || files[0]);
  }

  const proc = gyp.spawn(config.variables.msbuild_path, msbuildArgs);
  await new Promise((resolve, reject) => proc.on('exit', (code, signal) => {
    if (code !== 0) {
      return reject(new Error(config.variables.msbuild_path + ' failed with exit code: ' + code));
    }
    if (signal) {
      return reject(new Error(config.variables.msbuild_path + ' got signal: ' + signal));
    }
    resolve();
  }));
}

require.cache[buildModulePath].exports = boxedBuild;
require(nodeGypCli);
```

## Why this wrapper is preferred

This wrapper is preferred because it:

1. leaves dependency source untouched
2. keeps the fix in the bootstrap / control-plane layer
3. only changes the Windows build-phase behavior
4. avoids sandbox isolation weakening
5. remains easy to review and regenerate

## Current validation status

The currently validated direct proof surface is:

```powershell
node-gyp rebuild --verbose
```

inside the boxed project shell through the wrapper-published `node-gyp` command surface.

That direct rebuild path is validated as green.

Full end-to-end reinstall / dependency-tree revalidation remains a separate surface and must be tracked independently.

## Related

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\current-state.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\bootstrap-integration.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`
- `upstream\sandboxie-docs\docs\Content\NoSecurityIsolation.md`
- `upstream\sandboxie-docs\docs\Content\NoSecurityFiltering.md`
- `upstream\sandboxie-docs\docs\Content\DisableFileFilter.md`
- `upstream\sandboxie-docs\docs\Content\DropChildProcessToken.md`
- `upstream\sandboxie-docs\docs\Content\OpenPipePath.md`
- `upstream\sandboxie-docs\docs\Content\OpenFilePath.md`
