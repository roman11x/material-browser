# Firefox baselines

This directory holds the record of the exact Firefox source revisions this project is developed and
released against.

**Canonical file: `docs/baselines/firefox-baselines.md` — created and populated by Issue #3.** It does
not exist yet; nothing here has been verified against a local checkout.

Scope, per [ADR-0002](../adr/0002-release-baseline-esr153.md) and Addendum v1.2 §8:

- the source-verification metadata block for each inspected branch (repository, branch, commit,
  `browser/config/version.txt` and `version_display.txt`, date inspected, build mode, relevant
  preferences);
- the remote-inspection record from Architecture Pack v1.1, which becomes a *development baseline*
  only after local verification with `git rev-parse HEAD`, `git status`, `git branch --show-current`
  and both version files;
- the baseline-update procedure: a local tip differing from the recorded baseline is never silently
  substituted — a dated baseline-update entry explains the change.

Evidence format: [`docs/agents/evidence-requirements.md`](../agents/evidence-requirements.md).
