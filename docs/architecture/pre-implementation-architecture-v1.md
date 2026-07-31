# Pre-Implementation Architecture Pack v1

Status: for joint review. No implementation has started. Incorporates Architecture Review v1, your D1–D8 approvals and refinements, the launcher activation contract, and the workspace-memo directive.

**Source verification note.** Per your instruction, every Firefox source path in this pack was verified against the real `mozilla-firefox/firefox` repository (branch `esr140`, and `main` where noted) at the time of writing — not recalled from memory. Verification produced one major finding, flagged throughout as **[F1]**:

> **[F1] The classic URL-bar frontend is being replaced upstream.** On branch `esr140`, `UrlbarController.sys.mjs`, `UrlbarView.sys.mjs`, and `UrlbarInput.sys.mjs` all exist as expected. On `main`, all three are **gone**: the build file now lists `UrlbarParentController.sys.mjs`, `UrlbarMuxerStandard.sys.mjs`, and a `Smartbar…` module, and the entire urlbar test suite is dual-tracked as `browser-nova.toml` / `browser-proton.toml` — Mozilla is mid-migration to a new address-bar frontend. Consequence: the "URL-bar upstream churn" risk is **confirmed live, not speculative**. Our launcher work must target the `esr140` view/controller structure, and we should *plan* for a significant launcher-view rework at the next ESR generation rather than hope to avoid one. All launcher-adjacent decisions below are written with this in mind.

---

## Section 1 — Architecture Decision Records

Each ADR is recorded as: Status · Context · Decision · Consequences. All are **Accepted** per your review unless marked otherwise.

**ADR-0001 — Upstream base: raw Mozilla Firefox source.**
Context: full ownership of the visual token layer and frontend architecture; a derivative base (e.g. Zen) would interpose a second theme/maintenance layer we don't control. Decision: base directly on `mozilla-firefox/firefox`. Consequences: we own every rebase; we owe upstream-maintainability discipline (narrow patches, ADRs, conflict log).

**ADR-0002 — Release baseline: current Firefox ESR, re-verified at branch time.**
Context: ESR reduces rebase frequency to ~yearly; the currently verified ESR line is 140 (branch `esr140` confirmed to exist upstream), with mainline at 153 and a new ESR generation imminent. Decision: release patches target the current supported ESR, with the exact version re-checked immediately before the release branch is created — no ESR number is permanently encoded in architecture documents. Development may read `main` for foresight ([F1] came from exactly that), but release patches must apply and build against the selected ESR. Consequences: first ESR transition is treated as a scheduled maintenance exercise with recorded conflicts, measured effort, and isolation improvements before public release.

**ADR-0003 — Build strategy: Desktop Artifact Mode for frontend development.**
Context: all planned M2–M8 work is browser-chrome JS/CSS/XHTML. Decision: Artifact Mode is the default development mode; full builds are a separate parallel track required only for official branding, baked policies, and distributable binaries. Consequences: fast iteration now; Widevine Case B and packaging blocked on the full-build track by design (ADR-0009).

**ADR-0004 — Vertical tabs: native sidebar implementation first.**
Context: Firefox's sidebar component on `esr140` contains a maintained vertical-tabs implementation (verified: `browser/components/sidebar/` with `SidebarController.toggleVerticalTabs`, a `#vertical-tabs` element, and `sidebarVerticalTabsEnabled` state). Decision: spike the native implementation first; adopt it if the approved design (right side, ~64px compact / ~248px expanded, hover + keyboard expansion, frosted overlay, workspace area, accent bar, approved geometry, bottom cluster, reliable a11y) is reachable with a maintainable patch. A custom rail requires a **documented technical limitation**; "easier to rewrite" is explicitly insufficient. Consequences: we inherit upstream tab semantics, a11y, DnD, overflow, session handling, and bug fixes; we accept some styling constraint risk, resolved by the spike.

**ADR-0005 — Workspaces: substrate deliberately undecided pending source inspection.**
Context: our workspaces are independent active tab *sets*; native tab groups (verified present on `esr140`: `browser/components/tabbrowser/content/tabgroup.js`) primarily organize tabs within one visible strip. Decision: no substrate is chosen yet. An options memo (Section 4 framework) comparing custom state over `gBrowser`, show/hide layering, native tab groups as substrate, and a hybrid will be produced after the M1 source inspection. Consequences: `WorkspaceService` design waits; M1 inspection gains a defined third target (tab visibility + session-store semantics).

**ADR-0006 — Launcher: alternate presentation layer over Firefox URL-bar infrastructure.**
Context: Firefox's query stack (providers, ranking, Places, open tabs, search engines, extension results) must not be reimplemented; [F1] makes controller/view coupling a known churn surface. Decision: architecture is `Firefox providers & ranking → our launcher adapter/controller → our centred launcher view`. Upstream systems kept intact: URL classification, search aliases/engines, suggestions, open-tab switching, Places, extension results, ranking, navigation actions. We own: presentation, group rendering, selection, keyboard input, activation lifecycle, command provider, workspace provider, error presentation, Material/glass styling. Consequences: the adapter boundary is the designated absorption point for [F1]-class upstream refactors — when the view/controller side changes at ESR-next, the adapter is rewritten, not the launcher.

**ADR-0007 — Launcher activation contract (firm).**
Decision, on Enter with a valid selection: (1) duplicate-activation guard engages, (2) intended action is captured, (3) launcher closes immediately, (4) action executes or is handed off, (5) focus transfers appropriately. Validate-before-close is permitted only where the action requires it (e.g. palette parsing) — validate, then close, then apply. Synchronous failure: stay open, preserve input, inline error. Post-handoff asynchronous failure: never reopen the launcher; surface via the standard browser notification mechanism. Required tests: single-Enter-single-execution; stale query callbacks cannot alter selection after activation; close cancels/detaches the live view query safely; Enter on an open-tab result focuses that tab; Shift+Enter performs the new-tab action and closes; mouse activation follows the identical close contract.

**ADR-0008 — Theming: manual DMS synchronization through a semantic token layer.**
Decision: `dank-pywalfox.json → palette reader/validator → semantic --browser-* tokens → all chrome`. No file watching, no automatic recolour, no component reads raw DMS keys, no DMS colours injected into websites. The mockup's OKLCH values are the built-in provisional dark palette and visual-regression fixture — explicitly *not* the DMS schema. Consequences: RFC (Section 5) governs mapping; fixture-dependent sections blocked on your palette files.

**ADR-0009 — Widevine: Case A / Case B split.**
Decision: Case A (development & personal-build feasibility) begins in M0–M1 across stock-control, clean-profile, Artifact-Mode, Fedora/Wayland, with Niri and Hyprland where compositor behaviour may matter. Case B (rebranded distributable feasibility) begins once a full branded build exists and must separately report CDM download, CDM initialization, EME support, legal-test-content playback, representative commercial-service playback, and distribution/licensing status. Case B alone controls public DRM claims. Prohibited: copied proprietary binaries, identity spoofing, undocumented workarounds.

**ADR-0010 — Desktop integration: standard Wayland + XDG portals only.**
Decision: no core dependency on `hyprctl`, Niri IPC, X11 tools, GNOME Shell or KDE APIs, or compositor-specific positioning. One browser for Niri and Hyprland; compositor-specific extras, if ever, are optional and post-1.0.

---

## Section 2 — Mockup Addendum Specification

Scope: only the missing or corrected states. The approved main mockup is untouched; every addendum state reuses its existing vocabulary (hue-292 tonal ladder, 3px accent indicator, 40px icon buttons, hairlines at 6–12% white, radii 9–20, Inter). Values marked *(provisional)* are placeholders until the RFC's contrast rules finalize them; geometry is binding, opacity numbers are not.

### 2.1 Security chip states

Same geometry as approved (top-left pill, lock glyph + registrable domain). Four states:

- **prominent** — full opacity; backing at ≥ launcher-panel solidity *(provisional: 92% alpha)*; entered on navigation commit and registrable-domain change; holds for a short dwell *(provisional: 3s)* then decays to ambient.
- **ambient** — restrained but readable *without hover*; resting opacity is a token defined by the RFC's composited-contrast rule, not a fixed 40%; hover/focus still elevates to full.
- **sensitive** — as prominent, plus a subtle accent ring *(1px, --browser-focus-ring)*; entered while a password, authentication, or payment-related field holds focus; persists for the duration of focus; no decay.
- **warning** — solid or near-solid error-container surface with on-error-container text; covers insecure origin, certificate errors, dangerous permissions; **never glass, never decays, never hover-gated**.

Addendum must show all four, plus the transition prominent→ambient.

### 2.2 Keyboard-focused rail

Rail in expanded state with a visible focus ring (--browser-focus-ring, 2px, radius following the row) on one workspace pill and, in a second frame, on one tab row. The focused row shows the same tonal fill as launcher selection, establishing that keyboard focus and launcher selection share one visual grammar. An ESC-style hint chip may appear at the rail's top edge while rail-focus mode is active.

### 2.3 Rail modes

Three frames: **Compact** (≈64px reserved; page content ends at the rail; hover/keyboard expansion overlays the page — this is the approved mockup, now with content actually reserving space), **Docked** (≈248px reserved, always expanded, no overlay behaviour), **Overlay** (0px reserved; rail absent at rest; slides over content on edge-hover or shortcut, full frosted treatment).

### 2.4 Workspace monograms and icons

Pills at approved geometry showing the three identity forms: user-selected icon; icon with colour role applied to the pill container; generated two-character monogram fallback ("Mu" / "Ma" resolving the Music/Manga collision). One frame shows the tooltip with the full workspace name; launcher workspace results always render full names.

### 2.5 Bottom rail utility cluster

Anchored at the rail's bottom edge, mirroring the launcher button's language (40×40, radius 12, tonal container on hover/active). Items: downloads, DMS sync, menu/settings. Required frames:

- **Collapsed rail**: three stacked icon buttons.
- **Expanded rail**: same buttons gain 13px labels, row-style like tabs.
- **Download activity**: downloads button with a thin circular progress ring in --browser-primary; badge dot for completed-unseen. Whether the button hides when idle is left to interaction design — show both variants.
- **DMS sync success**: brief check-glyph swap + one accent pulse *(≤300ms, animation-level respecting)*.
- **DMS sync failure**: error-container fill on the button + small inline toast anchored above the cluster ("Palette invalid — kept current theme"), reusing launcher-error styling (2.8).
- **Hover and keyboard focus**: hover = 5.5% wash (existing rule); focus = focus-ring token, consistent with 2.2.

### 2.6 Ctrl+L launcher state

Approved launcher frame with the full current URL populated and fully selected (selection colour from the mockup's ::selection value, tokenized). Results reflect URL-editing context (heuristic navigate result first). Everything else identical to Ctrl+K.

### 2.7 Command result

Launcher showing a `> ` query with a COMMANDS group: rows use a chevron-terminal glyph in place of favicon, command name at 13px, optional shortcut hint right-aligned in the ESC-chip style. Selection identical to other rows.

### 2.8 Inline launcher error

After a failed synchronous activation: input preserved, selection preserved, and a single-line error strip between input row and results — error-container background, on-error-container text, 12.5px, radius 10, with a compact retry affordance. No modal, no shake animation.

### 2.9 Solid / reduced-transparency mode

The approved launcher + rail rendered with all glass alphas raised to 100% on the equivalent surface-container tones, blur removed, hairlines retained. Establishes that the design survives with zero transparency (this doubles as the forced-fallback appearance for contrast failures and `prefers-reduced-transparency`).

---
## Section 3 — Milestone 1 Source-Reading Plans

All paths below verified to exist on branch `esr140` of `mozilla-firefox/firefox` (✓), with `main`-branch divergence noted where found. Verification method: direct retrieval from the repository, not memory.

### Spike A — Native vertical tabs / sidebar

**Verified entry points (`esr140`):**

- `browser/components/sidebar/` ✓ — component root; `moz.build` lists `SidebarManager.sys.mjs` ✓ and `SidebarState.sys.mjs`.
- `browser/components/sidebar/browser-sidebar.js` ✓ — defines `SidebarController` (confirmed: `toggleVerticalTabs()`, `sidebarVerticalTabsEnabled`).
- `browser/components/sidebar/sidebar-main.mjs` ✓ — main sidebar custom element; references a `#vertical-tabs` element and the enable-vertical-tabs context-menu flow (confirmed by content inspection).
- `browser/components/tabbrowser/content/tabbrowser.js` ✓, `tabs.js` ✓, `tab.js` ✓ — tab strip and tab element logic shared by horizontal and vertical presentations.
- `browser/themes/shared/sidebar.css` ✓ and `browser/themes/shared/tabbrowser/tabs.css` ✓ — the styling surfaces the restyle must work through.
- `browser/base/content/browser.xhtml` ✓ — where the sidebar/tab structure is instantiated; read for structure, patch minimally.

**Questions the spike must answer:**

1. Which prefs/state place the sidebar with vertical tabs on the **right**, and is that placement first-class or CSS-mirrored?
2. What are the native expanded/collapsed widths, and are they CSS-custom-property-driven (ideal) or hard-coded (patch required)? Can ≈64px / ≈248px be reached without breaking hit targets?
3. How is expand-on-hover implemented, and can a keyboard expansion path (D2's shortcut → focus-into-rail → arrow navigation → Enter → Escape-restore) be added without forking the interaction code?
4. Can the sidebar render as an overlay (no reserved space) — does any native mode exist, or is overlay a positioning patch?
5. Where do favicon/title/audio/mute/pinned/loading states surface in the vertical presentation, and does the row DOM allow the 3px accent bar and approved row geometry via CSS alone?
6. Is there an insertion point (slot, container, or stable sibling) for a **workspace pill area above** the tab list and a **utility cluster below** it, without rewriting `sidebar-main.mjs`?
7. What is the a11y tree of the vertical tab list (roles, keyboard model), and does restyling preserve it?
8. How do tab groups (`tabgroup.js` ✓) render inside vertical tabs — relevant to the workspace memo.
9. What does the native rail do in multiple windows and private windows?

**Minimal experimental patch:** enable vertical tabs on the right in a dev profile; apply a chrome-CSS-only restyle pass targeting collapsed width 64px, expanded 248px, hue-292 surface, accent bar on the selected tab. No JS changes. Screenshot against the mockup.

**Acceptance criteria:** every D3 checklist item classified as *reachable via CSS* / *reachable via narrow JS patch* / *blocked*, each with the file and mechanism named; the experimental patch demonstrates at minimum right-placement + both widths + accent bar; a11y tree confirmed intact under the restyle (screen-reader smoke + `about:debugging` a11y panel).

**Custom-rail fallback triggers (documented limitation required):** right placement impossible or X11-era-mirroring-only; widths locked in JS with no stable override point; hover-expand architecture prevents keyboard expansion without forking; no viable insertion point for workspace/utility areas; a11y regressions inherent to the restyle. Any single trigger is escalated with evidence before the fallback is chosen — per D3, convenience is not a trigger.

**ESR-rebase documentation output:** list of every native file the restyle touches or depends on (selectors, IDs, prefs), so future ESR diffs can be scanned against it; note that `main` already shows sidebar test churn (a `legacy/` test split exists upstream), so the sidebar is active territory — record the upstream bug component ("Firefox :: Sidebar", from `moz.build`) for tracking.

### Spike B — URL-bar infrastructure

**Verified entry points (`esr140`):**

- `browser/components/urlbar/UrlbarController.sys.mjs` ✓, `UrlbarView.sys.mjs` ✓, `UrlbarInput.sys.mjs` ✓ — the classic trio. **[F1]: all three are absent on `main`** (replaced by `UrlbarParentController.sys.mjs`, `UrlbarMuxerStandard.sys.mjs`, `Smartbar…`, with nova/proton dual test manifests). The spike reads `esr140` only; `main` is read solely to characterize what ESR-next will likely bring.
- `browser/components/urlbar/UrlbarProvidersManager.sys.mjs` ✓ — provider registration/query orchestration; the seam our adapter attaches to.
- `browser/components/urlbar/UrlbarProviderOpenTabs.sys.mjs` ✓, `UrlbarProviderPlaces.sys.mjs` ✓ — reference providers for open-tab switching and history/bookmarks.
- `browser/components/urlbar/ActionsProviderQuickActions.sys.mjs` ✓ — Firefox's own command-palette-style provider; the closest existing model for our `>` command provider (note: the older name `UrlbarProviderQuickActions.sys.mjs` no longer exists on `esr140` — verified 404 — an instance of why paths must be checked, not remembered).
- `browser/components/urlbar/UrlbarUtils.sys.mjs` ✓, `browser/components/urlbar/docs/index.rst` ✓ — result types, muxer/ranking documentation.

**Questions the spike must answer:**

1. What is the minimal contract between `UrlbarInput`, `UrlbarController`, and `UrlbarView` — can a second view implementation register against the controller, or does the controller assume the one view (defining our adapter's thickness)?
2. How does a query lifecycle run (start, provider fan-out, muxer, view update, cancel) and where do stale-result guards already exist that ADR-0007's tests can build on?
3. What does result *activation* actually call (pickResult → navigation/tab-switch paths), and where must our close-then-execute sequence hook to satisfy the activation contract without duplicating navigation logic?
4. How are restriction tokens / search-mode prefixes implemented, and can our `>` `@` `%` `#` prefixes ride the same mechanism?
5. What does a provider implement (interface surface of `UrlbarProvider`-derived classes per `ActionsProviderQuickActions`), and do custom results support custom view rendering hooks?
6. How does the heuristic result work for Ctrl+L's populated-URL state?
7. What breaks if the persistent urlbar element is hidden but kept alive versus removed — which approach keeps `UrlbarInput` functional as our launcher's input backend?
8. From `main`: one-page characterization of the new architecture's shape (Parent/child controller split, muxer change) — *not* to target it now, but so the adapter boundary (ADR-0006) is drawn where ESR-next damage will be contained.

**Minimal experimental patch:** in a dev build, open a bare centred panel on Ctrl+K that issues a query through the existing `UrlbarProvidersManager` and logs structured results (no styling, no activation). Proves the provider seam is usable from an alternate surface.

**Acceptance criteria:** written M2 design note specifying the adapter interface (inputs: query string + context; outputs: grouped result list + activation callbacks), the chosen hide-vs-remove strategy for the stock urlbar, the activation hook point satisfying ADR-0007 steps 1–5, and the prefix mechanism decision — each with `esr140` file/class references.

**Fallback triggers:** controller hard-binds to the single view with no tolerable seam → adapter thickens to drive `UrlbarProvidersManager` directly (still no forked providers; ADR-0006's prohibition stands); providers demand a live `UrlbarInput` → keep a hidden functional input as backend and document it as deliberate.

**ESR-rebase documentation output:** the adapter-touched upstream API list; an explicit "[F1] watch item" stating that the launcher view-binding is *expected* to need rework at ESR-next, with the adapter file named as the sole intended blast radius.

### Spike C (small, feeds Section 4) — Tab visibility & session semantics

Read `browser/components/tabbrowser/content/tabbrowser.js` ✓ for hide/show tab APIs and their session-store interaction; `tabgroup.js` ✓ for group membership, collapse, and persistence semantics. Output: the factual inputs the workspace memo's comparison matrix needs. No recommendation.

---

## Section 4 — Workspace Substrate Options Memo (framework only)

Per your directive: comparison framework prepared now; recommendation only after Spike C.

**Options under comparison:**

- **O1 — Custom workspace state over `gBrowser`**: our own store maps workspace → tab set; switching manipulates tab visibility/order directly.
- **O2 — Show/hide layering over normal Firefox tabs**: workspaces are views over one real tab set using native hide/show, with our store owning membership.
- **O3 — Native tab groups as substrate**: each workspace is (or owns) a native group; switching collapses/hides other groups; persistence rides group persistence.
- **O4 — Hybrid**: e.g. O2 visibility mechanics + O3 group objects for membership/persistence, or O1 store with native groups as an import/export boundary.

**Comparison dimensions** (your list, each scored with evidence from Spike C, not intuition): upstream maintenance · session restoration · tab visibility semantics · pinned tabs · multiple windows · private windows · Firefox Sync implications · dragging tabs between workspaces · compatibility with native vertical tabs · accessibility · risk of tab loss · testability.

**Scoring template per cell:** mechanism (file/API), confidence (verified in source / inferred / unknown), and failure mode. Hard gates that veto an option regardless of totals: any credible tab-loss path on crash mid-switch; private-window state ever persisting; incompatibility with the Spike A rail outcome.

**Evidence Spike C must supply:** exact hide/show API surface and its guarantees; what SessionStore records for hidden tabs and collapsed groups; group behaviour across windows; how pinned tabs interact with visibility and groups; what the vertical-tab presentation does with hidden tabs and groups.

**Deliberately out of scope for the memo:** UI design (rail pill area is fixed by the addendum), launcher commands (fixed by plan), sync *implementation* (only implications are assessed).

---
## Section 5 — DMS Token Mapping RFC, Draft 0.1

`docs/rfcs/0001-dms-token-mapping.md`. Sections marked **⛔ BLOCKED** await your fixtures (Section 8 lists them). Everything else is draftable now and drafted here.

### RFC §1 — Status & scope

Governs the pipeline `dank-pywalfox.json → DMS palette reader/validator → semantic --browser-* tokens → browser chrome`. Owns: parsing, validation, normalization, mapping, derivation, contrast enforcement, fallback, last-known-good. Non-goals: watching the palette file; automatic recolour on wallpaper change; exposing raw DMS keys to any component; injecting colours into web content; website dark mode (separate system).

### RFC §2 — Source contract ⛔ BLOCKED

To be written verbatim from real fixtures: observed key names, value formats (hex/rgb/alpha?), light/dark scheme presence, ordering guarantees, any schema/version marker, file size and write pattern (depends on your answer about which DMS component writes it and when — determines whether the validator must defend against partial writes). **No key names may be invented; this section stays empty until fixtures arrive.**

### RFC §3 — Normalization

Canonical internal colour space: **OKLCH** (matches the mockup; perceptually uniform lightness makes the surface ladder and contrast math sane). All inbound values parse → sRGB → OKLCH. Out-of-gamut results clamp chroma at constant L/H. Parse failures are per-key fatal for that palette: a palette with any unparseable colour is rejected whole (atomic apply, RFC §12) with the specific key and value named in the error.

### RFC §4 — Semantic token taxonomy

Adopted from the plan's candidate list, with one-line usage contracts (excerpt; full table in the RFC file):

- `--browser-surface`, `--browser-surface-dim/-bright`, and the five `--browser-surface-container-*` steps: the tonal ladder; every opaque chrome background is one of these — no component defines its own background colour.
- `--browser-primary` / `--browser-on-primary` / container pair: accent family; used only for the D-list (active-tab bar, focus, selected result, progress, toggles, workspace selection, badges).
- `--browser-on-surface` / `--browser-on-surface-variant`: primary and secondary text.
- `--browser-outline` / `--browser-outline-variant`: hairlines; alpha-carrying (see §7).
- `--browser-error*` quartet: warning chip, launcher error strip, sync-failure states; **never rendered on glass**.
- `--browser-focus-ring`: the only permitted focus indicator colour; shared by rail focus, launcher selection dot, sensitive-chip ring.
- `--browser-scrim`, `--browser-shadow`: launcher backdrop and elevation.

Rule: components consume tokens only; the adapter is the only writer. Token list may shrink but not grow ad hoc — additions require an RFC amendment.

### RFC §5 — Mapping table ⛔ BLOCKED

DMS key → token, written only from RFC §2's observed schema.

### RFC §6 — Derived-token formulas

Where DMS supplies fewer roles than the taxonomy needs, derive in OKLCH from a base surface S(L₀,C₀,H₀) and accent A:

- Surface ladder as lightness offsets from L₀, targeting the mockup's proven shape: dim −0.02 · surface 0 · container-low +0.02 · container +0.04 · container-high +0.06 · container-highest +0.09 · bright +0.12 *(offsets provisional; validated against fixtures and contrast rules)*. Chroma held ≤0.03 for surfaces regardless of source chroma — the "tonal, not tinted" rule from the approved direction, enforced numerically.
- `on-` colours: fixed-L text tones (≈0.92 primary / 0.80 secondary on dark surfaces) hue-matched to their surface, then contrast-checked (§10) with L adjusted along the hue until passing.
- Container pairs: accent-hued, L pushed toward the surface pole (containers ≈ L 0.32–0.42 dark) with `on-container` at the opposite pole — mirroring the mockup's launcher-selection and workspace-pill values.
- Focus ring: accent at L ≥ 0.75, C ≥ 0.10 — bright enough to pass 3:1 against every ladder step (checked, not assumed).

### RFC §7 — Alpha & glass

Only these tokens may carry alpha: outlines (6–12%), state layers (§9), scrim, and the three **glass surface** derivations. Glass presets bind alpha+blur pairs, fixture values from the mockup: chip ambient 55%/14px · rail 72%/20px · launcher 92%/28px. Presets Solid/Soft/Frosted/Custom map onto these; Solid sets all three to 100%/0px on the equivalent ladder step. No other token may be translucent — prevents accidental double-alpha compositing.

### RFC §8 — Focus & accent derivation

Accent source preference order (final order ⛔ pending schema): explicit DMS primary/accent role if present → highest-chroma mid-L palette entry → provisional-palette accent as last resort. Selected accent is then constrained to the §6 focus-ring bounds for focus uses while the raw accent may be used for fills; both recorded as separate tokens so a low-chroma wallpaper cannot produce an invisible focus ring.

### RFC §9 — State layers

Hover +5.5% white · pressed +9% · selected = container fill (not a wash) · disabled = on-surface at 38% with interaction removed · danger-hover = error-container. One systemwide rule, tokenized as `--browser-state-hover` etc.; components never hard-code washes. (Generalizes the mockup's observed 5–6% values into law.)

### RFC §10 — Contrast validation

Numeric minima (WCAG 2.x ratios as the baseline metric): body/launcher-input/selected-result/error text 4.5:1 · large text 3:1 · UI components, focus ring, active-tab indicator 3:1 · security chip text and warning states 4.5:1 **in every state including ambient** · disabled text exempt but must remain distinguishable from enabled.

**Composited methodology for glass:** every text-over-glass pairing is validated against four canonical worst-case backdrops — white, black, mid-gray (#808080), saturated (#ff3b30) — composited at the configured alpha, blur ignored (blur never increases contrast; assuming otherwise would be unsound). A pairing passes only if all four pass. This is the rule that will set the ambient chip's real opacity floor and likely raise it well above the mockup's 40%.

### RFC §11 — Failure adjustment hierarchy

Deterministic, in order, per failing component: (1) raise that surface's glass alpha in 5% steps to 100% → (2) move one ladder step toward the contrasting pole → (3) adjust foreground L along its hue → (4) insert local scrim behind the text → (5) disable glass for the component → (6) whole-theme fallback to the built-in palette. Security/auth/warning UI skips 1–4: it is born at step 5 conditions (solid) and can only escalate to 6. Every adjustment is logged with component, step taken, and measured before/after ratios.

### RFC §12 — Fallback & last-known-good

Built-in accessible palette = frozen snapshot of the provisional palette (§15 table): the browser with no DMS, a broken DMS, or a contrast-unrecoverable DMS always looks like the approved mockup. LKG: last fully-validated token set persisted (profile-scoped JSON, atomic write); restore triggers: parse failure, validation failure, user `> restore theme`. Apply is atomic — a candidate palette either passes §3+§10 entirely and replaces the live set in one operation, or nothing visible changes and the error surfaces (§14).

### RFC §13 — Light palettes ⛔ BLOCKED

Depends on whether DMS emits light schemes (your fixture item 3). Policy-if-absent placeholder: browser remains dark-only until a light source exists; no synthetic light derivation in v1.

### RFC §14 — Versioning & errors

Schema detection per §2's marker (⛔ mechanism blocked). User-facing strings, final wording:

- file missing → "No DMS palette found at ⟨path⟩ — kept current theme."
- unreadable → "Couldn't read the DMS palette (permissions?) — kept current theme."
- malformed → "DMS palette isn't valid JSON — kept current theme."
- unknown schema → "DMS palette format not recognized — kept current theme."
- contrast-unrecoverable → "Palette fails readability checks — kept current theme."

All surfaced at the sync trigger point (button toast per Addendum 2.5, launcher inline error for `> sync dms`). Never a modal; never a silent failure.

### RFC §15 — Test fixtures & the provisional reference palette

Checked-in fixtures: real palettes (⛔ yours), edge palettes (near-monochrome, low-contrast, saturated-light — synthesized once §2 defines the schema), malformed set (truncated JSON, wrong types, out-of-range values), and expected token-output snapshots per input.

**Provisional built-in dark palette** (extracted from the approved mockup; binding as the fallback and regression reference):

| Token | Value |
|---|---|
| surface (window backdrop) | `oklch(0.16 0.015 292)` |
| surface-container (rail base) | `oklch(0.20 0.02 292)` |
| surface-container-high (launcher panel) | `oklch(0.22 0.02 292)` |
| primary-container (launcher btn / selection) | `oklch(0.32 0.05–0.06 292)` |
| primary-container-high (active workspace) | `oklch(0.42 0.09 292)` |
| primary / accent-indicator | `oklch(0.78 0.14 292)` |
| on-primary-container | `oklch(0.88 0.08 292)` |
| on-surface | `oklch(0.90–0.94 0.01 292)` |
| on-surface-variant | `oklch(0.82 0.01 292)` |
| on-surface-muted (labels) | `oklch(0.55 0.02 292)` |
| outline / outline-variant | `oklch(1 0 0 / 0.06–0.12)` |
| state-hover | `oklch(1 0 0 / 0.055)` |
| glass alphas (chip/rail/launcher) | 0.55 / 0.72 / 0.92 |
| scrim | `oklch(0.10 0.01 292 / 0.55)` |

**Visual regression method:** (a) token snapshot tests — adapter output for each fixture diffed against stored JSON snapshots (cheap, runs everywhere); (b) reference screenshots — the browser rendered with the provisional palette in defined states (launcher open with fixed results, rail compact/expanded, chip states) captured via Firefox's browser-chrome screenshot facilities and pixel-diffed within tolerance; failures attach diffs. (b) runs on the fixed provisional palette only, so design drift and mapping bugs are distinguishable: (a) catches mapping regressions, (b) catches styling regressions.

### RFC §16 — Accessibility

`prefers-reduced-transparency` and the browser's own reduced-transparency setting force Solid preset (Addendum 2.9). Forced-colors/HCM: token layer defers entirely to system colours; glass disabled. No information by colour alone — the 3px indicator always pairs position+colour; audio/mute/pinned states pair glyph+colour; sync failure pairs glyph+toast text. Animation tokens respect `prefers-reduced-motion` (rail expansion and sync pulse become instant).

---
## Section 6 — Initial Repository Issue Set (Milestones 0–1)

Template per issue: **Goal · Scope · Non-goals · Acceptance criteria (AC) · Test evidence (TE) · Documentation output (Doc) · Dependencies (Dep) · Maintenance notes (MN)**. Sized for focused commits and single-sitting reviews.

### Milestone 0

**#1 — Repository and ADR bootstrap**
Goal: versioned home for decisions before any code. Scope: repo layout (`docs/adr/`, `docs/rfcs/`, `docs/design/`, `patches/`), ADR template, Section 1's ADR-0001…0010 committed. Non-goals: any Firefox source, any tooling automation. AC: all ten ADRs render, numbered, cross-linked to this pack. TE: n/a (docs). Doc: the ADRs are the doc. Dep: none. MN: ADR-0002 contains the "re-verify ESR before branching" instruction — future-us reads it there.

**#2 — Design reference import**
Goal: the approved baseline is reproducible forever. Scope: mockup HTML, §1 decision inventory (from Review v1), provisional palette table (RFC §15), reference screenshots of the mockup's states. Non-goals: addendum states (that's #13). AC: a newcomer can state the approved design from repo contents alone. TE: n/a. Doc: `docs/design/baseline.md`. Dep: #1. MN: marked immutable-except-by-decision; addenda extend, never edit.

**#3 — Policy profile: extensions**
Goal: dedicated Firefox profile with uBlock Origin + SponsorBlock via enterprise policy. Scope: `policies.json` with `ExtensionSettings`, IDs `uBlock0@raymondhill.net` / `sponsorBlocker@ajay.app`, `installation_mode: "normal_installed"`; **`install_url` values copied from AMO at commit time and recorded with retrieval date — guessing is a review-rejection**. Non-goals: `force_installed`; Dark Reader (deferred); baked-in policies (full-build track). AC: fresh profile auto-installs both; both disableable in about:addons; both function on a test page/video. TE: screenshots of about:policies (active), about:addons (enabled→disabled→enabled), working block + segment skip. Doc: profile setup guide. Dep: none. MN: `Policies.sys.mjs` path verified on esr140; policy schema changes at ESR transitions are a known watch item (Risk R9).

**#4 — Wayland/compositor baseline (stock Firefox)**
Goal: baseline behaviour before we change anything. Scope: stock Firefox on Fedora/Wayland under **both** Niri and Hyprland: portals file dialog, screen share, notifications, clipboard, DnD, fullscreen video, PiP, fractional scaling. Non-goals: fixing anything; multi-monitor depth (M9). AC: per-compositor findings table, pass/fail/notes per item. TE: the table + screenshots of anomalies. Doc: `docs/compat/baseline-⟨date⟩.md`. Dep: none. MN: this table is the regression reference every future compositor bug is compared against.

**#5 — Widevine Case A: stock control run**
Goal: the DRM control datapoint (ADR-0009). Scope: stock Firefox, clean profile, Fedora/Wayland: CDM download observed, plugin state in about:plugins/about:support, EME init, one legal public EME test page, one commercial service with a legally held account; repeat the playback step under Niri and Hyprland. Non-goals: Case B; any workaround. AC: findings recorded per ADR-0009's report axes. TE: about:support media section capture, console/media logs. Doc: opens `docs/drm/spike.md` (the living Case A/B document). Dep: none. MN: this file is the release-gate input; every later branding state appends a column.

**#6 — ESR strategy ADR + rebase scaffolding**
Goal: make the first ESR transition boring. Scope: ADR-0002 finalized with the branch-time verification step; rebase checklist v0 (the plan's 8 steps expanded to concrete commands); empty conflict log with schema (file · upstream change · our patch · resolution · minutes spent). Non-goals: performing a rebase. AC: checklist executable by future-us without archaeology. TE: n/a. Doc: `docs/maintenance/`. Dep: #1. MN: [F1] pre-registered in the conflict log as an *expected* ESR-next conflict — the first entry is written before the conflict exists.

### Milestone 1

**#7 — Bootstrap Artifact Mode build**
Goal: reproducible dev environment on the Legion. Scope: current bootstrap flow, Desktop Artifact Mode, `./mach run` with the #3 profile; record exact commands, mozconfig, disk/time cost. Non-goals: full build (tracked in #12). AC: clean re-run from the doc alone succeeds. TE: build log excerpt + running-browser screenshot. Doc: `docs/dev/build.md`. Dep: #3. MN: bootstrap flow itself churns; doc carries a "verified against revision X on date Y" header.

**#8 — Hello-chrome patch + toolchain check**
Goal: prove the edit-run-test loop. Scope: one small visible chrome change; `./mach lint` and a browser-chrome test run (`./mach test` on an existing urlbar browser-test) verified working. Non-goals: keeping the change. AC: change visible; lint and test commands documented with real output. TE: screenshot + command transcripts. Doc: appended to `docs/dev/build.md`. Dep: #7. MN: identifies the test harness our M2 tests will live in.

**#9 — Spike A: native vertical tabs** — executes Section 3 Spike A verbatim. AC/TE/Doc/fallback-triggers as written there. Dep: #7. MN: output gates the D3 decision; upstream bug component "Firefox :: Sidebar" recorded for watching.

**#10 — Spike B: URL-bar infrastructure** — executes Section 3 Spike B verbatim, including the one-page `main`-branch [F1] characterization. Dep: #7. MN: output is the M2 design note; adapter blast-radius file named.

**#11 — Spike C: tab visibility & session semantics** — executes Section 3 Spike C; output feeds the Section 4 memo. Dep: #7. MN: hard-gate evidence (tab-loss paths) explicitly collected.

**#12 — Widevine Case A on the dev build + Artifact Mode limits doc**
Goal: second DRM datapoint; full-build backlog. Scope: repeat #5's protocol on the Artifact Mode build; diff against control; enumerate which planned work exceeds Artifact Mode (branding, baked policies, Case B, packaging). Non-goals: performing a full build. AC: diff table appended to `docs/drm/spike.md`; `docs/dev/full-build-backlog.md` exists. TE: same evidence class as #5. Dep: #5, #7. MN: creates the M-Full-Build track's charter.

**#13 — Mockup addendum pass**
Goal: the Section 2 states exist as design artifacts. Scope: addendum frames for 2.1–2.9 in the mockup's language. Non-goals: touching the approved baseline; final opacity numbers (RFC-owned). AC: every 2.1–2.9 state depicted; baseline unchanged. TE: side-by-side with baseline confirming vocabulary consistency. Doc: `docs/design/addendum-01.md`. Dep: #2. MN: addendum values feed RFC §15 fixtures where applicable.

---

## Section 7 — Risk Register

Likelihood/Impact: L=low M=medium H=high. Each row: early detection · mitigation · fallback.

| # | Risk | L'hood | Impact | Early detection | Mitigation | Fallback |
|---|---|---|---|---|---|---|
| R1 | **URL-bar upstream churn** — **[F1] confirmed**: view/controller trio already deleted on `main` (Nova/Smartbar migration) | **H (certain at ESR-next)** | H | `main` watched quarterly; nova/proton test manifests tracked; conflict-log pre-entry from #6 | Adapter boundary (ADR-0006) drawn so only the adapter binds upstream view/controller; providers seam is the stable contract | Rewrite adapter against new architecture at ESR-next; launcher view untouched; if provider seam also breaks, adapter drives ProvidersManager directly |
| R2 | **Native-sidebar churn** — sidebar is active development (test-suite `legacy/` split visible upstream) | M–H | M | "Firefox :: Sidebar" bug component watched; Spike A's touched-file list scanned at each ESR diff | Restyle via CSS + narrow patches only; insertion points chosen at stable IDs per Spike A | Documented-limitation path to custom rail (D3 discipline applies in reverse: migrate only with evidence) |
| R3 | **ESR rebase cost** | H (recurring) | M–H | Effort measured at first transition (#6 scaffolding) | Narrow patches, conflict log, ADRs, integration tests around every touched subsystem | Skip one ESR generation on the release branch only if security backports remain available — never ship an EOL base |
| R4 | **Widevine** — Case B unknowns for a branded distributable | M | H (claims), M (project) | Case A data at M0–M1; Case B immediately when full build exists | ADR-0009 protocol; no claims before per-service tests | Classifications "personal builds only" / "unsupported pending licensing"; public claims altered, project continues |
| R5 | **Full-build requirements** — branding/policies/Case B/packaging all exceed Artifact Mode | H (certain) | M | #12's limits doc | Parallel M-Full-Build track charter early; first full build well before M10 | Timeline slip absorbed by track parallelism, not milestone re-ordering |
| R6 | **Glass performance** — backdrop blur over video, fractional scaling, two compositors | M | M | Perf capture during M3/M4 on video-heavy pages; both compositors | Blur radii bounded; Solid preset is also the perf escape; overlay rail only blurs when open | Default preset demoted to Soft/Solid; glass becomes opt-in |
| R7 | **Accessibility & contrast** — glass over arbitrary pages defeats nominal contrast | M | H | RFC §10 composited checks in CI from M3 | Deterministic §11 hierarchy; security UI exempt from glass by rule | Forced Solid (§11 step 6); the provisional palette guarantees an accessible floor |
| R8 | **Workspace/session corruption** — tab loss on crash mid-switch | L–M | **H (trust-destroying)** | Spike C hard-gate evidence; crash-mid-switch test in M7 suite | Substrate chosen partly on loss-safety; atomic workspace state writes; session-store integration over parallel storage | LKG workspace snapshot + "restore workspace" command; worst case degrades to plain Firefox session restore |
| R9 | **Extension policy changes** — AMO URLs, policy schema, WebExtension surface drift | L–M | M | #3's dated `install_url` records; policy docs diffed at ESR transitions | `normal_installed` (not force) limits blast radius; extensions vanilla by requirement | Manual install instructions ship as documented fallback |
| R10 | **Fedora/portal changes** — portal backends, Wayland stack updates | M | L–M | #4 baseline table re-run per Fedora release | Standard portals only (ADR-0010); no exotic integration | Pin known-good portal packages in docs; report upstream |
| R11 | **Niri/Hyprland divergence** — compositor-specific rendering/input differences | M | M | Dual-compositor testing from M0 (#4) onward, per milestone | Zero compositor-specific code paths (ADR-0010) | A per-compositor *workaround* doc (never a per-compositor build); worst case: known-issues list per compositor |

Standing review: register revisited at each milestone close and each ESR transition; any risk that fires gets a post-mortem line item here.

---

## Section 8 — Status Classifications

**Confirmed from the attached mockup** (read directly from source): full decision inventory of Review v1 §1 — layout, chip geometry, rail geometry and glass values, launcher structure and glass values, grouped results, flat selection model, Enter-activates-and-closes, Escape/scrim behaviour, the OKLCH hue-292 tonal system, and the provisional palette values now frozen in RFC §15.

**Confirmed from current Firefox source/documentation** (verified against the live repository this session): branch `esr140` exists; on it, the urlbar trio (`UrlbarController/View/Input.sys.mjs`), `UrlbarProvidersManager`, `UrlbarProviderOpenTabs`, `UrlbarProviderPlaces`, `ActionsProviderQuickActions`, the sidebar component with vertical-tabs support (`browser-sidebar.js` with `toggleVerticalTabs`, `sidebar-main.mjs` with `#vertical-tabs`), tab groups (`tabgroup.js`), `Policies.sys.mjs`, `browser.xhtml`, and the shared sidebar/tabs CSS all exist at the paths cited in Section 3. **[F1]** on `main`: the urlbar trio is removed and a nova/proton migration is in progress. Also confirmed externally: ESR 140 is the current supported line with 2026 point releases; mainline is at 153, so ESR-next is imminent. `UrlbarProviderQuickActions.sys.mjs` does **not** exist on esr140 (renamed to `ActionsProviderQuickActions.sys.mjs`) — recorded as a concrete example of why remembered paths are rejected.

**Assumptions requiring a source spike**: native sidebar can reach the approved rail design (Spike A); a second view can bind `UrlbarController`, or the adapter can drive `UrlbarProvidersManager` directly (Spike B); restriction-token mechanism can host our prefixes (Spike B); hide-vs-remove strategy for the stock urlbar is viable (Spike B); native hide/show + session semantics support loss-safe workspaces (Spike C); Artifact Mode suffices for all M2–M8 changes (#12 verifies incrementally).

**Blocked on files from you**: RFC §2 (source contract), §5 (mapping table), §8 accent-preference finalization, §13 (light palettes), real-palette fixtures in §15 — unblocked by: your active `dank-pywalfox.json`, several substantially different wallpaper palettes, a light-leaning palette or confirmation none exists, the writer/timing answer, and any adjacent DMS palette files.

**Decisions already approved**: D1–D8 as refined in your review; the launcher separation architecture and activation contract (ADR-0006/0007); the workspace-memo directive (ADR-0005); Case A/B split (ADR-0009); all recorded in Section 1.

**Decisions still open**: rail implementation A/B (after Spike A); workspace substrate O1–O4 (after Spike C + memo); stock-urlbar hide-vs-remove (after Spike B); downloads-button idle visibility (addendum shows both variants); exact ambient-chip opacity floor, ladder offsets, and all ⛔ RFC content (after fixtures); ESR-next adoption timing (at branch time per ADR-0002).

---

*Pack ends here. Nothing has been implemented. Next joint-review inputs: your read of Sections 1–7, and — whenever convenient — the fixture files, which unblock the last RFC sections independently of everything else.*