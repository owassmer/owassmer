# Engine-semantics probes (run each in the viewer BEFORE B2 code depends on it)

Pattern for all probes: tiny program, encode result in registers, run a practice map,
click an ant, read registers. One probe per run; note result in SESSION_LOG.md and
promote the fact to the verified list.

Syntax below is a best-effort reconstruction — check every line against the
reference tab before running; fix mnemonics/operand order there, not semantics.

## Probe A — MARK with register amount

Decides: distance-graded nest-field deposits (register) vs banded literals.

```
; expect: if register MARK works, r5 climbs to ~77+ (additive, minus decay)
SET r4 77
MARK CH_GREEN r4
SNIFF CH_GREEN HERE r5
; park forever: no MOVE, so ant stays put and keeps re-marking
JMP 0            ; or loop label — check reference for jump syntax
```
Read r5 after ~10 ticks. r5 >= 76 and climbing → register MARK works.
r5 == 0 or parse error → use banded literals in B2 (already the default plan).

## Probe B — PROBE with register direction

Decides: whether safe_move can PROBE the rolled direction directly, or must
branch to 4 literal-direction PROBEs (costs ~6 extra ops/attempt).

```
SET r1 2            ; E
PROBE r1 r5         ; register dir — may not parse
PROBE E r6          ; literal dir — control
; r5 == r6 (and program parses) → register PROBE ok
JMP 0
```

## Probe C — .const with numeric values

```
.const NEST_BAND 12
SET r5 NEST_BAND
JMP 0
```
Parses and r5 == 12 → named numeric dials allowed (B5 tuning much cleaner).
Parse error → inline literals with a comment block listing dial values.

## Probe D — ants spawn at the nest (tick 0)

Dead-reckoning zero-point assumption. Program: do nothing (`JMP 0`) and visually
confirm all 200 ants sit on the nest cell(s) at tick 0. Also confirms whether the
nest is a single cell or a region (matters for DROP-on-nest reset of (dx,dy)).

## Probe E — SENSE ANT and ID register

```
SENSE ANT r5        ; direction of nearest ant? check reference for operand form
; ID: reference claims ants differ by ID — find how ID is read (register? op?)
JMP 0
```
Needed only for B6 role splits; low priority.

## Probe F (cheap, same run as D) — register value range / signedness

SET r5 0, then SUB r5 1 (or DEC) once at tick 0. Read r5: 255 → 8-bit wrap
(use offset-128 encoding for dx,dy); -1 → signed ok (use raw dx,dy).
B2's dead-reckoning band selector depends on this.
