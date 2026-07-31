# Architecture Decision Records

ADR-0001 through ADR-0012 are the current decision surface of Material Browser. Each file quotes its
decision **verbatim** from the accepted architecture documents preserved in
[`docs/architecture/`](../architecture/README.md) and cites the exact document and section. Nothing
here paraphrases, strengthens, or resolves a decision.

## Reading rule

Decisions are assembled along the amendment chain:

```text
v1 decision text → v1.1 amendments/replacements → v1.2 amendments → v1.2.1 amendments
```

The latest accepted amendment wins. Where v1.1 replaced a v1 decision, the v1 text appears in the
ADR only under **Superseded (historical)** and is not current architecture.

## Index

| ADR | Title | Status | Governing text | Gate |
|---|---|---|---|---|
| [0001](0001-upstream-base-raw-firefox-source.md) | Upstream base: raw Mozilla Firefox source | Accepted | v1 §1 (unchanged in v1.1) | — |
| [0002](0002-release-baseline-esr153.md) | Release baseline: ESR 153, with recorded revisions | Accepted (revised in v1.1) | v1.1 §1 | Re-verify supported ESR at release-branch time; Issue #3 local verification |
| [0003](0003-build-strategy-artifact-mode.md) | Build strategy: Artifact Mode; corrected full-build boundary | Accepted (revised in v1.1) | v1.1 §1 | Issue #14 full-build boundary document |
| [0004](0004-vertical-tabs-native-sidebar-first.md) | Vertical tabs: native sidebar implementation first | Accepted (evidence rebased in v1.1) | v1 §1 + v1.1 §1 | **Spike A** (Issue #10) |
| [0005](0005-workspace-substrate-undecided.md) | Workspaces: substrate deliberately undecided | Accepted; substrate open | v1 §1 (unchanged in v1.1) | **Spike C** (Issue #12) → memo → human decision |
| [0006](0006-launcher-browser-query-adapter.md) | Launcher: alternate presentation layer, frontend-agnostic | Accepted (revised in v1.1, amended in v1.2) | v1.1 §1 + v1.2 §2 | **Spike B** (Issue #11) |
| [0007](0007-launcher-activation-contract.md) | Launcher activation contract (firm) | Accepted — firm | v1 §1 (unchanged in v1.1) | Hook point from Spike B; six tests embedded in M2 issues |
| [0008](0008-theming-manual-dms-semantic-tokens.md) | Theming: manual DMS sync through a semantic token layer | Accepted | v1 §1 (unchanged in v1.1) | RFC ⛔ sections blocked on human-owner fixtures |
| [0009](0009-widevine-case-a-case-b-split.md) | Widevine: Case A / Case B split | Accepted | v1 §1 (unchanged in v1.1) | Case B blocked on the full-build track; controls all public DRM claims |
| [0010](0010-desktop-integration-wayland-xdg-portals.md) | Desktop integration: standard Wayland + XDG portals only | Accepted | v1 §1 (unchanged in v1.1) | — |
| [0011](0011-extension-actions-without-traditional-toolbar.md) | Extension actions without a traditional toolbar | Accepted (direction); mechanism gated | v1.2 §3 + v1.2.1 §2 | **Spike D** (Issue #16) |
| [0012](0012-page-actions-and-identity-without-address-bar.md) | Page actions and identity without an address bar | Accepted (direction); mechanism gated | v1.2.1 §3 | **Spike F** (Issue #19) |

## Amendment provenance

| ADR | v1 | v1.1 | v1.2 | v1.2.1 |
|---|---|---|---|---|
| 0001 | original | unchanged | — | — |
| 0002 | original | **revised (governs)** | — | — |
| 0003 | original | **revised (governs)** | Issue #8 evidence AC | Issue #8 evidence AC |
| 0004 | original decision | evidence rebased to `esr153` | — | — |
| 0005 | original | unchanged; Spike C rebased | Spike C list expanded | — |
| 0006 | original | **revised** | **amended (governs)** | — |
| 0007 | original (incl. the six required tests) | unchanged | — | — |
| 0008 | original | unchanged; RFC §10/§15/§16 revised | — | — |
| 0009 | original (incl. Case B's six report axes) | unchanged | — | — |
| 0010 | original | unchanged | — | — |
| 0011 | — | — | **original** | **corrected (badge ownership)** |
| 0012 | — | — | — | **original** |

## Adding or amending an ADR

New ADRs use [`adr-template.md`](adr-template.md) and follow the escalation procedure in
[`docs/agents/escalation.md`](../agents/escalation.md): a source fact that contradicts an accepted
ADR is documented, escalated as an `architecture-question` issue, decided by the human owner, and
then recorded as a dated amendment entry — never as a silent rewrite.
