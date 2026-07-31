# ADR-0002 — Release baseline: ESR 153, with recorded revisions

**Status:** Accepted (revised in v1.1; supersedes the v1 formulation)
**Governing text:** Pre-Implementation Architecture Pack v1.1, Section 1
**Amendment chain:** v1 (original) → **v1.1 (revised — governs)** → v1.2 (unaffected) → v1.2.1 (unaffected)

## Decision (verbatim, v1.1 — current)

> **ADR-0002 — Release baseline: ESR 153, with recorded revisions. (revised)**
> Context: Firefox 153 is the new ESR generation (`esr153` verified: 153.1.0esr). ESR 140 remains supported through the transition window but is not the correct baseline for a project that has not begun implementation. Decision: implementation and release patches target `esr153`; `esr140` is retained as the Nova/Smartbar comparison point, rebase-rehearsal material, and inter-ESR-delta evidence; `main` is forward observation only. Every source spike records exact commit hashes for all inspected branches and the browser version from `browser/config/version.txt` (or equivalent authoritative source) at that revision; train numbers are never used as repository identifiers. The concrete baseline for this pack is the metadata block above. At release-branch creation time, the then-current supported ESR is re-verified — no ESR number is permanent in architecture documents. Consequences: the 140→153 delta observed this session (F1, sidebar test restructuring) doubles as free evidence of per-generation rebase cost; the first *forward* transition (153→next) inherits v1's scheduled-maintenance-exercise treatment: record conflicts, measure effort, identify high-churn patches, improve isolation before public release.

Source: [`pre-implementation-architecture-v1.1.md`](../architecture/pre-implementation-architecture-v1.1.md), "Section 1 — Architecture Decision Records", ADR-0002.

"The metadata block above" is v1.1's "Source-verification metadata" block, in the same document.
Issue #3 turns that remote-inspection record into locally verified development baselines in
`docs/baselines/firefox-baselines.md`; per Addendum v1.2 §8, remote hashes become development
baselines only after local verification, and a divergent local tip is never silently substituted.

## Branch roles (verbatim, v1.1)

> Branch roles (ADR-0002): `esr153` = implementation and release baseline · `esr140` = comparison point for the Nova/Smartbar transition, rebase-rehearsal material, and inter-ESR-delta evidence · `main` = forward observation, never a version label ("main is version X" claims are prohibited; versions come from `version.txt` at a recorded commit).

Source: [`pre-implementation-architecture-v1.1.md`](../architecture/pre-implementation-architecture-v1.1.md), "Source-verification metadata".

## Superseded (historical — not current architecture)

The v1 formulation below was replaced by the v1.1 text above. It is retained for provenance only;
its ESR 140 baseline is **not** current architecture.

> **ADR-0002 — Release baseline: current Firefox ESR, re-verified at branch time.**
> Context: ESR reduces rebase frequency to ~yearly; the currently verified ESR line is 140 (branch `esr140` confirmed to exist upstream), with mainline at 153 and a new ESR generation imminent. Decision: release patches target the current supported ESR, with the exact version re-checked immediately before the release branch is created — no ESR number is permanently encoded in architecture documents. Development may read `main` for foresight ([F1] came from exactly that), but release patches must apply and build against the selected ESR. Consequences: first ESR transition is treated as a scheduled maintenance exercise with recorded conflicts, measured effort, and isolation improvements before public release.

Source: [`pre-implementation-architecture-v1.md`](../architecture/pre-implementation-architecture-v1.md), "Section 1", ADR-0002.

## Status and gates

- No spike gate. Standing obligation: the supported ESR is re-verified at release-branch creation
  time; no ESR number is permanent.
- Open decision carried in v1.1 Section 8: *next-ESR adoption timing (per ADR-0002 at branch time)*.
- Baseline changes require a recorded baseline-update entry (Addendum v1.2 §8, Issue #3).

## Related risks and readiness

- **R1** URL-bar frontend contract migration · **R2** native-sidebar churn · **R3** ESR rebase cost ·
  **R14** artifact source/binary mismatch.
- Readiness (Addendum v1.2): *ESR baseline & rebase machinery — Ready for Milestone 0* (#3 local
  verification, #7 scaffolding).
