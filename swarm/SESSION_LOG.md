# SWARM Ant Colony Challenge — Experiment Log

Running record of (program version, panel results, score, verdict). Never repeat an experiment.

## Status

| Version | Description | Panel (fortress/gauntlet/pockets/maze/bridge + open) | Score | Verdict |
|---------|-------------|------------------------------------------------------|-------|---------|
| default | Engine default program | — | 45 | baseline |
| B1 | Validated movement (PROBE-before-MOVE, reroll ≤5, SENSE EMPTY fallback) | — | 74 | shipped (prior session) |
| B2 | Nest-distance field on CH_BLUE (P1 nest-rooted) | — | — | DRAFT, ungated — see B2_DRAFT.md |

Target: 856.

## Session 2026-07-20 (remote container) — BLOCKED: no network access to challenge host

- Environment: Claude Code remote container, outbound HTTPS via org egress proxy.
- `https://dev.moment.com/swarm` → CONNECT tunnel rejected with **403 policy denial**
  by the egress gateway (verified via `curl --cacert /root/.ccr/ca-bundle.crt` and
  the proxy status endpoint's `recentRelayFailures`). All external hosts tested are
  denied; this session's egress policy is effectively no-internet.
- Consequence: no viewer access → no probes, no panel runs, no scores. Per operating
  principles (empiricism-second, gated stages), **no program changes can ship from
  this session**.
- Work done instead (all offline, clearly marked unvalidated):
  - `PROBES.md` — minimal probe programs for every UNVERIFIED engine semantic.
  - `B2_DRAFT.md` — full B2 (nest field) design traced to P1/P2, with register
    allocation and op-budget accounting. Syntax UNCHECKED against the reference tab.
- Unblock options: run the session from an environment whose network policy allows
  `dev.moment.com` (e.g. "trusted network access" or a custom allowlist including
  that host), or iterate locally in a browser and use this repo only as the log.

## Verified engine facts (axioms — do not re-derive)

- H = 2000 ticks; practice viewer == eval config.
- Every action (MOVE/PICKUP/DROP) consumes the tick even on failure; blocked MOVE
  loses the tick; MOVE 0 burns a tick stationary.
- Non-action ops free up to 64 ops/tick; hitting the cap ends the tick, PC persists.
- PC and registers persist across ticks; registers init to 0.
- SENSE/SMELL direction codes: N=1 E=2 S=3 W=4, 0 = none, ties random.
- MARK additive, cap 255, decay 1/tick, no diffusion, walls hold no pheromone.
- Baselines: default 45, B1 74. ρ = S(500)/S(0) = 0.84 → plateau regime.

## UNVERIFIED (probe before any code depends on it)

1. MARK with register amount (vs banded literals).
2. PROBE with register direction.
3. `.const` with numeric values.
4. Ants spawn at nest at tick 0.
5. SENSE ANT / ID-based role splits behave as documented.
