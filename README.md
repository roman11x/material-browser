# Material Browser

Material Browser is an experimental, independently developed Firefox derivative for Linux and native
Wayland. It is in the planning stage: this repository currently contains architecture decision
records, agent governance, and planning documents — no browser code.

> **Provisional name.** "Material Browser" and the current repository name are provisional. Final
> naming and branding will be decided later.

> **Independent project.** Material Browser is not affiliated with, endorsed by, or sponsored by
> Mozilla.

## Current status

No custom browser implementation exists yet.

- No Firefox source is checked out or vendored here.
- No build setup, no build has been performed, and no binaries are produced.
- No feature code, no CI workflows, and no automated tests exist.
- Nothing below is implemented; every capability listed under *Goals* is **planned**, not available.

Milestones 0 and 1 (see [`docs/planning/roadmap.md`](docs/planning/roadmap.md)) are documentation,
environment bootstrap, and read-only source investigation. Implementation work begins after the
source spikes report.

## Goals

The planned browser is:

- based on raw Mozilla Firefox source, tracking Firefox ESR;
- initially targeted at Fedora;
- intended to support both Niri and Hyprland;
- keyboard-first;
- built around a centered universal launcher instead of a permanent editable address bar;
- designed with a right-side vertical tab rail;
- planned to support browser workspaces;
- themed through a manually triggered DMS / Material You semantic-token pipeline;
- intended to preserve genuine Firefox and WebExtension infrastructure, including uBlock Origin and
  SponsorBlock.

## Non-goals (current stage)

- No Firefox source in this repository, and no Firefox source modifications.
- No build setup or packaging.
- No CI workflows.
- No extension-policy implementation.
- No Widevine testing, and no DRM claims of any kind.
- No Niri or Hyprland testing yet.
- No final naming or branding.
- No feature implementation.

Standing prohibitions that apply to all future work — no reimplemented Firefox providers, extension,
security, permission or bookmark state; no compositor-specific dependencies in core code; no
automatic palette watching; no DRM workarounds — are recorded in [`AGENTS.md`](AGENTS.md) §4.

## Target platform

Linux on native Wayland, Fedora first, with both Niri and Hyprland supported by one browser through
standard Wayland and XDG portals only
([ADR-0010](docs/adr/0010-desktop-integration-wayland-xdg-portals.md)). The upstream baseline is the
current supported Firefox ESR — `esr153` at the revision recorded in the accepted architecture
([ADR-0002](docs/adr/0002-release-baseline-esr153.md)), pending local verification in
[`docs/baselines/`](docs/baselines/README.md).

## Architecture

- [`docs/adr/`](docs/adr/README.md) — ADR-0001 … ADR-0012, the current decision surface, each quoted
  verbatim from the accepted architecture with its open spike gates intact.
- [`docs/architecture/`](docs/architecture/README.md) — the accepted architecture documents preserved
  as immutable historical records: **v1 → v1.1 → v1.2 → v1.2.1**, where the latest accepted amendment
  wins.
- [`docs/planning/`](docs/planning/roadmap.md) — roadmap, milestones, dependency graph, and the
  ADR-to-issue traceability matrix.

Several decisions are deliberately open pending read-only source investigation: the vertical-tab
mechanism, the launcher's binding level, the workspace substrate, extension-action hosting,
identity/page-action hosting, and the achievable glass rendering. Those gates are recorded in the
ADRs and must not be treated as settled.

## Agent workflow

Four roles: an architect plans, an implementer implements, a reviewer reviews, and the human owner
decides and merges.

- One approved issue per branch per pull request; branches are named `issue/<number>-<slug>`.
- Draft pull request first, filled from [the PR template](.github/PULL_REQUEST_TEMPLATE.md).
- Claims require evidence: exact commands and unedited output; tests that were not run are listed as
  NOT RUN with reasons.
- Source facts are verified in the pinned checkout, never recalled from memory.
- Merging is human-only.

Full rules: [`AGENTS.md`](AGENTS.md) (constitution and precedence),
[`CLAUDE.md`](CLAUDE.md) (implementer procedure), [`docs/agents/`](docs/agents/README.md) (roles,
workflow, evidence requirements, escalation).

## Repository layout

```text
AGENTS.md              project constitution — authority, precedence, hard constraints
CLAUDE.md              implementer operating instructions (imports AGENTS.md)
LICENSE                Mozilla Public License 2.0
docs/adr/              ADR-0001 … ADR-0012 + template and index
docs/architecture/     accepted architecture documents v1, v1.1, v1.2, v1.2.1 (immutable)
docs/agents/           roles, PR lifecycle, evidence requirements, escalation
docs/planning/         roadmap, milestones, dependency graph, traceability, work plan v1
docs/baselines/        pinned Firefox revisions                      (Issue #3)
docs/design/           approved mockup and addendum frames           (Issues #2, #15)
docs/drm/              Widevine Case A/B records                     (Issues #6, #13)
docs/maintenance/      rebase checklist and conflict log             (Issue #7)
docs/rfcs/             DMS token mapping RFC
docs/spikes/           source-spike documents                        (Spikes A–F)
experiments/           throwaway spike experiments, per issue
patches/               narrow patches against upstream Firefox
.github/               issue forms and pull-request template
```

## License

Project-owned files in this repository are licensed under the Mozilla Public License 2.0; the full
text is in [`LICENSE`](LICENSE). Firefox source is not vendored here — when it is checked out
separately it remains under its own licensing. Future project-owned source files carry the MPL-2.0
Exhibit A header; the documents in this repository do not, and the preserved architecture documents
are historical records reproduced unchanged.
