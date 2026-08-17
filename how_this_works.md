# How this works

This repo is a **Claude Code plugin marketplace**. It's what you just added and installed from with:

```
/plugin marketplace add comenerv/controlled-mp
/plugin install npv-portfolio-valuation@controlled-mp
```

This doc explains what those two commands actually did, and how the pieces fit together.

## The three concepts

Claude Code has three layers here, each one file (or folder) in this repo:

| Concept | What it is | Where it lives |
|---|---|---|
| **Marketplace** | A catalog — a list of plugins and where to fetch each one from | `.claude-plugin/marketplace.json` |
| **Plugin** | An installable bundle — metadata plus a set of components (skills, commands, agents, etc.) | `.claude-plugin/plugin.json` |
| **Skill** | The actual instructions Claude reads and follows for a specific task | `skills/npv-portfolio-valuation/SKILL.md` |

A marketplace can list many plugins. A plugin can bundle many skills (or commands, or agents). Right now `controlled-mp` is the simplest case: one marketplace, one plugin, one skill, all in the same repo.

## What each command did

### `/plugin marketplace add comenerv/controlled-mp`

Claude Code fetched this GitHub repo and read `.claude-plugin/marketplace.json`. That file says, in effect: *"this marketplace is called `controlled-mp`, and it offers one plugin, `npv-portfolio-valuation`, whose source is this same repo."* Nothing gets installed yet — this step just registers the marketplace as a known source, the way adding a package registry doesn't install any packages by itself.

### `/plugin install npv-portfolio-valuation@controlled-mp`

Claude Code looked up `npv-portfolio-valuation` in the `controlled-mp` marketplace it just added, read `.claude-plugin/plugin.json` for that plugin's manifest, and installed it. Because `plugin.json` doesn't list a `skills` field explicitly, Claude Code used the default convention — everything under `skills/` — and picked up `skills/npv-portfolio-valuation/SKILL.md`.

That's the "Installed NPV Portfolio Valuation. Plugin is now active." message: the skill is now loaded and available to Claude Code in this environment.

## How the skill itself gets used

`SKILL.md` starts with YAML frontmatter:

```yaml
name: npv-portfolio-valuation
description: Calculate Net Present Value (NPV) and discounted cash flow (DCF)
  based valuations for one or more portfolio companies from 10-K/10-Q
  financial data provided as a CSV. Use this skill whenever...
```

Claude Code doesn't run this like a function you call by name. Instead, the `description` acts as a standing trigger condition: whenever you give Claude a prompt, it checks whether the situation matches any installed skill's description. If you upload a CSV of 10-K/10-Q line items and ask what a company or portfolio is worth, this skill's description matches, and Claude reads the rest of `SKILL.md` — the workflow, formulas, and assumptions — before answering, instead of improvising a valuation from scratch.

`/skills` (which you ran and got "No changes") lists which installed skills are currently active/inactive in this session — it's a status check, not an install step.

## How this maps to the files in the repo

```
controlled-mp/
├── .claude-plugin/
│   ├── marketplace.json   # the catalog: what plugins exist, where to fetch them
│   └── plugin.json        # this plugin's manifest: name, version, metadata
├── skills/
│   └── npv-portfolio-valuation/
│       └── SKILL.md        # the actual instructions Claude follows
├── docs/
│   └── index.html          # a designed landing page, served via GitHub Pages —
│                            # this is documentation *about* the marketplace,
│                            # separate from the marketplace mechanics above
└── README.md               # GitHub-native front page, links to docs/index.html
```

The `docs/` site (https://comenerv.github.io/controlled-mp/) is purely descriptive — nothing there is read by `/plugin` commands. Claude Code only ever reads `.claude-plugin/*.json` and whatever paths those manifests point to.

## Updating

If `SKILL.md` or the manifests change upstream, your local install won't pick it up automatically:

```
/plugin marketplace update      # re-fetch the marketplace listing
/plugin install npv-portfolio-valuation@controlled-mp   # re-install to get the new version
```

## Adding a second plugin

Because everything currently lives at the repo root (`source: "./"` in `marketplace.json`), a second plugin needs its own subdirectory with its own `plugin.json` and `skills/`, then a new entry in `marketplace.json`'s `plugins` array pointing at that subdirectory. See the "Adding a plugin" section in `README.md` for the exact steps.
