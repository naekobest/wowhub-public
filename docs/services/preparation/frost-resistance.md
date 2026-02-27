# Frost Resistance

**Service key:** `frost_resistance`
**Category:** Preparation
**Expansions:** Vanilla Classic (Naxxramas)
**Contributes to scoring:** Informational — does not currently feed into the Preparation score

---

## What it measures

Whether players had sufficient frost resistance gear equipped for encounters where resistance is required. The service reads the combatant info snapshot at fight start, sums the frost resistance values from all equipped items (excluding shirt and tabard/ranged slots), and compares the total against a configurable minimum target.

Results record per player:
- Total frost resistance from equipped items
- Whether the target was met
- Which items contributed resistance

---

## Encounter gating

The service only runs on boss fights whose encounter ID appears in the configured resistance encounter list. For all other bosses in the same log, the service produces no output and does not affect any score. This means a Naxxramas log where only Sapphiron is a resistance check will show frost resistance results only for that fight.

---

## Why it matters

Entering a resistance check fight without the required gear causes substantially increased magic damage taken from frost abilities. In Sapphiron, undergeared players will take excessive damage from Blizzard and Ice Bolt, creating an avoidable healing burden that can cause healer mana exhaustion before the enrage timer. The service makes it possible to identify which players arrived without the required gear before the attempt begins, rather than diagnosing it from death logs after a wipe.

---

## WCL data used

Combatant info gear data (Tier 2). WarcraftLogs records each item equipped per slot in the combatant info snapshot, including item stats.

---

## Implementation notes

Resistance is summed from item stats only. Aura-based resistance buffs (e.g. Frost Protection Potions, Frost Resistance Aura) are not counted — only base item stats are reliable across all log formats. This means the threshold should be set against base gear values, not the buffed total a player would have in combat.

Shirt and tabard slots are excluded from the item stat scan as they do not contribute resistance in practice and would introduce noise for items with cosmetic resistance values.
