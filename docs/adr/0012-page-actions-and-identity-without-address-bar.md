# ADR-0012 — Page actions and identity without an address bar

**Status:** Accepted (direction); mechanism gated on Spike F
**Governing text:** Architecture Patch v1.2.1 §3
**Amendment chain:** v1.2.1 §3 (original — governs)

## Decision (verbatim, v1.2.1 §3)

> **ADR-0012 — Page actions and identity without an address bar**
>
> **Status:** Accepted (direction); mechanism gated on Spike F.
> **Context:** removing the address bar removes the visual home of Firefox's built-in page controls: tracking-protection status/controls, origin identity and connection security, site permissions and indicators, Reader Mode, bookmark star/state, translation, PiP/media page indicators where applicable, zoom state, container/contextual-identity indication, Firefox-provided page actions, WebExtension page actions, temporary permission indicators, and any URL-bar actions future ESRs introduce. None of these may silently disappear. The origin chip covers identity/security display but not how the full set of panels and actions is invoked.
> **Decision:**
>
> ```text
> Firefox identity, permission, and page-action infrastructure
>         ↓
> BrowserPageActionAdapter owned by us
>         ↓
> origin chip + contextual page-actions panel + launcher discovery
> ```
>
> - Reuse Firefox's real identity, permission, tracking-protection, bookmarking, Reader Mode, translation, and page-action infrastructure; never recreate the underlying security or permission state.
> - The origin chip is the primary anchor for identity, security, tracking protection, and site permissions; activating it opens the real browser-owned identity/security panel, or an adapter-hosted presentation preserving its semantics.
> - A contextual **Page Actions** control provides non-security actions relevant to the current page; it may appear beside the chip only when relevant (`[ shield/lock github.com ] [ page actions ⋯ ]`), or the two may merge into one expanded origin panel **if** source inspection shows that preserves clarity and upstream behaviour — this choice is not finalized before Spike F.
> - The launcher may expose page actions for keyboard discovery/activation but is never the sole interface for important permission or security state.
> - Contextual actions retain tab-specific visibility and enabled state; built-in and extension-provided page actions remain distinguishable; unknown future upstream actions degrade into a discoverable generic panel rather than vanish.
>
> **Consequences:** the chip's role expands from indicator to anchor; Spike F becomes a gate for no-address-bar daily-driver mode alongside Spikes B and D; stock-toolbar removal must stay reversible until Spike F and #16 resolve (§6, #19 relationships).

Source: [`architecture-patch-v1.2.1.md`](../architecture/architecture-patch-v1.2.1.md) §3.

## Status and gates

- **Gate: Spike F** (Patch v1.2.1 §4; executed by Issue #19). Required recommendation is exactly one
  of four allowed outcomes; reimplementing security, permission, tracking-protection, or bookmark
  state is prohibited.
- **No-address-bar daily-driver mode is not ready until #16 and #19 both resolve**; launcher
  foundation work (M2) may begin before it, but **stock-toolbar removal remains reversible until the
  result is known** — a revert path is part of M2's definition of done (Patch v1.2.1 §6).
- The origin chip's *visual* role is approved; its complete Firefox integration is blocked on
  Spike F (Patch v1.2.1 §8).
- Escalation trigger (Work Plan #19): *stock panels cannot re-anchor without content copying →
  architecture-question on ADR-0012 before M2/M3 chip implementation.*

## Readiness deltas (verbatim, v1.2.1 §8)

> - **Security chip visual design and navigation/domain prominence contract** — Ready for Milestone 0.
> - **Identity, tracking-protection, permission-panel anchoring, and complete page-action integration** — Ready for source spike; blocked on Spike F before implementation is considered complete.
> - **Built-in identity/security host** — Ready for source spike (visual role approved, mechanism open).
> - **Built-in page actions** — Blocked on source spike (F).
> - **WebExtension page actions** — Blocked on Spikes D and F.
> - **No-address-bar daily-driver mode** — Blocked on Spikes B, D, and F.
> - **Origin chip design** — Approved.
> - **Origin chip complete Firefox integration** — Blocked on Spike F.

## Related risks

- **R16** built-in page actions and identity controls become inaccessible.
- Related: [ADR-0011](0011-extension-actions-without-traditional-toolbar.md) (extension actions;
  shared meaning-colour rule and shared toolbar-removal reversibility constraint).
