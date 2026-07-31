# Accepted architecture documents (historical records)

This directory holds the accepted architecture documents exactly as they were accepted. They are
**historical records, immutable in place**. Corrections arrive as a new numbered addendum or patch
document — never as an edit to an accepted one. That is the mechanism v1.1, v1.2 and v1.2.1 already
use, each carrying its own merge guide.

## Lineage

| Document | Role |
|---|---|
| [`pre-implementation-architecture-v1.md`](pre-implementation-architecture-v1.md) | **v1 — the original architecture pack.** |
| [`pre-implementation-architecture-v1.1.md`](pre-implementation-architecture-v1.1.md) | **v1.1 supersedes and amends v1.** |
| [`architecture-addendum-v1.2.md`](architecture-addendum-v1.2.md) | **v1.2 amends v1.1.** |
| [`architecture-patch-v1.2.1.md`](architecture-patch-v1.2.1.md) | **v1.2.1 amends v1.1 and v1.2.** |

**v1 remains preserved because v1.1 refers to unchanged decisions without restating all their
details.** ADR-0001, ADR-0004, ADR-0005, ADR-0007, ADR-0008, ADR-0009 and ADR-0010 exist in full
only in v1; v1.1 carries them forward by reference. ADR-0007's six required tests and ADR-0009's
Case B report axes are examples of detail that lives only in v1.

**When documents conflict, the latest accepted amendment wins.** Read any decision in this order:

```text
v1 decision text → v1.1 amendments/replacements → v1.2 amendments → v1.2.1 amendments
```

[`docs/adr/`](../adr/README.md) is the navigable current-decision surface; it applies this order for
every decision and labels superseded text as such. No merged "current architecture" document exists —
merging is performed by reading, guided by each amendment document's own merge guide (v1.2 "Merge
guide — where v1.2 slots into v1.1"; v1.2.1 §9).

## Superseded v1 claims — not current architecture

v1 is preserved for its unchanged decisions and its provenance. The following v1 claims were
replaced by later documents and must never be cited as current:

| Superseded v1 claim | Replaced by |
|---|---|
| Release baseline is the current ESR line, "currently verified" as **ESR 140** | v1.1 ADR-0002: baseline is **`esr153`**; `esr140` is comparison material only |
| `[F1]` — the urlbar view/controller/input trio is "gone" on `main` | v1.1 "Finding F1 (corrected)": the modules were **moved and repackaged as browser-content modules**; v1's moz.build-only inference is recorded as methodologically wrong and replaced by the multi-source verification rule |
| "Baked policies" listed as a full-build justification (ADR-0003) | v1.1 ADR-0003: distribution-scoped policy testing is **build-independent** |
| ADR-0006's launcher adapter bound to the URL-bar view/controller layer | v1.1 ADR-0006 (frontend-agnostic, provider-level preference) as amended by v1.2 §2 (provider-contract stability is a **hypothesis under test** by Spike B) |
| `esr140` file-path evidence throughout Sections 3 and 8 | v1.1 Sections 3 and 8, re-verified on `esr153 @ f815328b` |
| Issue set numbered #1–#13 | v1.1 Section 6 (#1–#15), Addendum v1.2 §8 (#16–#18), Patch v1.2.1 §6 (#19) |
| Spikes A–C only | Spikes D and E added in v1.2 §4–§5; Spike F added in v1.2.1 §4 |

## Provenance and approved corrections

The documents in this directory are byte-identical copies of the accepted originals and are never
edited. The **generated** operational documents in this repository —
[`AGENTS.md`](../../AGENTS.md), [`CLAUDE.md`](../../CLAUDE.md), [`docs/agents/`](../agents/README.md),
[`docs/planning/`](../planning/roadmap.md) and the GitHub templates — are extracted verbatim from the
accepted *Agent Collaboration and Implementation Work Plan v1*, preserved byte-identically at
[`docs/planning/agent-collaboration-work-plan-v1.md`](../planning/agent-collaboration-work-plan-v1.md).

Six corrections were applied to those generated files under **direct human-owner authority**, and to
those files only. The preserved documents keep the original text.

| # | Correction | Files affected | Before | After | Authority |
|---|---|---|---|---|---|
| 1 | Canonical baseline-record path | `AGENTS.md` §3 and §6; `CLAUDE.md` "Before editing anything" step 2; `docs/agents/evidence-requirements.md`; `docs/planning/milestone-0.md` | `docs/baselines.md` | `docs/baselines/firefox-baselines.md` | Human-owner decision on Issue #1 (2026-07-31) |
| 2 | Issue-template label | `.github/ISSUE_TEMPLATE/source-spike.yml` | `labels: ["spike", "needs-approval"]` | `labels: ["source-spike", "needs-approval"]` | Human-owner decision on Issue #1 (2026-07-31); the repository label is `source-spike` and no `spike` label exists |
| 3 | Quoted dropdown label, so the form parses under both local YAML parsers | `.github/ISSUE_TEMPLATE/implementation.yml` | `{ label: Dependencies closed?, … }` | `{ label: "Dependencies closed?", … }` | Human-owner correction on draft PR #2 (2026-07-31) |
| 4 | CI bootstrap rule — CI is not an unconditional merge gate while no CI is configured | `AGENTS.md` §12 and §15; `CLAUDE.md` "Validation and PR"; `docs/agents/workflow.md` step 5 and hard rules | "CI green + human approval are hard gates" · "CI green" · "CI runs" · "request Codex review after CI is ready" | "All configured required status checks must pass. Until CI is configured, documented local validation and human approval are the merge gates." Codex review is requested after the configured checks complete, or immediately when none are configured; Codex remains advisory and never a hard GitHub merge gate; human approval remains mandatory | Human-owner correction on draft PR #2 (2026-07-31) |
| 5 | Baseline bootstrap rule — Firefox baseline verification is scoped to baseline-dependent work instead of gating every issue on a file that Issue #3 has not yet created | `CLAUDE.md` "Before editing anything" steps 2–4; `docs/agents/evidence-requirements.md` "Baseline verification (local)" | One step: "Verify the environment against `docs/baselines/firefox-baselines.md`" before editing anything, and a single combined baseline-verification evidence section | Repository verification (branch, commit, working-tree state, dependencies and required inputs) is required for **every** issue; **Firefox baseline** verification is required before baseline-dependent Firefox source, build, packaging or compatibility work; baseline-independent documentation or design work records it as `NOT APPLICABLE` with a short reason; Issue #3 is the bootstrap exception — it creates `docs/baselines/firefox-baselines.md` and is never blocked by that file's absence | Human-owner acceptance of Codex review round 1 finding P1 on PR #2 (2026-07-31) |
| 6 | Escalation order — the `architecture-question` issue is opened before Codex's independent verification | `docs/agents/escalation.md` steps 1–8 | Step 3 Codex verifies → step 4 issue opened | Step 3 issue opened → step 4 Codex verifies, matching AGENTS.md §14; where Codex discovers the contradiction first, its finding supplies the step-4 evidence and the issue is opened and linked before architectural resolution proceeds | Human-owner acceptance of Codex review round 1 finding P2 on PR #2 (2026-07-31) |

No other generated file differs from the work plan's proposed text.
