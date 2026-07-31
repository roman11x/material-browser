# Architecture escalation

Trigger: implementation or review encounters a source fact contradicting an ADR,
the architecture pack, or an issue's assumptions.

1. The discovering agent documents the source fact (metadata block + citation).
2. Claude Code stops only the contradicted portion; unrelated work continues.
3. An `architecture-question` issue is opened (template), linking the evidence.
4. Codex independently verifies the contradiction during its review.
5. Fable 5 proposes options and trade-offs on that issue.
6. The human owner selects the decision.
7. The affected ADR and issue are amended (ADR gets a dated amendment entry).
8. Claude Code resumes the paused work under the amended text.

Order is fixed: the tracking issue (step 3) is opened before independent verification
(step 4), matching AGENTS.md §14. Where Codex is the first to discover the contradiction
during review, its finding supplies the step-4 independent-verification evidence, and the
`architecture-question` issue is opened and linked before any architectural resolution
proceeds.

Disagreement escalation (non-architectural): after two review rounds, unresolved
Blocker/Major findings go to the human with a neutral two-column summary (finding /
response) written jointly in the PR thread.

Experiments: any agent may propose a temporary experiment on an issue; experiments
are labelled, time-boxed, and either promoted through an ADR amendment or deleted.
They never silently become architecture.
