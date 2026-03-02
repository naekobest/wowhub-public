# DPS

**Service key:** `dps`
**Category:** Performance
**Expansions:** All
**Contributes to scoring:** Yes — class-based median comparison feeds into the Performance category score

---

## What it measures

Damage per second for each DPS player across three segments:

| Segment | Coverage |
|---------|----------|
| Trash | All non-boss combat in the log |
| Boss | All boss pull windows aggregated |
| Total | Full log including both boss and trash |

Per-boss results are also available showing DPS for each specific encounter.

Healers and tanks are excluded from the ranking. Pets and NPCs are filtered using the `knownPlayerNames` set from the WCL roster. Role assignments come from the WCL player roster, which derives roles from talent specialisation.

---

## Calculation

For each segment, DPS is computed as total damage divided by total active fight time for that segment. Trash time is the sum of all non-boss combat windows; boss time is the sum of all boss pull durations. Gaps between pulls are not included in the time denominator.

---

## Scoring

DPS is scored using a class-based median comparison model. For each DPS player, their output is compared against the median DPS for their class within the same raid. A player at or above the class median by the configured threshold margin receives a score of 100. Players below the median score proportionally lower.

This approach avoids penalizing players for choosing lower-DPS specs and avoids rewarding players for being in raids with lower overall output. A Warrior doing 300 DPS in a raid where the Warrior median is 280 scores higher than a Warrior doing 350 in a raid where the Warrior median is 400.

The median threshold is configurable per expansion via `dps_median_score` (default 75). Per-player score badges using WoW quality tier colors (Poor through Legendary) are displayed on the result card.

---

## Why it matters

DPS output is partly a downstream result of Execution and Preparation, but class median comparison isolates the player-skill component. A player who brought all their consumables and world buffs (good Preparation score) but still underperforms their class median has a mechanical or rotation issue that the class median score surfaces independently.

---

## WCL data used

DamageDone table (Tier 2). A separate trash pull pass is performed when `needsTrashData` is declared.

---

## Implementation notes

The three-segment model (trash, boss, total) is consistent across DPS, healing, and damage taken. This allows direct comparison between segments on the report card.

Role exclusion is implemented at the roster level. The service receives pre-segmented player groups (tanks, healers, DPS) from the pipeline context and only processes the DPS group.

Pet and NPC filtering uses the `knownPlayerNames` set derived from the WCL roster's combatant info. Any entry in the damage table whose name is not in the known player set is excluded from rankings and median calculations.
