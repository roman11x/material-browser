# Source spikes

This directory will hold source-spike documents. It is empty today — no spike has been run and no
Firefox checkout exists.

Every spike document opens with the source-verification metadata block defined in
[`docs/agents/evidence-requirements.md`](../agents/evidence-requirements.md), locates paths in the
pinned checkout rather than guessing them, and ends with the required recommendation chosen only from
its allowed set.

| Spike | Subject | Issue | Gates |
|---|---|---|---|
| A | Native vertical tabs / sidebar (`esr153`) | #10 | Rail mechanism, [ADR-0004](../adr/0004-vertical-tabs-native-sidebar-first.md) |
| B | URL-bar / Nova / Smartbar infrastructure and the provider-contract comparison | #11 | Adapter binding level and the M2 design note, [ADR-0006](../adr/0006-launcher-browser-query-adapter.md) |
| C | Tab visibility, tab groups, SessionStore | #12 | Workspace substrate memo, [ADR-0005](../adr/0005-workspace-substrate-undecided.md) |
| D | WebExtension action and unified-extensions infrastructure | #16 | Extension hosting choice, [ADR-0011](../adr/0011-extension-actions-without-traditional-toolbar.md); launcher daily-driver readiness |
| E | Browser-chrome glass and compositor feasibility | #17 | Achievable glass rung; no frosted-blur release claim until resolved |
| F | Built-in page actions, identity and permission surfaces | #19 | Identity/page-action hosting choice, [ADR-0012](../adr/0012-page-actions-and-identity-without-address-bar.md) |

A further investigation, the sensitive-field signal investigation (Issue #18), produces a document
here on the same terms; its outcome selects the security chip's sensitive-state trigger rung.

Experiment code belonging to a spike is throwaway, lives under
[`experiments/`](../../experiments/README.md), and never ships or is imported by product code.
