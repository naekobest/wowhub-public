# Healing Metrics

**Service key:** `healing`
**Category:** Performance
**Expansions:** All
**Contributes to scoring:** Yes — overhealing percentage is a weighted input to the Performance category score

---

## What it measures

Healing per second (HPS) and overhealing percentage per healer, segmented across three windows:

| Segment | Coverage |
|---------|----------|
| Trash | All non-boss combat in the log |
| Boss | All boss pull windows aggregated |
| Total | Full log including both boss and trash |

Only players assigned a healer role by the WCL roster are included.

### Overhealing

Overhealing percentage is raw healing minus effective healing, expressed as a fraction of raw healing. A value of 60% means 60% of healing output was absorbed by targets already at full health.

### Per-fight data

For each boss fight, the service records effective HPS, raw HPS, and overhealing percentage per healer. These can be inspected independently of the raid-wide summary.

---

## Scoring

The overhealing percentage across all healers averaged to a raid-wide value feeds the Performance category score. The score is inverted: 0% overhealing scores 100, and overhealing at the configured maximum threshold scores 0. The threshold is configurable per expansion.

Lower overhealing means healers were casting reactive, efficient heals rather than pre-emptively flooding the raid.

---

## Why it matters

In WoW Classic, healer mana is a genuine resource constraint on longer boss fights. Healers who overheal by 60% are spending significantly more mana per point of effective healing than healers at 30%. This accelerates mana exhaustion, which can lead to healer downtime near enrage timers. The overhealing percentage is the most objective measure of healing efficiency available from log data alone.

---

## WCL data used

Healing table (Tier 2), with a separate trash data pass when `needsTrashData` is declared.

---

## Implementation notes

The three-segment model (trash, boss, total) is shared with DPS and damage taken, allowing consistent comparison across performance categories.

Absorb-based healing and shield applications are not included in the WCL healing table in the same way as direct heals. Absorb contributions may appear as a separate row or not at all depending on the expansion and WCL API version. The service does not attempt to merge absorb data with heal data.
