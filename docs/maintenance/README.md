# ESR maintenance

This directory will hold the machinery that makes ESR transitions routine. It is empty today.

**Created by Issue #7:**

- `docs/maintenance/rebase-checklist.md` — the ESR transition procedure expanded into concrete
  commands, executable without archaeology.
- `docs/maintenance/conflict-log.md` — the conflict watch set and its schema (file · upstream change ·
  our patch · resolution · minutes spent), seeded before the first conflict exists with the
  pre-registered F1 entries: the Nova default flip and the arrival of the parent/child controller
  split.

Standing rules, from [`AGENTS.md`](../../AGENTS.md) §6:

- every touched upstream file enters the conflict watch set;
- ESR transitions follow the rebase checklist;
- baselines change only via a recorded baseline-update entry in
  [`docs/baselines/`](../baselines/README.md), never silently.

Governing decisions: [ADR-0002](../adr/0002-release-baseline-esr153.md) (baseline and re-verification
at branch time) and risks R1–R3 in the accepted architecture.
