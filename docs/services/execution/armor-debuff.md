# Armor Debuff

**Service key:** `armor_debuff`
**Category:** Execution
**Expansions:** All
**Contributes to scoring:** Yes — weighted input to the Execution category score

---

## What it measures

Uptime of Sunder Armor and Expose Armor on the primary boss target, plus per-player cast data for every player who contributed stacks. Results are available per boss, per zone (for multi-zone raids), and as a raid-wide summary.

### Uptime

For each boss fight, the service calculates what percentage of the fight duration Sunder Armor and Expose Armor were active on the target. Data comes from the WCL debuff aura table, which records time bands when each debuff was present.

### Cast data

Per-player cast data records how many effective Sunder Armor or Expose Armor casts each player landed and their casts-per-minute rate. This lets a guild leader distinguish between a coverage gap caused by one tank losing focus and a gap caused by a coordination problem across multiple players.

---

## Scoring

Uptime is normalised against a configurable target (default 95%). A 95% combined uptime scores 100; lower values scale linearly toward 0. The Execution category score is an average across debuff uptime, armor debuff uptime, and interrupt coverage.

Gap severity classification:

| Gap | Classification |
|-----|---------------|
| Under 5 seconds | Minor |
| 5 to 10 seconds | Major |
| Over 10 seconds | Critical |

---

## Why it matters

In WoW Classic raid content, full Sunder Armor stacks on the main tank target reduce the boss's physical damage reduction significantly. For physical DPS-heavy compositions the uptime of this debuff is a multiplier on every swing landed by every melee player and every physical DPS ability. Warriors spend global cooldowns on Sunder; if the debuff still falls off, either the tank rotation is misconfigured or the assignment is unclear. The cast breakdown makes it possible to have a specific conversation with the specific player responsible.

---

## WCL data used

- Debuff aura table (Tier 2) — for uptime bands on the boss target.
- Cast events for Sunder Armor and Expose Armor (Tier 3) — for per-player cast rate and effective cast counts.

---

## Implementation notes

The effective cast filter matches only casts that landed (hit or partially resisted); misses are excluded from the CPM calculation because a missed Sunder does not refresh the debuff stack and should not count as meaningful contribution.

Zone-wide uptime and CPM are computed by aggregating all per-boss results within each zone. Zone grouping only appears in the output when more than one distinct zone is present in the log.
