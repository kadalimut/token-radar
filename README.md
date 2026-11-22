# Token Launch Radar

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/kadalimut/token-radar/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/kadalimut/token-radar/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

An entry for the make.fun bounty [Build the best token launch radar](https://make.fun/bounty/token-radar-01).

**Token Launch Radar** reads a sealed snapshot of every protocol DefiLlama
tracks (758 protocols, snapshot of 2026-11-14), works out which ones still
record no token, and ranks them by what their launch would be worth — then
hands in a web dashboard with the board, a detail view for each protocol,
and the steps to farm it.

## What it ranks

- **Watchlist (40 protocols):** every protocol with at least $50M of TVL
  and no token on record, not a wrapped, staked or bridged product of a
  protocol whose token already trades.
- **Early radar (160 protocols):** the pre-token protocols below the $50M
  floor with the most TVL.
- **Board: 200 rows**, ranks 1–200, every watchlist slug included.
- Live-token protocols are set aside with their market caps shown, never
  mixed into the ranking: a market cap is a fact, not a model.

## The model (radar-2)

One function scores every row — no protocol name is special-cased:

```
implied = round(tvl_usd × 0.30 × growth × maturity × category × chains × stability)
  growth    = 1 + clamp(change_7d, -50%, +150%) / 100 × 0.60
  maturity  = 0.75 + min(age_days, 1460) / 1460 × 0.45   (unknown age: 365 days)
  category  = Dexs 1.25, Lending and Derivatives 1.20, Yield / CDP / RWA 1.15,
              Risk Curators and Allocators 1.10, Bridge 0.55,
              Liquid Staking / Restaking 0.60, everything else 1.00–1.05
  chains    = 1 + min(chains − 1, 4) × 0.03
  stability = 1 − min(|growth|, 60%) / 100 × 0.15   (null growth: 0.97)
```

Why these factors: TVL is the launch's raw material, so it enters
linearly. Fee-earning, user-facing protocols (DEXs, lending, derivatives)
can monetise a token through fees and governance, so they score higher.
Wrapped, bridged and staked products are claims on a token that already
trades, so a standalone launch is worth much less. Older protocols have
survived longer with no token, so a launch is more "overdue"; violent
7-day swings are discounted as unstable. Every other factor is bounded,
so no single week's spike or one extra chain can dominate the board.

Each row also carries a concrete farm plan (6 steps), generated from the
protocol's own category, chains, TVL and age — including a second-chain
step for multi-chain protocols and a category-specific weekly action.

## The dashboard

`dashboard/` is a static, offline page (no network, no external assets):
a ranked board table, a detail view for each protocol (TVL, 7-day growth,
category, chains, age, every multiplier, implied value, farm plan), and
filters — search by name/slug, category, chain, minimum TVL — plus sorting
by implied value, TVL, 7-day growth or name. A live-token tab shows the
protocols that already trade.

## Run it

Requires Node.js 22.18 or later (tested on Node 24.22.1).

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <kit>/input/snapshot.json
```

That writes `out/board.json`, `out/data.js`, `out/dashboard/`,
`out/screenshots/` and `out/dossier.json`. Run it twice and the outputs
are byte-identical: the program reads no clock and no network, and ties
break by slug.

## Layout

- `main.ts` — entrypoint: reads the snapshot, writes the outputs.
- `lib/model.ts` — the radar-2 model, the board builder and farm plans.
- `vendor/makefun-board.ts` — make.fun's dossier library, copied verbatim.
- `dashboard/` — the offline dashboard (`index.html`, `app.css`, `app.js`).
- `screenshots/` — board, detail view and filtered view of the dashboard.

Implied value is a model, not a price and not a promise that any protocol
will issue a token.

---

<sub>Built for the make.fun bounty by @sukengopi. Sign up with my referral link: https://make.fun/r/8npzgr73 · code <code>8npzgr73</code></sub>
