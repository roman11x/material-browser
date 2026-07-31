# Architecture Addendum v1.2

Status: targeted addendum to the accepted Pre-Implementation Architecture Pack v1.1. Only additions and revisions appear here; unaffected v1.1 sections are not repeated. No code has been written.

Source-verification note: new entry points cited below were verified against `esr153 @ f815328b072efaaca17d1ae7c7c1756cdab9f7a0` (153.1.0esr) this session, per the v1.1 metadata block and the multi-source verification rule. One candidate path did not verify and is marked accordingly.

## 1. Change summary

1. Provider-stability wording downgraded from established fact to tested hypothesis; Spike B gains a cross-revision contract-comparison mandate (→ §2).
2. New ADR-0011: extension actions without a traditional toolbar; new Spike D with verified esr153 entry points (→ §3, §4).
3. New Spike E: browser-chrome glass and compositor feasibility, with its own fallback hierarchy and risk (→ §5).
4. Sensitive-field detection constrained to trusted existing signals; new investigation (#18) with verified entry points; no content-DOM scanning from chrome (→ §6).
5. Mockup addendum extended with extension-action frames (→ §7).
6. Issues #16–#18 added; #8 gains Artifact Mode revision-compatibility acceptance criteria; #15 gains the extension frames; #3 gains local reproducible baseline verification (→ §8).
7. Risks R12–R15 added (→ §9).
8. Status classifications updated; readiness classification added (→ §10, end).

## 2. Revised provider-stability wording (replaces the claim in F1's closing sentence, ADR-0006, and R1)

Adopted wording, verbatim where the old claim appeared:

> The provider and ranking layer is the most promising available integration seam because its major modules remain recognizable across the inspected branches. Its API stability is not yet established and is an explicit output of Spike B.

**Spike B mandate extension.** Compare the provider-level contracts at the pinned `esr140`, `esr153`, and `main` revisions where practical, recording for each: constructor and registration model · query-context structure · provider start/cancel lifecycle · result object representation · muxer integration · listener/callback interface · activation data supplied by results · process ownership · material differences between revisions · **which subset can be wrapped behind `BrowserQueryAdapter`**. The adapter-wrappable subset is the spike's primary deliverable; presence-of-files is explicitly insufficient evidence of contract stability.

**ADR-0006 amendment.** Provider-level integration remains the preferred binding — now stated as *a hypothesis under test by Spike B*, not a verified stable boundary. The fallback ladder (controller-level binding → hidden stock input backend) is unchanged and becomes the live path if the hypothesis fails.

**R1 amendment.** Early-detection column gains: "Spike B cross-revision contract diff"; the mitigation column's "verified stable across all three branches" is replaced by "most recognizable seam; stability under test (Spike B)".

## 3. ADR-0011 — Extension actions without a traditional toolbar

**Status:** Accepted (direction); implementation mechanism gated on Spike D.
**Context:** the browser removes the ordinary navigation toolbar, but real WebExtensions expose browser/MV3 actions, page actions, popups, badges, per-tab enabled state, context menus, pinned state, permission warnings, and management controls. uBlock Origin (popup/dashboard, per-site toggle, element picker and zapper, logger, settings/filter lists, badge counts) and SponsorBlock (current-video status, popup and category controls, segment submission where supported, voting and skip controls, per-site/per-video behaviour) must remain **fully usable**, not merely active in the background.
**Decision:** host and invoke the real extension actions and popups — never reproduce their interfaces:

```text
Firefox WebExtension action infrastructure
        ↓
our extension-action host
        ↓
rail action buttons / extensions panel / launcher commands
```

The browser provides: an Extensions button in the bottom rail utility cluster; an extensions panel preserving Firefox permission and management semantics; optional pinned extension actions in compact and expanded rail; badge rendering; popup anchoring; keyboard access; context-menu access; a launcher result for finding and invoking extension actions; a reliable route to `about:addons`. The launcher may aid discovery but is never the only interface — popups need an understandable visual anchor and mouse users need a discoverable control.
**Constraints:** it is *not assumed* that stock popups can anchor to arbitrary custom chrome — Spike D verifies in source. A completely custom reimplementation of extension popups or state is not an acceptable outcome at any rung.
**Consequences:** the bottom utility cluster (Addendum 2.5) gains the Extensions entry as a first-class member; launcher gains an EXTENSION result group (guarded, §7); daily-driver readiness of the launcher is gated on Spike D (#16 dependency rule, §8).

## 4. Spike D — WebExtension action and unified-extensions infrastructure (esr153)

**Verified entry points (`esr153`):** `browser/base/content/browser-unified-extensions.js` ✓ (Unified Extensions panel logic) · `browser/components/extensions/ExtensionPopups.sys.mjs` ✓ (popup creation/anchoring) · `browser/components/extensions/parent/ext-browserAction.js` ✓ and `ext-pageAction.js` ✓ (action semantics, badges, per-tab state) · `browser/components/customizableui/CustomizableUI.sys.mjs` ✓ (widget placement and customization state — central to Q3) · `browser/themes/shared/addons/unified-extensions.css` ✓ (panel styling surface). Panel markup location: the guessed `unified-extensions-panel.inc.xhtml` does **not** exist (verified 404); the actual markup file is located via `browser-unified-extensions.js` references during the spike, not guessed.

**Inspection list:** Unified Extensions panel · browser/action widgets · page-action handling · popup creation and anchoring · badge rendering · context menus · pinned-state persistence · toolbar customization state · per-window and per-tab state · keyboard activation · private-window behaviour · extension permission messaging.

**Questions (verbatim from review):**
1. Can a browser action be hosted in a non-toolbar custom rail while preserving normal Firefox behaviour?
2. Can the standard popup anchor to a rail button?
3. Does moving an action widget preserve extension assumptions and customization state?
4. Can unpinned actions remain available through a restyled standard panel?
5. How are page actions handled when no editable address bar exists?
6. Can launcher results invoke actions that have no popup?
7. Can launcher results open actions that do have a popup, and where would that popup anchor?
8. How do badges and tab-specific action states update?
9. What would break uBlock Origin's popup, picker, logger, or SponsorBlock's popup?
10. Which upstream files would become rebase dependencies?

**Minimal experiment:** keep Firefox's action infrastructure intact; place one real extension action in a temporary rail host; open its genuine popup; verify badge, context menu, per-tab state, and keyboard activation; repeat with uBlock Origin and SponsorBlock; verify continued function after tab switching and in a second browser window.

**Required recommendation — exactly one of:** rehost standard widgets in the rail · keep a hidden/minimal standard action host and expose a custom proxy · retain and restyle the standard unified extensions panel · a documented hybrid. Custom reimplementation of popups/state is out of bounds.

**Rebase output:** dependency list per Q10, appended to the conflict-log watch set.

## 5. Spike E — Browser-chrome glass and compositor feasibility

**Premise (honest limitation of the mockup):** the HTML mockup proves the intended appearance inside one DOM. It does not prove that Firefox browser chrome can sample and blur *webpage content* across the real chrome/content compositing boundary. This is a distinct question from R6 (glass performance) and gets its own spike and risk.

**Timing:** Milestone 1, immediately after the hello-chrome patch (#9).

**Environments:** Fedora · Niri · Hyprland · native Wayland · hardware acceleration enabled · fractional scaling · light page · dark page · video playback · scrolling content.

**Test matrix (separately):** (1) translucent chrome surface without blur · (2) `backdrop-filter` or the current supported equivalent · (3) launcher with scrim · (4) overlay rail without full-window scrim · (5) Solid/reduced-transparency mode · (6) open/close animation · (7) fullscreen and picture-in-picture interactions.

**Determinations required:** does the chrome panel actually blur webpage content, or only other chrome layers? · do page and chrome live in compositing layers that prevent the intended effect? · do clipping and rounded corners work? · GPU and frame-time impact · behaviour during video · behaviour under fractional scaling · Niri vs Hyprland differences · visual artifacts, excessive power use, or input-region problems.

**Fallback hierarchy (design remains valid at every rung):**
1. Genuine chrome-over-content blur.
2. Translucent tinted surface with local scrim, no blur.
3. Opaque tonal Material surface.
4. Experimental deeper compositor/platform work — only after the browser is otherwise usable, and desktop compositor rules are never part of the core solution.

**Consequences elsewhere:** RFC §7's presets map onto whichever rung the spike establishes (Frosted may compile to rung 2 on some stacks); RFC §10's blur-excluded contrast model already anticipates rungs 2–3, so no contrast rework is needed if blur fails; marketing/claims: frosted blur must not become a release claim until this spike resolves (#17 dependency rule, §8).

---
## 6. Sensitive-field detection: constrained mechanism + investigation (#18)

**Constraint adopted:** no arbitrary content-DOM scanning from browser chrome, and no broad page-inspection service is created for this animation state. The `sensitive` chip state (Addendum 2.1) keeps its approved *appearance*; its *trigger* is now mechanism-gated.

**Verified entry points (`esr153`):** `toolkit/components/passwordmgr/LoginManagerChild.sys.mjs` ✓ and `LoginManagerParent.sys.mjs` ✓ (password-field detection and its existing content→parent messaging) · `toolkit/components/formautofill/FormAutofillParent.sys.mjs` ✓ and `shared/FormAutofillHeuristics.sys.mjs` ✓ (credit-card/address field classification and autofill signals).

**Investigation scope:** existing password-field and Login Manager signals · existing credit-card/autofill signals · trusted focus/form-fill notifications crossing the content/chrome boundary · Fission/process-boundary implications · private browsing · sandboxed and cross-origin frames · false-positive/false-negative behaviour · whether a generic payment-field signal actually exists at all.

**Preferred hierarchy (adopted verbatim):**
1. Existing trusted Firefox security/autofill signal.
2. Narrow existing content-to-parent notification.
3. Explicitly limited support for password fields only.
4. Drop automatic sensitive-field triggering; retain manual/prominent origin access.

**Invariant:** the origin chip becomes prominent on navigation commit and registrable-domain change regardless of the sensitive-trigger outcome — the security floor does not depend on this investigation.

**Addendum 2.1 amendment:** the sensitive-state trigger description now reads "entered on a trusted platform signal per #18's outcome (at minimum password-field focus if rung 3; absent if rung 4)" instead of naming password/auth/payment focus as assumed capabilities.

## 7. Mockup addendum extension — extension actions (new frames; approved baseline untouched)

**Compact rail:** Extensions button in the bottom utility cluster (same 40×40/radius-12 language) · uBlock Origin pinned action with badge (count badge, 10px, error-container fill per uBO's blocking-count convention — badge *content* comes from the extension, only the badge *frame* is ours) · SponsorBlock pinned action · keyboard-focus state (focus-ring token) · update/permission-attention indicator (small accent dot at the button corner, distinct from the download badge position).

**Expanded rail:** pinned extension action rows — icon, name, badge, optional state text (e.g. per-site disabled) in on-surface-variant · Extensions button opening the full panel · context-menu state on an action row (standard menu, themed frame).

**Extension popup (hosting concept):** genuine popup anchored **beside the right rail** rather than the removed top toolbar; popup shows the extension's own content; the browser-owned frame/shadow may follow our theme; extension content itself is not recoloured or rewritten unless Firefox normally themes it. One frame each for a rail-button anchor and a panel-row anchor.

**Launcher — EXTENSION group example:**

```text
EXTENSION
uBlock Origin — Open controls
SponsorBlock — Open controls
Manage extensions
```

Guard: frames and copy must not imply that invoking every extension from the launcher is guaranteed until Spike D proves popup anchoring (Q7); "Open controls" rows render in the addendum with a footnote marker tied to #16.

## 8. Issue-set changes

**#3 (amended) — local, reproducible baseline verification.** The v1.1 remote-inspection hashes are preserved as the *inspection record*; they become *development baselines* only after verification in the actual local checkout with evidence equivalent to: `git rev-parse HEAD` · `git status` · `git branch --show-current` · `cat browser/config/version.txt` · `cat browser/config/version_display.txt`. Recorded per checkout: remote URL, branch, commit hash, version files, checkout date, tree-clean status. If the local tip differs from the recorded baseline, it is never silently replaced — a baseline-update entry explains the change.

**#8 (amended) — Artifact Mode availability acceptance criteria.** Acceptance now requires: checkout at the intended ESR 153 baseline with `git rev-parse HEAD` evidence · successful artifact discovery/download · record of the artifact source revision actually selected · confirmation that source and artifact revisions are compatible · successful `./mach build` · successful `./mach run` · successful relevant browser-chrome test. Strict pinning uses the supported revision mechanism with failures recorded honestly. If the chosen revision has no usable artifact: do not pretend the build is pinned; find the nearest suitable official artifact-producing revision; record both revisions; decide (recorded) whether to move the source baseline or perform a full build; never continue with unexplained source/binary mismatch (→ R14).

**#15 (amended)** — gains the §7 extension-action frames; the Spike-D guard footnote is part of acceptance.

**#16 — Extension-action and unified-extensions spike (new).** Executes Spike D (§4). Dep: #8, #9. MN: **the launcher is not daily-driver-ready until #16 resolves** — recorded as a hard gate on Milestone 9 entry, not on Milestone 2 development.

**#17 — Browser-chrome glass feasibility spike (new).** Executes Spike E (§5). Dep: #9. MN: **frosted blur must not become a release claim until #17 resolves**; RFC §7 preset compilation depends on the established rung.

**#18 — Security-chip signal investigation (new).** Executes §6. Dep: #8; may run independently of the visual addendum. MN: outcome selects the sensitive-trigger rung; chip prominence invariant unaffected.

## 9. Risk-register additions

| # | Risk | L'hood | Impact | Early detection | Mitigation | Fallback |
|---|---|---|---|---|---|---|
| R12 | **Extension action incompatibility** — removing/replacing toolbar surfaces makes extension buttons, popups, badges, page actions, or context menus inaccessible or unreliable | M | H (product-defining extensions) | Spike D with real uBlock Origin and SponsorBlock | Preserve standard Firefox action infrastructure; rehost or restyle, never reimplement | Retain a minimal standard-compatible extension panel/action host |
| R13 | **Browser-chrome glass infeasibility** — chrome may not blur web content across the real compositing boundary as the standalone mockup does, or costs are unacceptable | M | M (visual identity), L (function) | Spike E | Translucent and Solid designs are first-class modes; contrast model already blur-independent | Ship tonal transparency without blur, or fully opaque Material surfaces |
| R14 | **Artifact source/binary mismatch** — checkout and downloaded artifact diverge, causing crashes or invalid test conclusions (Mozilla explicitly warns of difficult-to-diagnose failures) | M | M–H (silently wrong conclusions) | Both revisions recorded per #8; clean startup + representative tests after every baseline move | Supported artifact-producing revisions only; clean objdir on baseline change | Move to a compatible revision or perform a full build |
| R15 | **Sensitive-field signal unavailable** — no reliable existing signal for generic authentication/payment-field focus | M | L–M (one chip state) | #18 | Trusted existing signals only; no DOM-scanning service | Password-fields-only, or omit automatic sensitive mode; navigation/domain prominence retained regardless |

## 10. Updated status classifications (deltas to v1.1 Section 8 only)

**Confirmed from current Firefox source** (additions, `esr153 @ f815328b`): `browser-unified-extensions.js`, `ExtensionPopups.sys.mjs`, `CustomizableUI.sys.mjs`, `ext-browserAction.js`, `ext-pageAction.js`, `unified-extensions.css`, `LoginManagerChild/Parent.sys.mjs`, `FormAutofillParent.sys.mjs`, `FormAutofillHeuristics.sys.mjs` all exist at the cited paths. Negative result recorded: `unified-extensions-panel.inc.xhtml` does not exist at the guessed path; panel markup located via call sites in Spike D.

**Downgraded from confirmed to hypothesis:** provider-layer *contract* stability (was overclaimed in v1.1's F1/ADR-0006; §2 wording governs).

**New assumptions requiring a source spike:** standard extension popups can anchor to rail-hosted buttons (Spike D Q2); action widgets can move without breaking customization state (Q3); page actions have a viable home without an editable address bar (Q5); chrome can blur web content across the compositing boundary (Spike E); a trusted generic payment-field signal exists (#18 — expected uncertain).

**Blocked on files from you:** unchanged (five fixture items; ⛔ RFC sections).

**Decisions still open (additions):** Spike D's hosting recommendation (four allowed outcomes) · Spike E's glass rung · #18's trigger rung · popup-anchor position finalization (beside-rail concept approved; exact geometry after Spike D).

## Merge guide — where v1.2 slots into v1.1

- §2 wording → replaces the final sentence of Finding F1, amends ADR-0006's binding rationale, and edits R1's detection/mitigation cells.
- §3 ADR-0011 → appended to v1.1 Section 1 after ADR-0010.
- §4 Spike D and §5 Spike E → appended to v1.1 Section 3 after Spike C.
- §6 → amends Addendum 2.1 (trigger sentence) and adds the investigation as a Section 3 companion item.
- §7 → appended to v1.1 Section 2 as new addendum frames under 2.5/2.10; #15 scope updated.
- §8 → v1.1 Section 6: #3, #8, #15 amended in place; #16–#18 appended to Milestone 1.
- §9 → appended to v1.1 Section 7 as R12–R15.
- §10 → deltas applied to v1.1 Section 8.
- Readiness classification below → appended after v1.1's closing section.

## Architecture readiness after v1.2

- **Launcher/query architecture** — *Ready for source spike* (Spike B; provider-contract stability is the open question). Implementation blocked on the spike.
- **Vertical tab rail** — *Ready for source spike* (Spike A). Design fixed; mechanism (native vs custom) blocked on the spike.
- **Workspaces** — *Blocked on source spike* (Spike C feeds the substrate memo; no design work proceeds past the memo framework).
- **Extension actions** — *Ready for source spike* (Spike D; ADR-0011 direction accepted, mechanism open).
- **Glass/frosted rendering** — *Blocked on source spike* (Spike E determines the achievable rung; translucent/solid designs proceed regardless).
- **Security chip: navigation/domain prominence** — *Ready for Milestone 0* (design and trigger fully specified; no upstream unknowns).
- **Security chip: sensitive state** — *Blocked on source spike* (#18 selects the trigger rung).
- **DMS token pipeline: structure, taxonomy, derivations, fallback, LKG** — *Ready for Milestone 0* (RFC drafted; adapter implementation gated on RFC approval per plan).
- **DMS token pipeline: source contract, mapping, light handling, real fixtures** — *Blocked on user fixture* (the five fixture items).
- **Screenshot/visual regression** — *Blocked on source spike* (RFC §15 spike); token snapshots and computed-style assertions *Ready for Milestone 0/1 CI*.
- **Extension policy + profile model** — *Ready for Milestone 0* (#4, distribution-scoped, build-independent).
- **Widevine Case A** — *Ready for Milestone 0* (#6). **Case B** — blocked on the full-build track by design (not a spike; a build prerequisite).
- **ESR baseline & rebase machinery** — *Ready for Milestone 0* (#3 local verification, #7 scaffolding).
- **Wayland/compositor integration baseline** — *Ready for Milestone 0* (#5).
- **Desktop-through transparency (real desktop behind chrome)** — *Deferred post-1.0* (unchanged from the original plan; Spike E explicitly excludes it from core).

*Addendum ends. This is intended as the final architecture-only pass; the next outputs are spike results, unless a spike exposes a fundamental contradiction.*