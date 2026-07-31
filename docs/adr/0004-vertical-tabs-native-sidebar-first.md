# ADR-0004 — Vertical tabs: native sidebar implementation first

**Status:** Accepted (decision unchanged since v1; evidence rebased in v1.1)
**Governing text:** Pre-Implementation Architecture Pack v1, Section 1 (decision) + v1.1, Section 1 (evidence)
**Amendment chain:** v1 (original decision) → v1.1 (evidence rebased to `esr153`) → v1.2 (unaffected) → v1.2.1 (unaffected)

## Decision (verbatim, v1)

> **ADR-0004 — Vertical tabs: native sidebar implementation first.**
> Context: Firefox's sidebar component on `esr140` contains a maintained vertical-tabs implementation (verified: `browser/components/sidebar/` with `SidebarController.toggleVerticalTabs`, a `#vertical-tabs` element, and `sidebarVerticalTabsEnabled` state). Decision: spike the native implementation first; adopt it if the approved design (right side, ~64px compact / ~248px expanded, hover + keyboard expansion, frosted overlay, workspace area, accent bar, approved geometry, bottom cluster, reliable a11y) is reachable with a maintainable patch. A custom rail requires a **documented technical limitation**; "easier to rewrite" is explicitly insufficient. Consequences: we inherit upstream tab semantics, a11y, DnD, overflow, session handling, and bug fixes; we accept some styling constraint risk, resolved by the spike.

Source: [`pre-implementation-architecture-v1.md`](../architecture/pre-implementation-architecture-v1.md), "Section 1 — Architecture Decision Records", ADR-0004.

## Evidence as rebased in v1.1 (verbatim — current)

> **ADR-0004 — Vertical tabs: native sidebar implementation first.** *(decision unchanged; evidence rebased)*
> Verified on `esr153`: `browser/components/sidebar/` with `browser-sidebar.js`, `sidebar-main.mjs` (18 vertical-tabs references), `SidebarManager.sys.mjs`, `SidebarTreeView.sys.mjs`. Spike A (Section 3) is the gate; custom rail only with a documented technical limitation; "easier to rewrite" remains insufficient.

Source: [`pre-implementation-architecture-v1.1.md`](../architecture/pre-implementation-architecture-v1.1.md), "Section 1 — Architecture Decision Records", ADR-0004.

The `esr140` paths quoted in the v1 context above are **superseded evidence**: the current baseline
is `esr153` ([ADR-0002](0002-release-baseline-esr153.md)) and the entry points verified there are
the ones listed in v1.1 Section 3, Spike A.

## Status and gates

- **Gate: Spike A** (v1.1 Section 3, as rebased on `esr153`; executed by Issue #10). Until Spike A
  reports, the rail *mechanism* — native sidebar versus custom rail — is not settled; the rail
  *design* is settled.
- Custom-rail fallback requires a documented technical limitation with evidence, escalated to the
  human owner before the fallback is chosen (v1 Spike A, "Custom-rail fallback triggers").
- Open decision recorded in v1.1 Section 8: *rail A/B (Spike A)*.

## Related risks and readiness

- **R2** native-sidebar churn.
- Readiness (Addendum v1.2): *Vertical tab rail — Ready for source spike (Spike A). Design fixed;
  mechanism (native vs custom) blocked on the spike.*
