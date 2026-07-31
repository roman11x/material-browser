# ADR-0001 — Upstream base: raw Mozilla Firefox source

**Status:** Accepted
**Governing text:** Pre-Implementation Architecture Pack v1, Section 1
**Amendment chain:** v1 (original) → v1.1 (carried over unchanged) → v1.2 (unaffected) → v1.2.1 (unaffected)

## Decision (verbatim, v1)

> **ADR-0001 — Upstream base: raw Mozilla Firefox source.**
> Context: full ownership of the visual token layer and frontend architecture; a derivative base (e.g. Zen) would interpose a second theme/maintenance layer we don't control. Decision: base directly on `mozilla-firefox/firefox`. Consequences: we own every rebase; we owe upstream-maintainability discipline (narrow patches, ADRs, conflict log).

Source: [`pre-implementation-architecture-v1.md`](../architecture/pre-implementation-architecture-v1.md), "Section 1 — Architecture Decision Records", ADR-0001.

## Restatement in v1.1 (verbatim)

> **ADR-0001 — Upstream base: raw Mozilla Firefox source.** *(unchanged from v1)*
> Base directly on `mozilla-firefox/firefox`; full ownership of the token layer and frontend architecture; we owe upstream-maintainability discipline.

Source: [`pre-implementation-architecture-v1.1.md`](../architecture/pre-implementation-architecture-v1.1.md), "Section 1 — Architecture Decision Records", ADR-0001.

## Status and gates

No open spike gate. The exact upstream revision this decision is applied against is governed by
[ADR-0002](0002-release-baseline-esr153.md) and recorded in `docs/baselines/firefox-baselines.md`
(created by Issue #3).

## Related risks and readiness

- Risk **R3 — ESR rebase cost** (v1 Section 7; v1.1 Section 7 carries R3 forward with the
  153→next transition as the measurement point).
- Traceability: [`docs/planning/architecture-traceability.md`](../planning/architecture-traceability.md)
  maps this ADR to Milestone 0, Issues #1 and #3.
