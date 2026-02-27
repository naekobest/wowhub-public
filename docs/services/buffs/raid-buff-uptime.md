# Raid Buff Uptime

**Service key:** `buff_uptime`
**Category:** Buffs
**Expansions:** All
**Contributes to scoring:** Yes — primary input to the Buffs category score

---

## What it measures

Uptime of persistent raid buffs across all boss fights, expressed as a percentage of each fight's duration per buff. Tracked buffs are divided into two groups:

**Reliable buffs** are spells the raid should maintain at close to 100% regardless of class composition, because at least one player of the required class is almost always present and the spell has a duration long enough to cover most fights. Examples: Blessing of Kings, Mark of the Wild, Fortitude, Arcane Intellect. These buffs are tracked and scored.

**Class buffs** are provided by classes that may not always be present in the raid. Their uptime is tracked for informational purposes but missing class buffs are not penalised when the relevant class is absent.

---

## Scoring

For each reliable buff, uptime is normalised against a target of 95%. A buff at 95% or above scores 100 for that slot. Lower values scale linearly toward 0. The Buffs category score is derived from the average uptime across all tracked reliable buffs.

---

## Why it matters

Persistent raid buffs that should be at 100% uptime are among the easiest things to fail at without noticing — a Paladin casting Kings and forgetting to rebuff after a wipe, or a Druid's Mark of the Wild falling off mid-fight because the caster died. Each percentage point of downtime represents a small but real multiplier loss for the affected players. In the Buffs category specifically, the issue is never individual skill — it is always process: buff assignments, rebuff habits after wipes, and awareness of expiration.

---

## WCL data used

Buff aura table (Tier 2). Per-boss aura data includes time bands for each spell and the number of players affected.

---

## Implementation notes

The buff tracking uses the WclAura abstraction layer to normalise the dual-key aura field structure returned by the WCL API. Uptime is calculated from aura time bands directly rather than from cast events, which are not reliable for buff tracking because buffs can be refreshed without a visible cast event in the log.

Per-boss results aggregate to a raid-wide average uptime per buff. The Buffs score uses the raid-wide average.
