# ADR-0011 — Extension actions without a traditional toolbar

**Status:** Accepted (direction); implementation mechanism gated on Spike D
**Governing text:** Architecture Addendum v1.2 §3, as corrected by Architecture Patch v1.2.1 §2
**Amendment chain:** v1.2 §3 (original) → **v1.2.1 §2 (badge-ownership correction — governs)**

## Decision (verbatim, v1.2 §3)

> **ADR-0011 — Extension actions without a traditional toolbar**
>
> **Status:** Accepted (direction); implementation mechanism gated on Spike D.
> **Context:** the browser removes the ordinary navigation toolbar, but real WebExtensions expose browser/MV3 actions, page actions, popups, badges, per-tab enabled state, context menus, pinned state, permission warnings, and management controls. uBlock Origin (popup/dashboard, per-site toggle, element picker and zapper, logger, settings/filter lists, badge counts) and SponsorBlock (current-video status, popup and category controls, segment submission where supported, voting and skip controls, per-site/per-video behaviour) must remain **fully usable**, not merely active in the background.
> **Decision:** host and invoke the real extension actions and popups — never reproduce their interfaces:
>
> ```text
> Firefox WebExtension action infrastructure
>         ↓
> our extension-action host
>         ↓
> rail action buttons / extensions panel / launcher commands
> ```
>
> The browser provides: an Extensions button in the bottom rail utility cluster; an extensions panel preserving Firefox permission and management semantics; optional pinned extension actions in compact and expanded rail; badge rendering; popup anchoring; keyboard access; context-menu access; a launcher result for finding and invoking extension actions; a reliable route to `about:addons`. The launcher may aid discovery but is never the only interface — popups need an understandable visual anchor and mouse users need a discoverable control.
> **Constraints:** it is *not assumed* that stock popups can anchor to arbitrary custom chrome — Spike D verifies in source. A completely custom reimplementation of extension popups or state is not an acceptable outcome at any rung.
> **Consequences:** the bottom utility cluster (Addendum 2.5) gains the Extensions entry as a first-class member; launcher gains an EXTENSION result group (guarded, §7); daily-driver readiness of the launcher is gated on Spike D (#16 dependency rule, §8).

Source: [`architecture-addendum-v1.2.md`](../architecture/architecture-addendum-v1.2.md) §3.

## Correction — extension badge ownership (verbatim, v1.2.1 §2 — current)

> v1.2 §7 proposed rendering the uBlock Origin badge in our `error-container` fill. **That was wrong** — it assigned our palette a piece of state the WebExtension API explicitly gives to the extension. Verified on esr153: `ext-browserAction.js` implements extension-set badge background colour (`badgeBackgroundColor` handling present), alongside badge text, badge text colour, and per-tab/per-window badge state.
>
> **Adopted rule:**
>
> ```text
> Extension supplies badge visual state
>         ↓
> our host preserves it
>         ↓
> our rail controls placement, not meaning
> ```
>
> Our chrome may control only: badge geometry, minimum/maximum size, placement relative to the action icon, clipping, legibility fallback **when the extension provides no colour**, and focus/hover/disabled framing around the complete action button. We never recolour uBlock Origin's, SponsorBlock's, or any extension's badge to fit the DMS palette. When no badge colours are supplied, Firefox's standard fallback behaviour is preserved unless Spike D establishes a safe browser-owned fallback.

Source: [`architecture-patch-v1.2.1.md`](../architecture/architecture-patch-v1.2.1.md) §2.

The meaning-colour rule that generalizes this correction (Patch v1.2.1 §5, binding for all frames):

> **Meaning-colour rule (binding for all frames):** DMS accent colour never replaces established security, warning, permission, or container colours where those colours carry meaning. Container colours, permission-warning colours, protection states, and extension badge colours (§2) render with their semantic values; our tokens style only the surrounding frames and surfaces.

## Status and gates

- **Gate: Spike D** (Addendum v1.2 §4, extended by Patch v1.2.1 §2 with Q11 and the six badge
  checks; executed by Issue #16). Required recommendation is exactly one of four allowed outcomes;
  custom reimplementation of popups or state is a prohibited outcome.
- **The launcher is not daily-driver-ready until #16 resolves** (Addendum v1.2 §8; hard gate on
  Milestone 9 entry, not on Milestone 2 development).
- Stock-toolbar removal stays reversible until Spikes D and F resolve
  ([ADR-0012](0012-page-actions-and-identity-without-address-bar.md); [`AGENTS.md`](../../AGENTS.md) §4).

## Related risks and readiness

- **R12** extension action incompatibility.
- Readiness: *Extension actions — Ready for source spike (Spike D; ADR-0011 direction accepted,
  mechanism open)* (Addendum v1.2). *Extension badge state — Source mechanism exists (verified:
  `ext-browserAction.js` badge-colour handling on esr153); custom-host rendering blocked on Spike D*
  · *WebExtension page actions — Blocked on Spikes D and F* (Patch v1.2.1 §8).
