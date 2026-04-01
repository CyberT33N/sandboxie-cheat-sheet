# Global Hotkeys Without Focus

## Scope

This document covers sandboxed applications that need to react to key presses even when their own window is **not** focused and the key press occurs on the host desktop.

Typical examples:

- global hotkeys,
- background tray tools,
- accessibility tools,
- push-to-talk or voice-trigger tooling,
- and applications that must observe or receive host keyboard events while running boxed.

## Practical outcome

For this scenario, `NoSecurityIsolation=y` must be enabled.

```ini
NoSecurityIsolation=y
```

In the tested scenario, `OpenWinClass=*` alone was **not** sufficient.

## Why `OpenWinClass=*` alone is not enough

The upstream documentation for `OpenWinClass` explains host-window communication, but it does **not** state that the setting fully re-enables global host hotkey behavior.

- Upstream: [Open Win Class](https://sandboxie-plus.github.io/sandboxie-docs/Content/OpenWinClass/)

This distinction matters:

- `OpenWinClass=*` opens communication with host windows,
- but global hotkeys without focus are a deeper compatibility problem tied to Sandboxie security restrictions.

## What the upstream and GitHub evidence says

The strongest evidence comes from the Sandboxie GitHub issue tracker.

### Core issue

- GitHub issue: [#192 All shortcuts/hotkeys conflicts/occupied when a program is running inside sandboxie?](https://github.com/sandboxie-plus/Sandboxie/issues/192)

### Maintainer statements

- GitHub issue comment: [#192 comment 726091032](https://github.com/sandboxie-plus/Sandboxie/issues/192#issuecomment-726091032)
  - states that this is a **security restriction of Sandboxie**.

- GitHub issue comment: [#192 comment 726133948](https://github.com/sandboxie-plus/Sandboxie/issues/192#issuecomment-726133948)
  - states that, at first glance, allowing hotkeys appears to require disabling one of Sandboxie’s **core security guarantees**.
  - also explains that a clean secure solution would require a helper service to register hotkeys on behalf of boxed applications and relay the events back.

- GitHub issue comment: [#192 comment 726136814](https://github.com/sandboxie-plus/Sandboxie/issues/192#issuecomment-726136814)
  - gives `OpenWinClass=*` only as a risky workaround and explicitly says it is **not recommended**.

### Related issue showing side effects of `OpenWinClass=*`

- GitHub issue: [#826 Microsoft Teams: No tray icon with OpenWinClass=*](https://github.com/sandboxie-plus/Sandboxie/issues/826)
- GitHub issue comment: [#826 comment 1327740710](https://github.com/sandboxie-plus/Sandboxie/issues/826#issuecomment-1327740710)

This issue is relevant because it shows that `OpenWinClass=*` is not a clean universal answer. It may help with host-window communication but still creates tray/UI side effects and does not automatically solve global shortcut behavior in a robust way.

## Architectural explanation

### The core conflict

Global hotkeys without focus require the boxed application to participate in host-desktop input behavior while it is not the active foreground window.

Sandboxie’s security model is designed to stop sandboxed applications from freely integrating into host-side desktop communication primitives.

That is why this problem is architectural and not just a missing allow-rule.

### Why this is deeper than clipboard or window access

Clipboard and host-window paste mainly require:

- clipboard availability,
- host-window accessibility,
- and outbound interaction toward a target host window.

Global hotkeys without focus are different:

- the host desktop is the place where the key event is generated,
- the boxed process must still receive or effectively participate in that event flow,
- and that crosses a core Sandboxie security boundary.

### Why `NoSecurityIsolation=y` changes the result

The upstream documentation for `NoSecurityIsolation` explains that Application Compartment mode disables Sandboxie’s normal token-based security isolation and prioritizes compatibility.

- Upstream: [No Security Isolation](https://sandboxie-plus.github.io/sandboxie-docs/Content/NoSecurityIsolation/)
- Upstream: [Compartment Mode](https://sandboxie-plus.github.io/sandboxie-docs/PlusContent/compartment-mode/)

It specifically relaxes:

- token filtering,
- privilege restrictions,
- job-object restrictions,
- and parts of security-oriented path behavior.

That broader compatibility relaxation is consistent with the observed result: host-global input behavior starts working only after the core isolation barrier is reduced.

## Security impact

`NoSecurityIsolation=y` is a much larger security concession than `OpenWinClass=*`.

According to the upstream documentation, Application Compartment mode keeps virtualization but drops the stronger isolation guarantees that normally make Sandboxie a security boundary.

This means:

- compatibility improves,
- but security isolation is substantially reduced.

For trusted local productivity tools this may be acceptable.
For untrusted applications this is the wrong trade-off.

## Recommended interpretation

Use this document when the problem statement is:

- “The application must react to a key pressed on the host system even when the application window is not focused.”

For that problem class, the tested and documented conclusion is:

- `OpenWinClass=*` alone is not enough,
- `NoSecurityIsolation=y` must be enabled.

## Minimal paired conclusion for the two scenarios

### Clipboard / host-window communication

Use:

```ini
OpenWinClass=*
```

### Global host hotkeys without focus

Use:

```ini
NoSecurityIsolation=y
```

In real-world cases where a tool needs **both** capabilities, both settings may be required together.
