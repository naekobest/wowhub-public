# Damage Taken

**Service key:** `damage_taken`
**Category:** Performance
**Expansions:** All
**Contributes to scoring:** Yes — per-player relative scoring feeds into the Performance category score

---

## What it measures

Total damage taken per player across three segments:

| Segment | Coverage |
|---------|----------|
| Trash | All non-boss combat in the log |
| Boss | All boss pull windows aggregated |
| Total | Full log including both boss and trash |

The service also aggregates the top damage sources raid-wide so guild leaders can identify which boss abilities dealt the most cumulative damage to the raid group.

---

## Scoring

Each non-tank player is scored relative to the raid average boss damage taken. Players who took less damage than average score higher; players who took more score lower. Tanks are excluded from scoring (score = null) since their damage intake is role-inherent and not comparable to DPS or healers.

The raid-wide damage taken score is the average of all non-tank player scores.

---

## Why it matters

Damage taken scoring surfaces players who are consistently standing in avoidable mechanics or not using defensive cooldowns, even when they survive. The deaths service catches the fatal outcome; damage taken catches the pattern before a player dies.

- **Identifying standouts:** A DPS player who took significantly more damage than peers on the same boss is likely failing a personal avoidance mechanic.
- **Raid-wide source ranking:** Knowing which ability dealt the most total damage across all players helps prioritise healing assignments, defensive cooldown timing, and resist gear requirements for future progression.

---

## WCL data used

DamageTaken table (Tier 2), with a separate trash data pass when `needsTrashData` is declared.

---

## Implementation notes

The three-segment model is consistent with DPS and healing for side-by-side comparison. The raid-wide top source ranking is computed from the boss segment by default, where source distribution is most meaningful.
