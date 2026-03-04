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

| Threshold | Severity |
|-----------|----------|
| Under 2 seconds | Filtered (WCL timing artifact) |
| 2–5 seconds | Short |
| 5–15 seconds | Medium |
| Over 15 seconds | Long |

Gaps under 2 seconds are silently discarded. Short cast transitions between a Rogue applying and a Warrior re-applying Expose Armor produce sub-2s debuff gaps that do not represent real coverage failures and would inflate drop counts.

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

### Expose Armor drop detection

When Expose Armor uptime falls below 100%, the service calculates coverage gaps per boss: number of drops, total gap duration, and longest single gap. The raid summary aggregates drop totals across all boss fights. Trash fights are excluded from raid summary drop totals to avoid noise from pull transitions.

Gaps under 2 seconds are filtered as WCL timing artifacts before counting or scoring. Remaining gaps are classified by severity (Short / Medium / Long) and the raid summary includes an `exposeScore` that penalises the raid proportionally to their frequency and duration.

A segmented timeline bar visualizes Expose Armor coverage per boss. Gap segments are color-coded by severity: yellow for short, orange for medium, red for long. Bosses with Expose Armor uptime below 20% and no gap data are flagged as "Not maintained" rather than showing empty gap counts.
