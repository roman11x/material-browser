# ADR-0006 — Launcher: alternate presentation layer over Firefox query infrastructure, frontend-agnostic

**Status:** Accepted (revised in v1.1, amended in v1.2); provider-level binding is a hypothesis under test by Spike B
**Governing text:** Pre-Implementation Architecture Pack v1.1, Section 1, as amended by Addendum v1.2 §2
**Amendment chain:** v1 (original) → **v1.1 (revised)** → **v1.2 §2 (amended — governs)** → v1.2.1 (unaffected)

## Decision (verbatim, v1.1)

> **ADR-0006 — Launcher: alternate presentation layer over Firefox query infrastructure, frontend-agnostic. (revised)**
> Architecture: `Firefox query/provider infrastructure → BrowserQueryAdapter (ours) → centred launcher (ours)`. Upstream systems kept intact as in v1 (classification, engines/aliases, suggestions, open-tab switching, Places, extension results, ranking, navigation actions); we own presentation, grouping, selection, keyboard input, activation lifecycle, command and workspace providers, error presentation, and styling. New constraints from F1: **the launcher must not know which upstream URL-bar frontend (legacy-compatible or Nova/Smartbar) is active**; that knowledge is confined to `BrowserQueryAdapter`, which may carry ESR-specific internal implementations if unavoidable. The adapter preferentially binds at the provider level (`UrlbarProvidersManager`), the layer verified stable across esr140/esr153/main, rather than to either stock view. We do not begin by cosmetically relocating the stock address bar if that binds us to a frontend Mozilla is replacing — Spike B decides the exact binding with evidence.

Source: [`pre-implementation-architecture-v1.1.md`](../architecture/pre-implementation-architecture-v1.1.md), "Section 1 — Architecture Decision Records", ADR-0006.

## Amendment (verbatim, Addendum v1.2 §2 — current)

The claim that the provider layer is "verified stable across esr140/esr153/main" is **downgraded to a
hypothesis under test**. The adopted replacement wording, verbatim:

> The provider and ranking layer is the most promising available integration seam because its major modules remain recognizable across the inspected branches. Its API stability is not yet established and is an explicit output of Spike B.

> **ADR-0006 amendment.** Provider-level integration remains the preferred binding — now stated as *a hypothesis under test by Spike B*, not a verified stable boundary. The fallback ladder (controller-level binding → hidden stock input backend) is unchanged and becomes the live path if the hypothesis fails.

> **Spike B mandate extension.** Compare the provider-level contracts at the pinned `esr140`, `esr153`, and `main` revisions where practical, recording for each: constructor and registration model · query-context structure · provider start/cancel lifecycle · result object representation · muxer integration · listener/callback interface · activation data supplied by results · process ownership · material differences between revisions · **which subset can be wrapped behind `BrowserQueryAdapter`**. The adapter-wrappable subset is the spike's primary deliverable; presence-of-files is explicitly insufficient evidence of contract stability.

Source: [`architecture-addendum-v1.2.md`](../architecture/architecture-addendum-v1.2.md) §2.

## Fallback ladder (verbatim, v1.1 Section 3, Spike B)

> **Fallback ladder:** provider-level binding unworkable → controller-level binding with the adapter absorbing controller-contract churn → (last resort) hidden stock input as backend, documented as deliberate. Forked providers remain prohibited at every rung.

## Superseded (historical — not current architecture)

The v1 formulation below binds the adapter to the URL-bar view/controller layer and rests on v1's
`[F1]` reading, which v1.1 records as methodologically wrong (see
[`docs/architecture/README.md`](../architecture/README.md)). It is retained for provenance only.

> **ADR-0006 — Launcher: alternate presentation layer over Firefox URL-bar infrastructure.**
> Context: Firefox's query stack (providers, ranking, Places, open tabs, search engines, extension results) must not be reimplemented; [F1] makes controller/view coupling a known churn surface. Decision: architecture is `Firefox providers & ranking → our launcher adapter/controller → our centred launcher view`. Upstream systems kept intact: URL classification, search aliases/engines, suggestions, open-tab switching, Places, extension results, ranking, navigation actions. We own: presentation, group rendering, selection, keyboard input, activation lifecycle, command provider, workspace provider, error presentation, Material/glass styling. Consequences: the adapter boundary is the designated absorption point for [F1]-class upstream refactors — when the view/controller side changes at ESR-next, the adapter is rewritten, not the launcher.

Source: [`pre-implementation-architecture-v1.md`](../architecture/pre-implementation-architecture-v1.md), "Section 1", ADR-0006.

## Status and gates

- **Gate: Spike B** (v1.1 Section 3 as rewritten, plus the v1.2 §2 cross-revision contract mandate;
  executed by Issue #11). Its output is the M2 design note.
- Escalation trigger (Work Plan #11): *provider seam materially unstable across the pinned revisions
  → architecture-question on ADR-0006 before M2 issues are written.*
- Open decisions recorded in v1.1 Section 8: *adapter binding level and hide-vs-remove (Spike B)* ·
  *Nova-targeting posture within ESR 153's lifespan (Spike B Q2/Q8)*.

## Related risks and readiness

- **R1** URL-bar frontend contract migration (F1, corrected).
- Readiness (Addendum v1.2): *Launcher/query architecture — Ready for source spike (Spike B;
  provider-contract stability is the open question). Implementation blocked on the spike.*
- Related: [ADR-0007](0007-launcher-activation-contract.md) (activation contract the adapter must satisfy).
