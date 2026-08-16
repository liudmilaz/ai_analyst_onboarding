# Data guide — for the mentor

**This file contains answers. If you are the trainee, stop here** and read
`README.md` and `data/README.md` instead.

Everything below was recomputed from the CSVs in `data/`. It exists so the
mentor can recognise a correct answer, **not so it can be handed over.** See the
spoiler-control rule at the top of `AGENTS.md`.

---

## 1. The three traps, and what each looks like when hit

### Minor units

All money is in cents. Missed, every figure is 100× too large — obvious enough
that it self-corrects quickly.

### Local currency, and the join path

`mrr_local` and `spend_amount` are local; `raw_markets.eur_fx` converts them.
The join must route `raw_subscriptions → raw_merchants.country_code →
raw_markets`, because subscriptions carry no `country_code`.

The tempting shortcut, `JOIN raw_markets USING (currency)`, **fans 117 rows into
300** and inflates total MRR from €1,697.75 to €4,433.36 — **2.611×**. Four
markets share EUR (DE, FR, IT, ES); non-euro rows match exactly one market each
and spot-check perfectly, so sampling a few rows will not catch it.

### The stock among the flows

`raw_operating_costs.cost_category` has six values, but only five are costs.
`cash_balance_eom` is an end-of-month **balance** — 93.8% of the column by value.

| | Monthly |
|---|---|
| Unfiltered `SUM(amount_eur)` | **€54,814** |
| Costs only | **€3,419** |

A **16× error** that still looks like a plausible opex number, so nothing about
the result signals a mistake. This is the best teaching moment in the dataset:
it rewards sanity-checking a magnitude against a known anchor (€1,500/month of
revenue), which no test will do for them.

## 2. Expected results

Recomputed from the CSVs. **If a trainee's model disagrees, work out which is
right** — do not assume the table wins. Several of these depend on a modelling
choice, marked below.

| Metric | Value | Depends on |
|---|---|---|
| MRR, Jan 2024 | €1,137.85 | inclusive `end_date`, no proration |
| MRR, Dec 2024 | €1,640.77 | — |
| MRR, Dec 2025 | €1,509.78 | — |
| Exit ARR | €18,117.34 | — |
| Blended gross margin | 84.9% | MRR-weighted; see §3 |
| Logo churn, paying base | **9 of 95 = 9.5%** | see §3 |
| Operating costs | €3,419/month | excluding `cash_balance_eom` |
| Acquisition spend, 24m | €32,439 | — |
| Net burn | **€3,253/month** | including acquisition spend; see §3 |
| Cash balance, latest | €57,235 | as recorded — see §4 |
| Implied runway | **~17.6 months** | at the burn above |

## 3. Four places where the obvious answer is wrong

### Net burn and runway

Burn is `revenue − operating costs − acquisition spend`:

```
€1,517 − €3,419 − €1,352 = −€3,253/month
```

Acquisition spend is **not** a subset of the `sales_and_marketing` operating-cost
line — at €1,352/month it is *larger* than that line's €932/month, so the two are
independent and both are real cash out. Omitting it gives €1,909/month and a
runway of 30 months instead of **17.6** — the difference between "two and a half
years of cash" and "under eighteen months."

### Logo churn

13 merchants carry `status = 'churned'`, but only **9** ever appear in
`raw_subscriptions`; and 160 is every merchant, while only **95** ever held a
subscription. On the paying base: **9 / 95 = 9.5%**, not 8.1%.

Two related wrinkles: 42 merchants read `plan_type = 'free'` while holding a paid
subscription (the column records the plan at signup), and two merchants
(Café Coffee, Pulse Studio) are `active` with a null `churn_date` while every
subscription they hold has ended — so merchant-derived churn finds 13 while
subscription-derived churn finds 15.

### CAC — the denominator moves it as much as the numerator

Spend is €32,439. Divided by the **46** merchants who signed up in the window it
gives €705; divided by the **32** who actually subscribed it gives **€1,013.72**.

The numerator is ambiguous too: the last merchant signed up **2024-06-29**, yet
€16,856 of spend is booked across 2025 — money that acquired nobody. Restricting
to months that acquired someone gives €238.24.

Both are defensible; they answer different questions. **LTV:CAC comes out at
3.84 or 0.90** depending on the choice, and 0.90 is below 1.0 — meaning that on
full spend, a customer costs more to win than they return. A trainee who notices
that their modelling decision flipped the business conclusion has understood the
most important thing in this dataset.

Whichever they pick, **LTV must use the same population** — it is built from
ARPA over paying merchants, so pairing it with a CAC over signups is incoherent.

### Gross margin — 84.9% is one of several answers

| Method | Result |
|---|---|
| `AVG(gross_margin_pct)` over all 5 SKUs | 51.50% |
| Simple average over the 3 subscribed SKUs | 85.83% |
| Catalogue price-weighted | 85.52% |
| **MRR-weighted over live subscriptions** | **84.92%** |

51.5% comes from SW-003 and SW-004 storing `0.0` — they are free plans, and
averaging them in drags the figure 33 points off.

Recomputing from source has its own trap: `(price_eur - cogs_eur) / price_eur`
on the raw **integer** columns returns **0** for every paid SKU, and raises
**division by zero** on the two free ones, which fails the build outright. Cast
before dividing.

## 4. The cash ledger contradicts the P&L

Recorded cash *rises* €49,785 → €57,235 across 2024–25, while the P&L implies a
€3,253/month burn that would leave the company insolvent from May 2025 — a
divergence of **€85,519**.

Both cannot be true. Cash rising while a P&L shows losses is the normal
signature of a funded startup, so this is worth investigating rather than
assuming an error. The evidence settles it:

| Test | Result |
|---|---|
| Monthly cash change ~ linear in time | **R² = 0.971** (`d_cash = −227.7 + 50.1·t`) |
| Cash level ~ quadratic in time | R² = 0.9992 |
| corr(cash change, **time**) | **+0.985** |
| corr(cash change, revenue) | +0.401 |

The series is a function of **time**, not of the business — a synthetic curve. No
real mechanism produces a smooth monotonic ramp: financing is lumpy, accrual
timing oscillates, non-cash expenses are flat or step-shaped.

This is a data-forensics exercise, not a SQL exercise, and it is the single best
thing in the dataset. **Do not hand it over.** Ask whether the two numbers can
both be true, and let them find out.

## 5. Conventions the data will not tell them

**`end_date` is inclusive of its own month, and there is no proration.** 14 of
15 end dates are mid-month, so this is a real choice — and the checkpoints only
partly discriminate:

| Checkpoint | Inclusive | Exclusive |
|---|---|---|
| Jan 2024 | **1,137.85** ✓ | 1,108.86 |
| Dec 2024 | 1,640.77 | 1,640.77 |
| Dec 2025 | 1,509.78 | 1,509.78 |

Only the first tells them apart. A trainee who picks exclusive passes two of the
three checks and ships a series wrong in 8 of 24 months — with every reason to
trust it. Proration fails both of the first two (Jan 1,081.70; Dec 1,637.09).

**`raw_products.type` is a label, not a classifier.** All three revenue-bearing
SKUs are recurring, but only SW-001 is typed `subscription`; SW-002 (€9.00) and
SW-005 (€9.99) are typed `software`. Filtering `type = 'subscription'` drops
Dec-2025 MRR from €1,509.78 to €1,083.00 — **28.3% of that month, 30.6% across
the whole series, with no error.**

The docs now say revenue is *recurring* and warn that the column does not encode
that, so this is a documented quirk rather than a contradiction — but it still
catches anyone who trusts a category column without checking a total against an
anchor. That is the lesson; do not pre-empt it.

**`previous_plan_sku` is not a foreign key.** All 9 populated rows read `free`,
which is a `plan_type` value, not a `raw_products.sku`. A relationships test
against the product table fails on all 9. It also means every recorded transition
is free→paid, with no paid→paid moves at all — so expansion and contraction MRR
are not merely thin, they are **undefined**, and NRR should be reported with that
caveat or not at all.

## 6. Known caveats worth naming

- **Scale is small.** €18k ARR. Internally consistent, but no big-number
  dashboards.
- **MRR declines through 2025.** Real, not a bug — no merchant signed up after
  2024-06-29, so churn outpaces new business. Worth explaining rather than
  smoothing over.
- **Coverage differs by table.** Costs cover 2024–25; merchants and subscriptions
  reach back to 2022. Scoping a window over entities rather than events drops
  Jan-2024 MRR from €1,137.85 to €47.00.
