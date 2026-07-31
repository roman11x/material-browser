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

- Raw Mozilla Firefox source; release baseline: current ESR per `docs/baselines/firefox-baselines.md`
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

- Implementation targets the pinned baseline in `docs/baselines/firefox-baselines.md`; baselines change
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
