# Watchful: A Second Opinion for Coding Agents

**A Claude Code plugin that reads your session transcripts and makes your coding agent cheaper and
better at your own work.**

Install it by pasting one line; it works in the terminal, the desktop app, and VS Code. Coding agents
leave a complete trace of every session on disk, and nobody reads it. Watchful does, read-only and
on the developer's own laptop. Codex support comes next, in v1.5. Built in TypeScript on Node.js.

## Value propositions

1. **Writes skills for you.** It spots requests you keep repeating and drafts a skill for each one.
   It tests each draft against Claude without the skill and keeps only the winners.
2. **Cuts token spend.** About 64% of spend is Claude re-reading context on every turn. Watchful
   tells you when to compact, what a handoff should keep, and when to start fresh.
3. **Cleans up your setup.** It flags stale or conflicting CLAUDE.md lines, memories, and unused
   skills, so every session loads less.
4. **Safety, second.** Tripwires that can pause a risky action and alert you.

## What the data said first

Before writing any product code, Watchful's first scanner ran over 30 days of the owner's own
sessions (57 sessions, 160 transcript files) and reshaped the pitch:

- **Context carry is the money.** Cache reads were about 64 percent of spend and cache writes about
  27 percent. The five most expensive sessions ran 1,000 to 1,900 turns, re-sending 265k to 590k
  tokens of context on every turn.
- **Mistakes cost time, not money.** Wrong guesses at paths, repeated file reads, and failure loops
  were about 0.1 to 0.2 percent of tokens. Their real cost is derailment.
- **The trace finds real risk.** 22 recursive deletes (one wiped a repo's `.git`), 12 force pushes,
  and 4 `.env` reads, most of them intended but never recorded as decisions.

So the product leads with efficiency and quality, and safety is the reason to keep it running. A
later audit caught that the scanner double-counted tokens (one API message is stored as several
transcript records). The figures above are corrected, and the erratum is logged in the decision
record.

## The AI feature: skills that have to prove they help

Watchful spots recurring requests in a developer's history ("ship it", "give me a link to test")
and drafts a Claude Code skill for each. No skill installs on a hunch:

```
recurring pattern found in the user's own sessions
        │
        ▼
a headless Claude session drafts the skill (the "challenger")
        │
        ▼
replayed against Claude WITHOUT the skill on a case library built
from real past tasks, split 70/30 so the builder never sees holdout cases
        │
        ▼
scored on end state, weighted tokens, time, and safety; a blinded
pairwise judge (both orders) for text skills
        │
        ▼
pre-registered verdict with intervals:
WIN · CHEAPER · NO_EFFECT · WORSE · REJECT · INSUFFICIENT
        │
        ▼
promotion gated by blast radius: read-only and local-write skills
promote themselves; anything that pushes, deploys, or deletes
needs the owner's approval. Every promotion is one-command reversible.
```

A worked example: the owner's `ship` skill (commit, push, merge) reached version 6. On the owner's
picks it won 7 of 8 comparisons and passed 5 of 5 holdout cases, with a posterior of 0.947 against
the 0.97 bar for a skill that pushes to GitHub. The next batch would have cost about 490k weighted
tokens to confirm a decision the owner would make the same way, so it shipped as a logged, one-off
override instead of lowering the bar for every future skill.

## Product notes

- **Plain words for a non-technical user.** The core user builds by asking Claude and does not read
  its commands. Every notification states what happened, why it matters, and ends with one
  recommendation, with headers under 40 characters.
- **Themes, not alerts.** 34 raw findings collapse into about 11 themes, one per fix. The local page
  leads with the top 3, each with an Approve button that writes the fix and an Undo that restores
  the old text. Banners are reserved for live moments (a failure loop, a risky command), capped at 3
  a day.
- **First run in seconds.** A new user's first session shows one line, then a 30-day backfill builds
  their page for zero model tokens. Measured on a clean install: hook returns in 0.10 s, page ready
  9.2 s later. On the owner's machine it scanned 97 sessions and led with "26 sessions got too
  heavy" (up to 965k tokens re-sent in a single turn).
- **A kill switch only the human can pull.** `/watchful-off` stops every hook, banner, page, and
  background job on the next tool call. Claude cannot invoke it on its own. Approved fixes stay,
  because they are plain text and settings that keep working without Watchful.

## Engineering notes

- **Stack:** TypeScript on Node.js, a CLI shipped as a Claude Code plugin (SessionStart, PostToolUse, and Stop
  hooks), a local web page server, and a transient worker that runs queued evals only after 2 idle
  minutes and yields within 10 seconds of new activity.
- **Token-lean by design.** One ledger tracks every token Watchful spends, weighted by API price
  (input 1, output 5, cache write 1.25, cache read 0.1), with caps on background work. Detection is
  mechanical tripwires; models are only called where a judgment is needed.
- **Hard resource budget** set for a 24 GB laptop: under 6 GB of disk, no always-on daemon, no hooks
  without an explicit install, transcripts read-only, credentials never touched, everything
  reversible.
- **Decision discipline:** 37 architecture decision records, 470 passing tests, a portability test
  that fails if a path specific to the owner's machine leaks into the source, and docs whose every
  cross-reference is checked by the test suite.

## Vision

Today the plugin only runs when a hook or command starts it. Next it becomes an always-on daemon (a
background process) that:

- Coaches you live ("compact now") instead of only reporting after the fact.
- Watches every session at once and catches risky patterns that per-session vendor checks miss.
- Covers unattended agent swarms: it blocks dangerous actions and pages you.
- Tests skills in the background while your machine is idle.
- Applies a type of fix on its own, such as a skill or a memory rule, once its fixes have a
  measured track record.

After that comes a team tier that shares a playbook across machines.

**Status:** the owner is running it daily (dogfood week, 2026-10-07 to 10-13). Next is a friends
test: 2 to 3 developers installing from one pasted line.

---

*The Watchful codebase is proprietary. All rights reserved. Text shared for evaluation purposes
only. See the repository [NOTICE](../NOTICE.md).*
