# ADR-0005 — Workspaces: substrate deliberately undecided pending source inspection

**Status:** Accepted — and deliberately undecided on substrate
**Governing text:** Pre-Implementation Architecture Pack v1, Section 1
**Amendment chain:** v1 (original) → v1.1 (unchanged; Spike C rebased to `esr153`) → v1.2 §8 (Spike C inspection list expanded) → v1.2.1 (unaffected)

## Decision (verbatim, v1)

> **ADR-0005 — Workspaces: substrate deliberately undecided pending source inspection.**
> Context: our workspaces are independent active tab *sets*; native tab groups (verified present on `esr140`: `browser/components/tabbrowser/content/tabgroup.js`) primarily organize tabs within one visible strip. Decision: no substrate is chosen yet. An options memo (Section 4 framework) comparing custom state over `gBrowser`, show/hide layering, native tab groups as substrate, and a hybrid will be produced after the M1 source inspection. Consequences: `WorkspaceService` design waits; M1 inspection gains a defined third target (tab visibility + session-store semantics).

Source: [`pre-implementation-architecture-v1.md`](../architecture/pre-implementation-architecture-v1.md), "Section 1 — Architecture Decision Records", ADR-0005.

## Restatement in v1.1 (verbatim)

> **ADR-0005 — Workspaces: substrate deliberately undecided pending source inspection.** *(unchanged; Spike C rebased on esr153, where tab-group behaviour has evolved and must be re-inspected rather than inherited from 140-era conclusions)*

Source: [`pre-implementation-architecture-v1.1.md`](../architecture/pre-implementation-architecture-v1.1.md), "Section 1 — Architecture Decision Records", ADR-0005.

The `esr140` `tabgroup.js` citation in the v1 context is superseded evidence; the current baseline is
`esr153` ([ADR-0002](0002-release-baseline-esr153.md)) and **no 140-era conclusions may be imported**
into the Spike C reading (v1.1 Section 3, Spike C).

## Substrate options under comparison (verbatim, v1 Section 4)

> - **O1 — Custom workspace state over `gBrowser`**: our own store maps workspace → tab set; switching manipulates tab visibility/order directly.
> - **O2 — Show/hide layering over normal Firefox tabs**: workspaces are views over one real tab set using native hide/show, with our store owning membership.
> - **O3 — Native tab groups as substrate**: each workspace is (or owns) a native group; switching collapses/hides other groups; persistence rides group persistence.
> - **O4 — Hybrid**: e.g. O2 visibility mechanics + O3 group objects for membership/persistence, or O1 store with native groups as an import/export boundary.

Source: [`pre-implementation-architecture-v1.md`](../architecture/pre-implementation-architecture-v1.md), "Section 4 — Workspace Substrate Options Memo (framework only)".

## Hard veto gates (verbatim, v1 Section 4)

> **Scoring template per cell:** mechanism (file/API), confidence (verified in source / inferred / unknown), and failure mode. Hard gates that veto an option regardless of totals: any credible tab-loss path on crash mid-switch; private-window state ever persisting; incompatibility with the Spike A rail outcome.

## Status and gates

- **Gate: Spike C** (v1.1 Section 3 as expanded by Addendum v1.2 §8; executed by Issue #12), then
  the Section 4 options memo, then a **human decision** on the substrate.
- No workspace design work proceeds past the memo framework (Addendum v1.2 readiness map).
- Open decision recorded in v1.1 Section 8: *workspace substrate (Spike C + memo)*.

## Related risks and readiness

- **R8** workspace/session corruption (tab loss on crash mid-switch).
- Readiness (Addendum v1.2): *Workspaces — Blocked on source spike (Spike C feeds the substrate
  memo; no design work proceeds past the memo framework).*
