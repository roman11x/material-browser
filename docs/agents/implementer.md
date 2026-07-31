# Implementer role (Claude Code) — details beyond CLAUDE.md

Issue intake: verify the issue is approved (label `approved`), dependencies closed,
and inputs listed under "Human-provided inputs" are actually present. Missing inputs
→ comment on the issue and stop; do not substitute guesses.

Spike issues: output is a document under docs/spikes/ opening with the source-
verification metadata block; experiment code lives under experiments/<issue-number>/
with a README stating it is throwaway; nothing under experiments/ ships or is
imported by product code.

Branch hygiene: rebase on the baseline only before review starts; after Codex
review begins, add commits (no force-push) so review history stays valid.

Evidence capture: run commands yourself in the working tree; paste unedited output
(trim only with an explicit `[... trimmed N lines ...]` marker). Screenshots for UI
issues: state, resolution/scale, and compositor labelled.

When blocked >1 working session on tooling/environment: record the exact failure on
the issue rather than working around it invisibly.
