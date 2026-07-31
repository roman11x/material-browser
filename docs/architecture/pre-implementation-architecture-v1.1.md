# Pre-Implementation Architecture Pack v1.1

Status: revision of v1 for joint review, incorporating the peer-review corrections. No implementation has started. Accepted material from v1 is preserved; every change is listed in the change log and mapped in "Peer-review corrections resolved" at the end.

## Source-verification metadata

All source claims in this document were re-verified this session against the branches below. Every source-reading document produced by this project must begin with a block in this form (one per branch when comparing).

```text
Repository:        github.com/mozilla-firefox/firefox
Branch:            esr153
Commit:            f815328b072efaaca17d1ae7c7c1756cdab9f7a0
Firefox version:   153.1.0 (browser/config/version.txt) / 153.1.0esr (version_display.txt)
Date inspected:    2026-07-31
Artifact/full build: none (remote source inspection only)
Relevant feature preferences: urlbar "smartbar" result-group mode observed in UrlbarPrefs.sys.mjs (placeholder config); exact selection pref to be pinned in Spike B

Branch:            esr140          (transition/reference)
Commit:            a8effc86687c00dcc71cf24d6bc26f66117e85df
Firefox version:   140.14.0 / 140.14.0esr
Date inspected:    2026-07-31

Branch:            main            (forward-looking observation only)
Commit:            fdd583cd5a10d051053acda8b760c3bd5d800034
Firefox version:   155.0a1
Date inspected:    2026-07-31
```

Branch roles (ADR-0002): `esr153` = implementation and release baseline · `esr140` = comparison point for the Nova/Smartbar transition, rebase-rehearsal material, and inter-ESR-delta evidence · `main` = forward observation, never a version label ("main is version X" claims are prohibited; versions come from `version.txt` at a recorded commit).

## Change log v1 → v1.1

1. Release baseline moved from ESR 140 to **ESR 153**; all paths, spikes, issues, risks, and status classifications rebased and **re-verified against `esr153`** (peer-review §1, §8).
2. **F1 rewritten**: URL-bar frontend contract *migration*, not module disappearance; v1's moz.build-only inference acknowledged as methodologically wrong and replaced with a multi-source verification rule (§2).
3. Spike B rewritten around the esr153 transitional architecture (controller + content-module input/view + Smartbar + Nova manifests), with the seven added questions and a provider-level minimal experiment (§3).
4. Policies/profile model corrected: `policies.json` is distribution-scoped, not profile-scoped; Artifact-vs-full-build reasoning corrected ("baked policies" removed as a full-build justification) (§4).
5. Reduced-transparency: the browser's own setting is authoritative; system signals optional and verified-before-use; no silent global enabling of experimental platform prefs (§5).
6. RFC contrast methodology now models full per-component compositing stacks; backdrop fixtures justified; light and dark saturated fixtures added (§6).
7. Screenshot regression demoted from settled mechanism to open spike; token snapshots + computed-style assertions are the CI baseline (§7).
8. Spikes A and C rebased on esr153 with the expanded sidebar/tab-group inspection lists (§8).
9. Issue set resequenced to the 15-issue M0/M1 plan, including the new baseline-recording issue (§9).
10. Source-verification metadata block added and mandated (§10).
11. New sections: "Changed conclusions after ESR 153 inspection" and "Peer-review corrections resolved".

## Finding F1 (corrected)

> **F1 — The URL-bar frontend contract is actively migrating upstream.**
> ESR 140 uses the older system-module arrangement (`UrlbarController/View/Input.sys.mjs` all as system modules — verified on `esr140`). ESR 153 already carries the transitional Nova/Smartbar architecture alongside legacy-compatible pieces — verified on `esr153`: `UrlbarController.sys.mjs` and `UrlbarMuxerStandard.sys.mjs` remain system modules, while the input and view are repackaged as browser-content modules (`content/UrlbarInput.mjs`, `content/UrlbarView.mjs`, plus `content/UrlbarResult.mjs`, `content/UrlbarEventBufferer.mjs`, `content/UrlbarShared.mjs`, `content/SearchModeSwitcher.mjs`, all listed in `jar.mn`), Smartbar components exist (`content/SmartbarInput.mjs`, `content/SmartbarInputController.mjs`, `content/SmartbarInputUtils.mjs`, `SmartbarMentionsPanelSearch.sys.mjs`), Nova test manifests (`browserNova.toml`) run alongside the default suite, and `UrlbarPrefs.sys.mjs` contains a "smartbar" result-group mode explicitly marked as a temporary placeholder. Current `main` separates controller responsibilities further (`UrlbarParentController.sys.mjs`; the monolithic controller module is gone there). The **provider layer remains recognizable and stable across all three branches** (`UrlbarProvidersManager`, `UrlbarProviderOpenTabs`, `UrlbarProviderPlaces`, `ActionsProviderQuickActions` verified on `esr153`), but the input/view/controller integration contract is **not stable across ESR generations**.

**Verification rule (replacing v1's flawed inference):** absence from one `moz.build` list is never proof a component is gone. Existence and packaging are established from, jointly: `moz.build`, `jar.mn`, `content/` listings, imports and call sites, test manifests, and generated/packaged module locations. (v1's claim that the input/view "are gone on main" failed this rule — they are packaged via `jar.mn` as content modules, which the moz.build check could not see.)

---

## Section 1 — Architecture Decision Records

Format unchanged. ADR-0001, -0004…-0010 carry over from v1 with edits noted; ADR-0002, -0003, -0006 are revised.

**ADR-0001 — Upstream base: raw Mozilla Firefox source.** *(unchanged from v1)*
Base directly on `mozilla-firefox/firefox`; full ownership of the token layer and frontend architecture; we owe upstream-maintainability discipline.

**ADR-0002 — Release baseline: ESR 153, with recorded revisions. (revised)**
Context: Firefox 153 is the new ESR generation (`esr153` verified: 153.1.0esr). ESR 140 remains supported through the transition window but is not the correct baseline for a project that has not begun implementation. Decision: implementation and release patches target `esr153`; `esr140` is retained as the Nova/Smartbar comparison point, rebase-rehearsal material, and inter-ESR-delta evidence; `main` is forward observation only. Every source spike records exact commit hashes for all inspected branches and the browser version from `browser/config/version.txt` (or equivalent authoritative source) at that revision; train numbers are never used as repository identifiers. The concrete baseline for this pack is the metadata block above. At release-branch creation time, the then-current supported ESR is re-verified — no ESR number is permanent in architecture documents. Consequences: the 140→153 delta observed this session (F1, sidebar test restructuring) doubles as free evidence of per-generation rebase cost; the first *forward* transition (153→next) inherits v1's scheduled-maintenance-exercise treatment: record conflicts, measure effort, identify high-churn patches, improve isolation before public release.

**ADR-0003 — Build strategy: Artifact Mode for frontend development; corrected full-build boundary. (revised)**
Decision: Artifact Mode (against the pinned `esr153` revision) is the default for all chrome JS/CSS/XHTML work. A full build is required for: final application identity, branding configuration, compiled distributables, and meaningful Widevine Case B testing. A full build is **not** required merely to test `policies.json`: distribution-scoped policy (a `distribution/` directory alongside the application) can be exercised against a development or repackaged application independently. Consequences: the full-build track's charter (#14) lists only genuine full-build needs; policy testing moves earlier and cheaper (#4).

**ADR-0004 — Vertical tabs: native sidebar implementation first.** *(decision unchanged; evidence rebased)*
Verified on `esr153`: `browser/components/sidebar/` with `browser-sidebar.js`, `sidebar-main.mjs` (18 vertical-tabs references), `SidebarManager.sys.mjs`, `SidebarTreeView.sys.mjs`. Spike A (Section 3) is the gate; custom rail only with a documented technical limitation; "easier to rewrite" remains insufficient.

**ADR-0005 — Workspaces: substrate deliberately undecided pending source inspection.** *(unchanged; Spike C rebased on esr153, where tab-group behaviour has evolved and must be re-inspected rather than inherited from 140-era conclusions)*

**ADR-0006 — Launcher: alternate presentation layer over Firefox query infrastructure, frontend-agnostic. (revised)**
Architecture: `Firefox query/provider infrastructure → BrowserQueryAdapter (ours) → centred launcher (ours)`. Upstream systems kept intact as in v1 (classification, engines/aliases, suggestions, open-tab switching, Places, extension results, ranking, navigation actions); we own presentation, grouping, selection, keyboard input, activation lifecycle, command and workspace providers, error presentation, and styling. New constraints from F1: **the launcher must not know which upstream URL-bar frontend (legacy-compatible or Nova/Smartbar) is active**; that knowledge is confined to `BrowserQueryAdapter`, which may carry ESR-specific internal implementations if unavoidable. The adapter preferentially binds at the provider level (`UrlbarProvidersManager`), the layer verified stable across esr140/esr153/main, rather than to either stock view. We do not begin by cosmetically relocating the stock address bar if that binds us to a frontend Mozilla is replacing — Spike B decides the exact binding with evidence.

**ADR-0007 — Launcher activation contract (firm).** *(unchanged from v1: guard → capture → close → execute/hand off → focus; validate-first only where required; sync failure stays open with inline error; async post-handoff failure surfaces via browser notifications, never reopens; the six required tests stand.)*

**ADR-0008 — Theming: manual DMS synchronization through a semantic token layer.** *(unchanged from v1)*

**ADR-0009 — Widevine: Case A / Case B split.** *(unchanged from v1)*

**ADR-0010 — Desktop integration: standard Wayland + XDG portals only.** *(unchanged from v1)*

---

## Section 2 — Mockup Addendum Specification

**Carried over from v1 unchanged** (accepted in peer review: security-chip state model 2.1, keyboard-focused rail 2.2, Compact/Docked/Overlay 2.3, monograms/icons 2.4, bottom utility cluster 2.5, Ctrl+L state 2.6, command result 2.7, inline error 2.8) — with one revision:

**2.9 Solid / reduced-transparency mode (revised).** The frames are unchanged (all glass alphas to 100% on equivalent ladder tones, blur removed, hairlines retained). The *trigger model* is corrected per peer-review §5: the browser's own "Reduce transparency" setting is the authoritative switch and always works; forced-colors/high-contrast always disables unsafe glass; a system reduced-transparency signal is honoured only where verified to function in our build and Fedora/Wayland environment. The design never depends on `prefers-reduced-transparency` being available, and no experimental platform preference is silently enabled globally to theme our chrome (side effects on web content must be assessed first). Full behaviour in RFC §16.

---
## Section 3 — Milestone 1 Source-Reading Plans (rebased on esr153)

All paths verified on `esr153 @ f815328b` this session (✓); `esr140`/`main` cited only for deltas. Spike documents must open with the Source-verification metadata block.

### Spike A — Native vertical tabs / sidebar (esr153)

**Verified entry points (`esr153`):** `browser/components/sidebar/browser-sidebar.js` ✓ · `sidebar-main.mjs` ✓ (vertical-tabs handling confirmed by content inspection) · `SidebarManager.sys.mjs` ✓ · `SidebarTreeView.sys.mjs` ✓ · `browser/components/tabbrowser/content/{tabbrowser,tabs,tab}.js` ✓ · `browser/themes/shared/sidebar.css` ✓ · `browser/base/content/browser.xhtml` ✓. Delta note: `SidebarCollapsedWindows.sys.mjs` exists on `main` but **not** on `esr153` (verified 404) — the sidebar is still evolving; the spike documents esr153 as-is and lists main-only modules as ESR-next watch items. (This also corrects one peer-review detail: that module is not part of the esr153 surface.)

**Questions (v1's nine, retained, now answered against esr153) plus the expanded inspection list:**

10. Does the newer sidebar already supply **stable top and bottom slots** usable for the workspace pill area and utility cluster without structural patches?
11. What is the current hover-expansion implementation and its configurability?
12. Right placement: first-class or mirrored?
13. Keyboard navigation: what exists natively; what does D2's shortcut model still need?
14. What do the **sidebar tree/view abstractions** (`SidebarTreeView.sys.mjs`) own, and are they the sanctioned extension surface for custom areas?
15. What is the **migration path away from legacy sidebar tests** (the legacy test split observed upstream), and does it signal imminent removal of anything we'd depend on?

**Minimal experimental patch, acceptance criteria, fallback triggers, and rebase-documentation output: as in v1**, executed against the pinned esr153 revision; the touched-file list additionally marks any file that differs from `esr140` (early churn signal) and any `main`-only sibling.

### Spike B — URL-bar / Nova / Smartbar infrastructure (esr153) — rewritten

**Verified entry points (`esr153`):**

- System modules: `UrlbarController.sys.mjs` ✓ · `UrlbarProvidersManager.sys.mjs` ✓ · `UrlbarMuxerStandard.sys.mjs` ✓ · `UrlbarPrefs.sys.mjs` ✓ (contains `makeSmartBarGroups` and a `"smartbar"` result-group mode marked "temporary placeholder until smartbar gets its own config") · `SmartbarMentionsPanelSearch.sys.mjs` ✓.
- Browser-content modules (per `jar.mn` ✓): `content/UrlbarInput.mjs` ✓ · `content/UrlbarView.mjs` ✓ · `content/UrlbarResult.mjs` · `content/UrlbarEventBufferer.mjs` · `content/UrlbarShared.mjs` · `content/SearchModeSwitcher.mjs` · `content/SmartbarInput.mjs` ✓ · `content/SmartbarInputController.mjs` · `content/SmartbarInputUtils.mjs`.
- Providers (stable layer): `UrlbarProviderOpenTabs.sys.mjs` ✓ · `UrlbarProviderPlaces.sys.mjs` ✓ · `ActionsProviderQuickActions.sys.mjs` ✓ (command-provider model).
- Test manifests: default suite plus `browserNova.toml` variants across `tests/browser-*` (dual-frontend tracking, verified in `moz.build`).
- Deltas: `esr140` has the input/view as system modules (comparison reading for transition understanding); `main` has `UrlbarParentController.sys.mjs` and no monolithic controller (ESR-next preview).

**Questions — v1's set rebased, plus the seven peer-review additions:**

1. *(v1 Q1–7 retained, re-asked against the esr153 arrangement: controller↔content-module contract; query lifecycle and stale-result guards; activation paths for URL, search, tab-switch, and action results and where close-then-execute hooks; restriction-token/search-mode mechanism — note `SearchModeSwitcher.mjs` is now a distinct content module; provider interface per `ActionsProviderQuickActions`; heuristic result for Ctrl+L; hide-vs-remove of the stock urlbar element.)*
2. Is **Nova expected to be default or permanent** within ESR 153's lifespan? (Evidence: the frontend-selection pref/mechanism, its default value at the pinned revision, and upstream signals; the `UrlbarPrefs` "smartbar" placeholder suggests active flux.)
3. Should `BrowserQueryAdapter` target the **Nova contract, the legacy contract, or the lower provider-level contract shared by both**? (ADR-0006 predisposes provider-level; the spike must confirm feasibility with evidence, not preference.)
4. Can the launcher be a **new frontend surface over `UrlbarProvidersManager`**, avoiding dependence on either stock view?
5. Which **result and query objects cross process boundaries**, and what are the parent/content responsibilities of the split (`.sys.mjs` vs `content/*.mjs`)?
6. Which **activation operations require a live stock input or view** (and would therefore force a hidden-but-functional stock element as backend)?
7. Can **one adapter support both frontend modes** during development tests (Nova on/off), so our test matrix survives upstream default flips?
8. Which architecture is **most likely to survive the next ESR**, given `main`'s parent/child controller split?

**Minimal experiment (revised, provider-level):**

```text
Launcher test surface (bare centred panel, Ctrl+K)
    ↓
project-owned query adapter (BrowserQueryAdapter skeleton)
    ↓
shared Firefox provider/ranking layer (UrlbarProvidersManager on esr153)
    ↓
structured results logged or rendered (no styling, no activation)
```

Run with the Nova mechanism in both states if togglable. Explicit non-goal: relocating or restyling the stock address bar — no binding to a frontend Mozilla is replacing.

**Acceptance criteria:** the M2 design note specifies the adapter interface, the chosen binding level with evidence for Q3, the hide-vs-remove strategy, the activation hook satisfying ADR-0007, the prefix mechanism decision, process-boundary notes (Q5), and both-mode test feasibility (Q7) — each with esr153 file/class references and the metadata block.

**Fallback ladder:** provider-level binding unworkable → controller-level binding with the adapter absorbing controller-contract churn → (last resort) hidden stock input as backend, documented as deliberate. Forked providers remain prohibited at every rung.

**ESR-rebase documentation output:** adapter-touched upstream API list; F1 watch items (Nova default status, `UrlbarParentController` arrival, "smartbar gets its own config" landing); the adapter named as sole intended blast radius.

### Spike C — Tab visibility, tab groups, and SessionStore (esr153) — expanded

Entry points: `tabbrowser.js` ✓, `tabgroup.js` ✓, `tabs.js` ✓, plus SessionStore integration points found from their call sites. v1's brief (hide/show APIs, session semantics, factual memo inputs) stands, expanded per peer review — esr153 tab-group behaviour is inspected fresh, with **no conclusions imported from the 140-era reading**:

persistence model · saved/closed groups · collapsed-group accessibility · hover previews · active-tab handling on collapse/switch · split-view interaction if present · rendering inside vertical tabs · movement between windows · pinned-tab interactions · hidden-tab SessionStore records · private-window behaviour.

Output feeds the Section 4 memo unchanged in structure; hard gates (tab-loss paths, private-state persistence, rail compatibility) unchanged.

---

## Section 4 — Workspace Substrate Options Memo (framework only)

**Carried over from v1 unchanged** — options O1–O4, the twelve comparison dimensions, evidence-per-cell scoring, and the three hard veto gates — with one revision: all evidence now comes from the esr153 Spike C, and any cell whose answer differs between esr140 and esr153 records both values (inter-generation drift in tab-group behaviour is itself maintenance evidence for the "upstream maintenance" dimension). Recommendation still deferred until after inspection.

---
## Section 5 — DMS Token Mapping RFC, Draft 0.2

Carried over from v1 draft 0.1 with three revised sections. Unchanged and still standing: §1 scope, §2 ⛔, §3 normalization, §4 taxonomy, §5 ⛔, §6 derivations, §7 alpha/glass, §8 accent (⛔ ordering), §9 state layers, §11 adjustment hierarchy, §12 fallback/LKG, §13 ⛔, §14 versioning/errors, and the §15 provisional palette table and token-snapshot method.

### RFC §10 — Contrast validation (revised)

Numeric minima unchanged from draft 0.1. The compositing methodology is corrected to model the **complete per-component layer stack**, not nominal foreground-vs-panel pairs:

```text
Launcher:      webpage → launcher scrim → translucent launcher surface → text/icon
Overlay rail:  webpage → translucent rail surface → text/icon
Security chip: webpage → optional local scrim → chip surface → security text/icon
```

Each translucent component declares its stack; validation composites every layer at configured alpha in order, then measures foreground contrast against the true composite. Docked/compact rail surfaces over reserved space composite against the window backdrop token instead of webpage fixtures.

**Backdrop fixtures, with rationale (each exists to defeat a specific failure mode):**

- white `#ffffff` — worst case for light-on-dark glass text (washes dark surfaces bright);
- black `#000000` — worst case for scrim/shadow-dependent separation and any dark-on-dark drift;
- mid-gray `#808080` — minimizes composite distance from mid-tone surfaces, the hardest case for hairlines and secondary text;
- saturated dark `#0a3cff`-class — hue-shifts translucent tints; catches chroma interactions the neutrals miss;
- saturated light `#ff3b30`-class — combines luminance pressure with hue shift; catches accent-on-glass collisions.

A pairing passes only if it passes on **all** fixtures. Blur is excluded from the model and may never be relied on to improve contrast: contrast must pass without it. Security chip and authentication/security warnings remain solid whenever their glass stack cannot guarantee readability — they do not descend the §11 ladder (unchanged rule, restated with the stack model).

### RFC §15 — Test fixtures, reference palette, and regression method (revised)

Palette fixtures and the provisional OKLCH reference table: unchanged from draft 0.1.

**Regression testing, two settled layers + one open spike:**

- *Settled, CI baseline:* (a) **token snapshot tests** — adapter output per fixture diffed against stored JSON snapshots; (b) **computed-style assertions** — browser-chrome tests asserting that rendered components carry the expected token-derived computed styles (colour, alpha, radius) in defined states. Both run everywhere, deterministically.
- *Open spike (not settled):* the screenshot/pixel-diff mechanism. The spike must: investigate current Firefox browser-chrome screenshot and visual-regression facilities and prefer existing Mozilla infrastructure when suitable; otherwise define deterministic capture with documented font, scaling, compositor, and animation controls. **No promise of stable raw pixel diffs across arbitrary Fedora/Niri/Hyprland environments.** Structural browser-chrome tests are separated from optional visual-regression tests; screenshot comparisons, if adopted, run only in a controlled reference environment. Until the spike concludes, screenshots are documentation aids, not gates.

### RFC §16 — Accessibility (revised)

- **Authoritative control:** the browser's own **Reduce transparency** setting → Solid chrome, always, unconditionally functional.
- **Forced-colors / high-contrast:** always disables unsafe glass; token layer defers to system colours (unchanged).
- **System signals:** a system reduced-transparency indication is honoured *only where verified* to work in the selected build and Fedora/Wayland environment; the design never depends on `prefers-reduced-transparency` availability; no experimental web-platform preference is enabled globally for chrome theming without a documented assessment of website side effects.
- No-information-by-colour-alone and `prefers-reduced-motion` handling: unchanged from draft 0.1 (motion handling may likewise fall back to a browser-owned animation-level setting where the system signal is unavailable).

---

## Section 6 — Initial Repository Issue Set (resequenced)

Template unchanged (Goal · Scope · Non-goals · AC · TE · Doc · Dep · MN). Renumbered per peer review; content revised where corrections apply, otherwise carried from v1.

### Milestone 0

**#1 — Repository and ADR bootstrap** — as v1 #1, committing the v1.1 ADRs.

**#2 — Approved mockup import** — as v1 #2 (baseline immutable; addenda extend).

**#3 — Record exact Firefox baselines (new)**
Goal: pin the source-of-truth revisions. Scope: record `esr153` implementation commit, `esr140` comparison commit, `main` observation commit, with `version.txt` values and dates, as `docs/baselines.md` in the metadata-block format; all spike docs must reference it. Non-goals: any checkout yet. AC: this pack's metadata block committed and designated canonical; update procedure defined (baselines move only by recorded decision). TE: n/a. Dep: #1. MN: prevents "main is version X" drift permanently.

**#4 — Distribution-scoped extension-policy test + isolated profile (revised)**
Goal: *test a dedicated browser installation or development package with a distribution-scoped policy, launched against an isolated development profile.* Scope: `policies.json` placed in `<browser-installation>/distribution/` (or a system policy location when deliberately testing system-wide deployment) — never in the profile; profile holds prefs, session data, extension user state, test bookmarks/history, workspace test state, nothing more. `ExtensionSettings` with the two verified IDs, `normal_installed`, AMO `install_url` copied at commit time with retrieval date. Non-goals: `force_installed`; Dark Reader; final branding. AC: about:policies shows the policy active from the distribution scope; both extensions auto-install, function, and remain user-disableable; profile demonstrably contains no policy file. TE: as v1 #3 plus a listing showing policy location. Doc: corrected profile-vs-distribution guide. Dep: none. MN: policy schema drift at ESR transitions remains watch item R9; distribution-dir packaging into a dev application is testable without a full build (ADR-0003).

**#5 — Stock Firefox Wayland baseline under Niri and Hyprland** — as v1 #4.

**#6 — Widevine stock control** — as v1 #5.

**#7 — ESR maintenance/rebase scaffolding** — as v1 #6, with the conflict-log pre-entry updated to the corrected F1 (expected 153→next churn: Nova default flip, parent/child controller split arrival).

### Milestone 1

**#8 — Bootstrap Artifact Mode against the pinned esr153 revision** — as v1 #7, targeting `esr153 @ f815328b` (or the then-current recorded baseline per #3); doc header carries the metadata block.

**#9 — Hello-chrome edit/test loop** — as v1 #8, noting the urlbar test run must name whether it executed the default or `browserNova` manifest.

**#10 — ESR 153 native-sidebar spike** — executes Section 3 Spike A.

**#11 — ESR 153 URL-bar/Nova/Smartbar spike** — executes Section 3 Spike B.

**#12 — ESR 153 tab visibility, tab groups, and SessionStore spike** — executes Section 3 Spike C.

**#13 — Artifact-build Widevine Case A** — the Case A half of v1 #12: repeat #6's protocol on the Artifact Mode build; diff appended to `docs/drm/spike.md`.

**#14 — Full-build requirements and packaging-boundary document** — the boundary half of v1 #12, corrected per ADR-0003: enumerates what genuinely requires a full build (application identity, branding, compiled distributable, Case B) versus what does not (distribution-scoped policies, chrome work); creates the full-build track charter. Dep: #4, #8.

**#15 — Mockup addendum** — as v1 #13 (states 2.1–2.9, revised 2.9 trigger model).

Sequencing rule made explicit per peer review: **no development checkout of `esr140` is created as a working base** — no debt is manufactured that must migrate to `esr153`. `esr140` is read remotely or checked out read-only for comparison purposes only.

---

## Section 7 — Risk Register (revised rows only)

Structure and rows R3–R11 carried over from v1 unchanged (with R3's "measure at first transition" now referencing the 153→next transition, and the free 140→153 delta recorded as calibration data). Revised rows:

| # | Risk | L'hood | Impact | Early detection | Mitigation | Fallback |
|---|---|---|---|---|---|---|
| R1 | **URL-bar frontend contract migration (F1, corrected)** — input/view/controller integration contract unstable across ESR generations: esr140 system-module trio → esr153 controller + content modules + Smartbar + Nova manifests + placeholder config → main parent/child controller split | **H (in progress now; further change at ESR-next near-certain)** | H | Nova default status and the "smartbar gets its own config" landing watched on `main`; `browserNova` manifest evolution tracked; conflict-log pre-entry from #7 | Adapter binds at the provider level (verified stable across all three branches); launcher never knows the active frontend; adapter may carry ESR-specific internals | Adapter reimplemented per ESR generation if contracts diverge; launcher untouched; last-resort hidden stock input backend, documented |
| R2 | **Native-sidebar churn** — evolving on both sides of the ESR boundary (legacy test split on esr153; `SidebarCollapsedWindows` main-only) | M–H | M | Spike A touched-file list diffed at each ESR; main-only sidebar modules tracked as arrival watch items | as v1 | as v1 |

---

## Section 8 — Status Classifications (rewritten for esr153)

**Confirmed from the attached mockup:** unchanged from v1 (decision inventory; provisional palette).

**Confirmed from current Firefox source/documentation** (verified this session at the commits in the metadata block): `esr153` exists at 153.1.0esr; on it: `UrlbarController.sys.mjs`, `UrlbarProvidersManager.sys.mjs`, `UrlbarMuxerStandard.sys.mjs`, providers (`OpenTabs`, `Places`, `ActionsProviderQuickActions`), content modules per `jar.mn` (`UrlbarInput.mjs`, `UrlbarView.mjs`, `UrlbarResult.mjs`, `UrlbarEventBufferer.mjs`, `UrlbarShared.mjs`, `SearchModeSwitcher.mjs`, Smartbar trio), Nova test manifests, the `UrlbarPrefs` "smartbar" placeholder mode, the sidebar component (`browser-sidebar.js`, `sidebar-main.mjs` with vertical-tabs handling, `SidebarManager`, `SidebarTreeView`), tab groups (`tabgroup.js`), `Policies.sys.mjs`, `browser.xhtml`, shared sidebar/tabs CSS. Deltas: `esr140` carries the input/view as system modules; `main` (155.0a1) has `UrlbarParentController.sys.mjs`, no monolithic controller module, and `SidebarCollapsedWindows.sys.mjs` (absent from esr153). `policies.json` is application/distribution-scoped on Linux, not profile-scoped.

**Assumptions requiring a source spike:** provider-level adapter binding is feasible on esr153 (Spike B Q3–Q4); activation paths reachable without a live stock view, or a hidden input backend is tolerable (Q6); one adapter can serve both frontend modes in tests (Q7); native sidebar reaches the approved rail design on esr153, including slot suitability (Spike A Q10, Q14); esr153 tab-group/SessionStore semantics support loss-safe workspaces (Spike C — no 140-era conclusions imported); Artifact Mode suffices for M2–M8 (#14 verifies).

**Blocked on files from you:** unchanged — RFC §2, §5, §8 ordering, §13, real-palette fixtures; unblocked by the five fixture items.

**Decisions already approved:** v1's list, plus the peer review's accepted-portions list, as refined by v1.1's corrections.

**Decisions still open:** rail A/B (Spike A) · workspace substrate (Spike C + memo) · adapter binding level and hide-vs-remove (Spike B) · Nova-targeting posture within ESR 153's lifespan (Spike B Q2/Q8) · screenshot-regression mechanism (RFC §15 spike) · downloads-button idle visibility · ambient-chip opacity floor, ladder offsets, all ⛔ RFC content (fixtures) · next-ESR adoption timing (per ADR-0002 at branch time).

---

## Changed conclusions after ESR 153 inspection

1. **v1's F1 was overbroad and its method was flawed.** The input/view were moved and repackaged as browser-content modules, not removed; the corrected F1 stands above, with the multi-source verification rule replacing moz.build-only inference.
2. **The implementation baseline changed** from esr140 to esr153; v1's "trio exists on our baseline" conclusion no longer holds — on esr153 the controller is a system module while input/view are content modules, so Spike B's design and the adapter's binding-level question changed substantively (provider-level binding promoted from mitigation to default posture, ADR-0006).
3. **The transitional architecture is already in our baseline** (Smartbar modules, Nova manifests, placeholder config on esr153) — v1 treated Nova as an ESR-next event; it is partially a *now* event, which strengthens rather than weakens the frontend-agnostic launcher requirement.
4. **`ActionsProviderQuickActions` persists on esr153** — the command-provider model carries over intact.
5. **One peer-review detail corrected back:** `SidebarCollapsedWindows.sys.mjs` is `main`-only, not part of the esr153 sidebar surface; it is tracked as an arrival watch item, not a current dependency.
6. **"Baked policies" was wrongly listed as a full-build justification in v1**; distribution-scoped policy testing is build-independent (ADR-0003, #4, #14).

## Peer-review corrections resolved

| Correction | Resolved in |
|---|---|
| §1 ESR 153 baseline | Metadata block · ADR-0002 · Sections 3, 6 (#3, #7–#14), 7, 8 · Changed conclusions 2–3 |
| §2 F1 rewording + verification method | Finding F1 (corrected) · Changed conclusions 1 · verification rule |
| §3 Launcher spike reframed for esr153 | Section 3 Spike B · ADR-0006 · R1 |
| §4 Policies/profile/full-build model | ADR-0003 · Issues #4, #14 · Changed conclusions 6 |
| §5 Reduced-transparency authority | RFC §16 · Addendum 2.9 |
| §6 Compositing methodology | RFC §10 |
| §7 Screenshot regression as open spike | RFC §15 |
| §8 Sidebar/tab-group re-inspection on esr153 | Section 3 Spikes A and C · ADR-0004/0005 · Changed conclusions 5 |
| §9 Issue resequencing | Section 6 (15-issue set, no-esr140-checkout rule) |
| §10 Source-verification metadata | Metadata block (mandated for all spike docs) · Issue #3 |

---

*v1.1 ends here. No code has been written. Awaiting joint review; fixture files still unblock the ⛔ RFC sections independently.*