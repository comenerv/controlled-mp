# controlled-mp

A [Claude Code](https://claude.com/product/claude-code) plugin marketplace — a curated set of installable plugins that extend Claude Code with domain-specific skills.

## What is this?

This repo is both a **marketplace** (`.claude-plugin/marketplace.json`, which lists available plugins) and, for now, a **single plugin** (`.claude-plugin/plugin.json`) bundling one skill. As more plugins are added, each will live in its own subdirectory under `plugins/` and get its own entry in the marketplace listing.

## Install

Add this marketplace to Claude Code, then install whichever plugin you want from it:

```
/plugin marketplace add comenerv/controlled-mp
/plugin install npv-portfolio-valuation@controlled-mp
```

This is a **private** repo, so you'll need GitHub access to `comenerv/controlled-mp` (and to already be authenticated with `gh`/git) for the add step to succeed.

To pick up updates after the marketplace has changed:

```
/plugin marketplace update
```

## Available plugins

| Plugin | Description |
|---|---|
| [`npv-portfolio-valuation`](skills/npv-portfolio-valuation/SKILL.md) | Calculates NPV / DCF valuations for one or more portfolio companies from 10-K/10-Q financial data provided as a CSV — per-holding valuation plus portfolio-level roll-up. |

### npv-portfolio-valuation

Turns raw 10-K/10-Q line items (revenue, operating cash flow, capex, debt, cash, shares outstanding) into a discounted cash flow valuation.

- Normalizes quarterly filings to an annual/TTM basis
- Computes historical FCF and growth rate, with explicit, overridable assumptions for growth rate, discount rate, and terminal growth
- Projects a 5-year DCF and bridges enterprise value to equity value (NPV) and value/share
- Rolls up multiple holdings into a portfolio-level NPV when position weights are available
- Flags negative/volatile FCF, thin history, and other data-quality caveats rather than presenting a false-precision number

This is an illustrative model for quick estimation, not a substitute for a full equity research model or investment advice — see the [skill definition](skills/npv-portfolio-valuation/SKILL.md) for the full methodology and assumptions.

## Repo layout

```
controlled-mp/
├── .claude-plugin/
│   ├── marketplace.json   # lists all plugins in this marketplace
│   └── plugin.json        # manifest for the npv-portfolio-valuation plugin
└── skills/
    └── npv-portfolio-valuation/
        └── SKILL.md        # the skill Claude loads when triggered
```

## Contributing a new plugin

1. Add a new directory (e.g. `plugins/<name>/` with its own `.claude-plugin/plugin.json` and `skills/`), or extend this single-plugin layout if it stays a one-plugin marketplace.
2. Add an entry for it to `plugins` in `.claude-plugin/marketplace.json`.
3. Add a row to the **Available plugins** table above.
