# Debuff Uptime

**Service key:** `debuff_uptime`
**Category:** Execution
**Expansions:** All
**Contributes to scoring:** Yes — weighted input to the Execution category score

---

## What it measures

Uptime of key raid debuffs on the primary boss target, expressed as a percentage of each fight's duration. The set of tracked debuffs and the scoring thresholds are configurable per expansion.

In WoW Classic the tracked debuffs are:

| Debuff | Class required | Condition |
|--------|---------------|-----------|
| Faerie Fire | Druid | Only tracked when at least one Druid is in the raid |
| Curse of Recklessness | Warlock | One or more Warlocks |
| Curse of Elements | Warlock | Two or more Warlocks |
| Curse of Shadow | Warlock | Three or more Warlocks |

Debuffs that require a class absent from the raid are silently excluded. They receive no score of zero — they simply do not appear in the results for that log.

---

## Scoring

Each debuff's raw uptime percentage is normalised against a configurable target. A 95% uptime scores 100; lower values scale linearly toward 0.

The Execution category score is an average of all scored metrics in that category. Debuff uptime contributes alongside armor debuff uptime and interrupt coverage.

Gap severity is also classified for each debuff:

| Gap | Classification |
|-----|---------------|
| Under 5 seconds | Minor |
| 5 to 10 seconds | Major |
| Over 10 seconds | Critical |

These labels appear in per-boss results to help identify when uptime dropped and how badly.

---

## Why it matters

Raid debuffs in WoW Classic amplify the damage output of entire damage types — fire and frost mages both benefit from Curse of Elements, and Faerie Fire adds physical damage output for the entire melee and physical DPS group. A single debuff dropping for 30 seconds on a 3 minute boss can cost more damage than a single DPS player deals in the same window.

---

## WCL data used

Debuff aura table (Tier 2). Per-boss, per-fight windows are fetched by the Query Planner and stored in the report context. Each aura entry includes time bands indicating when the debuff was active on the target.

---

## Implementation notes

The service is composition-aware at query time. Before fetching data, it checks the raid roster for the required classes and filters the tracked debuff list accordingly. This prevents the service from scoring a 0% uptime for a debuff that the raid literally could not apply.

Per-boss results feed into a raid-wide average. The raid average is the primary input to scoring. Zone-grouped results are produced for multi-zone raid logs.
