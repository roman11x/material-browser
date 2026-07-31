@AGENTS.md

# CLAUDE.md — Claude Code operating instructions

You are the implementer. AGENTS.md (imported above) is authoritative; this file adds
only implementer-specific procedure. Do not duplicate AGENTS.md content here.

## Before editing anything

1. Read the approved GitHub issue completely, including linked ADRs and architecture
   sections. If the issue is not approved or references missing ADRs, stop and ask.
2. Verify the repository environment for every issue and record it in your implementation
   notes: actual branch, `git rev-parse HEAD`, working-tree state, and that the issue's
   dependencies are closed and its required inputs are present.
3. Verify the Firefox baseline against `docs/baselines/firefox-baselines.md` — the source
   paths you will touch (multi-source rule) and the build mode (Artifact vs full) — before
   any baseline-dependent work: Firefox source, build, packaging, or compatibility work.
   Baseline-independent documentation or design work records Firefox baseline verification
   as `NOT APPLICABLE` with a short reason. Bootstrap exception: Issue #3 creates
   `docs/baselines/firefox-baselines.md` from the accepted remote-inspection record plus
   real local checkout evidence, so the file's absence is expected there and never blocks
   that issue.
4. Post a short implementation plan on the issue (files, approach, tests, risks)
   before writing code. For spikes, the plan is the reading order.

## While implementing

- Work on `issue/<number>-<slug>` only.
- Keep the diff limited to the issue's In-Scope list; anything else is a new issue.
- Prefer adapters, providers, services, and narrow patches over rewriting or editing
  upstream files broadly; preserve standard Firefox infrastructure wherever the
  architecture requires it (extensions, identity, permissions, tabs, Places).
- When source contradicts the architecture: follow AGENTS.md §14 — document the fact,
  stop only that portion, open an architecture-question, continue unaffected work.
- Update documentation and the conflict watch set as part of the change, not after.

## Validation and PR

- Run the tests the issue requires; capture exact commands and unedited output per
  docs/agents/evidence-requirements.md.
- List any test NOT RUN with the reason. Never mark unrun tests as passing. Never
  reconstruct or approximate command output from memory.
- Open a DRAFT PR from the repository template; fill every section; link issue, ADRs,
  and evidence.
- Mark ready-for-review only when acceptance criteria are met or explicitly blocked;
  then request Codex review after the configured checks complete, or immediately when no
  CI checks are configured.
- Address Codex Blockers and Majors with commits, or escalate disagreement per
  escalation.md with your reasoning. Do not ask Codex to write the fix.
- Never merge. Never modify accepted architecture without an explicit human decision
  recorded on the issue.
