# GPU acceleration

## Single source of truth

This document is the architectural entry point for GPU-backed Python workloads in the `python-general` toolchain area.

It defines:

- what GPU support depends on
- why global host Python results are not sufficient as proof
- how the tested `torch` and CUDA findings should be interpreted
- where installation, uninstall, and troubleshooting procedures live

## Core architectural rule

GPU availability must be validated inside the exact runtime boundary that will execute the workload.

That means the following interpreter is the relevant test target for the `python-general` toolchain:

```text
C:\shared\sandbox-toolchains\python-general\python\3.12.9\python.exe
```

The global host command `python` is a different runtime boundary and may therefore produce different results.

## What the tested case proved

The tested case showed the following:

- the global host interpreter returned a CUDA-enabled `torch` build and `torch.cuda.is_available() == True`
- the copied `python-general` toolchain originally had a CPU-only `torch` build with `torch.version.cuda == None`
- because of that mismatch, GPU was unavailable both outside the sandbox and inside the sandbox when the toolchain interpreter was used
- after reinstalling the CUDA-enabled `torch` build into the exact toolchain-specific Python environment, the same toolchain interpreter returned `True`

## Main conclusion

GPU support in this repository context is not determined by Sandboxie alone.

The decisive factors are:

- the exact Python version inside the toolchain
- the exact `torch` build installed for that Python version
- the exact user-base and cache paths used during installation
- the exact reinstall flow for large CUDA wheels

## Practical policy

### Valid reasoning order

1. Test the exact toolchain interpreter on the host.
2. Confirm that the toolchain has a CUDA-enabled `torch` build.
3. Only after that, use the same interpreter inside the sandbox.

### Invalid reasoning order

Do not assume that GPU should work in the toolchain just because the global host `python` command reports `True`.

## GPU area split

This area is split into four focused documents:

- install: `docs/applications/programming-languages/python/gpu/install.md`
- uninstall: `docs/applications/programming-languages/python/gpu/uninstall.md`
- troubleshooting: `docs/applications/programming-languages/python/gpu/troubleshooting.md`

## Alternative when different hardware isolation is required

If stronger hardware-oriented isolation is required and GPU execution must remain available, a virtualization platform with explicit GPU and hardware support may be a better fit, for example VMware.

That is an architectural alternative, not the primary baseline for the `python-general` Sandboxie workflow.

## Related documents

- docs/applications/programming-languages/python/general.md
- docs/applications/programming-languages/python/dependencies.md
- docs/applications/programming-languages/python/gpu/install.md
- docs/applications/programming-languages/python/gpu/uninstall.md
- docs/applications/programming-languages/python/gpu/troubleshooting.md
