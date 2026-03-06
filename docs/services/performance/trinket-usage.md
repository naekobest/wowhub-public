# Trinket Usage

**Service key:** `trinket_usage`
**Category:** Performance
**Expansions:** Vanilla Classic
**Contributes to scoring:** Yes

---

## What it measures

Whether players activated their on-use trinkets and racial abilities on cooldown during boss fights. Trinkets that sit unused in a player's bags reduce their effective output without appearing in any damage or healing meter.

Results record per player per boss:
- Which tracked trinkets and racials were detected
- How many times each was used
- How many uses were expected given the fight duration and cooldown
- An efficiency score per trinket (actual / expected, capped at 100%)
- An overall trinket score for the player on that boss

---

## Detection

Two data sources are combined:

**Buff events (`applybuff`):** On-use trinkets that apply a buff are detected when the buff appears on the player. This is the primary detection method for trinkets.

**Cast events:** Racial abilities (Blood Fury, Berserking, etc.) are detected via cast events, since they do not always produce a visible buff aura.

**CombatantInfo gear check:** Gear slots 12 and 13 (trinket slots) from the combatant info snapshot are scanned against the configured item ID list. If a tracked trinket is equipped but never used during a fight, it appears in the results with 0 uses and 0% efficiency. Without this check, unused trinkets would be invisible — there is no absence-of-event to detect.

---

## Scoring

Per-trinket score: `actual_uses / expected_uses`, capped at 100%.

Expected uses are derived from `fight_duration / trinket_cooldown`. A fight lasting 150 seconds with a 90-second cooldown produces an expected count of 1 (floored), so a single use scores 100%.

Racial abilities are tracked but not scored — they are informational only, since their cooldown and benefit vary significantly by situation.

The player's overall trinket score for a fight is the average of all scored trinket efficiencies. Players with no tracked trinkets equipped receive no score.

---

## Why it matters

On-use trinkets are among the largest per-player throughput cooldowns available in Vanilla Classic. A DPS trinket like Zandalarian Hero Charm or Badge of the Swarmguard, used twice in a boss fight where three uses were possible, represents a non-trivial throughput loss that does not appear in the DPS meter — the player's damage is lower, but it looks like a bad pull rather than a missed cooldown. The trinket usage service makes this visible.

Equipped-but-unused detection is particularly important: it catches players who forgot to swap trinkets between pulls, or who brought situational trinkets to a fight where they should have equipped general-purpose items.

---

## Configured trinkets (Vanilla Classic)

| Trinket | Type | Cooldown |
|---------|------|----------|
| Zandalarian Hero Charm | DPS | 120s |
| Zandalarian Hero Medallion | Healer | 120s |
| Earthstrike | DPS | 120s |
| Badge of the Swarmguard | DPS | 180s |
| Jom Gabbar | DPS | 90s |
| Kiss of the Spider | DPS | 120s |
| Slayer's Crest | DPS | 120s |
| Renataki's Charm of Beasts | DPS | 180s |
| Core of Ar'kelos | Healer | 90s |
| Royal Seal of Eldre'Thalas | Healer | 120s |
| Scarab Brooch | Tank | 45s |
| Onyxia Blood Talisman | Tank | 60s |
| Blood Fury (Orc racial) | Racial | 120s |
| Berserking (Troll racial) | Racial | 180s |
| Stoneform (Dwarf racial) | Racial | 300s |

---

## WCL data used

- Buffs aura table for on-use trinket buff detection (Tier 2)
- Cast events for racial ability detection (Tier 3)
- CombatantInfo gear data for equipped-but-unused detection (Tier 2)