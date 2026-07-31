# Architecture traceability matrix

Source: [`docs/planning/agent-collaboration-work-plan-v1.md`](agent-collaboration-work-plan-v1.md)
§9, committed verbatim as that plan's §5 directs ("committed verbatim and kept current by the
architect at each milestone close"). The matrix below is the work plan's text; nothing is added,
removed, or reworded.

---

Format: ADR → milestone → issue(s) → expected PR(s) → required tests/verification → relevant risks. PRs are named by their issue (`PR-#n`). "Later" = issues not yet written by design (M2+ issues are authored after their gating spikes, per the accepted workflow).

| ADR | Milestone | Issue(s) | Expected PR | Verification | Risks |
|---|---|---|---|---|---|
| 0001 raw source | M0 | #1, #3 | PR-#1, PR-#3 | baseline transcripts; ADRs committed | R3 |
| 0002 ESR 153 baseline | M0→ongoing | #3, #7 | PR-#3, PR-#7 | local verification; checklist + seeded log | R1–R3, R14 |
| 0003 Artifact/full-build boundary | M1 | #8, #14 | PR-#8, PR-#14 | amended #8 AC set; charter | R5, R14 |
| 0004 native tabs first | M1→M4 | #10; M4 later | PR-#10 | Spike A classification + a11y evidence | R2 |
| 0005 workspace substrate open | M1→M7 | #12; memo + M7 later | PR-#12 | matrix evidence; hard gates | R8 |
| 0006 launcher/adapter | M1→M2 | #11; M2 later | PR-#11 | contract comparison; M2 design note | R1 |
| 0007 activation contract | M2 | **later (gap, planned)** | — | the six named tests, written into M2 issues verbatim | R1 |
| 0008 DMS tokens/manual sync | M6 | **blocked on fixtures**; RFC ⛔ | — | token snapshots + computed-style CI; fixture tests | R7 |
| 0009 Widevine A/B | M0/M1 + full-build track | #6, #13; Case B later | PR-#6, PR-#13 | DRM evidence sets; Case B classification | R4 |
| 0010 Wayland/portals | M0 | #5 | PR-#5 | per-compositor findings table | R10, R11 |
| 0011 extension actions | M1→M9 gate | #16, #15 | PR-#16, PR-#15 | Spike D recommendation + badge checks | R12 |
| 0012 page actions/identity | M1→M9 gate | #19, #15 | PR-#19, PR-#15 | Spike F recommendation + control lists | R16 |

**Product-defining decisions currently lacking an implementation/verification issue (called out per instruction; all are *deliberate sequencing*, not oversights, but they are real gaps until written):**
1. **ADR-0007 activation contract** — no issue exists yet; M2 issues must embed its six tests verbatim. Owner: architect, after #11.
2. **ADR-0008 adapter implementation** — RFC ⛔ sections and the M6 adapter have no issues; blocked on user fixtures, then architect writes them.
3. **D1 security-chip state machine and D2 rail keyboard model** — design accepted, implementation issues arrive with M3/M4 after #10/#19.
4. **Workspace implementation (post-memo)** — awaits the human substrate decision.
5. **Glass presets → rung compilation (RFC §7 after #17)** — M3 issue to be written with the rung known.
6. **Screenshot-regression spike (RFC §15)** — approved as a spike but not yet numbered; recommend adding as **#20** when M1 issues are opened, or folding into M3 entry. Flagged for the human.
7. **Case B execution** — charter comes from #14; the executing issues belong to the full-build track and are unwritten.
