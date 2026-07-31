# Patches

Project patches against upstream Firefox source will live here. The directory is empty today: no
Firefox source has been checked out and no patch has been written.

Rules, from [`AGENTS.md`](../AGENTS.md) §6:

- patches are narrow, modular and rebase-friendly; prefer adapters, providers, services and new
  modules over edits inside `browser.xhtml` or upstream classes;
- every touched upstream file enters the conflict watch set in
  [`docs/maintenance/`](../docs/maintenance/README.md);
- patches target the pinned baseline recorded in
  [`docs/baselines/`](../docs/baselines/README.md); baselines change only via a recorded
  baseline-update entry.

No Firefox source is vendored into this repository
([ADR-0001](../docs/adr/0001-upstream-base-raw-firefox-source.md)).
