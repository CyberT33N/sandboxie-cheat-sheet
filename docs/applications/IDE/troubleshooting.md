## Ram Disk Preferred

The preferred setup is to keep `UseRamDisk=y` enabled. A RAM disk keeps sandbox storage in system memory, which makes sandboxed file operations very fast and keeps the box volatile. This is usually the best option for short-lived development sessions, testing, and workflows where fast cleanup is important.

## Typical Error

One common failure pattern for large Node.js or monorepo workflows is:

```text
ERR_PNPM_ENOSPC ENOSPC: no space left on device
```

In this situation, the problem is usually not `pnpm` itself. The sandbox storage is full because `UseRamDisk=y` redirects boxed file writes into the RAM-backed sandbox storage, and a large install such as `pnpm install` can create thousands of files and temporary copies.

## Option A

If the machine has enough free system memory, increase the RAM disk capacity and keep `UseRamDisk=y` enabled. This is the preferred solution because it preserves the performance and volatility benefits of RAM-backed sandbox storage.

## Option B

If the workload grows into a multi-gigabyte install, build, or cache footprint, and the host machine cannot safely reserve that amount of RAM, disable `UseRamDisk` for that sandbox. At that point, a normal file-based sandbox root is the safer and more stable configuration.

## Rule Of Thumb

Keep the RAM disk enabled as long as the host has enough memory for the sandbox workload without harming system stability. Once the sandbox needs several gigabytes of storage and that RAM reservation becomes unreasonable for the machine, `UseRamDisk` should be turned off for that box.
