# ADR-0010 — Desktop integration: standard Wayland + XDG portals only

**Status:** Accepted
**Governing text:** Pre-Implementation Architecture Pack v1, Section 1
**Amendment chain:** v1 (original) → v1.1 (carried over unchanged) → v1.2 (unaffected) → v1.2.1 (unaffected)

## Decision (verbatim, v1)

> **ADR-0010 — Desktop integration: standard Wayland + XDG portals only.**
> Decision: no core dependency on `hyprctl`, Niri IPC, X11 tools, GNOME Shell or KDE APIs, or compositor-specific positioning. One browser for Niri and Hyprland; compositor-specific extras, if ever, are optional and post-1.0.

Source: [`pre-implementation-architecture-v1.md`](../architecture/pre-implementation-architecture-v1.md), "Section 1 — Architecture Decision Records", ADR-0010.

## Restatement in v1.1 (verbatim)

> **ADR-0010 — Desktop integration: standard Wayland + XDG portals only.** *(unchanged from v1)*

Source: [`pre-implementation-architecture-v1.1.md`](../architecture/pre-implementation-architecture-v1.1.md), "Section 1 — Architecture Decision Records", ADR-0010.

## Status and gates

- No spike gate. Issue #5 records the stock Firefox Wayland baseline under **both** Niri and
  Hyprland (portals file dialog, screen share, notifications, clipboard, drag-and-drop, fullscreen
  video, picture-in-picture, fractional scaling) as the regression reference.
- Addendum v1.2 §5 (Spike E) states explicitly that desktop compositor rules are never part of the
  core solution, and that desktop-through transparency stays deferred post-1.0.
- Enforced as a hard constraint in [`AGENTS.md`](../../AGENTS.md) §4: no compositor-specific
  dependencies in core code.

## Related risks and readiness

- **R10** Fedora/portal changes · **R11** Niri/Hyprland divergence (mitigation: zero
  compositor-specific code paths; worst case is a per-compositor *workaround* document, never a
  per-compositor build).
- Readiness (Addendum v1.2): *Wayland/compositor integration baseline — Ready for Milestone 0 (#5).*
