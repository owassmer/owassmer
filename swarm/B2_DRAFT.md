# B2 — Nest field on CH_BLUE (DRAFT, ungated, syntax unchecked)

Theory trace: P1 nest-rooted geodesic. Replace L1-potential homing (spurious minima
on every separating obstacle boundary — the 74→856 gap) with ascent of an
approximate nest-distance field written by the ants themselves. P2 gate: no ant
pinned/orbiting on fortress/bridge; CH_BLUE overlay zero inside pockets.

## Field construction

- Every ant, every tick it MOVEs, deposits on CH_BLUE an amount that DECREASES with
  its dead-reckoned L1 distance from nest. Ascending CH_BLUE then points nestward
  along *walked corridors* (graph paths, not L1 rays) — that is the whole point:
  the field only exists on cells ants actually traversed, so its ascent cannot
  lead into a wall pocket the way the L1 potential does.
- Deposit bands (literals, pending Probe A; L1 = |dx| + |dy| from dead reckoning):
  - L1 <  8  → MARK CH_BLUE 10
  - L1 < 24  → MARK CH_BLUE 6
  - L1 < 56  → MARK CH_BLUE 3
  - else     → MARK CH_BLUE 1
  Rationale (P4): decay is 1/tick globally, cap 255. Near-nest cells see high
  traffic × high deposit → will saturate; acceptable because the gradient only
  needs to be informative far from the nest where deposits are small. If the
  overlay shows a saturated plateau extending past ~L1 15, halve the top band
  before touching anything else (one dial at a time).
- Dead reckoning is ONLY a band selector + last-resort fallback here. Its L1 error
  from reroll drift is tolerable for banding (bands are 16–32 wide); navigation
  authority is the field. Re-zero (dx,dy) on every successful DROP at nest.

## Homing (CARRYING == 1)

1. On nest → DROP, reset dx=dy=128 (offset encoding pending Probe F), state=search.
2. Jitter: with prob ~1/8 (low 3 bits of an LCG/RNG register == 0), take a random
   PROBE-validated direction instead of the gradient step. P2 escape hatch for
   residual field defects; do NOT raise above 1/8 to paper over a bad deposit rule.
3. Otherwise SNIFF CH_BLUE at HERE and 4 dirs; move to the max neighbor that is
   (a) strictly > HERE (validated ascent — prevents oscillation on flat field),
   (b) PROBE-passable.
4. If no neighbor qualifies (field absent/flat): fallback = safe_move toward
   sign-direction of dead reckoning (B1 behavior).

## Search (CARRYING == 0) — unchanged from B1 in this stage

SMELL food; if found and adjacent PICKUP; else B1 wander via safe_move.
(Do NOT fold in CH_RED recruitment or CH_GREEN dispersal — those are B3/B4,
separately gated.)

## Register allocation

| reg | use |
|-----|-----|
| r0  | dx (offset 128 pending Probe F) |
| r1  | dy (offset 128) |
| r2  | scratch (PROBE results, SNIFF values) |
| r3  | subroutine return (flat CALLs only — B1 convention) |
| r4  | scratch / best-dir accumulator |
| r5  | best-value accumulator in gradient scan |
| r6  | RNG/jitter state |
| r7  | safe_move retry counter |

## Op budget sketch (worst-case tick, cap 64)

- band select + MARK: ~8
- 5× SNIFF + max-scan compare chain: ~22
- strict-ascent check + PROBE validation + possible reroll: ~12
- jitter branch + RNG update: ~6
- MOVE + dead-reckoning update: ~5
- total ≈ 53 — inside budget; no room for casual additions (anti-bloat).

## Gate (must pass before B3)

Panel: fortress, gauntlet, pockets, maze, bridge + open canary, fixed seeds.
Ship iff strict improvement in summed delivered AND no open regression AND
P2 diagnostic clean: tag a homer on fortress/bridge and watch it round the
obstacle; CH_BLUE overlay hugs corridors, zero inside pockets.
