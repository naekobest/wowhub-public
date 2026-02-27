# Gear Enchants

**Service key:** `gear_enchants`
**Category:** Preparation
**Expansions:** All
**Contributes to scoring:** Yes — weighted input to the Preparation category score

---

## What it measures

Whether all enchantable gear slots carry a valid permanent enchant. Each slot is checked against a known valid enchant list. Slots with no enchant or with a known bad enchant (a lower-rank version or a PvP enchant on a raid slot) are both flagged — with a distinction between missing and incorrect.

Per-player score is the fraction of enchantable slots that carry a valid enchant, expressed as a percentage. The raid summary reports the average enchant score and the most common missing or bad enchant slot across the roster.

---

## Slot configuration

Slots and their severity weight are configurable per expansion. A missing weapon enchant is weighted more harshly than missing boots, reflecting the relative DPS impact. Configured slot examples:

| Slot | Severity |
|------|----------|
| Mainhand | High |
| Offhand | High |
| Head | Medium |
| Chest | Medium |
| Shoulders | Medium |
| Legs | Medium |
| Hands | Low |
| Feet | Low |

---

## Two-handed weapon handling

When the mainhand item ID is recognised as a two-handed weapon, the offhand slot check is skipped. This prevents warriors, shamans, or paladins using two-handers from being penalised for an empty offhand slot they cannot fill.

Offhand items that cannot receive weapon enchants (held-in-offhand books, off-hand frills) are also excluded via a per-slot `enchantable_item_ids` filter. Only items in this list are required to have an enchant; all other off-hand items pass the check unconditionally.

---

## Scoring

Each player's enchant coverage fraction feeds a weighted average. Slots with higher severity weights contribute proportionally more to the score. The raid-wide average across all players and all enchantable slots is the input to the Preparation category score.

---

## Why it matters

Unenchanted gear is a recurring gap at all guild experience levels. A weapon missing an enchant is a permanent DPS loss for the entire night — unlike a forgotten consumable, it cannot be corrected mid-raid. The automated audit replaces the practice of manually checking armory links before every raid night.

---

## WCL data used

Combatant info, specifically gear slot data with `permanentEnchant` IDs (Tier 2). WarcraftLogs records the enchant ID applied to each slot in the combatant info snapshot.

---

## Data quality limitations

The enchant check depends on the quality of the expansion's known enchant ID lists. New enchants added in patches may appear as unknown IDs until the list is updated, causing them to be treated as bad enchants rather than valid ones. This is a maintenance concern rather than a detection failure.
