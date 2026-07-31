# PR lifecycle

1. Human approves or creates an issue (templates in .github/ISSUE_TEMPLATE/).
2. Claude Code creates `issue/<number>-<slug>` from the current baseline.
3. Claude Code implements and tests per the issue and CLAUDE.md.
4. Claude Code opens a DRAFT PR with the full template.
5. Configured status checks run. Until CI is configured, the issue-required local
   validation is run instead and its evidence recorded in the PR.
6. Claude Code marks the PR ready only when acceptance criteria are met or clearly
   marked blocked (with reasons) in the PR body.
7. Codex reviews independently per AGENTS.md §16.
8. Claude Code addresses accepted blocking findings with commits; disagreements are
   answered in-thread with evidence or escalated.
9. Codex performs one re-review (round 2).
10. Human owner merges or requests further work.

Hard rules: default maximum two Codex rounds, then human escalation with both
positions summarized · Codex is advisory and never a hard GitHub merge gate · all
configured required status checks must pass and human approval is mandatory; until CI
is configured, documented local validation and human approval are the merge gates ·
Codex never implements fixes; Claude Code owns corrections · nobody but the human
merges.
