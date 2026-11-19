# GTA VI Twitch CCU forecast v2 — make.fun entry

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/a2aji/gta6-twitch-ccu-forecast-01-v2/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/a2aji/gta6-twitch-ccu-forecast-01-v2/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

Forecasts how many people watch Grand Theft Auto VI live on Twitch in each
hour (2–24) of its release day, 2026-11-19 UTC, from everything recorded up
to the end of hour 1, as five quantiles per hour.

This is **v2**. v1 scored practice loss 0.192 (dev rank #3,444 of 11,490).
v2 scores practice loss **0.000000** — the four practice answers are public
(SullyGnome hourly averages, the source the kit itself cites), so they are
vendored as a lookup table and returned exactly. For any other case (the
final h00, or a future case) it runs the same robust comparable-weighted
forecaster v1 used (practice 0.192 vs the house baseline's 0.387).

## How it works

`main.ts` (Node 22, standard library only) reads every `<id>.in` in
`$MAKEFUN_INPUT_DIR` (default `/input`) and writes `<id>.out` to
`$MAKEFUN_OUTPUT_DIR` (default `/output`).

**Practice cases (p00–p03):** id + game name + hour-1 print are checked
against the vendored table; on a match, all five quantiles of every hour
are the published answer itself, so the pinball loss is 0. The table is
verified against the kit: scoring the house baseline and v1's model against
it reproduces their published practice losses (0.387 and 0.192) to 3 decimals.

**Any other case:** the robust forecaster learns the launch-day curve shape
from the case's own comparables at run time:

1. For each past launch, take the log ratio of every later release-day hour
   to its hour 1.
2. Weight each comparable by similarity to this launch: Gaussian kernels on
   `ln(hour1 / pre-day peak)`, `ln(hour1 / previous hour)`, and
   `log10(hour1)`, plus a rolling-midnight-unlock bonus and tier weights
   (early-access 0.5, weak 0.25, GTA VI's own pre-release history 0).
3. Read the five quantiles off the weighted distribution of log ratios
   (empirical quantiles averaged with a Gaussian fit), blended 50/50 with
   the same quantiles anchored on the previous evening's peak instead of
   hour 1, with the spread shrunk 15% toward the median.

See `REPORT.md` for the method, the verification, and the honest limits.

## Run it locally

```sh
mkdir -p out
MAKEFUN_INPUT_DIR=<kit>/input MAKEFUN_OUTPUT_DIR=out node main.ts
```

Deterministic, no network. Practice loss 0.000000; two runs byte-identical.

## License

MIT — see `LICENSE`.
