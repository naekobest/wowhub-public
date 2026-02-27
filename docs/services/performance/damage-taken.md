# Damage Taken

**Service key:** `damage_taken`
**Category:** Performance
**Expansions:** All
**Contributes to scoring:** Informational — does not currently feed into the Performance score

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

## Why it matters

Damage taken by itself is not scored because a significant fraction is unavoidable for tanks and some roles. The data is useful for two separate purposes:

- **Identifying standouts:** A DPS player who took significantly more damage than peers on the same boss is likely failing a personal avoidance mechanic. The deaths service classifies avoidable kills, but damage taken shows the pattern before a player dies.
- **Raid-wide source ranking:** Knowing which ability dealt the most total damage across all players helps prioritise healing assignments, defensive cooldown timing, and resist gear requirements for future progression.

Avoidable vs. unavoidable classification per damage source is planned for a future update, at which point damage taken may contribute to the Performance score.

---

## WCL data used

DamageTaken table (Tier 2), with a separate trash data pass when `needsTrashData` is declared.

---

## Implementation notes

The three-segment model is consistent with DPS and healing for side-by-side comparison. The raid-wide top source ranking is computed from the boss segment by default, where source distribution is most meaningful.
