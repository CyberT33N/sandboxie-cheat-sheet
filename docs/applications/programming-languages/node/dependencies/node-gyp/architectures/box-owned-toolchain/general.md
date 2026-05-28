# Box-Owned-Toolchain

## Architectural status

This document captures the explored but **not validated** `node-gyp` architecture track where Microsoft Visual Studio Build Tools should be installed from inside a sandbox instead of being consumed from the host through the preferred host-sync model.

This track is currently:

- exploratory
- not validated
- not recommended as the primary method for this repository

The preferred method remains:

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\general.md`

## Definition

In this architecture, the install box (or a duplicated test box derived from the install-box baseline) would own the Microsoft C++ build toolchain instead of consuming the host-installed Build Tools.

The intended model was:

- host-side download of the official Visual Studio Build Tools bootstrapper
- host-side creation of an offline layout under the shared toolchain area
- execution of the installer from inside the sandbox
- installation either:
  - into a host-visible shared path, or
  - into a box-local path used only inside the same sandbox

## Why this architecture was explored

The motivation was straightforward:

- the repository already centralizes selected toolchain binaries under shared roots
- `node-gyp` needs more than `node.exe` and `pnpm.cmd`; it also needs:
  - Python
  - MSBuild
  - MSVC compiler / linker tools
  - Windows SDK
- if those Microsoft build tools could be made sandbox-owned, then the install box would depend less on the host system for native addon builds

## Official Microsoft distribution model used for the test

Official Microsoft bootstrapper:

- [Visual Studio Build Tools 2022 bootstrapper](https://aka.ms/vs/17/release/vs_buildtools.exe)

Official Microsoft documentation used during validation:

- [Use command-line parameters to install Visual Studio](https://learn.microsoft.com/en-us/visualstudio/install/use-command-line-parameters-to-install-visual-studio?view=vs-2022)
- [Create an offline installation package of Visual Studio](https://learn.microsoft.com/en-us/visualstudio/install/create-an-offline-installation-of-visual-studio?view=vs-2022)
- [Visual Studio Build Tools workload and component IDs](https://learn.microsoft.com/en-us/visualstudio/install/workload-component-id-vs-build-tools?view=visualstudio)

Important architectural note:

- Microsoft documents an offline **layout** and an **installation** from that layout
- Microsoft does **not** document a small portable build-tools runtime comparable to the shared Python binary
- that means this track was always an installer-driven architecture, not a true portable-binary architecture

## Test scope

The explored test path was:

1. create a duplicate sandbox as a dedicated playground instead of changing the validated install box directly
2. keep the current hardened install-box baseline as the starting point
3. add only the minimal extra rules needed for the Visual Studio installer chain
4. test both:
   - installation to a host-visible shared path
   - installation to a box-local path

## Host-side preparation that succeeded

### Step 1 - create the offline layout in the shared toolchain area

The following command path was used successfully on the host side:

```powershell
$bootstrapper = "C:\Users\yourusername\Downloads\vs_buildtools.exe"
$layout = "C:\shared\sandbox-toolchains\dev\vs-buildtools-layout"

& $bootstrapper `
  --layout $layout `
  --lang en-US `
  --add Microsoft.VisualStudio.Workload.VCTools `
  --includeRecommended
```

This step succeeded and produced the expected layout area under:

```text
C:\shared\sandbox-toolchains\dev\vs-buildtools-layout\
```

## Sandbox config additions that were tested

Only the relevant additions are listed here.

The duplicate test box received these extra rules on top of the hardened install-box baseline:

```ini
# --- Shared dev toolchain root ---
NormalFilePath=node.exe,C:\shared\sandbox-toolchains\dev\
NormalFilePath=powershell.exe,C:\shared\sandbox-toolchains\dev\
NormalFilePath=cmd.exe,C:\shared\sandbox-toolchains\dev\
NormalFilePath=python.exe,C:\shared\sandbox-toolchains\dev\

# --- Visual Studio Build Tools bootstrapper from shared layout ---
NormalFilePath=vs_buildtools.exe,C:\shared\sandbox-toolchains\dev\vs-buildtools-layout\

# --- Visual Studio installer child process from layout / installer chain ---
NormalFilePath=setup.exe,C:\shared\sandbox-toolchains\dev\vs-buildtools-layout\

# --- Host-visible install target for the test ---
OpenFilePath=vs_buildtools.exe,C:\shared\sandbox-toolchains\dev\
OpenFilePath=setup.exe,C:\shared\sandbox-toolchains\dev\

# --- Second-stage bootstrapper extracted to temp ---
NormalFilePath=vs_setup_bootstrapper.exe,C:\Users\yourusername\AppData\Local\Temp\

# --- Second-stage bootstrapper still needs to read packages from the layout ---
NormalFilePath=vs_setup_bootstrapper.exe,C:\shared\sandbox-toolchains\dev\vs-buildtools-layout\

# --- Install target under shared dev root ---
OpenFilePath=vs_setup_bootstrapper.exe,C:\shared\sandbox-toolchains\dev\
```

## Tested installation commands

### Attempt A - install into a host-visible shared path

```powershell
$layout = "C:\shared\sandbox-toolchains\dev\vs-buildtools-layout"
$installPath = "C:\shared\sandbox-toolchains\dev\vs-buildtools-2022"

& "$layout\vs_buildtools.exe" `
  --noWeb `
  --quiet `
  --wait `
  --norestart `
  --installPath $installPath `
  --add Microsoft.VisualStudio.Workload.VCTools `
  --includeRecommended

Test-Path "C:\shared\sandbox-toolchains\dev\vs-buildtools-2022"
Test-Path "C:\shared\sandbox-toolchains\dev\vs-buildtools-2022\MSBuild\Current\Bin\MSBuild.exe"
Test-Path "C:\shared\sandbox-toolchains\dev\vs-buildtools-2022\Common7\Tools\VsDevCmd.bat"
```

Observed result:

- the command returned without a visible shell error
- the target path remained missing
- all three verification checks returned `False`

### Attempt B - install into a box-local path

```powershell
$layout = "C:\shared\sandbox-toolchains\dev\vs-buildtools-layout"
$installPath = "C:\BuildToolsTest"

& "$layout\vs_buildtools.exe" `
  --noWeb `
  --quiet `
  --wait `
  --norestart `
  --installPath $installPath `
  --add Microsoft.VisualStudio.Workload.VCTools `
  --includeRecommended

Test-Path "C:\BuildToolsTest"
Test-Path "C:\BuildToolsTest\MSBuild\Current\Bin\MSBuild.exe"
Test-Path "C:\BuildToolsTest\Common7\Tools\VsDevCmd.bat"
```

Observed result:

- the command again returned without a visible shell error
- the target path remained missing
- all verification checks returned `False`

### Attempt C - interactive mode

The interactive installer window appeared briefly and then closed again without a useful host-visible installation result.

This strongly suggests that the silent-mode behavior was not the root cause. The failure was structural and also appeared in the interactive path.

## Trace and log findings

### Bootstrapper logs

The Microsoft logs under `%TEMP%` showed that the first-stage bootstrapper and the extracted second-stage bootstrapper were launched successfully and exited with success for the **layout creation** path.

However, the later install attempts still did not materialize the requested target path.

### Resource trace conclusions

The Sandboxie resource trace showed that the installer chain did **not** stay confined to the original shared-layout executable path. Instead, execution continued from sandbox-owned copies and extracted temp paths.

Examples that appeared in the trace:

- sandbox-owned copy of the layout payload under:
  - `\Sandbox\...\drive\C\shared\sandbox-toolchains\dev\vs-buildtools-layout\...`
- sandbox-owned temp extraction of the second-stage bootstrapper under:
  - `\Sandbox\...\user\current\AppData\Local\Temp\...\vs_bootstrapper_d15\vs_setup_bootstrapper.exe`

The trace also recorded denied registry access for the relevant installer executables, for example:

- `Key X ...\Applications\vs_buildtools.exe`
- `Key X ...\Applications\vs_setup_bootstrapper.exe`

Finally, the trace did **not** show hits for either requested installation target:

- `vs-buildtools-2022`
- `BuildToolsTest`

That means the installer chain did not even progress to a meaningful write attempt into those target roots.

## Why the current hardened box is the problem

The critical architectural point is not merely "one more missing EXE allow rule".

The bigger issue is that the Visual Studio installer chain:

- extracts and relaunches helper executables from temp
- relies on installer-managed and registry-backed state
- behaves like a heavy Windows installer stack, not like a simple copied binary

That interacts badly with the current hardened sandbox baseline:

- `UseSecurityMode=y`
- `UsePrivacyMode=y`
- `ProtectHostImages=y`
- `HideNonSystemProcesses=y`
- highly targeted process-specific `NormalFilePath` / `OpenFilePath` rules

There is also an official Sandboxie limitation that matters here:

- `OpenFilePath` does not apply when the program executable itself resides within the sandbox

Once the Visual Studio installer chain transitions into sandbox-owned extracted executables, the current file-access model stops behaving like the normal host-sync model.

## Conclusion

This architecture track is currently **not validated**.

More specifically:

- the offline layout creation on the host side worked
- the attempt to install Microsoft Build Tools from inside the hardened sandbox did **not** produce a usable installation
- neither the host-visible shared target path nor the box-local test path materialized into a usable Build Tools tree

## Theoretical possibility

It is still plausible that a **different** sandbox architecture could make this work.

Likely requirements would include a dedicated, less hardened installer/toolchain box with broader permissions for:

- temp-extracted installer executables
- installer-related registry state
- installer-managed package / cache paths
- the final installation target

However:

- that broader box was **not** validated
- the exact minimum permission set is **not yet known**
- the current repository baseline should therefore **not** describe this path as working

## Preferred method

The preferred and validated method remains the host-sync architecture:

- host-installed Microsoft Visual Studio Build Tools
- host-provided Windows SDK
- central shared Python build helper binary
- install-box consumption of those host-provided tools through explicit visibility rules

Why this remains preferred:

- it is already validated end-to-end for direct `node-gyp` configure / rebuild flows
- it does not require weakening the same sandbox that executes external dependency build scripts
- it preserves the hardened install-box posture better than trying to turn that box into a full Microsoft installer environment

See:

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\general.md`

## Security interpretation

From a security perspective, weakening the install box so that it can host and complete a full Microsoft Build Tools installation would reduce the value of the sandbox boundary exactly where external dependency build scripts run.

That is why the host-sync method is not just the currently validated method, but also the currently preferred method.

## Related documents

- `docs\applications\programming-languages\node\dependencies\node-gyp\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\visual-studio-build-tools.md`
