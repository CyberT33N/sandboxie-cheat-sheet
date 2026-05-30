# tsx

## Architectural status

The older troubleshooting conclusion in this document came from an earlier host-mirror / box-only phase and is **not** a safe architectural rule for the current install-box materialization model.

What remains valid is the small `esbuild.exe` visibility overlay for projects that still exercise `esbuild` through `tsx`-adjacent toolchains.

Recommended architecture:

- `docs\applications\IDE\vscode\methods\host-sync\dependencies-installed-in-box.md`
- `docs\applications\IDE\vscode\methods\host-sync\templates\node-monorepo-materialized-dependencies.md`
- `docs\applications\programming-languages\node\dependencies\esbuild\general.md`

## Minimal overlay

```ini
# =========================
# tsx-adjacent esbuild surface
# =========================
NormalFilePath=esbuild.exe,C:\git\test\test-project\
```

## Legacy troubleshooting note

The statement "tsx watch does not work in a sandbox" should now be treated as a **legacy observation from a different architecture**, not as the current repository-wide recommendation.
