# DRM (Widevine) records

This directory will hold the living Widevine Case A / Case B document required by
[ADR-0009](../adr/0009-widevine-case-a-case-b-split.md).

**No DRM testing has been performed and no results exist.** Nothing in this repository states or
implies that DRM playback works.

Planned contents:

- **Issue #6 — Widevine Case A, stock control run.** Opens `docs/drm/spike.md` as the living Case A/B
  document: stock Firefox, clean profile, Fedora/Wayland, with the playback step repeated under Niri
  and Hyprland. Requires a legally held streaming account supplied by the human owner.
- **Issue #13 — Widevine Case A on the Artifact build.** Repeats #6's protocol on the Artifact Mode
  build and appends the diff against the control.
- **Case B** — rebranded-distributable feasibility. Not a spike: it is blocked on the full-build
  track by design, whose charter comes from Issue #14. Case B alone controls public DRM claims.

Evidence format for either case:
[`docs/agents/evidence-requirements.md`](../agents/evidence-requirements.md) — about:support media
section, plugin/component state, EME init result, legal test page result, per-service result with an
account-legality note, console/media logs.

Prohibited at every stage ([ADR-0009](../adr/0009-widevine-case-a-case-b-split.md),
[`AGENTS.md`](../../AGENTS.md) §4): copied proprietary CDM binaries, browser-identity spoofing,
undocumented DRM workarounds.
