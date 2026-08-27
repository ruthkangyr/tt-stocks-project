# TT Stocks Project — Stock Advisor Agent

An AI agent that turns a messy, human question about a stock ("is NVDA still worth holding
after that earnings drop?") into a structured, sourced, plain-English briefing — with an
explicit confidence level and an explicit list of what it does *not* know.

This README is the **plan**, not the finished product. It records what needs to be built and
in what order, so the build has a target before any code is written.

---

## Why this needs an LLM

The brief requires a task that cannot be handled reliably with simple parsing or a fixed
template. This one qualifies because:

- The **input** is unconstrained natural language — ticker, company name, or a vague vibe
  ("that chip company everyone's mad about"). No regex resolves that reliably.
- The **middle step** is judgement: reconciling price data against recent news that may
  agree, contradict, or be irrelevant, and deciding which facts actually bear on the question.
- The **output** is an argument, not a value. A template can print a P/E ratio; it cannot say
  *why* the ratio is misleading this quarter.

Anything a fixed template *can* do (fetching quotes, computing % change) stays in code.
The LLM is reserved for the part that genuinely needs reasoning.

---

## Scope: outcome metric & riskiest assumption

> Fill these in from the **grill-me** interview before writing build code. They drive
> every decision below.

- **Outcome metric:** _TBD — the one number that says this worked._
  Candidate: % of briefings a user rates "I'd actually act on this" out of 20 real queries.
- **Riskiest assumption:** _TBD._
  Candidate: that an LLM briefing over free-tier market data is more useful than the user
  just reading the top three headlines themselves.

**Not building:** buy/sell signals, portfolio management, backtesting, anything that reads
as licensed financial advice. Output is a briefing with sources, and it says so.

---

## Requirements checklist (from the brief)

| # | Requirement | Status |
|---|-------------|--------|
| 1 | Clarify the product with the **grill-me** skill (include outcome metric + riskiest assumption in the input) | ☐ Not started |
| 2 | App uses an **AI back-end** (server-side LLM call, OpenCode key, same pattern as Lunch Uncle) | ☐ Not started |
| 3 | One **extra-knowledge build** (skill / MCP / automation / evaluation), used on a real task | ☐ Not started |
| 4 | Scaffold + **one complete vertical slice**, with a real LLM call confirmed working before expanding | ☐ Not started |
| 5 | Small record of what was tried, what changed, one limitation or failed attempt | ☐ Not started |

---

## Architecture (intended)

```
user question
     │
     ▼
[ frontend ]  minimal — one input, one briefing view
     │
     ▼
[ worker / server ]
     ├── resolve: question ──► ticker            (LLM, cheap+strict)
     ├── fetch:   quote + recent price history   (deterministic code)
     ├── fetch:   recent headlines               (deterministic code)
     └── reason:  data + news ──► briefing       (LLM, the real call)
     │
     ▼
briefing: verdict-in-a-sentence · supporting facts · what I don't know · sources
```

Server-side LLM calls use the **OpenCode key** exactly as Lunch Uncle does. The key never
reaches the client. `.env` is gitignored; `.env.example` documents the variable names.

---

## The vertical slice

One path, working end to end, before anything is widened:

**Typed question → resolved ticker → live quote → real LLM briefing → rendered on screen.**

Done when a question typed into the browser returns a briefing built from a live LLM
response and live price data — no mocks, no hardcoded ticker. The LLM key gets confirmed
working (single smoke call) *before* the rest is built on top of it.

---

## Extra-knowledge build

Leaning toward **Evaluation**, because the riskiest assumption above is a quality question
and a guess won't settle it.

Plan: a small repeatable eval that runs the same ~15 questions through two briefing prompts
(terse-analyst vs. explain-the-reasoning), scored on whether the output is specific,
sourced, and honest about gaps. The result decides which prompt ships — that's the one real
project decision it feeds.

Alternative if the eval proves thin: an **MCP server** wrapping the market-data source, so
the agent can pull quotes and fundamentals as tools rather than through hardcoded fetches.

To record either way: what was tried, what changed as a result, and one limitation or
failed attempt — kept in `NOTES.md` for the pitch.

---

## Build order

1. Run **grill-me**, fill in the outcome metric and riskiest assumption above.
2. Scaffold the worker + `.env.example`; **smoke-test the LLM key** in isolation.
3. Ticker resolution (LLM) — the smallest real call.
4. Market data fetch (deterministic).
5. Briefing generation (the real LLM call) + minimal frontend.
6. Extra-knowledge build, used on a real task from step 5.
7. Write up `NOTES.md`: what changed, one limitation.

---

## Setup

_To be filled in once the scaffold exists._

```bash
cp .env.example .env   # add the OpenCode key
npm install
npm run dev
```

---

## Notes

Not financial advice. Every briefing carries its sources and an explicit uncertainty
section — if the agent can't support a claim, it's required to say so rather than round it
off into confidence.
