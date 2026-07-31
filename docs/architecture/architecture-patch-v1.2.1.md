# Architecture Patch v1.2.1

Status: narrowly scoped patch to accepted v1.1 + v1.2. Only the items below change; nothing else is repeated. No code has been written. New source citations verified this session against `esr153 @ f815328b` (153.1.0esr) under the multi-source rule; negatives recorded.

## 1. Change summary

1. Extension badge ownership corrected: extensions own badge meaning and colours; our chrome owns geometry and framing only (→ §2).
2. New ADR-0012: page actions and identity without an address bar; `BrowserPageActionAdapter` architecture (→ §3).
3. New Spike F: built-in URL-bar actions, identity, and permission surfaces, with verified starting call sites and honest negatives (→ §4).
4. Mockup addendum gains origin-chip/permission/tracking-protection/page-actions/container frames, with a meaning-colours-are-not-DMS-colours rule (→ §5).
5. Issue #19 added; Issue #8 gains `./mach artifact last` evidence requirements (→ §6).
6. Risk R16 added (→ §7).
7. Readiness deltas applied, including the origin-chip readiness split (→ §8).
8. Merge guide (→ §9).

## 2. Extension badge ownership (correction)

v1.2 §7 proposed rendering the uBlock Origin badge in our `error-container` fill. **That was wrong** — it assigned our palette a piece of state the WebExtension API explicitly gives to the extension. Verified on esr153: `ext-browserAction.js` implements extension-set badge background colour (`badgeBackgroundColor` handling present), alongside badge text, badge text colour, and per-tab/per-window badge state.

**Adopted rule:**

```text
Extension supplies badge visual state
        ↓
our host preserves it
        ↓
our rail controls placement, not meaning
```

Our chrome may control only: badge geometry, minimum/maximum size, placement relative to the action icon, clipping, legibility fallback **when the extension provides no colour**, and focus/hover/disabled framing around the complete action button. We never recolour uBlock Origin's, SponsorBlock's, or any extension's badge to fit the DMS palette. When no badge colours are supplied, Firefox's standard fallback behaviour is preserved unless Spike D establishes a safe browser-owned fallback.

**Applied updates:**

- *v1.2 §7 (compact rail frame)* — reworded: "uBlock Origin pinned action with badge — badge text, background colour, and text colour are the extension's own, rendered verbatim; our frame supplies geometry, clipping, and focus/hover treatment only." The `error-container` sentence is struck.
- *Spike D questions* — add Q11: "Where do extension-set badge text/background/text-colour and per-tab/per-window badge state live, and can a rail host render them without transformation?"
- *Spike D experiment* — add the six badge checks: extension-defined badge text retained · extension-defined background retained · extension-defined text colour retained · tab-specific changes update after switching tabs · missing badge colour uses normal Firefox fallback · reduced-transparency and forced-colour modes do not destroy badge meaning.
- *R12* — mitigation cell gains: "badge meaning and colours remain extension-owned; host renders verbatim."
- *#15 acceptance criteria* — extension-action frames must depict extension-authored badge colours (e.g. uBO's own badge colour), not token colours; a frame note states the ownership rule.

## 3. ADR-0012 — Page actions and identity without an address bar

**Status:** Accepted (direction); mechanism gated on Spike F.
**Context:** removing the address bar removes the visual home of Firefox's built-in page controls: tracking-protection status/controls, origin identity and connection security, site permissions and indicators, Reader Mode, bookmark star/state, translation, PiP/media page indicators where applicable, zoom state, container/contextual-identity indication, Firefox-provided page actions, WebExtension page actions, temporary permission indicators, and any URL-bar actions future ESRs introduce. None of these may silently disappear. The origin chip covers identity/security display but not how the full set of panels and actions is invoked.
**Decision:**

```text
Firefox identity, permission, and page-action infrastructure
        ↓
BrowserPageActionAdapter owned by us
        ↓
origin chip + contextual page-actions panel + launcher discovery
```

- Reuse Firefox's real identity, permission, tracking-protection, bookmarking, Reader Mode, translation, and page-action infrastructure; never recreate the underlying security or permission state.
- The origin chip is the primary anchor for identity, security, tracking protection, and site permissions; activating it opens the real browser-owned identity/security panel, or an adapter-hosted presentation preserving its semantics.
- A contextual **Page Actions** control provides non-security actions relevant to the current page; it may appear beside the chip only when relevant (`[ shield/lock github.com ] [ page actions ⋯ ]`), or the two may merge into one expanded origin panel **if** source inspection shows that preserves clarity and upstream behaviour — this choice is not finalized before Spike F.
- The launcher may expose page actions for keyboard discovery/activation but is never the sole interface for important permission or security state.
- Contextual actions retain tab-specific visibility and enabled state; built-in and extension-provided page actions remain distinguishable; unknown future upstream actions degrade into a discoverable generic panel rather than vanish.

**Consequences:** the chip's role expands from indicator to anchor; Spike F becomes a gate for no-address-bar daily-driver mode alongside Spikes B and D; stock-toolbar removal must stay reversible until Spike F and #16 resolve (§6, #19 relationships).

## 4. Spike F — Built-in URL-bar actions, identity, and permission surfaces (esr153)

**Timing:** after the hello-chrome loop (#9), against the pinned baseline. May run in parallel with Spike D.

**Operating rule:** source paths are located and recorded from the pinned checkout via call sites, markup, manifests, and tests — never guessed. The following *starting* call sites were verified this session (existence only; ownership maps come from the spike): `browser/base/content/browser-siteIdentity.js` ✓ (identity box/panel logic) · `browser/base/content/browser-siteProtections.js` ✓ (tracking-protection panel) · `browser/base/content/browser-pageActions.js` ✓ (page-action handling — notable: this surface still exists on esr153 and must be characterized, not assumed vestigial) · `browser/modules/SitePermissions.sys.mjs` ✓ · `toolkit/components/reader/ReaderMode.sys.mjs` ✓ · `toolkit/components/translations/actors/TranslationsParent.sys.mjs` ✓ · `toolkit/components/contextualidentity/ContextualIdentityService.sys.mjs` ✓. Negatives recorded: `browser/components/translations/TranslationsParent.sys.mjs` and `browser/components/contextualidentity/…` do not exist (both live under `toolkit/`).

**Inspection list:** identity box and identity popup · tracking-protection icon and panel · permission indicators · browser-scoped temporary permissions · bookmark page action · Reader Mode action · translation action · zoom indicator/action · container/contextual-identity indicator · built-in page-action registry or equivalent · extension page actions · UITour or other code expecting named URL-bar targets · telemetry distinguishing URL-bar page actions · existing keyboard commands for these actions · private-window behaviour · multi-window and tab-specific state.

**Questions:** (verbatim from review)
1. Which controls are true page actions, and which are hard-wired parts of the identity or URL-bar structure?
2. Which controls can be rehosted as standard widgets?
3. Which panels assume an anchor inside the URL bar?
4. Can the real identity and tracking-protection panels anchor to the origin chip?
5. Can built-in page actions anchor to a custom Page Actions button or panel?
6. How are permission indicators updated per tab?
7. How are temporary permissions represented and cleared?
8. Which actions have existing keyboard commands and therefore need no permanent button?
9. Which actions are safety-critical and must remain immediately visible?
10. How should WebExtension `page_action` items coexist with built-in actions?
11. What happens when a future Firefox ESR adds an unknown page action?
12. Which UITour, telemetry, automated tests, or onboarding assumptions break when the normal URL-bar targets are absent?
13. Can stock panels be re-anchored without copying their contents?
14. Which upstream files enter our ESR conflict watch set?

**Minimal experiment:** a temporary browser-chrome host containing an origin/security button, tracking-protection state, one active permission indicator, the bookmark action, Reader Mode on a supported page, and one test WebExtension page action. Verify: genuine stock panels open · anchoring works · state updates after tab switches · security state cannot become stale · bookmark state updates · Reader Mode visibility updates · permission changes update immediately · keyboard activation works · second-window behaviour works · private-window state remains isolated.

**Required recommendation — exactly one:** rehost standard identity and page-action elements directly · keep stock backend elements alive and expose project-owned proxies · restyle and reposition a standard page-actions container · hybrid (rehost security-critical controls, proxy lower-priority actions). Reimplementing security, permission, tracking-protection, or bookmark state is prohibited.

## 5. Mockup addendum — new frames (baseline untouched)

- **Origin chip, ordinary secure page:** domain, security/tracking-protection glyph, no permission indicators.
- **Origin chip, active permission:** representative indicator (camera/microphone/location/notification), keyboard-focus state, and the expanded permission/security panel anchored to the chip.
- **Tracking protection:** normal protection · protection disabled for the current site · blocked-content attention state.
- **Contextual Page Actions button:** hidden when no actions exist · visible with Reader Mode · visible with bookmark state · multiple available actions · keyboard-focused state.
- **Page Actions panel** (visual example only; final content and grouping depend on Spike F, and frames carry that caption):

```text
PAGE ACTIONS
Add bookmark
Open Reader View
Translate page
Zoom: 110%
Test extension page action
```

- **Container identity:** how a container tab communicates its identity without the removed URL bar (e.g. container colour applied to a chip-adjacent marker and the tab row's identity strip — exact treatment provisional).
- **Meaning-colour rule (binding for all frames):** DMS accent colour never replaces established security, warning, permission, or container colours where those colours carry meaning. Container colours, permission-warning colours, protection states, and extension badge colours (§2) render with their semantic values; our tokens style only the surrounding frames and surfaces.

Applied as amendments to #15's scope and acceptance criteria.

## 6. Issue changes

**#19 — Built-in page-action, identity, and permission spike (new).** Executes Spike F (§4). Dep: #8, #9. Relationships: may run in parallel with #16 · origin-chip *implementation* depends on its identity/security findings · **no-address-bar mode is not daily-driver-ready until #16 and #19 both resolve** · launcher foundation work (M2) may begin before it, but **stock-toolbar removal remains reversible until the result is known** (a revert path is part of M2's definition of done). AC: source-path inventory · state and panel ownership map · minimal-experiment evidence · multi-tab/window/private-window tests · recommended hosting architecture · rebase dependency list · mockup implications · explicit list of controls safe to hide (reliable shortcut or launcher route exists) · explicit list of controls that must remain visually present.

**#8 (amended) — Artifact Mode evidence.** Add the official inspection command `./mach artifact last`, stored with build evidence. Acceptance evidence must include: source revision · artifact revision · artifact job/platform · artifact timestamp · `mach artifact last` output · clean/dirty object-directory state. Object-directory clobbers triggered by baseline or artifact changes are recorded as events.

## 7. Risk R16

| # | Risk | L'hood | Impact | Early detection | Mitigation | Fallback |
|---|---|---|---|---|---|---|
| R16 | **Built-in page actions and identity controls become inaccessible** — tracking protection, site identity, permission indicators, Reader Mode, bookmark state, translation, zoom, container identity, or Firefox/WebExtension page actions inaccessible or stale without the address bar | **H without deliberate integration** | **H** (security and permission state especially) | Spike F with real tab switching, permissions, Reader Mode, bookmarking, containers, and a test extension page action | Preserve Firefox state and panel implementations; rehost or proxy interfaces; generic handling for unknown future actions (ADR-0012) | Retain a minimal stock-compatible identity/page-action strip until the custom host is proven complete |

## 8. Readiness deltas

Replacing the v1.2 origin-chip line:

- **Security chip visual design and navigation/domain prominence contract** — Ready for Milestone 0.
- **Identity, tracking-protection, permission-panel anchoring, and complete page-action integration** — Ready for source spike; blocked on Spike F before implementation is considered complete.

Additions:

- **Built-in identity/security host** — Ready for source spike (visual role approved, mechanism open).
- **Built-in page actions** — Blocked on source spike (F).
- **WebExtension page actions** — Blocked on Spikes D and F.
- **No-address-bar daily-driver mode** — Blocked on Spikes B, D, and F.
- **Extension badge state** — Source mechanism exists (verified: `ext-browserAction.js` badge-colour handling on esr153); custom-host rendering blocked on Spike D.
- **Origin chip design** — Approved.
- **Origin chip complete Firefox integration** — Blocked on Spike F.

## 9. Merge guide

- §2 → v1.2 §7 (frame rewording, struck sentence), v1.2 §4 (Spike D Q11 + six badge checks), v1.1 Section 7 R12 cell, v1.2 §8 #15 AC.
- §3 ADR-0012 → v1.1 Section 1, after ADR-0011.
- §4 Spike F → v1.1 Section 3, after Spike E (as merged from v1.2).
- §5 frames + meaning-colour rule → v1.1 Section 2 / v1.2 §7 as additional 2.10-series frames; #15 scope.
- §6 → v1.1 Section 6: #19 appended to Milestone 1; #8 amended in place (stacking on the v1.2 amendment); M2 definition of done gains the toolbar-removal revert path.
- §7 R16 → v1.1 Section 7, after R15.
- §8 → v1.2 readiness classification, edited in place.

---

*Patch ends. Architecture is complete enough to begin Milestone 0. Further architecture changes arise only from concrete source-spike findings.*