# Clipboard and Host Window Communication

## Scope

This document covers sandboxed applications that:

- copy text into the clipboard,
- paste or send that text into a host window,
- or otherwise need to communicate with windows that exist outside the sandbox.

This is a host-window communication problem, not only a clipboard problem.

## Practical outcome

If a sandboxed application needs to paste text into a host window, `OpenWinClass=*` may be required:

```ini
OpenWinClass=*
```

## Why `OpenWinClass=*` is required in this scenario

The upstream documentation explains that Sandboxie normally does **not** allow a sandboxed program to access or communicate with windows outside the sandbox.

- Upstream: [Open Win Class](https://sandboxie-plus.github.io/sandboxie-docs/Content/OpenWinClass/)

`OpenWinClass=*` changes that behavior and allows full communication with all windows in the system.

For clipboard-driven automation, this matters because the full workflow is typically:

1. the sandboxed application writes text into the clipboard,
2. the sandboxed application triggers a paste action or window-directed input,
3. the target window exists on the host and not inside the sandbox.

Clipboard access by itself is therefore not enough. The host window must also be reachable from the boxed process.

## Why `OpenClipboard` alone is not sufficient

The upstream documentation for `OpenClipboard` only says that clipboard access can be disabled with:

```ini
OpenClipboard=n
```

- Upstream: [Open Clipboard](https://sandboxie-plus.github.io/sandboxie-docs/Content/OpenClipboard/)

This means:

- if `OpenClipboard=n` is present, clipboard access is explicitly disabled,
- but if it is **not** present, clipboard access alone still does not guarantee host-window communication.

In other words:

- `OpenClipboard` is about clipboard availability,
- `OpenWinClass=*` is about host-window accessibility.

For host paste scenarios, the second one is the decisive setting.

## Architectural explanation

The compatibility boundary in this scenario is between two different execution surfaces:

### 1. Boxed application surface

The application runs under Sandboxie and owns:

- its own process context,
- its own window objects,
- and its own ability to request clipboard or UI actions.

### 2. Host desktop surface

The target window exists outside the sandbox and is therefore protected by Sandboxie window isolation.

### Consequence

Without an explicit exception, the boxed process may hold clipboard data but still fail to deliver the intended interaction to the host window.

That is why this is not just a clipboard-access question. It is a cross-boundary window-communication question.

## Security impact

`OpenWinClass=*` is a broad host-window exception.

The upstream documentation states that it:

- allows full communication with all windows outside the sandbox,
- reduces separation,
- and may affect other window-related behavior.

- Upstream: [Open Win Class](https://sandboxie-plus.github.io/sandboxie-docs/Content/OpenWinClass/)

This is a narrower loss than disabling Sandboxie security isolation entirely, but it is still a meaningful reduction of host protection.

## Related GitHub issue material

The following Sandboxie issue material supports the interpretation that `OpenWinClass=*` is a real but risky host-communication workaround:

- GitHub issue: [#826 Microsoft Teams: No tray icon with OpenWinClass=*](https://github.com/sandboxie-plus/Sandboxie/issues/826)
- GitHub issue comment: [#192 comment 726136814](https://github.com/sandboxie-plus/Sandboxie/issues/192#issuecomment-726136814)

Issue `#826` is not clipboard-specific, but it documents the same class of trade-off: applications that need host-window communication may require `OpenWinClass=*`, and that workaround can create side effects.

## Recommended interpretation

Use this document when the problem statement is:

- “The application can run in the sandbox, but it cannot correctly interact with a host window.”

Typical symptoms include:

- paste into host windows does not work,
- clipboard-driven automation stops at the sandbox boundary,
- or host-window communication works only after `OpenWinClass=*` is enabled.
