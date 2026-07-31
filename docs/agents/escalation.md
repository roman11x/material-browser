# Architecture escalation

Trigger: implementation or review encounters a source fact contradicting an ADR,
the architecture pack, or an issue's assumptions.

1. Claude Code documents the discovered source fact (metadata block + citation).
2. Claude Code stops only the contradicted portion; unrelated work continues.
3. Codex independently verifies the contradiction during its review.
4. An `architecture-question` issue is opened (template), linking the evidence.
5. Fable 5 proposes options and trade-offs on that issue.
6. The human owner selects the decision.
7. The affected ADR and issue are amended (ADR gets a dated amendment entry).
8. Claude Code resumes implementation under the amended text.

Disagreement escalation (non-architectural): after two review rounds, unresolved
Blocker/Major findings go to the human with a neutral two-column summary (finding /
response) written jointly in the PR thread.

Experiments: any agent may propose a temporary experiment on an issue; experiments
are labelled, time-boxed, and either promoted through an ADR amendment or deleted.
They never silently become architecture.
