# Sandboxie Observations And Signals

## Scope

This document captures the key Sandboxie- and VS Code-specific observations that shaped the final boxed-owned-toolchain method.

## `PrintCompositorLPAC`

### Symptom

CLI-driven extension commands or internal VS Code server flows failed with messages such as:

```text
bad option: --disable-features=PrintCompositorLPAC
```

### Root cause

This was traced to:

- `SpecialImage=chrome,Code.exe`
- and related Chromium / Chrome compatibility handling

### Important negative finding

`argv.json` was checked and was **not** the source of the injected flag.

## Project box failure mode

When `SpecialImage=chrome,Code.exe` was active in the project box:

- the GUI could appear more cooperative
- but JSON/ESLint/language-server processes failed

The critical architectural insight was that `Code.exe` is shared by both:

- the GUI plane
- internal server processes

This makes Chromium special-image classification unsafe for the project-box authoring runtime.

## Maintenance box failure mode

When the same Chromium-oriented rules remained in the Maintenance Box, `code.cmd`-based extension operations failed for the same reason.

That is why the Maintenance Box also had to remove those rules.

## `SBIE2190`

After removing `SpecialImage`, `SBIE2190` can appear and should be interpreted as a secondary signal rather than the primary blocker.

The current method prioritizes:

- functioning GUI
- functioning internal language/server processes

over trying to preserve a Chromium-special-image classification for `Code.exe`.

## `SBIE2205` / `ConsoleInit`

`SBIE2205 Service not implemented: ConsoleInit` should currently be treated as a lower-priority warning unless it is shown to cause a concrete function break.

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\sandboxie\maintenance-box.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\sandboxie\project-box.md`
