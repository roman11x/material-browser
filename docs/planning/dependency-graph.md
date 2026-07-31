# Dependency graph (issues #1–#19)

#1 ──► #2 ──► #15
 │ └──► #3
 └────► #7
#4 (independent)      #5 (independent)      #6 (independent) ──► #13
#3 ─► #8 ──► #9 ──► #10
      │      ├────► #11
      │      ├────► #12
      │      ├────► #16 ◄─(also #8)
      │      ├────► #17
      │      └────► #19 ◄─(also #8)
      ├────► #13
      ├────► #14 (+ #4 evidence)
      └────► #18

Decision joins (block later milestones, not M1 issues):
Spike A(#10) → rail mechanism · Spike B(#11) → M2 design · Spike C(#12) → substrate
memo → human decision · Spike D(#16)+F(#19) → no-address-bar daily-driver gate ·
Spike E(#17) → glass claims gate · #14 → M-Full-Build charter → Case B → M10 claims.
User fixtures → RFC ⛔ sections → M6 adapter (independent of all of the above).
