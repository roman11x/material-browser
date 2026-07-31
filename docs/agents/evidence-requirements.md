# Evidence requirements

## Source-verification metadata block (all spike docs, all source claims)
Repository / Branch / Commit / Firefox version (version.txt + version_display.txt) /
Date inspected / Artifact or full build / Relevant feature preferences.
One block per branch when comparing.

## Baseline verification (local)
`git rev-parse HEAD` · `git status` · `git branch --show-current` ·
`cat browser/config/version.txt` · `cat browser/config/version_display.txt`
Recorded: remote URL, branch, commit, version files, checkout date, tree-clean state.
Local tip differing from docs/baselines/firefox-baselines.md → baseline-update entry, never a silent
replacement.

## Artifact Mode build evidence (every Artifact build claim)
Source revision · artifact revision · artifact job/platform · artifact timestamp ·
`./mach artifact last` output · objdir clean/dirty · clobber events recorded.
Source/artifact mismatch handling per Issue #8 rules; never continue unexplained
(Risk R14).

## Test evidence
Exact command · unedited output (explicit trim markers only) · environment
(compositor, scaling) for UI tests · NOT-RUN list with reasons.

## DRM evidence (Case A/B)
about:support media section · plugin/component state · EME init result · legal test
page result · per-service result with account-legality note · console/media logs.
