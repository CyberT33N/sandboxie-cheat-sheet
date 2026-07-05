# Electron-Vite Command Surfaces In The Boxed-Owned-Toolchain

## Scope

This document is the `electron-vite`-specific boxed-owned-toolchain reference for one **example implementation** of a deeper process-spawning problem.

It is intentionally **not** the primary place for the generic architecture explanation.

The primary generic reference lives here:

- `docs\troubleshooting\sandboxie\process-spawning\nested-child-process-orchestration.md`

This document only owns:

1. the `electron-vite`-specific example chain
2. the `electron-vite`-specific bootstrap/shim/rewrite implementation
3. the boundary between normal framework usage and tooling-owned command-surface interception

## Important boundary

### Normal `electron-vite` usage

The normal and preferred `electron-vite` posture is:

- use the project-owned package scripts
- use the framework-native dev command surfaces
- let the package manager and framework expose the intended entrypoints
- do **not** manually dig into `node_modules` just to launch `electron-vite`

In other words:

- normal projects should **not** treat `node_modules\.bin\electron-vite.CMD`
- or `node_modules\electron-vite\bin\electron-vite.js`

as the primary user-facing launch contract.

### Tooling-owned usage

There are still valid architectures where a project-owned tooling/orchestration layer must drive the framework indirectly.

Typical examples:

1. monorepos with a project-owned runtime runner
2. profile-based launchers
3. enterprise bootstrap/control-plane layers
4. canonical serve commands that must wait for other services before the desktop runtime starts

In those architectures it can be valid for the tooling layer to reach a local dependency command surface such as:

- `electron-vite.CMD`
- or directly `electron-vite.js`

That is the exact example documented here.

## What this example proved

The validated repository example proved all of the following:

1. `electron-vite.CMD` can work in isolation in the current Sandboxie architecture
2. direct `node.exe -> electron-vite.js` can work in isolation
3. the inner tooling runner can work in isolation
4. the problem was therefore **not** "Electron-Vite is generally broken in the box"
5. the real problem was the **final launch edge inside the canonical orchestration path**

So this document must not be misread as:

- "Electron-Vite requires direct Node launch by default"

The correct statement is:

- a specific tooling-owned orchestration chain required a control-plane-owned rewrite and stabilized spawn policy

## Example command chain

This is the validated **example only**:

```text
pnpm exec nx run desktop-app:serve --no-tui --verbose -- --profile=desktop-dev
  -> Nx task graph
    -> sibling watch/readiness tasks
      -> pnpm exec tsx tooling/run/cli.ts serve --profile=desktop-dev
        -> child_process.spawn(cmd.exe /d /s /c electron-vite.CMD dev --watch)
          -> electron-vite.CMD
            -> node.exe
              -> electron-vite.js
                -> Electron-Vite build/watch
                  -> Electron boot
```

The generic architecture explanation for why this can fail even when the inner pieces work is kept in:

- `docs\troubleshooting\sandboxie\process-spawning\nested-child-process-orchestration.md`

## What worked in isolation in this example

The validated example established these green surfaces:

1. `node .\node_modules\electron-vite\bin\electron-vite.js dev --watch`
2. `& $env:ComSpec /d /s /c '"...\electron-vite.CMD" dev --watch'`
3. `pnpm exec tsx tooling/run/cli.ts serve --profile=desktop-dev`
4. the actual Electron runtime could start when the lower command surface was reached correctly

That means:

- the framework was not generically broken
- the dependency wrapper was not generically broken
- the runtime was not generically broken

## What did not solve the example by itself

The same validated example also established that the following were not sufficient on their own:

1. blaming `electron-vite.CMD` as the only cause
2. blaming `cmd.exe` as the only cause
3. rewriting only to direct `node.exe -> electron-vite.js` without also fixing the spawn-handle strategy
4. keeping the canonical path but leaving the final launch on inherited console handles

That is why the final solution has two parts:

1. the wrapper-path rewrite
2. the explicit stabilized `stdio`/handle strategy on that same rewritten spawn

## Current boxed-owned-toolchain implementation

### 1. Electron terminal entrypoint

The current project terminal entrypoint for the `electron-vite` serve surface is:

```powershell
param(
  [string]$RepoPath,
  [switch]$EnableComSpecTraceProxy
)

$ErrorActionPreference = 'Stop'
Set-StrictMode -Version Latest

$launcher = Join-Path $PSScriptRoot 'Start-testMonoVSCode.ps1'

if (-not (Test-Path -LiteralPath $launcher)) {
  throw "Project launcher not found: $launcher"
}

& $launcher `
  -Action OpenTerminal `
  -RepoPath $RepoPath `
  -OpenTerminalIntent ElectronServe `
  -EnableComSpecTraceProxy:$EnableComSpecTraceProxy
```

This matters because the terminal must be opened in the **ElectronServe** intent so the project bootstrap can publish the Electron-specific shell surface before the canonical command is run.

### 2. Electron-Vite shim refresh

The project bootstrap still publishes a local `electron-vite.CMD` shim.

That shim is retained because:

1. the isolated/manual wrapper surface is still useful
2. it is a valid fallback/diagnostic surface
3. the productive canonical fix should not depend on a missing local dependency wrapper

Current implementation:

```powershell
function Ensure-testElectronViteCmdShim {
  param(
    [Parameter(Mandatory = $true)]
    [string]$RepoPath,

    [Parameter(Mandatory = $true)]
    [string]$FallbackNodeRoot
  )

  $electronViteCmd = Join-Path $RepoPath 'apps\desktop-app\node_modules\.bin\electron-vite.CMD'
  $electronViteJs = Join-Path $RepoPath 'apps\desktop-app\node_modules\electron-vite\bin\electron-vite.js'

  if (-not (Test-Path -LiteralPath $electronViteCmd) -or -not (Test-Path -LiteralPath $electronViteJs)) {
    return $false
  }

  $fallbackNodeExe = Join-Path $FallbackNodeRoot 'node.exe'
  $shimContent = @(
    '@echo off'
    'setlocal EnableExtensions'
    'set "_electronViteJs=%~dp0..\electron-vite\bin\electron-vite.js"'
    'set "_shimLogDir=%LOCALAPPDATA%\SandboxToolchains\Logs\test-mono"'
    'if not exist "%_shimLogDir%" mkdir "%_shimLogDir%" >nul 2>nul'
    'set "_shimLog=%_shimLogDir%\electron-vite-shim.log"'
    '>>"%_shimLog%" echo [%DATE% %TIME%] start cwd=%CD% boxed_node_root=%BOXED_NODE_ROOT% args=%*'
    'if defined BOXED_NODE_ROOT if exist "%BOXED_NODE_ROOT%\node.exe" goto :boxedNodeRoot'
    ('if exist "{0}" goto :fallbackNode' -f $fallbackNodeExe)
    'goto :pathNode'
    ':boxedNodeRoot'
    '>>"%_shimLog%" echo [%DATE% %TIME%] using BOXED_NODE_ROOT "%BOXED_NODE_ROOT%\node.exe"'
    '"%BOXED_NODE_ROOT%\node.exe" "%_electronViteJs%" %*'
    'set "_shimExit=%ERRORLEVEL%"'
    '>>"%_shimLog%" echo [%DATE% %TIME%] exit_code=%_shimExit%'
    'exit /b %_shimExit%'
    ':fallbackNode'
    ('>>"%_shimLog%" echo [%DATE% %TIME%] using fallback "{0}"' -f $fallbackNodeExe)
    ('"{0}" "%_electronViteJs%" %*' -f $fallbackNodeExe)
    'set "_shimExit=%ERRORLEVEL%"'
    '>>"%_shimLog%" echo [%DATE% %TIME%] exit_code=%_shimExit%'
    'exit /b %_shimExit%'
    ':pathNode'
    '>>"%_shimLog%" echo [%DATE% %TIME%] using PATH node'
    'node "%_electronViteJs%" %*'
    'set "_shimExit=%ERRORLEVEL%"'
    '>>"%_shimLog%" echo [%DATE% %TIME%] exit_code=%_shimExit%'
    'exit /b %_shimExit%'
  ) -join [Environment]::NewLine

  [System.IO.File]::WriteAllText($electronViteCmd, $shimContent, [System.Text.Encoding]::ASCII)
  return $true
}
```

Important architectural note:

- the shim is **not** the productive root cause fix by itself
- the productive canonical fix happens later on the final spawn boundary

### 3. Spawn tracer activation in the project terminal

The project terminal bootstrap injects the tracer and publishes the active stabilizer mode:

```powershell
[System.IO.File]::WriteAllText($LogPath, '', [System.Text.Encoding]::ASCII)

$currentNodeOptions = [string]$env:NODE_OPTIONS
$normalizedTracerPathForNode = $TracerPath -replace '\\', '/'
$requireOption = '--require="{0}"' -f $normalizedTracerPathForNode

if ($currentNodeOptions.IndexOf($TracerPath, [System.StringComparison]::OrdinalIgnoreCase) -lt 0 -and
    $currentNodeOptions.IndexOf($normalizedTracerPathForNode, [System.StringComparison]::OrdinalIgnoreCase) -lt 0) {
  $env:NODE_OPTIONS = if ([string]::IsNullOrWhiteSpace($currentNodeOptions)) {
    $requireOption
  }
  else {
    "$currentNodeOptions $requireOption"
  }
}

$env:BOXED_CHILD_PROCESS_DEBUG_ENABLED = 'true'
$env:BOXED_CHILD_PROCESS_DEBUG_LOG = $LogPath
$env:BOXED_CHILD_PROCESS_SPAWN_TRACER = $TracerPath
$env:BOXED_SPAWN_STABILIZE_ELECTRON_VITE_CMD = 'true'
$env:BOXED_SPAWN_STABILIZE_ELECTRON_VITE_CMD_MODE = 'direct-node-js-with-full-pipes'
```

The header then exposes:

- `ChildProcessSpawnTracer`
- `ChildProcessDebugLog`
- `ElectronViteCmdSpawnStabilizer`
- `ElectronViteCmdSpawnStabilizerMode`

That makes the active control-plane lane auditable before the canonical command is run.

### 4. Final child-process detection and direct Node rewrite

The generated tracer identifies the problematic Windows wrapper spawn and rewrites it to direct `node.exe -> electron-vite.js`:

```javascript
function shouldStabilizeElectronViteCmdSpawn(command, args) {
  if (process.platform !== 'win32') {
    return false;
  }

  if ((process.env.BOXED_SPAWN_STABILIZE_ELECTRON_VITE_CMD ?? '').toLowerCase() !== 'true') {
    return false;
  }

  const commandText = command === undefined || command === null ? '' : String(command);
  if (!/[\\/]cmd\.exe$/iu.test(commandText)) {
    return false;
  }

  const commandLine = toStringArray(args).join(' ');
  return /electron-vite\.cmd/iu.test(commandLine) && /\bdev\b/iu.test(commandLine) && /--watch/iu.test(commandLine);
}

function createElectronViteDirectNodeRewrite(command, args) {
  if (!shouldStabilizeElectronViteCmdSpawn(command, args)) {
    return null;
  }

  const normalizedArgs = toStringArray(args);
  const commandLine = normalizedArgs[normalizedArgs.length - 1] ?? '';
  const commandTokens = splitWindowsCommandLine(commandLine);
  if (commandTokens.length === 0) {
    return null;
  }

  const electronViteCmdPath = commandTokens[0];
  const electronViteArgs = commandTokens.slice(1);
  const boxedNodeRoot = process.env.BOXED_NODE_ROOT;
  const nodeExe = boxedNodeRoot ? path.join(boxedNodeRoot, 'node.exe') : process.execPath;
  const electronViteJs = path.resolve(path.dirname(electronViteCmdPath), '..', 'electron-vite', 'bin', 'electron-vite.js');

  if (!fs.existsSync(nodeExe) || !fs.existsSync(electronViteJs)) {
    return null;
  }

  return {
    args: [electronViteJs, ...electronViteArgs],
    command: nodeExe,
    electronViteArgs,
    electronViteCmdPath,
    electronViteJs,
    nodeExe
  };
}
```

This is the repository-specific part that turns:

```text
cmd.exe /d /s /c electron-vite.CMD dev --watch
```

into:

```text
node.exe electron-vite.js dev --watch
```

inside the control plane.

### 5. Stabilized `stdio` / handle strategy on the same rewritten spawn

The final successful lane did **not** stop at the rewrite.

The same rewritten spawn also receives an explicit `stdio`/handle strategy:

```javascript
function createStableElectronViteSpawnOptions(options) {
  const normalized = options && typeof options === 'object'
    ? { ...options }
    : {};

  const stabilizerMode = (process.env.BOXED_SPAWN_STABILIZE_ELECTRON_VITE_CMD_MODE ?? '')
    .trim()
    .toLowerCase();

  if (stabilizerMode === 'inherit-stdin-pipe-output' || stabilizerMode === 'direct-node-js-with-stable-fallback') {
    normalized.stdio = ['inherit', 'pipe', 'pipe'];
  }
  else {
    // Fully isolated pipes are the current default because the inherited
    // stdin variant still blocked on the final direct node -> electron-vite
    // spawn edge in the canonical orchestration lane.
    normalized.stdio = ['pipe', 'pipe', 'pipe'];
  }

  if (!Object.prototype.hasOwnProperty.call(normalized, 'windowsHide')) {
    normalized.windowsHide = true;
  }

  return normalized;
}

function mirrorChildPipesToParent(child) {
  if (child.stdout && typeof child.stdout.on === 'function') {
    child.stdout.on('data', (chunk) => {
      process.stdout.write(chunk);
    });
  }

  if (child.stderr && typeof child.stderr.on === 'function') {
    child.stderr.on('data', (chunk) => {
      process.stderr.write(chunk);
    });
  }
}
```

The important architectural point is:

- direct rewrite alone was not enough
- stabilized `stdio` alone was not enough in earlier lanes
- the productive fix required **both** on the same final launch edge

### 6. Spawn patch that combines rewrite and stabilization

The tracer then combines both decisions on the final `spawn(...)` call:

```javascript
const normalizedArgs = toStringArray(args);
const electronViteDirectNodeRewrite = createElectronViteDirectNodeRewrite(command, normalizedArgs);
const rewriteElectronViteCmdSpawn = electronViteDirectNodeRewrite !== null;
const stabilizeElectronViteCmdSpawn = rewriteElectronViteCmdSpawn || shouldStabilizeElectronViteCmdSpawn(command, normalizedArgs);
const effectiveCommand = rewriteElectronViteCmdSpawn
  ? electronViteDirectNodeRewrite.command
  : command;
const effectiveArgs = rewriteElectronViteCmdSpawn
  ? electronViteDirectNodeRewrite.args
  : args;
const effectiveOptions = stabilizeElectronViteCmdSpawn
  ? createStableElectronViteSpawnOptions(options)
  : options;

appendEvent({
  event: 'spawn-before',
  args: normalizedArgs,
  command: command === undefined || command === null ? null : String(command),
  rewriteElectronViteCmdSpawn,
  rewrittenArgs: rewriteElectronViteCmdSpawn ? electronViteDirectNodeRewrite.args : null,
  rewrittenCommand: rewriteElectronViteCmdSpawn ? electronViteDirectNodeRewrite.command : null,
  rewrittenElectronViteJs: rewriteElectronViteCmdSpawn ? electronViteDirectNodeRewrite.electronViteJs : null,
  stabilizedElectronViteCmdSpawn: stabilizeElectronViteCmdSpawn,
  options: normalizeOptions(effectiveOptions)
});

const child = originalSpawn.call(this, effectiveCommand, effectiveArgs, effectiveOptions);

if (rewriteElectronViteCmdSpawn) {
  appendEvent({
    event: 'spawn-electron-vite-rewritten',
    electronViteArgs: electronViteDirectNodeRewrite.electronViteArgs,
    electronViteCmdPath: electronViteDirectNodeRewrite.electronViteCmdPath,
    electronViteJs: electronViteDirectNodeRewrite.electronViteJs,
    nodeExe: electronViteDirectNodeRewrite.nodeExe
  });
}

if (stabilizeElectronViteCmdSpawn) {
  appendEvent({
    event: 'spawn-stdio-stabilized',
    originalStdio: normalizeStdio(options && typeof options === 'object' ? options.stdio : undefined),
    patchedStdio: normalizeStdio(effectiveOptions.stdio),
    stabilizerMode: process.env.BOXED_SPAWN_STABILIZE_ELECTRON_VITE_CMD_MODE ?? null
  });

  mirrorChildPipesToParent(child);
}
```

This is the example-specific code path that actually solved the productive launch.

## Why the solution belongs here

This solution belongs in the `electron-vite` boxed-owned-toolchain area because it is the concrete example of:

1. a project-owned tooling runner
2. targeting the local `electron-vite` command surface
3. needing a repository-specific control-plane adaptation

The generic explanation of the failure class does **not** belong here.

That explanation belongs here:

- `docs\troubleshooting\sandboxie\process-spawning\nested-child-process-orchestration.md`

## Why this must not be over-generalized

This document must not be read as permission to always bypass framework-owned entrypoints.

The preferred rule remains:

1. use native framework scripts first
2. use project-owned tooling wrappers only when the architecture genuinely requires them
3. only apply wrapper bypass / direct Node launch in a control-plane layer when the canonical orchestrated surface has proven unstable and the lower surfaces are already validated

## Valid scenarios for this pattern

This example-specific pattern is architecturally valid when all of the following are true:

1. the project already has a tooling-owned launch layer
2. the framework is not being launched manually by an end user
3. the repository must preserve a canonical orchestrated command surface
4. the lower-level framework command was already proven healthy in isolation
5. the failing surface is the final launch edge inside that tooling-owned orchestration chain

## What not to claim

This document does **not** claim:

1. that every Electron-Vite project should run through `node_modules\.bin\electron-vite.CMD`
2. that every Electron-Vite project should directly call `electron-vite.js`
3. that `electron-vite` itself is defective
4. that every orchestration problem of this kind must be solved by a direct Node rewrite

## Related

- `docs\troubleshooting\sandboxie\process-spawning\nested-child-process-orchestration.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\lifecycle-and-command-surface.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
