# Agent Collaboration and Implementation Work Plan v1

Status: planning artifact for joint review, produced by the architect role. Nothing here has been executed: no files below exist in any repository, no branches, no PRs, no code. Every artifact is *proposed content* for the human owner to commit (or for Claude Code to commit under an approved issue).

Governing architecture: Pre-Implementation Architecture Pack v1.1 + Addendum v1.2 + Patch v1.2.1 (jointly, "the accepted architecture"), pinned to `esr153 @ f815328b` per the v1.1 metadata block.

---

## 1. Role definitions

**Fable 5 — Architect and planner.** Owns: product and technical architecture; ADRs; milestone definitions; dependency ordering; GitHub issue specifications; acceptance criteria; source-spike specifications; risk tracking; architecture-to-implementation traceability; shared agent instructions; dispute and escalation procedures. Does not own: repository implementation, Firefox patches, shell scripts, test implementation, branches, commits, PRs, or code-level responses to review comments. After this work plan and the agent instruction documents are accepted, the architect's involvement pauses and resumes only when: a source spike disproves an architectural assumption · Claude Code or Codex identifies an architecture-level contradiction · the human owner requests an architecture decision · an ESR transition requires architectural reconsideration.

**Claude Code — Implementer.** Owns: one approved issue at a time; dedicated branch per issue; inspecting current source before editing; implementation; tests; required validation; focused commits; draft PRs; test and build evidence; addressing valid Codex findings; updating affected documentation. Must not: change accepted architecture silently; expand issue scope without approval; merge its own PR; mark unrun tests as passing; invent command output; rewrite extension functionality instead of preserving Firefox infrastructure; resolve architecture disputes through implementation preference.

**Codex — Reviewer.** Owns: independent PR review; checking implementation against the approved issue; `AGENTS.md` compliance; regression and scope-creep detection; test adequacy including missing cases; upstream-maintainability review; security, permissions, privacy, and accessibility review; verifying claimed validation is actually evidenced; identifying architecture conflicts; separating blocking findings from suggestions. Must not: rewrite the implementation over style preference; approve ADR violations; treat unverified assumptions as facts; expand the issue during review; merge; directly alter Claude Code's branch unless the human explicitly changes the workflow.

**Human owner.** Final authority: approves issues, resolves escalations, merges PRs. CI plus human approval are the only merge gates.

---

## 2. Proposed `AGENTS.md` (repository root)

````markdown
# AGENTS.md — Project Constitution

Shared, authoritative repository-level instructions for all agents. Codex treats this
file as its primary review rulebook. Long procedures live in `docs/agents/`; this root
file stays concise so it always loads. Nested `AGENTS.md` files in subdirectories may
ADD rules for specific Firefox subsystems; they may never weaken or override root rules.

## 1. Project purpose

An independently branded, keyboard-first, Wayland-native Firefox derivative for Linux
(Fedora first; Niri and Hyprland both supported), tracking Firefox ESR, with a centred
launcher replacing the permanent address bar, a right-side vertical tab rail, browser
workspaces, and manual DMS/Material You theming through a semantic token layer.
Reference: `docs/adr/` and the accepted architecture pack in `docs/`.

## 2. Authority and precedence

1. Direct human-owner decision
2. Accepted ADRs
3. Accepted architecture pack and patches
4. AGENTS.md
5. Approved GitHub issue
6. Agent-specific instructions (CLAUDE.md, docs/agents/*)
7. Agent judgment

Higher rank wins. On conflict between any two sources, REPORT the conflict (see §14);
never silently choose.

## 3. Accepted architecture summary

- Raw Mozilla Firefox source; release baseline: current ESR per `docs/baselines.md`
  (currently esr153); no working checkout of older ESRs (ADR-0001/0002).
- Artifact Mode for frontend work; full builds only for identity, branding,
  distributables, Widevine Case B (ADR-0003).
- Native vertical-tabs/sidebar first; custom rail only with documented technical
  limitation (ADR-0004).
- Workspace substrate undecided until the Spike C options memo (ADR-0005).
- Launcher = our view + BrowserQueryAdapter over Firefox query/provider infrastructure;
  provider-level binding is a hypothesis under test (Spike B); the launcher never knows
  which URL-bar frontend is active (ADR-0006).
- Firm activation contract: guard → capture → close → execute → focus (ADR-0007).
- Manual DMS sync through semantic `--browser-*` tokens; no file watching; no raw DMS
  keys outside the adapter; no DMS colours injected into websites (ADR-0008).
- Widevine Case A (dev) / Case B (branded distributable) split; Case B alone controls
  public DRM claims (ADR-0009).
- Standard Wayland + XDG portals only; no compositor-specific core code (ADR-0010).
- Extension actions: host and invoke real WebExtension actions/popups; never
  reimplement them; badges keep extension-supplied meaning and colours (ADR-0011,
  Patch v1.2.1 §2).
- Page actions/identity: reuse real Firefox identity, permission, tracking-protection,
  bookmark, Reader Mode, translation, and page-action infrastructure via
  BrowserPageActionAdapter; never recreate security/permission state (ADR-0012).

## 4. Hard architectural constraints (Codex: violations are Blockers)

- No forked Firefox providers, history/bookmark/search reimplementations, parallel tab
  objects, or reimplemented extension/security/permission/bookmark state.
- No compositor-specific dependencies (hyprctl, Niri IPC, X11 tools, GNOME/KDE APIs)
  in core code.
- No automatic DMS palette watching or auto-recolour; no component reads DMS JSON
  directly; no DMS colours into web content.
- Enter-activation: exactly-once execution; launcher closes on success; failures follow
  ADR-0007.
- Security/auth/warning UI never rendered on low-opacity glass; security information is
  never sacrificed for minimalism.
- Meaning-colours rule: security, warning, permission, container, and extension-badge
  colours keep their semantic values; project tokens style frames and surfaces only.
- uBlock Origin and SponsorBlock run as genuine unmodified WebExtensions,
  `normal_installed`, user-disableable.
- Stock-toolbar removal stays reversible until Spikes D and F resolve.
- No copied proprietary CDM binaries, browser-identity spoofing, or undocumented DRM
  workarounds.

## 5. Source-verification requirements

- Never rely on remembered Firefox paths or APIs. Verify in the pinned checkout.
- File-existence claims require multi-source evidence: moz.build, jar.mn, content/
  listings, call sites, test manifests, packaged locations.
- Every source-reading document starts with the metadata block defined in
  `docs/agents/evidence-requirements.md` (repo, branch, commit, version.txt,
  date, build mode, relevant prefs).
- "Main is version X" style claims are prohibited; versions come from version files at
  recorded commits.

## 6. Upstream Firefox and ESR rules

- Implementation targets the pinned baseline in `docs/baselines.md`; baselines change
  only via a recorded baseline-update entry, never silently.
- Patches are narrow, modular, rebase-friendly; prefer adapters/providers/services/new
  modules over edits inside browser.xhtml or upstream classes.
- Every touched upstream file enters the conflict watch set
  (`docs/maintenance/conflict-log.md`).
- ESR transitions follow `docs/maintenance/rebase-checklist.md`.

## 7. Issue-scope rules

- One approved issue per branch per PR. No drive-by fixes; file a new issue instead.
- Scope expansion requires human approval recorded on the issue before implementation.
- Spike issues produce documents and throwaway experiments, never permanent code;
  experiment code must be clearly marked and excluded from release paths.

## 8. Testing and evidence requirements

- Required tests are listed per issue; the PR includes exact commands and real output
  per `docs/agents/evidence-requirements.md`.
- Tests not run are listed as NOT RUN with reasons; claiming unrun tests as passing is
  a Blocker and a trust violation.
- Build evidence for Artifact Mode includes source revision, artifact revision,
  job/platform, timestamp, `./mach artifact last` output, objdir clean/dirty state.

## 9. Security and privacy requirements

- Origin visibility invariants (prominent on navigation commit and registrable-domain
  change) may never regress.
- Sensitive-field detection uses trusted existing signals only; no content-DOM
  scanning services from chrome (Patch v1.2.1 §6 hierarchy).
- No telemetry additions, no external network calls beyond stock Firefox behaviour,
  without a human decision.
- Private-window state must remain isolated in every feature touching tabs,
  workspaces, extensions, or identity.

## 10. Accessibility requirements

- Keyboard path for every mouse path; focus visible via the focus-ring token.
- Reduced transparency: the browser's own setting is authoritative and always works;
  forced-colors/HCM disables unsafe glass.
- No information conveyed by colour alone.
- Restyles of native components must not degrade their a11y tree (verify, don't assume).

## 11. Git and branch rules

- Branch names: `issue/<number>-<slug>`. One issue per branch.
- Focused commits with imperative messages referencing the issue.
- No force-pushes after review has begun except by human instruction.
- Claude Code commits only to its own issue branches.

## 12. Pull-request requirements

- Draft PR first; use `.github/PULL_REQUEST_TEMPLATE.md` completely.
- Ready-for-review only when acceptance criteria are met or explicitly marked blocked.
- PR body links the issue, relevant ADRs, and evidence.
- Merging is human-only. CI green + human approval are hard gates.

## 13. Agent role separation

Fable 5 plans; Claude Code implements; Codex reviews; the human decides and merges.
Codex provides findings, never fixes; Claude Code owns corrections; neither agent
merges. Full definitions: `docs/agents/`.

## 14. Architecture escalation procedure

On discovering a source fact contradicting accepted architecture: document the fact →
stop only the contradicted portion → open an `architecture-question` issue (template
provided) → Codex independently verifies during review → Fable 5 proposes options →
human decides → ADR and issue amended → work resumes. Experiments never silently
become architecture. Details: `docs/agents/escalation.md`.

## 15. Definition of done

An issue is done when: acceptance criteria demonstrably met · required tests run with
evidence · documentation updated · conflict watch set updated for touched upstream
files · Codex blocking findings resolved or human-overruled · CI green · human merged.

## 16. Code Review Rules (Codex)

- Review against, in order: the approved issue's acceptance criteria; the ADRs it
  references; §4 hard constraints; §8–§10 requirements.
- Verify evidence, not claims: commands shown, outputs real, artifact/source revisions
  recorded, NOT-RUN tests honestly listed. Unevidenced validation claims are Blockers.
- Check upstream maintainability: does the diff touch upstream files more broadly than
  the issue requires? Are touched files added to the conflict watch set?
- Check scope: any change not traceable to the issue's In-Scope list is scope creep —
  Major, or Blocker if it alters architecture.
- Severity model (state severity, file/line, why it matters, evidence or failure
  scenario, smallest acceptable correction, and the ADR/criterion/rule violated):
  - **Blocker**: security problem, data-loss risk, architecture violation, broken
    build, missing required behaviour.
  - **Major**: likely regression, incorrect state handling, missing essential tests,
    serious upstream-maintenance problem.
  - **Minor**: localized correctness or maintainability issue that should be fixed.
  - **Suggestion**: optional improvement outside merge requirements.
- Style preference alone is never a Blocker and alone is at most a Suggestion.
- Maximum two review rounds by default; after two, unresolved disagreement escalates
  to the human owner with both positions summarized.
- Do not implement fixes, do not push to the branch, do not merge, do not expand the
  issue. Advisory only; the human decides.
````

---
## 3. Proposed `CLAUDE.md` (repository root)

````markdown
@AGENTS.md

# CLAUDE.md — Claude Code operating instructions

You are the implementer. AGENTS.md (imported above) is authoritative; this file adds
only implementer-specific procedure. Do not duplicate AGENTS.md content here.

## Before editing anything

1. Read the approved GitHub issue completely, including linked ADRs and architecture
   sections. If the issue is not approved or references missing ADRs, stop and ask.
2. Verify the environment against `docs/baselines.md`: actual branch, `git rev-parse
   HEAD`, tree cleanliness, source paths you will touch (multi-source rule), and the
   build mode (Artifact vs full). Record this in your implementation notes.
3. Post a short implementation plan on the issue (files, approach, tests, risks)
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
  then request Codex review after CI is ready.
- Address Codex Blockers and Majors with commits, or escalate disagreement per
  escalation.md with your reasoning. Do not ask Codex to write the fix.
- Never merge. Never modify accepted architecture without an explicit human decision
  recorded on the issue.
````

---

## 4. Proposed `docs/agents/` contents

### `docs/agents/README.md`

````markdown
# Agent documentation index

Load order for any agent joining the repository:
1. /AGENTS.md            — constitution and precedence (always load first)
2. Your role file        — implementer.md (Claude Code) or reviewer.md (Codex)
3. workflow.md           — the PR lifecycle both roles operate inside
4. evidence-requirements.md — what "proof" means here
5. escalation.md         — what to do when reality disagrees with the plan

Roles: Fable 5 = architect/planner (not present in day-to-day repo work);
Claude Code = implementer; Codex = reviewer; human owner = decisions and merges.
Nested AGENTS.md files may add subsystem rules; they never weaken root rules.
````

### `docs/agents/workflow.md`

````markdown
# PR lifecycle

1. Human approves or creates an issue (templates in .github/ISSUE_TEMPLATE/).
2. Claude Code creates `issue/<number>-<slug>` from the current baseline.
3. Claude Code implements and tests per the issue and CLAUDE.md.
4. Claude Code opens a DRAFT PR with the full template.
5. CI runs.
6. Claude Code marks the PR ready only when acceptance criteria are met or clearly
   marked blocked (with reasons) in the PR body.
7. Codex reviews independently per AGENTS.md §16.
8. Claude Code addresses accepted blocking findings with commits; disagreements are
   answered in-thread with evidence or escalated.
9. Codex performs one re-review (round 2).
10. Human owner merges or requests further work.

Hard rules: default maximum two Codex rounds, then human escalation with both
positions summarized · Codex is advisory; CI green + human approval are the only
merge gates · Codex never implements fixes; Claude Code owns corrections · nobody
but the human merges.
````

### `docs/agents/implementer.md`

````markdown
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
````

### `docs/agents/reviewer.md`

````markdown
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
````

### `docs/agents/escalation.md`

````markdown
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
````

### `docs/agents/evidence-requirements.md`

````markdown
# Evidence requirements

## Source-verification metadata block (all spike docs, all source claims)
Repository / Branch / Commit / Firefox version (version.txt + version_display.txt) /
Date inspected / Artifact or full build / Relevant feature preferences.
One block per branch when comparing.

## Baseline verification (local)
`git rev-parse HEAD` · `git status` · `git branch --show-current` ·
`cat browser/config/version.txt` · `cat browser/config/version_display.txt`
Recorded: remote URL, branch, commit, version files, checkout date, tree-clean state.
Local tip differing from docs/baselines.md → baseline-update entry, never a silent
replacement.

## Artifact Mode build evidence (every Artifact build claim)
Source revision · artifact revision · artifact job/platform · artifact timestamp ·
`./mach artifact last` output · objdir clean/dirty · clobber events recorded.
Source/artifact mismatch handling per Issue #8 rules; never continue unexplained
(Risk R14).

## Test evidence
Exact command · unedited output (explicit trim markers only) · environment
(compositor, scaling) for UI tests · NOT-RUN list with reasons.

## DRM evidence (Case A/B)
about:support media section · plugin/component state · EME init result · legal test
page result · per-service result with account-legality note · console/media logs.
````

---
## 5. Proposed `docs/planning/` contents

### `docs/planning/roadmap.md`

````markdown
# Roadmap (condensed; authoritative milestone detail in the architecture pack)

M0 Baseline            — issues #1–#7. Gate: baselines locally verified; policies,
                          Wayland, and DRM Case A control evidence recorded.
M1 Dev env + spikes    — issues #8–#19. Gate: Artifact build reproducible; Spikes
                          A–F documented with recommendations; addendum frames done.
M2 Launcher foundation — issues defined after Spike B (adapter design note).
                          Gate: ADR-0007 test suite green; toolbar removal reversible.
M3 Launcher visuals    — after M2 + provisional tokens. Includes chip state machine.
M4 Vertical tabs       — mechanism per Spike A decision (D3 discipline).
M5 Command provider    — model: ActionsProviderQuickActions (verified esr153).
M6 DMS RFC + adapter   — blocked on user fixtures; RFC ⛔ sections first.
M7 Workspaces          — blocked on substrate memo decision (Spike C → human).
M8 Website appearance  — per plan.
M-Full-Build (parallel)— charter from #14: identity, branding, distributable,
                          Widevine Case B. May start any time after M1.
M9 Hardening           — gates: #16 and #19 resolved (no-address-bar daily-driver),
                          #17 resolved (glass claims), Case B classification present.
M10 Packaging          — public claims constrained by Case B and compat reports.
Ongoing                — ESR transitions per docs/maintenance/; conflict log live.
````

### `docs/planning/milestone-0.md`

````markdown
# Milestone 0 — Baseline

Issues: #1 repo/ADR bootstrap · #2 mockup import · #3 baselines (local verification
rules per Patch v1.2.1 §6… as merged) · #4 distribution-scoped policy + isolated
profile · #5 Wayland/compositor baseline (Niri + Hyprland) · #6 Widevine Case A stock
control · #7 ESR maintenance scaffolding.

Entry: this work plan accepted; repository created by human owner.
Exit: all seven merged; docs/baselines.md verified against a local checkout;
evidence recorded per evidence-requirements.md.
Parallelism: #4, #5, #6 independent of each other; #2, #3, #7 depend on #1.
Human inputs: hardware/compositor access for #5; legal streaming account for #6.
````

### `docs/planning/milestone-1.md`

````markdown
# Milestone 1 — Dev environment and source spikes

Issues: #8 Artifact bootstrap (revision-compat AC + `mach artifact last` evidence) ·
#9 hello-chrome loop · #10 Spike A sidebar · #11 Spike B urlbar/Nova ·
#12 Spike C tabs/groups/SessionStore · #13 Case A on Artifact build ·
#14 full-build boundary doc · #15 mockup addendum (all frame sets incl. extensions,
identity, page actions) · #16 Spike D extension actions · #17 Spike E glass ·
#18 sensitive-signal investigation · #19 Spike F identity/page actions.

Ordering: #8 → #9 → {#10, #11, #12, #17} in any order; #16 and #19 after #9 (may run
in parallel with each other); #13 after #8; #18 after #8 (independent of visuals);
#14 after #8 (+#4 evidence); #15 after #2, informed by (not blocked on) spike
progress except where frames carry spike-gated guards.
Exit: every spike doc contains its required recommendation; readiness map updated;
any architecture-question issues resolved or explicitly parked by the human.
Spike outputs feeding decisions: A→rail mechanism (D3) · B→adapter binding + M2
design note · C→substrate memo → human decision · D→extension hosting choice ·
E→glass rung · F→identity/page-action hosting choice.
````

### `docs/planning/dependency-graph.md`

````markdown
# Dependency graph (issues #1–#19)

#1 ──► #2 ──► #15
 │ └──► #3
 └────► #7
#4 (independent)      #5 (independent)      #6 (independent) ──► #13
#3 ─► #8 ──► #9 ──► #10
      │      ├────► #11
      │      ├────► #12
      │      ├────► #16 ◄─(also #8)
      │      ├────► #17
      │      └────► #19 ◄─(also #8)
      ├────► #13
      ├────► #14 (+ #4 evidence)
      └────► #18

Decision joins (block later milestones, not M1 issues):
Spike A(#10) → rail mechanism · Spike B(#11) → M2 design · Spike C(#12) → substrate
memo → human decision · Spike D(#16)+F(#19) → no-address-bar daily-driver gate ·
Spike E(#17) → glass claims gate · #14 → M-Full-Build charter → Case B → M10 claims.
User fixtures → RFC ⛔ sections → M6 adapter (independent of all of the above).
````

### `docs/planning/architecture-traceability.md`

Content: the traceability matrix in §9 of this work plan, committed verbatim and kept current by the architect at each milestone close.

---

## 6. Proposed GitHub templates

### `.github/ISSUE_TEMPLATE/implementation.yml`

````yaml
name: Implementation issue
description: A scoped implementation task for Claude Code
labels: ["implementation", "needs-approval"]
body:
  - type: input
    id: objective
    attributes: { label: Objective, description: One sentence; outcome, not activity }
    validations: { required: true }
  - type: textarea
    id: arch-refs
    attributes: { label: Architecture references, description: "ADRs, pack sections, spike docs this work must conform to" }
    validations: { required: true }
  - type: textarea
    id: scope
    attributes: { label: In scope }
    validations: { required: true }
  - type: textarea
    id: non-goals
    attributes: { label: Out of scope / non-goals }
    validations: { required: true }
  - type: textarea
    id: files
    attributes: { label: Files/subsystems expected to be touched or investigated }
  - type: textarea
    id: acceptance
    attributes: { label: Acceptance criteria, description: Checkbox list; each must be evidenceable }
    validations: { required: true }
  - type: textarea
    id: evidence
    attributes: { label: Required test evidence, description: Commands/tests whose real output the PR must include }
    validations: { required: true }
  - type: textarea
    id: pr-boundaries
    attributes: { label: PR boundaries, description: One PR unless stated; what splits are allowed }
  - type: textarea
    id: review-focus
    attributes: { label: Codex review focus }
  - type: textarea
    id: escalation
    attributes: { label: Conditions requiring architecture escalation }
  - type: textarea
    id: inputs
    attributes: { label: Human-provided inputs, description: Must exist before work starts }
  - type: dropdown
    id: dependencies-met
    attributes: { label: Dependencies closed?, options: ["yes", "no — list in comment"] }
    validations: { required: true }
````

### `.github/ISSUE_TEMPLATE/source-spike.yml`

````yaml
name: Source spike
description: Read-only investigation of the pinned Firefox checkout producing a document
labels: ["spike", "needs-approval"]
body:
  - type: input
    id: spike-ref
    attributes: { label: Spike reference, description: "e.g. Architecture Pack Section 3 Spike B" }
    validations: { required: true }
  - type: textarea
    id: questions
    attributes: { label: Questions this spike must answer }
    validations: { required: true }
  - type: textarea
    id: entry-points
    attributes: { label: Verified starting entry points, description: Paths verified at the pinned commit; guesses prohibited }
  - type: textarea
    id: experiment
    attributes: { label: Minimal experiment, description: "Throwaway; lives under experiments/<issue>/" }
  - type: textarea
    id: outputs
    attributes: { label: Required outputs, description: Doc path, required recommendation set, rebase watch list }
    validations: { required: true }
  - type: checkboxes
    id: rules
    attributes:
      label: Spike rules acknowledged
      options:
        - { label: Doc opens with the source-verification metadata block, required: true }
        - { label: Experiment code is throwaway and excluded from product paths, required: true }
        - { label: Recommendation chosen only from the allowed set, required: true }
````

### `.github/ISSUE_TEMPLATE/bug.yml`

````yaml
name: Bug report
description: Defect in project code or docs (not upstream Firefox bugs)
labels: ["bug"]
body:
  - type: textarea
    id: expected
    attributes: { label: Expected behaviour (cite issue/ADR if applicable) }
    validations: { required: true }
  - type: textarea
    id: actual
    attributes: { label: Actual behaviour }
    validations: { required: true }
  - type: textarea
    id: repro
    attributes: { label: Reproduction steps }
    validations: { required: true }
  - type: textarea
    id: env
    attributes: { label: Environment, description: Baseline commit, build mode, compositor, scaling, profile state }
    validations: { required: true }
  - type: textarea
    id: evidence
    attributes: { label: Logs / screenshots }
````

### `.github/ISSUE_TEMPLATE/architecture-question.yml`

````yaml
name: Architecture question
description: A source fact contradicts accepted architecture, or a decision gap was found
labels: ["architecture-question", "needs-architect"]
body:
  - type: textarea
    id: fact
    attributes: { label: Discovered source fact, description: With metadata block and citation }
    validations: { required: true }
  - type: textarea
    id: contradiction
    attributes: { label: What it contradicts, description: ADR / pack section / issue assumption }
    validations: { required: true }
  - type: textarea
    id: stopped
    attributes: { label: Work stopped, description: Exactly which portion is paused; what continues }
    validations: { required: true }
  - type: textarea
    id: verification
    attributes: { label: Independent verification, description: Filled by Codex during review }
  - type: textarea
    id: options
    attributes: { label: Options and trade-offs, description: Filled by Fable 5 }
  - type: textarea
    id: decision
    attributes: { label: Human decision + ADR amendment link, description: Filled by the human owner }
````

### `.github/PULL_REQUEST_TEMPLATE.md`

````markdown
## Issue
Closes #<n>. Architecture references: <ADRs / pack sections>.

## What changed
<Focused summary tied to the issue's In-Scope list.>

## Acceptance criteria mapping
| Criterion | Where satisfied (file / doc) | Evidence |
|---|---|---|

## Baseline & build evidence
Branch/commit (`git rev-parse HEAD`): · Tree state: · Build mode: ·
Artifact evidence (if Artifact Mode): source rev / artifact rev / job / timestamp /
`./mach artifact last` output / objdir state:

## Test evidence
<Exact commands + unedited output. Explicit trim markers only.>
**Not run:** <tests + reasons, or "none">

## Upstream files touched
<List; confirm conflict watch set updated, or "none">

## Docs updated
<List, or "none required because …">

## Reviewer notes
<Known limitations, blocked criteria with reasons, escalations opened.>
````

---
## 7. Milestone roadmap

As specified in `docs/planning/roadmap.md` above (§5): M0 baseline (#1–#7) → M1 environment + six spikes (#8–#19) → M2 onward gated on spike outcomes, with the full-build track parallel from M1 and daily-driver/claims gates (#16+#19, #17, Case B) enforced at M9/M10. The roadmap file is the single condensed source; milestone detail remains in the architecture pack.

## 8. Implementation backlog — Issues #1–#19

**Backlog conventions (apply to every issue unless the entry overrides them):**
- *Classification key:* [DOC] documentation/setup · [SPIKE] source spike · [DESIGN] design artifact · [IMPL] Firefox implementation · [BUILD] full-build/packaging · [COMPAT] compatibility test. (Backlog #1–#19 contains no [IMPL] and no [BUILD] execution — by design, implementation begins at M2 and the full-build track begins from #14's charter.)
- *PR boundaries:* exactly one PR per issue, branch `issue/<n>-<slug>`, draft-first.
- *Required architecture references:* the accepted architecture stack is always in force; entries list only the sections that govern the specific work.
- *Codex review focus (baseline for all):* AC-to-evidence mapping, AGENTS.md §4/§5 compliance, scope containment; entries add issue-specific focus.
- *Escalation (baseline for all):* any source fact contradicting the referenced ADRs → architecture-question per escalation.md; entries add specific triggers.
- *Completion status:* all nineteen are **Not started** (specification approved via the architecture stack; awaiting human `approved` label per issue).
- *Human inputs:* "none" unless listed.

**#1 [DOC] Repository and ADR bootstrap.** Objective: versioned decision base exists before any code. Deps: repository created by human. Arch refs: v1.1 §1 as amended by v1.2 §3 (ADR-0011) and v1.2.1 §3 (ADR-0012). In scope: repo layout (docs/adr, docs/rfcs, docs/design, docs/planning, docs/agents, docs/maintenance, patches/, experiments/), ADR template, ADR-0001…0012 committed, AGENTS.md + CLAUDE.md + templates from this work plan committed. Out: any Firefox source; automation. Investigated: none. AC: all twelve ADRs render and cross-link; agent files load; templates appear in the GitHub issue chooser. Evidence: file listing, rendered links, screenshot of issue chooser. Review focus: ADR text fidelity to the accepted architecture (no silent rewording). Escalation: none expected. Inputs: repository + agent access (see §10).

**#2 [DESIGN] Approved mockup import.** Objective: the approved baseline is reproducible from the repo alone. Deps: #1. Arch refs: Review v1 §1 inventory; RFC §15 provisional palette. In scope: mockup HTML, decision inventory, palette table, reference screenshots. Out: addendum frames (#15). AC: newcomer can state the approved design from repo contents; baseline marked immutable-except-by-decision. Evidence: committed artifacts + rendered doc. Review focus: inventory matches the actual mockup file, not a paraphrase. Escalation: none.

**#3 [DOC] Record and locally verify Firefox baselines.** Objective: remote-inspection hashes become development baselines only through local verification. Deps: #1. Arch refs: ADR-0002; v1.2.1 §6 (as merged into #3); evidence-requirements.md. In scope: docs/baselines.md with the v1.1 metadata block as the inspection record; local checkout verification (`git rev-parse HEAD`, `git status`, `git branch --show-current`, both version files); baseline-update procedure. Out: builds. AC: local evidence recorded for esr153 (esr140/main may be verified remotely with that noted); divergence handled by baseline-update entry, never silent replacement. Evidence: command transcripts. Review focus: the recorded commit equals the transcript's commit; tree-clean state honest. Escalation: esr153 tip materially diverged from `f815328b` in ways affecting cited paths → architecture-question before adoption. Inputs: disk/bandwidth for the checkout.

**#4 [COMPAT] Distribution-scoped extension policy + isolated profile.** Objective: uBO + SponsorBlock auto-installed via distribution-scoped `policies.json`, user-disableable, against an isolated dev profile. Deps: none. Arch refs: v1.1 #4-as-revised (v1.2-era correction), ADR-0003, AGENTS.md §4 extension constraints. In scope: `distribution/policies.json` placement (or system policy location when deliberately testing system-wide), `ExtensionSettings` with verified IDs, `normal_installed`, AMO `install_url` copied at commit time with retrieval date; profile containing only user-scoped state. Out: `force_installed`; Dark Reader; branding. Investigated: `browser/components/enterprisepolicies/Policies.sys.mjs` (verified esr153) for behaviour questions only. AC: about:policies active from distribution scope; both extensions install, function, disable/re-enable; profile contains no policy file. Evidence: screenshots + working block/skip demonstration + directory listing. Review focus: **install_url provenance** (dated, from AMO, not guessed — guessing is review-rejection); policy scope correctness. Escalation: `normal_installed` cannot deliver the required default-installed-but-disableable behaviour. Inputs: none.

**#5 [COMPAT] Stock Wayland baseline under Niri and Hyprland.** Objective: pre-modification behaviour reference. Deps: none. Arch refs: ADR-0010; v1.1 #4 scope list. In scope: portals file dialog, screen share, notifications, clipboard, DnD, fullscreen video, PiP, fractional scaling — per compositor. Out: fixes; multi-monitor depth. AC: findings table pass/fail/notes per item per compositor. Evidence: table + anomaly screenshots labelled with compositor/scale. Review focus: coverage completeness; environment labelling. Escalation: none (findings feed risks, not architecture). Inputs: access to both compositors on the target hardware.

**#6 [COMPAT] Widevine Case A stock control.** Objective: DRM control datapoint. Deps: none. Arch refs: ADR-0009. In scope: stock Firefox, clean profile, Fedora/Wayland; CDM download, plugin/component state, EME init, one legal public EME page, one legally held commercial service; playback repeated under both compositors. Out: Case B; workarounds. AC: findings per ADR-0009 axes recorded in docs/drm/spike.md. Evidence: DRM evidence set per evidence-requirements.md. Review focus: no prohibited techniques anywhere in the method; account-legality noted. Escalation: none. Inputs: **legal streaming account** (human).

**#7 [DOC] ESR maintenance scaffolding.** Objective: first ESR transition pre-instrumented. Deps: #1. Arch refs: ADR-0002; v1.1 Section 7 R1/R3. In scope: rebase-checklist v0 (concrete commands), conflict-log schema, pre-registered F1 entry (Nova default flip, parent/child controller arrival). Out: performing any rebase. AC: checklist executable without archaeology; log seeded. Evidence: docs render. Review focus: checklist steps match the plan's 8-step ESR procedure. Escalation: none.

**#8 [DOC] Artifact Mode bootstrap at the pinned esr153 revision.** Objective: reproducible dev environment. Deps: #3. Arch refs: ADR-0003; v1.2 §8 #8 amendment; v1.2.1 §6 (#8 amendment); R14. In scope: bootstrap flow, Desktop Artifact Mode, `./mach run` with the #4 profile, reproducible build doc with metadata header. Out: full build. AC (full amended set): checkout at intended baseline with `git rev-parse HEAD` evidence · artifact discovery/download success · selected artifact source revision recorded · source/artifact compatibility confirmed · `./mach build` success · `./mach run` success · relevant browser-chrome test success · `./mach artifact last` output stored · objdir state recorded · pinning failures recorded honestly; no-artifact case handled by the four-step rule (never unexplained mismatch). Evidence: Artifact evidence block per evidence-requirements.md + build log excerpt + screenshot. Review focus: **revision arithmetic** — source rev vs artifact rev actually compared, not asserted. Escalation: Artifact Mode unusable at/near the baseline → decision needed on moving baseline vs full build (human). Inputs: none.

**#9 [DOC] Hello-chrome edit/test loop.** Objective: prove edit-run-test. Deps: #8. Arch refs: ADR-0003. In scope: one small visible chrome change (reverted); `./mach lint`; one existing urlbar browser-chrome test run, recording whether the default or `browserNova` manifest executed. Out: keeping the change. AC: change visible; commands documented with real output; manifest identity recorded. Evidence: transcripts + screenshot. Review focus: output authenticity. Escalation: none.

**#10 [SPIKE] Native vertical tabs / sidebar (Spike A).** Objective: rail mechanism decision input. Deps: #8, #9 recommended. Arch refs: ADR-0004; v1.1 Section 3 Spike A as rebased by v1.2 §8 (Q10–Q15). In scope: the fifteen questions; CSS-only restyle experiment (right side, 64/248px, hue-292, accent bar); a11y verification. Out: JS feature work; workspace implementation. Investigated: verified sidebar/tabbrowser/theme paths per the pack. AC: every D3 checklist item classified reachable-CSS / narrow-JS / blocked with file+mechanism; experiment screenshots vs mockup; a11y tree confirmed; rebase watch list. Evidence: spike doc with metadata block; experiment under experiments/10/. Review focus: fallback-trigger claims backed by evidence, not convenience (D3 rule). Escalation: any custom-rail trigger fires → evidence to the human before the fallback is chosen. Inputs: none.

**#11 [SPIKE] URL-bar / Nova / Smartbar (Spike B).** Objective: adapter binding decision + M2 design note. Deps: #8, #9. Arch refs: ADR-0006/0007; F1; v1.2 §2 provider-contract mandate; v1.1 Spike B as rewritten. In scope: v1's rebased questions + the seven Nova questions + the ten-point cross-revision provider-contract comparison (esr140/esr153/main where practical); provider-level minimal experiment (bare panel → adapter skeleton → ProvidersManager → logged results; Nova both states if togglable). Out: styling; activation implementation; relocating the stock bar. AC: M2 design note with adapter interface, binding level with evidence, hide-vs-remove strategy, ADR-0007 hook point, prefix mechanism, process-boundary notes, both-mode feasibility — plus the adapter-wrappable contract subset as primary deliverable. Evidence: spike doc; experiment under experiments/11/. Review focus: binding-level recommendation follows the evidence, not the ADR's preference (the hypothesis is allowed to fail). Escalation: provider seam materially unstable across the pinned revisions → architecture-question on ADR-0006 before M2 issues are written. Inputs: none.

**#12 [SPIKE] Tab visibility, tab groups, SessionStore (Spike C).** Objective: factual inputs for the substrate memo. Deps: #8. Arch refs: ADR-0005; v1.2 §8 expanded list. In scope: the eleven esr153 behaviours (persistence, saved/closed groups, collapsed a11y, previews, active-tab handling, split view if present, vertical rendering, cross-window movement, pinned interactions, hidden-tab SessionStore records, private windows); no 140-era conclusions imported. Out: the recommendation itself (memo + human decision follow). AC: every memo matrix cell fillable with mechanism/confidence/failure-mode; hard-gate evidence (tab-loss paths) explicitly collected. Evidence: spike doc. Review focus: evidence-per-cell honesty (verified vs inferred vs unknown). Escalation: credible unavoidable tab-loss path in all four substrate options. Inputs: none.

**#13 [COMPAT] Widevine Case A on the Artifact build.** Objective: second DRM datapoint. Deps: #6, #8. Arch refs: ADR-0009. In scope: repeat #6's protocol on the Artifact build; diff vs control appended to docs/drm/spike.md. Out: Case B. AC: diff table complete. Evidence: DRM set. Review focus: protocol identical to #6 (otherwise the diff is meaningless). Escalation: none. Inputs: same account as #6.

**#14 [DOC] Full-build requirements and packaging-boundary document.** Objective: honest full-build charter. Deps: #8; #4 evidence. Arch refs: ADR-0003 as corrected. In scope: what genuinely requires full builds (identity, branding, distributable, Case B) vs what does not (distribution policies, chrome work); M-Full-Build charter. Out: performing a full build. AC: charter exists; every claimed full-build necessity justified. Evidence: doc. Review focus: no "baked policies"-class overclaims recur. Escalation: none.

**#15 [DESIGN] Mockup addendum.** Objective: all corrective/missing states exist as design artifacts. Deps: #2; frame-level guards tied to #16/#19 where marked. Arch refs: v1.1 Section 2 + v1.2 §7 (badge-corrected) + v1.2.1 §5; meaning-colour rule. In scope: frames 2.1–2.9, extension frames (extension-authored badge colours), identity/permission/tracking/page-actions/container frames with spike-gated captions. Out: altering the approved baseline; final opacity numbers (RFC-owned). AC: every listed state depicted; baseline unchanged; badge ownership and meaning-colour rules visibly honoured; launcher-invocation guarantees not implied pre-Spike-D. Evidence: side-by-side with baseline. Review focus: vocabulary consistency; the two colour-ownership rules. Escalation: none. Inputs: none (design proceeds from the pack).

**#16 [SPIKE] Extension actions / unified extensions (Spike D).** Objective: extension hosting recommendation. Deps: #8, #9. Arch refs: ADR-0011; v1.2 §4 + v1.2.1 §2 (Q11 + six badge checks); R12. In scope: the eleven questions; minimal experiment with a real action rehosted, genuine popup, badge/context-menu/per-tab checks, then uBO and SponsorBlock, tab-switch and second-window verification; badge-ownership checks. Out: any custom popup/state reimplementation (prohibited outcome). AC: exactly one of the four allowed recommendations, with evidence; Q10 rebase list; badge checks pass/fail recorded. Evidence: spike doc; experiments/16/. Review focus: recommendation ∈ allowed set; popup-anchoring claims demonstrated, not asserted. Escalation: all four hosting options fail for uBO or SponsorBlock. Inputs: none. Gate exported: launcher not daily-driver-ready until resolved.

**#17 [SPIKE] Browser-chrome glass feasibility (Spike E).** Objective: establish the achievable glass rung. Deps: #9. Arch refs: v1.2 §5; R13; RFC §7/§10 interactions. In scope: the 7-case matrix across the listed environments; determinations list; rung recommendation. Out: compositor rules as core; desktop-through transparency. AC: rung established with per-environment evidence incl. video + fractional scaling + Niri/Hyprland diff; frame-time/GPU numbers captured. Evidence: spike doc with labelled captures + perf numbers; experiments/17/. Review focus: "blur works" claims show chrome-over-*content* blur specifically, not chrome-over-chrome. Escalation: none (all rungs are accepted designs). Inputs: both compositors on target hardware. Gate exported: no frosted-blur release claims until resolved.

**#18 [SPIKE] Sensitive-field signal investigation.** Objective: select the sensitive-trigger rung. Deps: #8; independent of visuals. Arch refs: v1.2 §6; R15; ADR constraints on DOM scanning. In scope: the investigation scope list over the verified passwordmgr/formautofill entry points; rung recommendation per the four-step hierarchy. Out: implementing the trigger; any scanning service. AC: rung chosen with evidence; false-positive/negative behaviour characterized; Fission/private/cross-origin implications documented. Evidence: spike doc. Review focus: rung justified by trusted-signal availability, not desirability. Escalation: none (rung 4 is an accepted outcome; chip prominence invariant unaffected). Inputs: none.

**#19 [SPIKE] Built-in page-action/identity/permission spike (Spike F).** Objective: identity/page-action hosting recommendation. Deps: #8, #9; parallel with #16 allowed. Arch refs: ADR-0012; v1.2.1 §4; R16. In scope: the fourteen questions; minimal experiment host (origin/security button, tracking-protection state, live permission indicator, bookmark, Reader Mode, test extension page action) with the ten verification points; locate-don't-guess path inventory from the verified starting call sites. Out: reimplementing security/permission/tracking/bookmark state (prohibited); finalizing the chip-vs-adjacent-button layout before findings. AC: one of the four allowed recommendations; ownership map; safe-to-hide vs must-remain-visible control lists; rebase list; mockup implications. Evidence: spike doc; experiments/19/. Review focus: staleness testing real (tab switches actually exercised); safety-critical list conservative. Escalation: stock panels cannot re-anchor without content copying → architecture-question on ADR-0012 before M2/M3 chip implementation. Inputs: none. Gates exported: origin-chip implementation depends on findings; no-address-bar daily-driver requires #16+#19; toolbar removal reversible until known.

---

## 9. Architecture traceability matrix

Format: ADR → milestone → issue(s) → expected PR(s) → required tests/verification → relevant risks. PRs are named by their issue (`PR-#n`). "Later" = issues not yet written by design (M2+ issues are authored after their gating spikes, per the accepted workflow).

| ADR | Milestone | Issue(s) | Expected PR | Verification | Risks |
|---|---|---|---|---|---|
| 0001 raw source | M0 | #1, #3 | PR-#1, PR-#3 | baseline transcripts; ADRs committed | R3 |
| 0002 ESR 153 baseline | M0→ongoing | #3, #7 | PR-#3, PR-#7 | local verification; checklist + seeded log | R1–R3, R14 |
| 0003 Artifact/full-build boundary | M1 | #8, #14 | PR-#8, PR-#14 | amended #8 AC set; charter | R5, R14 |
| 0004 native tabs first | M1→M4 | #10; M4 later | PR-#10 | Spike A classification + a11y evidence | R2 |
| 0005 workspace substrate open | M1→M7 | #12; memo + M7 later | PR-#12 | matrix evidence; hard gates | R8 |
| 0006 launcher/adapter | M1→M2 | #11; M2 later | PR-#11 | contract comparison; M2 design note | R1 |
| 0007 activation contract | M2 | **later (gap, planned)** | — | the six named tests, written into M2 issues verbatim | R1 |
| 0008 DMS tokens/manual sync | M6 | **blocked on fixtures**; RFC ⛔ | — | token snapshots + computed-style CI; fixture tests | R7 |
| 0009 Widevine A/B | M0/M1 + full-build track | #6, #13; Case B later | PR-#6, PR-#13 | DRM evidence sets; Case B classification | R4 |
| 0010 Wayland/portals | M0 | #5 | PR-#5 | per-compositor findings table | R10, R11 |
| 0011 extension actions | M1→M9 gate | #16, #15 | PR-#16, PR-#15 | Spike D recommendation + badge checks | R12 |
| 0012 page actions/identity | M1→M9 gate | #19, #15 | PR-#19, PR-#15 | Spike F recommendation + control lists | R16 |

**Product-defining decisions currently lacking an implementation/verification issue (called out per instruction; all are *deliberate sequencing*, not oversights, but they are real gaps until written):**
1. **ADR-0007 activation contract** — no issue exists yet; M2 issues must embed its six tests verbatim. Owner: architect, after #11.
2. **ADR-0008 adapter implementation** — RFC ⛔ sections and the M6 adapter have no issues; blocked on user fixtures, then architect writes them.
3. **D1 security-chip state machine and D2 rail keyboard model** — design accepted, implementation issues arrive with M3/M4 after #10/#19.
4. **Workspace implementation (post-memo)** — awaits the human substrate decision.
5. **Glass presets → rung compilation (RFC §7 after #17)** — M3 issue to be written with the rung known.
6. **Screenshot-regression spike (RFC §15)** — approved as a spike but not yet numbered; recommend adding as **#20** when M1 issues are opened, or folding into M3 entry. Flagged for the human.
7. **Case B execution** — charter comes from #14; the executing issues belong to the full-build track and are unwritten.

---

## 10. Open human inputs

1. **Repository creation and agent access** — GitHub repo (name can be provisional; final branding is an M10 concern, but the repo needs *a* name now), Claude Code and Codex wired to it, branch protection: merges by owner only.
2. **License decision** — the derivative inherits MPL-2.0 obligations for Firefox files; the project's own files need a declared license (MPL-2.0 throughout is the simplest coherent choice). Owner decision; blocks #1.
3. **DMS palette fixtures** — the five items (active `dank-pywalfox.json`, several contrasting wallpapers' palettes, light-leaning sample or confirmation, writer/timing answer, adjacent files). Blocks RFC ⛔ and M6; blocks nothing in M0/M1.
4. **Legal streaming account** for #6/#13.
5. **Hardware/compositor availability** for #5/#17 (Legion with both Niri and Hyprland sessions).
6. **AMO install_url retrieval** happens at #4 commit time (Claude Code retrieves; human merge attests).
7. **Decision on gap item 6** (§9): add screenshot-regression spike as #20 now, or defer to M3 entry.
8. **Approval labels** on #1–#19 to start the queue.

## 11. Readiness assessment

All artifacts in this plan are proposed content — nothing is created in any repository, and no commands have been run against one.

```text
Ready for coding-agent handoff
```

- **Documentation ready:** yes — AGENTS.md, CLAUDE.md, six docs/agents files, five docs/planning files, four issue templates, PR template: complete proposed content above, pending human commit via #1.
- **Agent instructions ready:** yes — role separation, workflow, severity model, escalation, and evidence rules are internally consistent with the accepted architecture and the two-round review policy.
- **Backlog ready:** yes — #1–#19 specified with the full field set; classifications preserved ([DOC]/[SPIKE]/[DESIGN]/[COMPAT]); no [IMPL]/[BUILD] execution in scope by design; one recommended addition (#20 screenshot-regression spike) awaiting the human decision in §10.7.
- **Human inputs still needed:** items 1–2 block #1 (repo + license); items 4–5 block #5/#6/#13/#17 only; item 3 blocks only M6/RFC ⛔; items 7–8 are decisions/labels.
- **First issue Claude Code should execute:** **#1** (repository and ADR bootstrap) — it unblocks #2/#3/#7 and materializes the constitution the rest of the workflow depends on. First *source-touching* issue afterward: #3, then #8.
- **First review Codex should perform:** **PR-#1** — reviewing the committed ADRs and agent files against this plan's proposed content is the cheapest possible calibration run for the finding format, severity model, and evidence rules before any Firefox source is in play.

*Work plan ends. The architect's involvement now pauses per §1, resuming on spike contradictions, architecture questions, human request, or ESR transition.*