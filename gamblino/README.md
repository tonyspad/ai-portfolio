# Gamblino — Social Sports Gaming

**A social sports gaming league for friends and family — points, not money. Born as a
29-player NFL pool in a spreadsheet; rebuilt as a real product without losing the spreadsheet's
soul.**

Every player gets a fixed budget of **16 points a week** to spread across that week's NFL games
against the spread — 2 to 8 games, with house rules that make allocation a genuine puzzle
(minimums per game, a cap on any single game, pushes refund). Correct picks pay double; two
league bonuses create the drama: a **perfect-week "parlay" bonus** whose multiplier scales
steeply with how many games you dared to count, and a **"whoopsie" bonus** for going one loss
short of perfection. Playoffs flip the whole league into a confidence-pool format. Everything
rolls up into a season-long leaderboard.

<p align="center">
  <img src="screenshots/betting-board.png" width="390" alt="Gamblino — the weekly betting board on mobile" />
</p>

*The core loop on mobile: budget meters up top (points bet / games bet), a jump strip of this
week's games, and per-game cards where tapping a side opens the legal point values — with
illegal allocations disabled and labeled with the reason. When the week locks, the same board
flips from betting to watching.*

## The AI feature — rules written in English, enforced by a machine

The most interesting engineering in Gamblino is how leagues customize scoring. House rules are
the lifeblood of a friends' pool — and no two pools agree — so instead of hard-coding rule
variants, Gamblino lets a league admin **describe a scoring rule in plain English** and turns
it into executable law:

```
admin's English description
        │
        ▼
LLM compiles it → a whitelisted JSON scoring DSL (no arbitrary code)
        │
        ▼
schema validation + a human-readable readback the admin reviews
        │
        ▼
the rule must PASS a scenario test suite (worked examples with
expected payouts) before it can be activated
        │
        ▼
activation freezes the rule's hash — what was tested is exactly
what runs, forever
        │
        ▼
a pure, zero-dependency, deterministic scoring engine executes it
(purity is enforced by an automated check in CI)
```

The architectural stance: **the LLM authors the law; a deterministic engine enforces it.** The
model never scores a single wager — it only drafts rules, and nothing it drafts can run until
it has survived validation and its full test suite. There's also **historical replay**: a
candidate rule can re-score entire past seasons so the league sees exactly how standings would
have shifted before adopting it.

## Product notes

- **Leagues → contests → markets → wagers.** League admins configure per-week betting rules
  from presets, correct scores with audited manual overrides, and manage members; a separate
  ops surface handles market data ingestion, settlements, and score generation.
- **Mobile-first by observation, not fashion** — nearly all play happens on phones, so every
  screen is designed at 390px first.
- **A design system with a sense of humor:** the UI is a deliberate homage to the spreadsheet
  the league came from — 1px gridlines, header-row fills, sheet tabs pinned to the bottom
  edge, tabular numerals throughout.
- **Live in production** at [gamblino.life](https://gamblino.life), running a real league
  through the NFL season.

## Engineering notes

- **Monorepo:** Turborepo + workspaces, TypeScript end to end — React 18 + Vite web app,
  Express + Prisma/PostgreSQL API on serverless AWS, a pure scoring-engine package, and a
  shared types/contracts package.
- **Auth & infra:** OAuth2/JWT auth, serverless deploys with environment guards that keep
  non-production configuration out of prod.
- **Quality:** CI runs typecheck, lint, unit + integration suites against a real database, and
  Playwright end-to-end tests on mobile device profiles; the scoring engine's zero-dependency
  purity is enforced automatically before every test run.

---

*The Gamblino codebase is proprietary — all rights reserved. Screenshots and text shared for
evaluation purposes only. Betting-market reference data via The Odds API; Gamblino is a
points-based social game, not a wagering service. See the repository [NOTICE](../NOTICE.md).*
