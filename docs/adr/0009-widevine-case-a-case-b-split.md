# ADR-0009 — Widevine: Case A / Case B split

**Status:** Accepted
**Governing text:** Pre-Implementation Architecture Pack v1, Section 1
**Amendment chain:** v1 (original) → v1.1 (carried over unchanged) → v1.2 (unaffected) → v1.2.1 (unaffected)

## Decision (verbatim, v1)

> **ADR-0009 — Widevine: Case A / Case B split.**
> Decision: Case A (development & personal-build feasibility) begins in M0–M1 across stock-control, clean-profile, Artifact-Mode, Fedora/Wayland, with Niri and Hyprland where compositor behaviour may matter. Case B (rebranded distributable feasibility) begins once a full branded build exists and must separately report CDM download, CDM initialization, EME support, legal-test-content playback, representative commercial-service playback, and distribution/licensing status. Case B alone controls public DRM claims. Prohibited: copied proprietary binaries, identity spoofing, undocumented workarounds.

Source: [`pre-implementation-architecture-v1.md`](../architecture/pre-implementation-architecture-v1.md), "Section 1 — Architecture Decision Records", ADR-0009.

## Case B report axes

Enumerated exactly as the decision states them; Case B must report each separately:

1. CDM download;
2. CDM initialization;
3. EME support;
4. legal-test-content playback;
5. representative commercial-service playback;
6. distribution/licensing status.

## Restatement in v1.1 (verbatim)

> **ADR-0009 — Widevine: Case A / Case B split.** *(unchanged from v1)*

Source: [`pre-implementation-architecture-v1.1.md`](../architecture/pre-implementation-architecture-v1.1.md), "Section 1 — Architecture Decision Records", ADR-0009.

## Evidence requirements

DRM evidence for either case is defined in
[`docs/agents/evidence-requirements.md`](../agents/evidence-requirements.md): about:support media
section · plugin/component state · EME init result · legal test page result · per-service result with
account-legality note · console/media logs. Findings are recorded in `docs/drm/` (Issue #6 creates
the living Case A/B document; see [`docs/drm/README.md`](../drm/README.md)).

## Status and gates

- Case A execution: Issue #6 (stock control) and Issue #13 (Artifact build), both requiring a legal
  streaming account supplied by the human owner.
- Case B is **not** a spike: it is blocked on the full-build track by design
  ([ADR-0003](0003-build-strategy-artifact-mode.md); charter from Issue #14).
- **No public DRM claim may be made before Case B reports** (Milestone 10 claim gate).
- Prohibited techniques are also hard constraints in [`AGENTS.md`](../../AGENTS.md) §4.

## Related risks and readiness

- **R4** Widevine / Case B unknowns for a branded distributable.
- Readiness (Addendum v1.2): *Widevine Case A — Ready for Milestone 0 (#6). Case B — blocked on the
  full-build track by design (not a spike; a build prerequisite).*
