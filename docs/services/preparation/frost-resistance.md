# Frost Resistance

**Service key:** `frost_resistance`
**Category:** Preparation
**Expansions:** Vanilla Classic (Naxxramas — Sapphiron and Kel'Thuzad)
**Contributes to scoring:** Informational — does not currently feed into the Preparation score

---

## What it measures

Whether players had sufficient frost resistance gear equipped for encounters where resistance is required. The service reads the combatant info snapshot at fight start, sums the frost resistance values from all equipped items and enchants, and compares the total against a role-based minimum target.

Results record per player:
- Total frost resistance from equipped items
- Total frost resistance from enchants on those items
- A per-slot breakdown showing item contribution and enchant contribution separately
- Whether the target was met

---

## Encounter gating

The service only runs on boss fights whose encounter ID appears in the configured resistance encounter list. For all other bosses in the same log, the service produces no output and does not affect any score. This means a Naxxramas log where only Sapphiron is a resistance check will show frost resistance results only for that fight.

Configured encounters for Vanilla Classic: Sapphiron (1118) and Kel'Thuzad (1119).

---

## Resistance targets

Targets are role-based rather than a single flat minimum:

| Role | Target FR |
|------|-----------|
| Melee (Warriors, Rogues, Paladins, Druids, Shamans) | 80 |
| Ranged (Mages, Warlocks, Hunters, Priests) | 100 |

Melee classes take less frost damage from Blizzard and Frost Bolt Volley in practice due to positioning, while ranged classes standing in the blizzard field or targeted by Kel'Thuzad's frost abilities need higher mitigation. The separate targets avoid over-penalizing melee and under-penalizing ranged.

---

## Why it matters

Entering a resistance check fight without the required gear causes substantially increased magic damage taken from frost abilities. In Sapphiron, undergeared players will take excessive damage from Blizzard and Ice Bolt, creating an avoidable healing burden that can cause healer mana exhaustion before the enrage timer. The service makes it possible to identify which players arrived without the required gear before the attempt begins, rather than diagnosing it from death logs after a wipe.

---

## WCL data used

Combatant info gear data (Tier 2). WarcraftLogs records each item equipped per slot in the combatant info snapshot, including item stats and enchant IDs.

---

## Implementation notes

Resistance is summed from both item stats and enchant values. Enchant FR values are looked up from a configurable table per enchant ID. This means the result accurately reflects a player's true pre-buff resistance total, including popular enchants like Greater Resistance enchants.

Shirt and tabard slots are excluded from the item stat scan as they do not contribute resistance in practice and would introduce noise for items with cosmetic resistance values.

The per-slot breakdown is returned as an array of objects (not a map keyed by slot ID), because PHP re-indexes sparse integer-keyed arrays during JSON serialization, which would cause slot IDs to be lost in the Inertia payload.