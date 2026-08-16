# SaaS Analytics Platform — an AI-assisted development workshop

A hands-on training project for data analysts. You start with six CSV files and
a spec, and finish with a working analytics platform **and a set of business
conclusions you can defend**. Both halves matter — the pipeline is the means,
not the goal.

## Getting started

Open your coding agent in this directory and say **"I'm ready."** It will walk
you through the project, explaining each step and checking you have understood
before moving on. You do not need to read anything else first.

Works with any agent that reads `AGENTS.md` — Claude Code, Cursor, Codex,
Copilot, Gemini CLI. Claude Code reads `CLAUDE.md`, which imports the same file.

**Work on a copy**, so the original stays available to compare against:

```bash
cp -r . ../workspace && cd ../workspace
```

Before Phase 3 you will need Docker and a container runtime — see
[`prerequisites.md`](prerequisites.md). Nothing else is needed to start.

## The company

**Invented Software** sells business-management software to small merchants —
cafés, bakeries, salons, florists, food trucks, gyms — across 8 markets.

**All revenue is recurring** — monthly fees on software plans and add-ons.
There are no transaction fees, no hardware and no payment processing, which
makes the standard SaaS metric stack the natural frame.

The product catalogue does not cleanly encode this: `raw_products.type` is not a
reliable way to tell recurring revenue from anything else, and filtering on it
will quietly cost you money. Work out which SKUs carry revenue from
`raw_subscriptions` instead, and check your total against a figure you trust.

> **The company is fictional and the data is synthetic.** Invented Software does
> not exist and every figure is generated. Treat the numbers as exercise
> material — never as a benchmark or a claim about any real market.

## The data

Six tables in [`data/`](data/). A field-by-field description, including the join
keys, is in [`data/README.md`](data/README.md).

| Table | Rows | What it is |
|---|---|---|
| `raw_merchants` | 160 | The customers |
| `raw_subscriptions` | 117 | Subscription periods — the only revenue source |
| `raw_products` | 5 | Plans and add-ons, with price and cost |
| `raw_markets` | 8 | Countries, currency, FX rate |
| `raw_acquisition_costs` | 768 | Marketing spend by market and channel |
| `raw_operating_costs` | 144 | Monthly P&L lines |

**Money is stored in minor units** — `price_eur = 1900` means €19.00 — and some
amounts are in local currency rather than euros. The details, and the join path
that conversion requires, are in [`data/README.md`](data/README.md).

The dataset is not tidy. Some of what you find will look wrong, and deciding
what to do about it is part of the work rather than an obstacle to it. Where
there is genuinely no single correct answer, say so in your write-up and defend
the choice you made — that is the skill being taught.

## What you will build

```
data/*.csv → database → dbt models → BI dashboard
```

Six phases, each with a written deliverable. The full spec, the architecture
constraints and the canonical list of KPIs are in
[`requirements.md`](requirements.md).

| Phase | Deliverable |
|---|---|
| 1 · Data discovery | Business requirements document |
| 2 · Architecture | Architecture decision record |
| 3 · Database | Working database, data loaded and validated |
| 4 · dbt modelling | Tested, documented models |
| 5 · BI platform | Executive dashboard and drill-downs |
| 6 · Automation | One-command startup and a refresh path |

The documents are not paperwork. Each one is the proof that you understood the
phase — the code alone is not.

## Files

| Path | What it is |
|---|---|
| `data/` | The six CSVs, plus a description of every field |
| `requirements.md` | The spec: phases, deliverables, constraints, KPI list |
| `prerequisites.md` | What to install, and when you will need it |
| `AGENTS.md` | Instructions for the mentoring agent. You do not need to read it |
| `data-guide.md` | Notes for the mentor. **Contains answers — skip it** |

If something goes sideways, tell your agent "let's back up." The pace is yours
to set.
