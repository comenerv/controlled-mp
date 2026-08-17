---
name: npv-portfolio-valuation
description: Calculate Net Present Value (NPV) and discounted cash flow (DCF) based valuations for one or more portfolio companies from 10-K/10-Q financial data provided as a CSV. Use this skill whenever the user uploads or references a CSV of 10-K/10-Q line items (revenue, operating cash flow, capex, debt, cash, shares outstanding, etc.) and asks for a valuation, NPV, DCF, intrinsic value, or "what is this company/portfolio worth" — even if they don't use the exact term "NPV". Also trigger for portfolio-level valuation roll-ups across multiple holdings in a single CSV.
---

# NPV Portfolio Valuation

A demo skill that turns raw 10-K/10-Q line items into a discounted cash flow (DCF) valuation — per holding and rolled up across a portfolio.

This is a simplified, transparent DCF model intended for quick estimation and illustration, not a substitute for a full equity research model. Always state assumptions explicitly in the output, since NPV is highly sensitive to growth and discount rate inputs.

## When to use this skill

Trigger this skill when the user provides (or points to) a CSV containing financial statement data pulled from 10-K (annual) or 10-Q (quarterly) filings, and wants:
- The intrinsic / NPV value of a single company
- A DCF valuation with explicit assumptions
- A portfolio-level valuation summary across multiple holdings in one file
- Sensitivity of value to discount rate or growth rate

## Expected input format

The CSV may come in different shapes. Before doing anything else, inspect the columns and confirm the mapping below rather than assuming. Common columns and their aliases:

| Concept | Typical column names |
|---|---|
| Entity identifier | `company`, `ticker`, `name` |
| Period | `period`, `fiscal_year`, `quarter`, `date` |
| Filing type | `filing_type` (10-K vs 10-Q) — if absent, infer from period (quarterly vs annual) |
| Revenue | `revenue`, `total_revenue`, `net_sales` |
| Operating cash flow | `operating_cash_flow`, `cfo`, `cash_from_operations` |
| Capital expenditures | `capex`, `capital_expenditures`, `purchases_of_ppe` |
| Total debt | `total_debt`, `long_term_debt` + `short_term_debt` |
| Cash & equivalents | `cash`, `cash_and_equivalents` |
| Shares outstanding | `shares_outstanding`, `diluted_shares` |
| Position size (portfolio weight) | `position_value`, `shares_held`, `weight` — optional, only present in portfolio-level files |

If a required column is missing or ambiguous, ask the user rather than guessing at financial figures.

## Workflow

### 1. Load and normalize the data

- Read the CSV and group rows by entity identifier.
- If the file mixes 10-K (annual) and 10-Q (quarterly) rows for the same company, annualize quarterly figures (multiply by 4, or sum trailing four quarters if at least four are present — prefer trailing-twelve-months when the data allows it) so all periods are on a comparable annual basis.
- Sort each entity's periods chronologically and check for gaps or duplicate periods before proceeding.

### 2. Compute historical Free Cash Flow (FCF)

For each period:

```
FCF = Operating Cash Flow − CapEx
```

Compute the historical FCF growth rate (year-over-year, or CAGR if 3+ years are available). Flag entities with negative or highly volatile FCF — these need a wider assumption range and an explicit caveat in the output, since a standard DCF is less reliable for them.

### 3. Set projection assumptions

Default to a 5-year explicit projection horizon. For each entity, propose (and let the user override):

- **Growth rate**: default to the historical FCF CAGR, capped at a reasonable range (e.g., 2%–25%) to avoid extrapolating an outlier year forward indefinitely.
- **Discount rate (r)**: if the user supplies a WACC or required return, use it. Otherwise use a default of 9–10% for a diversified equity portfolio and state clearly that this is a placeholder, not a company-specific WACC.
- **Terminal growth rate (g)**: default to a conservative long-run rate (e.g., 2.5%, roughly long-run GDP growth), always less than the discount rate.

State every assumption used in the final output — never bury them.

### 4. Project cash flows and discount to present value

Project FCF for years 1–5 using the growth rate, then compute terminal value with the Gordon Growth Model:

```
Terminal Value (Year 5) = FCF_year5 × (1 + g) / (r − g)
```

Discount each year's FCF and the terminal value back to present value:

```
PV(FCF_t) = FCF_t / (1 + r)^t
PV(Terminal Value) = Terminal Value / (1 + r)^5

Enterprise Value (EV) = Σ PV(FCF_1..5) + PV(Terminal Value)
```

### 5. Bridge to equity value (NPV per holding)

```
Equity Value = Enterprise Value − Total Debt + Cash & Equivalents
Value per Share = Equity Value / Diluted Shares Outstanding   (if shares data available)
```

`Equity Value` is the entity's NPV under this model.

### 6. Roll up to portfolio level (if multiple holdings)

If the CSV contains a position size / weight column:

```
Portfolio NPV = Σ (Equity Value_i × Ownership % or Position Weight_i)
```

If no weighting column exists, report each holding's NPV individually and the simple sum, and note that a weighted roll-up wasn't possible without position sizes.

### 7. Present the output

Always include:

1. **A summary table**: entity, EV, equity value (NPV), value/share (if available), implied vs. any market price the user supplied.
2. **Assumptions used**: growth rate, discount rate, terminal growth rate, projection horizon — per entity if they differ.
3. **Caveats**: flag entities with negative/volatile historical FCF, thin data history (fewer than 2–3 periods), or missing debt/cash/share data that forced a partial calculation.
4. **Optional sensitivity**: if the user wants it, show NPV at discount rates ±1–2 points from the base case, since this is usually the most consequential lever.

## Edge cases

- **Fewer than 2 periods of history**: can't compute a growth rate reliably. Ask the user for an assumed growth rate, or use an industry-average placeholder and flag it clearly.
- **Negative FCF in the base year**: don't silently grow a negative number to zero or positive — ask the user whether to model a turnaround (explicit path to positive FCF) or exclude the entity from the DCF roll-up with a note.
- **Multiple entities, inconsistent fiscal year-ends**: normalize to calendar-year buckets where possible; note any misalignment in the caveats section.
- **Discount rate ≤ terminal growth rate**: this breaks the Gordon Growth formula (division by zero or negative denominator) — never let this pass silently; adjust the terminal growth assumption down and flag it.

## Notes

This model is for illustration and quick estimation. It is not investment advice, and Claude should not present the output as a definitive valuation — frame results as "under these stated assumptions," and encourage the user to sanity-check discount rate and growth assumptions against their own view of the business.
