# ADR-0003 — Build strategy: Artifact Mode for frontend development; corrected full-build boundary

**Status:** Accepted (revised in v1.1; supersedes the v1 formulation)
**Governing text:** Pre-Implementation Architecture Pack v1.1, Section 1
**Amendment chain:** v1 (original) → **v1.1 (revised — governs)** → v1.2 §8 (evidence requirements for Issue #8) → v1.2.1 §6 (further Issue #8 evidence requirements)

## Decision (verbatim, v1.1 — current)

> **ADR-0003 — Build strategy: Artifact Mode for frontend development; corrected full-build boundary. (revised)**
> Decision: Artifact Mode (against the pinned `esr153` revision) is the default for all chrome JS/CSS/XHTML work. A full build is required for: final application identity, branding configuration, compiled distributables, and meaningful Widevine Case B testing. A full build is **not** required merely to test `policies.json`: distribution-scoped policy (a `distribution/` directory alongside the application) can be exercised against a development or repackaged application independently. Consequences: the full-build track's charter (#14) lists only genuine full-build needs; policy testing moves earlier and cheaper (#4).

Source: [`pre-implementation-architecture-v1.1.md`](../architecture/pre-implementation-architecture-v1.1.md), "Section 1 — Architecture Decision Records", ADR-0003.

## Correction recorded in v1.1 (verbatim)

> 6. **"Baked policies" was wrongly listed as a full-build justification in v1**; distribution-scoped policy testing is build-independent (ADR-0003, #4, #14).

Source: [`pre-implementation-architecture-v1.1.md`](../architecture/pre-implementation-architecture-v1.1.md), "Changed conclusions after ESR 153 inspection".

## Evidence obligations added later (verbatim)

Addendum v1.2 §8, amending Issue #8:

> **#8 (amended) — Artifact Mode availability acceptance criteria.** Acceptance now requires: checkout at the intended ESR 153 baseline with `git rev-parse HEAD` evidence · successful artifact discovery/download · record of the artifact source revision actually selected · confirmation that source and artifact revisions are compatible · successful `./mach build` · successful `./mach run` · successful relevant browser-chrome test. Strict pinning uses the supported revision mechanism with failures recorded honestly. If the chosen revision has no usable artifact: do not pretend the build is pinned; find the nearest suitable official artifact-producing revision; record both revisions; decide (recorded) whether to move the source baseline or perform a full build; never continue with unexplained source/binary mismatch (→ R14).

Patch v1.2.1 §6, amending Issue #8:

> **#8 (amended) — Artifact Mode evidence.** Add the official inspection command `./mach artifact last`, stored with build evidence. Acceptance evidence must include: source revision · artifact revision · artifact job/platform · artifact timestamp · `mach artifact last` output · clean/dirty object-directory state. Object-directory clobbers triggered by baseline or artifact changes are recorded as events.

Sources: [`architecture-addendum-v1.2.md`](../architecture/architecture-addendum-v1.2.md) §8;
[`architecture-patch-v1.2.1.md`](../architecture/architecture-patch-v1.2.1.md) §6.

## Superseded (historical — not current architecture)

The v1 formulation below was replaced by the v1.1 text above. Its "baked policies" full-build
justification is explicitly **withdrawn**.

> **ADR-0003 — Build strategy: Desktop Artifact Mode for frontend development.**
> Context: all planned M2–M8 work is browser-chrome JS/CSS/XHTML. Decision: Artifact Mode is the default development mode; full builds are a separate parallel track required only for official branding, baked policies, and distributable binaries. Consequences: fast iteration now; Widevine Case B and packaging blocked on the full-build track by design (ADR-0009).

Source: [`pre-implementation-architecture-v1.md`](../architecture/pre-implementation-architecture-v1.md), "Section 1", ADR-0003.

## Status and gates

- Open assumption recorded in v1.1 Section 8: *Artifact Mode suffices for M2–M8 (#14 verifies)*.
- Issue #14 produces the full-build requirements and packaging-boundary document.

## Related risks and readiness

- **R5** full-build requirements · **R14** artifact source/binary mismatch.
- Related: [ADR-0009](0009-widevine-case-a-case-b-split.md) (Case B requires the full-build track).
