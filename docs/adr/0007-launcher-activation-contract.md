# ADR-0007 — Launcher activation contract (firm)

**Status:** Accepted — firm
**Governing text:** Pre-Implementation Architecture Pack v1, Section 1
**Amendment chain:** v1 (original) → v1.1 (carried over unchanged) → v1.2 (unaffected) → v1.2.1 (unaffected)

## Decision (verbatim, v1)

> **ADR-0007 — Launcher activation contract (firm).**
> Decision, on Enter with a valid selection: (1) duplicate-activation guard engages, (2) intended action is captured, (3) launcher closes immediately, (4) action executes or is handed off, (5) focus transfers appropriately. Validate-before-close is permitted only where the action requires it (e.g. palette parsing) — validate, then close, then apply. Synchronous failure: stay open, preserve input, inline error. Post-handoff asynchronous failure: never reopen the launcher; surface via the standard browser notification mechanism. Required tests: single-Enter-single-execution; stale query callbacks cannot alter selection after activation; close cancels/detaches the live view query safely; Enter on an open-tab result focuses that tab; Shift+Enter performs the new-tab action and closes; mouse activation follows the identical close contract.

Source: [`pre-implementation-architecture-v1.md`](../architecture/pre-implementation-architecture-v1.md), "Section 1 — Architecture Decision Records", ADR-0007.

## The six required tests

Enumerated here exactly as the decision states them, for direct reuse in M2 issues:

1. single-Enter-single-execution;
2. stale query callbacks cannot alter selection after activation;
3. close cancels/detaches the live view query safely;
4. Enter on an open-tab result focuses that tab;
5. Shift+Enter performs the new-tab action and closes;
6. mouse activation follows the identical close contract.

## Restatement in v1.1 (verbatim)

> **ADR-0007 — Launcher activation contract (firm).** *(unchanged from v1: guard → capture → close → execute/hand off → focus; validate-first only where required; sync failure stays open with inline error; async post-handoff failure surfaces via browser notifications, never reopens; the six required tests stand.)*

Source: [`pre-implementation-architecture-v1.1.md`](../architecture/pre-implementation-architecture-v1.1.md), "Section 1 — Architecture Decision Records", ADR-0007.

## Status and gates

- No spike gate on the contract itself. The **hook point** that satisfies steps 1–5 is an output of
  Spike B ([ADR-0006](0006-launcher-browser-query-adapter.md); v1.1 Section 3, Spike B acceptance
  criteria).
- Recorded gap (Work Plan §9): no implementation issue exists yet; **M2 issues must embed the six
  tests verbatim**. See [`docs/planning/architecture-traceability.md`](../planning/architecture-traceability.md).

## Related risks and readiness

- **R1** URL-bar frontend contract migration (the adapter absorbs churn; the contract does not move).
