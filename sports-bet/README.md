# Sports-Bet — Odds-Movement Analysis & ML Signal Detection

**Does the way a betting line moves predict the outcome? A full-stack system built to answer
that question rigorously — per sport, with statistics first and models second, validated the
only honest way: walk-forward.**

Sports-Bet treats the betting market as a time series worth studying. It polls pre-game
consensus odds hourly across nine leagues (NFL, NBA, MLB, NCAAB, NCAAF, EPL, UCL, WNBA, NHL),
stores every snapshot, and mines the movement history for predictive structure: late moves,
sharp moves, reverse-line movement, key-number crossings.

> Deployed as a Railway backend (FastAPI + hourly cron worker) with Azure SQL storage and a
> Next.js frontend on Vercel. The system ran live through the 2025–26 seasons; the collector is
> currently paused between seasons, so this write-up is architecture-focused.

## The pipeline

```
The Odds API — hourly polling + a smart historical backfill
(coarse baseline cadence, densified near kickoff)
        │
        ▼
Azure SQL — event metadata + hourly consensus snapshots
(strictly pre-game: every query is lookahead-guarded at the timestamp level)
        │
        ▼
Feature engineering — 40+ movement features per event
open→close deltas · movement velocity · late-window moves · sharp-move and
reverse-line flags · variance regimes · key-number crossings — with every
threshold and bucket boundary calibrated per sport
        │
        ├──► Statistical layer — a battery of hypothesis tests per sport,
        │    each reporting p-value AND effect size, so "significant but
        │    tiny" edges are visible for what they are
        │
        ├──► Model layer — LogReg · Random Forest · GBM · XGBoost,
        │    trained per sport × target (ATS / total / moneyline), with
        │    cross-validated selection and feature-importance reporting
        │
        └──► Live signal detector — scores in-flight games with calibrated
             confidence, fires push alerts, then grades every alert against
             the actual outcome
```

## The methodology is the product

- **Statistics before models.** Before anything is "predicted," each candidate signal is put
  through per-sport hypothesis tests. A movement pattern only graduates to a model feature if
  the data says it deserves to — and the significance dashboard shows exactly which sports and
  which patterns carry real effect sizes.
- **Every sport is its own market.** Spread buckets align to each sport's key numbers (3/7/10/14
  in the NFL; 5/7/10 in the NBA), movement-magnitude bins are sport-calibrated, MLB is treated
  as a moneyline market (run lines barely move), and soccer/hockey run moneyline-only variants.
  Nothing is pooled across leagues that behave differently.
- **Walk-forward validation, no exceptions.** Models are evaluated only on games that occur
  after their training window, retrained as the window rolls forward — the backtest simulates
  deployment rather than leaking the future. In-sample accuracy is treated as marketing, not
  evidence.
- **Outcome-tracked alerting closes the loop.** Every live signal fired in production is graded
  after the game completes (correct / incorrect / push), so the system's real-world precision
  is measured on its actual alerts — retroactive model picks are shown alongside results for
  every completed game.
- **Regime awareness.** Playoff games are tagged per league (March Madness rounds, NFL
  playoffs, NBA play-in → Finals) and analyzable as separate regimes from the regular season.

## KenPom March Madness module

A dedicated NCAA-tournament ATS predictor built on six seasons of KenPom efficiency data
(2021–2026): feature-importance rankings across efficiency-metric deltas, seed-matchup ATS
grids, round-by-round splits, and year-by-year validation of the model's tournament picks — a
self-contained study in how far public efficiency metrics can be pushed against the spread.

## Engineering notes

- **Backend:** Python 3.12 · FastAPI · SQLAlchemy 2 · pandas / numpy / scipy · scikit-learn ·
  XGBoost · async httpx collector with batched upserts
- **Frontend:** Next.js 14 · TypeScript · Tailwind · Recharts — odds-movement line charts,
  statistical-significance heatmap grids, model-performance and feature-importance dashboards
- **Infra:** Railway (API + cron worker) · Azure SQL · Vercel

---

*The Sports-Bet codebase is proprietary — © Anthony Spadafino, all rights reserved. This
write-up is shared for evaluation purposes only. Odds data sourced from The Odds API under a
paid plan. See the repository [NOTICE](../NOTICE.md).*
