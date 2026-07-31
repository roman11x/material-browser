# ADR-0008 — Theming: manual DMS synchronization through a semantic token layer

**Status:** Accepted
**Governing text:** Pre-Implementation Architecture Pack v1, Section 1
**Amendment chain:** v1 (original) → v1.1 (carried over unchanged; RFC sections revised) → v1.2 (unaffected) → v1.2.1 (unaffected)

## Decision (verbatim, v1)

> **ADR-0008 — Theming: manual DMS synchronization through a semantic token layer.**
> Decision: `dank-pywalfox.json → palette reader/validator → semantic --browser-* tokens → all chrome`. No file watching, no automatic recolour, no component reads raw DMS keys, no DMS colours injected into websites. The mockup's OKLCH values are the built-in provisional dark palette and visual-regression fixture — explicitly *not* the DMS schema. Consequences: RFC (Section 5) governs mapping; fixture-dependent sections blocked on your palette files.

Source: [`pre-implementation-architecture-v1.md`](../architecture/pre-implementation-architecture-v1.md), "Section 1 — Architecture Decision Records", ADR-0008.

## Restatement in v1.1 (verbatim)

> **ADR-0008 — Theming: manual DMS synchronization through a semantic token layer.** *(unchanged from v1)*

Source: [`pre-implementation-architecture-v1.1.md`](../architecture/pre-implementation-architecture-v1.1.md), "Section 1 — Architecture Decision Records", ADR-0008.

## Governing RFC

The mapping pipeline is specified by the DMS Token Mapping RFC — draft 0.1 in v1 Section 5, revised
in v1.1 Section 5 (§10 contrast validation, §15 fixtures/regression method, §16 accessibility). The
RFC is not yet committed as a repository document; `docs/rfcs/` is reserved for it. Sections marked
⛔ BLOCKED in the RFC await the human owner's palette fixtures.

Two RFC rules that later documents restate as project-wide constraints:

> Rule: components consume tokens only; the adapter is the only writer. Token list may shrink but not grow ad hoc — additions require an RFC amendment.

Source: [`pre-implementation-architecture-v1.md`](../architecture/pre-implementation-architecture-v1.md), RFC §4.

> **Authoritative control:** the browser's own **Reduce transparency** setting → Solid chrome, always, unconditionally functional.

Source: [`pre-implementation-architecture-v1.1.md`](../architecture/pre-implementation-architecture-v1.1.md), RFC §16 (revised).

## Status and gates

- **Blocked on human-owner fixtures:** RFC §2 (source contract), §5 (mapping table), §8 accent
  ordering, §13 (light palettes), and the real-palette fixtures in §15. No DMS key names may be
  invented (v1 RFC §2).
- **Blocked on source spike:** the screenshot/visual-regression mechanism (v1.1 RFC §15). Token
  snapshots and computed-style assertions are the settled CI baseline.
- Related: the meaning-colour rule (Patch v1.2.1 §5) constrains where DMS colours may be applied at
  all — see [ADR-0011](0011-extension-actions-without-traditional-toolbar.md) and
  [ADR-0012](0012-page-actions-and-identity-without-address-bar.md).

## Related risks and readiness

- **R7** accessibility & contrast · **R13** browser-chrome glass infeasibility (Spike E establishes
  the achievable rung; the contrast model is already blur-independent).
- Readiness (Addendum v1.2): *DMS token pipeline: structure, taxonomy, derivations, fallback, LKG —
  Ready for Milestone 0* · *source contract, mapping, light handling, real fixtures — Blocked on
  user fixture*.
