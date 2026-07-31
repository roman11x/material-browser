# RFCs

This directory will hold project RFCs. It is empty today.

**Planned: `docs/rfcs/0001-dms-token-mapping.md`** — the DMS Token Mapping RFC governing
[ADR-0008](../adr/0008-theming-manual-dms-semantic-tokens.md). Draft 0.1 is recorded in Architecture
Pack v1 Section 5; draft 0.2 revises §10 (contrast validation over full compositing stacks), §15
(fixtures and regression method) and §16 (accessibility) in Architecture Pack v1.1 Section 5.

The RFC pipeline it governs: `dank-pywalfox.json → palette reader/validator → semantic --browser-*
tokens → browser chrome`.

Sections marked **⛔ BLOCKED** in the drafts stay blocked until the human owner supplies the palette
fixtures: §2 source contract, §5 mapping table, §8 accent-preference ordering, §13 light palettes,
and the real-palette fixtures in §15. **No DMS key names may be invented** — those sections stay
empty until fixtures arrive.

Settled and unblocked already: the token taxonomy, derivation formulas, alpha/glass rules, state
layers, adjustment hierarchy, fallback and last-known-good behaviour, versioning and error strings,
and the provisional built-in dark palette (which is the fallback and regression reference, explicitly
not the DMS schema).
