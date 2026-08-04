# Vite Test Troubleshooting In Boxed-Owned Toolchains

## Coverage Can Cause Sandbox Timeouts

Vite/Vitest test runs can become disproportionately slow inside a boxed-owned-toolchain sandbox when coverage is enabled.

Coverage may cause the test runner to:

- traverse many source and report paths
- collect V8 coverage data
- create temporary report trees
- generate JSON and HTML report files
- inspect, clean, and rewrite existing coverage output

Sandbox filesystem mediation and file-access policies apply to each of those operations. On large source trees this can produce a very high number of filesystem operations and can lead to long-running tests, shutdown delays, or timeouts.

This is a boxed-runtime concern. It is independent of a particular monorepo, Nx task graph, or application business code.

## Preferred Default

For normal boxed Vite test execution, prefer disabling coverage:

```text
--coverage.enabled=false
```

Coverage should be enabled only for a deliberate, isolated reporting run after the normal test path has been validated without coverage.

## Diagnostic Order

1. Run the same test command with coverage disabled.
2. Ensure Sandboxie tracing is disabled before assessing runtime performance.
3. If the coverage-disabled run succeeds, treat the coverage output and report traversal as the first performance investigation surface.
4. Re-enable coverage only when the required reporting result justifies the additional boxed filesystem cost.
