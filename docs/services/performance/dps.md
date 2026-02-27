# DPS

**Service key:** `dps`
**Category:** Performance
**Expansions:** All
**Contributes to scoring:** Informational — does not currently feed into the Performance score

---

## What it measures

Damage per second for each DPS player across three segments:

| Segment | Coverage |
|---------|----------|
| Trash | All non-boss combat in the log |
| Boss | All boss pull windows aggregated |
| Total | Full log including both boss and trash |

Per-boss results are also available showing DPS for each specific encounter.

Healers and tanks are excluded from the ranking. Role assignments come from the WCL player roster, which derives roles from talent specialisation.

---

## Calculation

For each segment, DPS is computed as total damage divided by total active fight time for that segment. Trash time is the sum of all non-boss combat windows; boss time is the sum of all boss pull durations. Gaps between pulls are not included in the time denominator.

---

## Why it matters

DPS numbers provide reference context alongside scored metrics. A guild leader reviewing the Execution or Preparation results can cross-reference whether performance gaps in mechanics correlate with DPS output or appear to be independent.

DPS is not scored because it is a downstream result of Execution (buff and debuff uptime, mechanic compliance) and Preparation (consumables, world buffs) rather than an independent input. Scoring DPS separately would double-count the factors already measured by those categories.

---

## WCL data used

DamageDone table (Tier 2). A separate trash pull pass is performed when `needsTrashData` is declared.

---

## Implementation notes

The three-segment model (trash, boss, total) is consistent across DPS, healing, and damage taken. This allows direct comparison between segments on the report card.

Role exclusion is implemented at the roster level. The service receives pre-segmented player groups (tanks, healers, DPS) from the pipeline context and only processes the DPS group.
