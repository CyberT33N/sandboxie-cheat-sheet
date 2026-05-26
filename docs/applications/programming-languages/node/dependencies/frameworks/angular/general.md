# Angular

## Architectural status

This short rule block reflects the older host-installed / host-mirror style where Angular ran against a host-visible dependency tree directly.

It may still be valid as a small rule fragment, but it is **not** the primary architecture anymore.

Recommended architecture:

- `docs\applications\IDE\vscode\methods\host-not-isolated\dependencies-installed-in-box.md`
- `docs\applications\IDE\vscode\methods\host-not-isolated\templates\node-monorepo-materialized-dependencies.md`
- `docs\applications\programming-languages\node\dependencies\esbuild\general.md`

## Minimal Angular-specific overlay

```ini
# =========================
# Angular build tool (esbuild)
# =========================

NormalFilePath=esbuild.exe,C:\git\test\test-project\
ReadFilePath=node.exe,C:\Users\yourusername\.config\angular\
```