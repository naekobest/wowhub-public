# WoW Hub

> Raid log analysis for WoW Classic, built for guild leads and powered by WarcraftLogs.

[![Status](https://img.shields.io/badge/status-beta-orange)](https://github.com/naekobest/wowhub-public)
[![Stack](https://img.shields.io/badge/stack-Laravel%2012%20%7C%20React%20%7C%20PostgreSQL-blue)](docs/architecture.md)

WoW Hub ingests your WarcraftLogs report and runs a structured analysis pipeline across four categories: **Execution**, **Preparation**, **Performance**, and **Buffs**. Every player gets scored individually. The raid gets an overall score. Results are graded on WoW's own quality scale, from Poor to Legendary.

No spreadsheets. No manual log scrubbing. Submit a WCL URL, wait a few seconds, get a full breakdown.

## Why Not Just Use WarcraftLogs?

WarcraftLogs is excellent for raw data: damage meters, cast timelines, event logs. What it doesn't do is tell you *what matters* and *who's responsible*. WoW Hub answers:

- Did your tanks keep Sunder/Expose Armor at full stacks all fight?
- Which healers were world buffed, which weren't?
- Who used their engineering items on cooldown?
- Are your DPS players running the right resistance gear?

WarcraftLogs shows you the events. WoW Hub scores the execution.

## What Gets Analyzed

### Execution

Mechanics that directly affect raid performance if missed.

- **Interrupts**: tracking rate vs. available windows per player
- **Armor debuff uptime**: Sunder Armor / Expose Armor coverage by tanks
- **Boss debuffs**: Curse of Recklessness, Curse of Elements, Faerie Fire uptime
- **Death analysis**: avoidable vs. unavoidable, per boss, per player
- **Ignite munching**: detects dropped Ignite rolling combos for fire mages — flags cases where a mage broke the chain before the 2.15 s window elapsed

### Preparation

What players bring to the raid before the first pull.

- **World buffs**: who had which buffs at pull, compliance rate per boss
- **Consumables**: flasks, elixirs, food buffs per player and role
- **Gear quality**: enchants, gems, item level relative to content
- **Professions**: relevant crafted gear and engineering presence
- **Resistance gear**: correct sets equipped for resistance check fights

### Performance

How effectively players use their toolkit during combat.

- **DPS / HPS**: tracked per player across three segments — trash pulls, boss fights, and full clear total
- **Damage taken**: total damage per player broken down by trash, boss, and full clear; avoidable classification planned
- **Engineering**: Goblin Sapper, grenades, on use trinkets tracked
- **Trinket usage**: on cooldown usage rate for major DPS/healing trinkets
- **Drums of Battle**: coverage and overlap analysis for Leatherworkers

### Buffs

Raid-wide buff infrastructure and uptime.

- **Raid buff uptime**: Blessing of Kings, Mark of the Wild, etc.
- **Combat buffs**: Bloodlust/Heroism coverage per fight phase
- **Player buff coverage**: who received which buffs and for how long

## Score System

Each category receives a score from 0 to 100%, then mapped to WoW's item quality scale:

| Score | Grade | Color |
|-------|-------|-------|
| 0–29% | Poor | Gray |
| 30–49% | Common | White |
| 50–64% | Uncommon | Green |
| 65–79% | Rare | Blue |
| 80–94% | Epic | Purple |
| 95–100% | Legendary | Orange |

An overall raid score is calculated from a weighted average across all four categories. The weighting is configurable per expansion, since what matters in Vanilla Classic differs from what matters in Mists of Pandaria.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Laravel 12, PHP 8.4 |
| Frontend | React 19, Inertia.js v2, TypeScript |
| Database | PostgreSQL (partitioned by expansion) |
| Queue | Redis + Laravel Horizon |
| Auth | WarcraftLogs OAuth 2.0 |
| Payments | Stripe (subscription tiers) |
| API | WarcraftLogs GraphQL v2 |

See [Architecture Overview](docs/architecture.md) for more detail.

## Expansion Support

| Expansion | Status |
|-----------|--------|
| WoW Classic (Vanilla) | Beta |
| The Burning Crusade Classic | Planned |
| Wrath of the Lich King Classic | Planned |
| Cataclysm Classic | Planned |
| Mists of Pandaria Classic | Planned |
| Season of Discovery | Planned |
| Retail | Planned |

Analysis services are expansion-aware. Each expansion has its own set of enabled checks, spell IDs, and scoring thresholds.

## Status

WoW Hub is currently in **Beta**. Classic Vanilla is the first supported expansion. If you want to follow development, watch this repo or join the Discord (link coming soon).

## Learn More

- [Architecture Overview](docs/architecture.md): stack, queue design, WCL API integration
- [Analysis Pipeline](docs/analysis-pipeline.md): how the scoring engine works
- [Roadmap](docs/roadmap.md): planned features and expansion timeline
- [Contributing](CONTRIBUTING.md): how to suggest features or new analysis checks
