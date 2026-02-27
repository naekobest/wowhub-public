# Consumables

**Service key:** `consumables`
**Category:** Preparation
**Expansions:** All
**Contributes to scoring:** Yes — weighted input to the Preparation category score

---

## What it measures

Flasks, elixirs, food buffs, and role-relevant consumables per player, as recorded in the WarcraftLogs combatant info snapshot at the start of each boss fight. The expected consumable set is configurable per expansion and supports class-specific and role-specific requirements.

Consumables are split into required and optional categories:

- **Required:** A missing required category reduces the player's score. Example: a physical DPS not having a Strength elixir.
- **Optional:** Present optional consumables add a smaller bonus weight. Example: engineering grenades, lesser potions.

This prevents players from being penalised for skipping genuinely minor consumables while still rewarding thoroughness.

---

## Scoring

Each player receives a score from 0 to 100 based on how many required categories they have covered. Optional categories can increase the score above the base value from required categories, up to 100. The raid-wide average is the primary input to the Preparation score.

The raid summary includes:
- How many players had at least one missing required category
- How many players were missing the majority of their expected consumables
- Per-category breakdown showing which consumable type was most commonly skipped across the roster

---

## Why it matters

Consumable compliance is one of the clearest indicators of individual preparation discipline and guild culture. Unlike world buffs, which require external content to be available, consumables are always obtainable. A player who shows up without flasks or food buffs on a progression boss is demonstrably less prepared than the data would show with any other metric.

---

## WCL data used

Combatant info (Tier 2). The buff aura snapshot at fight start includes all active buff effects, from which consumable buffs are identified by spell ID.

---

## Implementation notes

Consumable detection is entirely buff-based. A player who consumed a flask but whose buff wore off before the pull (possible in longer fights) may appear missing that consumable on later bosses. This is a known limitation of the WCL combatant info approach.

Some consumables share a buff ID with world effects or food from different sources. Spell ID lists are maintained per expansion to avoid false positives.
