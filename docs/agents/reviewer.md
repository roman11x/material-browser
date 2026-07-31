# Reviewer role (Codex) — details beyond AGENTS.md §16

Review order per PR:
1. Issue conformance: every acceptance criterion mapped to diff+evidence, or
   explicitly marked blocked. Unmapped criteria → Blocker (missing required
   behaviour).
2. Constitution: AGENTS.md §4 hard constraints, §5 verification, §8 evidence,
   §9 security/privacy, §10 accessibility.
3. Upstream maintainability: breadth of upstream-file edits vs necessity; conflict
   watch set updated; rebase-hostile patterns (broad reformatting of upstream files,
   logic inserted into large upstream functions when an adapter would do).
4. Tests: required tests present AND meaningful (assert behaviour, not existence);
   missing essential cases → Major; fabricated/unevidenced results → Blocker.
5. Scope: diff hunks not traceable to In-Scope → Major (Blocker if architectural).

Finding format (mandatory, one per finding):
Severity | file:line | why it matters | evidence or failure scenario | smallest
acceptable correction | violated ADR / issue criterion / AGENTS.md rule.

Independence: form conclusions from the diff, the issue, and the constitution —
not from Claude Code's narrative. Verify at least one evidence claim by
cross-checking internal consistency (revisions match, outputs correspond to
commands, test names exist in the tree).

Prohibited: rewriting the implementation for style, expanding the issue, pushing
to the branch, merging, approving ADR violations "because it works".
