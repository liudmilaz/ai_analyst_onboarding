# The data

Six CSVs describing **Invented Software**, a fictional B2B SaaS company selling
business-management software to small merchants across 8 markets.

> The company does not exist and every figure is generated. Treat the numbers as
> exercise material, never as a benchmark.

## Conventions

**All money is in minor units (cents).** `price_eur = 1900` means €19.00. Divide
by 100 once, as early as possible, and never again.

**`mrr_local` and `spend_amount` are in local currency.** Convert with
`raw_markets.eur_fx`: `amount_eur = amount_local * eur_fx`. Operating costs are
already in EUR.

**The FX join has to route through the merchant.** `raw_subscriptions` has no
`country_code`, so:

```
raw_subscriptions.merchant_id → raw_merchants.country_code → raw_markets
```

The only column subscriptions shares with markets is `currency`, and joining on
that **fans 117 rows into 300** — four markets use EUR (DE, FR, IT, ES), so
every euro-denominated subscription matches four times. Non-euro rows have
exactly one market each and spot-check perfectly, which is what makes the error
survive review.

## Coverage

The cost tables cover **2024–2025**. The entity tables do not:

| Table | Range | Predating 2024 |
|---|---|---|
| `raw_merchants.signup_date` | 2022-06-11 → 2024-06-29 | 114 of 160 |
| `raw_subscriptions.start_date` | 2022-06-30 → 2024-12-07 | 75 of 117 |

Scoping a reporting window over *entities* rather than *events* drops most of
the revenue. The reporting window and the entity history are not the same range.

---

## `raw_merchants` — 160 rows

One row per customer. **Not** every merchant is a paying customer: 65 never held
a subscription.

| Column | Notes |
|---|---|
| `merchant_id` | PK. Join key for `raw_subscriptions` |
| `merchant_name`, `business_type`, `city` | Descriptive |
| `country_code` | **FK → `raw_markets`.** The route to the FX rate |
| `signup_date` | See coverage above |
| `plan_type` | Records the plan **at signup**, not the current one — 42 merchants read `free` while holding a paid subscription |
| `status` | `active` or `churned` |
| `churn_date` | Null while active. Two merchants are `active` with a null `churn_date` while every subscription they hold has ended |

## `raw_subscriptions` — 117 rows

One row per subscription period. **The only source of revenue.**

| Column | Notes |
|---|---|
| `subscription_id` | PK |
| `merchant_id` | **FK → `raw_merchants`** |
| `plan_sku` | **FK → `raw_products.sku`** |
| `start_date` | 116 of 117 start mid-month |
| `end_date` | Null means still active. 14 of 15 end mid-month |
| `mrr_local` | Minor units, **local currency** |
| `currency` | Matches the merchant's market |
| `cancellation_reason` | Free text, nullable |
| `previous_plan_sku` | Populated on 9 rows, all reading `free`. **`free` is not a product** — it is a `plan_type` value, so a foreign-key test against `raw_products` fails on all 9 |

Turning these periods into a monthly series is the central modelling problem.
The convention: **a subscription is active for the whole month containing its
`end_date`, and there is no proration.**

## `raw_products` — 5 rows

| Column | Notes |
|---|---|
| `sku` | PK |
| `type` | A catalogue label, **not a reliable revenue classifier.** Only SW-001 is typed `subscription`; SW-002 and SW-005 are recurring plans typed `software`. Filtering `type = 'subscription'` silently drops **30.6%** of revenue. Derive revenue-bearing SKUs from `raw_subscriptions` instead |
| `price_eur`, `cogs_eur` | Minor units. **SW-003 and SW-004 are free** (`price_eur = 0`) |
| `gross_margin_pct` | Pre-computed. Averaging it across all five SKUs gives 51.5% because the two free plans store `0.0` |

Computing margin as `(price_eur - cogs_eur) / price_eur` on the raw integer
columns returns **0** for every paid SKU under integer division, and raises
**division by zero** on the two free ones. Cast first.

## `raw_markets` — 8 rows

| Column | Notes |
|---|---|
| `country_code` | PK |
| `currency` | **Not unique** — four markets are EUR |
| `eur_fx` | Multiplier to EUR: `amount_eur = amount_local * eur_fx` |
| `country_name`, `region`, `launch_year`, `vat_rate` | Descriptive |

## `raw_acquisition_costs` — 768 rows

| Column | Notes |
|---|---|
| `cost_id` | PK |
| `year_month` | First of month |
| `country_code` | **FK → `raw_markets`** |
| `channel` | `digital_marketing`, `field_sales`, `partnerships`, `referral` |
| `spend_amount` | Minor units, **local currency** |

Spend continues through 2025, but the last merchant signed up 2024-06-29.

## `raw_operating_costs` — 144 rows

Company-level P&L. Joins only to the calendar.

| Column | Notes |
|---|---|
| `cost_id` | PK |
| `year_month` | First of month |
| `cost_category` | Six values — but **only five are costs** |
| `amount_eur` | Minor units, already EUR |

`cost_category = 'cash_balance_eom'` is an end-of-month **balance**, not a cost,
and it is **93.8% of the column by value**. An unfiltered
`SUM(amount_eur) GROUP BY year_month` returns about **€54,814/month** against a
true **€3,419** — and €54,814 still looks like a plausible operating cost, so
nothing about the result signals a mistake.

A balance is a *stock*; the other five are *flows*. They cannot be summed
together, and no test will tell you so.
