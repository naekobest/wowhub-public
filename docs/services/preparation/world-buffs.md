# World Buffs

**Service key:** `world_buffs`
**Category:** Preparation
**Expansions:** All
**Contributes to scoring:** Yes — weighted input to the Preparation category score

---

## What it measures

Which world buffs each player had active at the start of the first boss pull. World buffs in WoW Classic are powerful consumable-strength effects obtained in the open world before the raid and cannot be reapplied during combat.

For each player, the service records which expected world buffs were present and which were missing, then calculates a per-player compliance percentage (buffs present / buffs expected). The raid-wide average compliance rate is the primary metric fed into the Preparation score.

---

## Tracked buffs (Vanilla Classic)

| Buff | Source |
|------|--------|
| Rallying Cry of the Dragonslayer | Head of Onyxia / Nefarian turn-in |
| Warchief's Blessing / Spirit of Zandalar | Bloodlust offensive / Zul'Gurub (Horde) or equivalent Alliance buff |
| Songflower Serenade | Felwood |
| Mol'dar's Moxie | Dire Maul Tribute |
| Slip'kik's Savvy | Dire Maul Tribute |
| Sayge's Fortune (applicable variants) | Darkmoon Faire |

The tracked set and DMF variants are configurable per expansion.

---

## Darkmoon Faire handling

DMF buffs are only evaluated when the Faire was open on the raid date. The service derives DMF availability from the real-world calendar: the Faire opens the Monday following the first Friday of each month and runs for seven days. Reports from non-DMF weeks do not penalise players for not having a buff that was unavailable.

---

## Scoring

Each player's compliance percentage (0 to 100) feeds a raid-wide average. That average is normalised against a target and contributes to the Preparation category score alongside consumables and gear enchants.

---

## Why it matters

World buffs are one of the largest DPS multipliers available in WoW Classic. A fully world-buffed player can deal 20 to 40% more damage than the same player without them. A guild that consistently raids at 70% world buff compliance is operating at a material disadvantage compared to one at 95%. The service makes compliance measurable and comparable across raid nights.

---

## WCL data used

Combatant info (Tier 2). WarcraftLogs captures each player's active buff aura list at the start of each fight. The service reads from the first boss pull snapshot.

---

## Implementation notes

Because combatant info is captured per-fight, players who join mid-raid or disconnect and reconnect may have different buff states on later bosses. Results are per-boss and can be aggregated to a raid-wide view or inspected per encounter.
