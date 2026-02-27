# Report Card

**Service key:** `report_card`
**Category:** None (meta-service)
**Expansions:** All
**Runs last** — priority 999, always executes after all category services

---

## What it does

`ReportCardService` runs after all other services have completed and persisted their results. It reads the raid-scoped results for each category from the database, computes a 0 to 100 score per category, derives an overall raid score from a weighted average, and maps each score to WoW's item quality grade scale.

The service makes no WCL API calls.

---

## Category scores

Each category score is computed from the metrics that contributed results:

**Execution (35% weight)**
Average of debuff uptime score, armor debuff uptime score, and interrupt coverage score. Each metric is normalised to 0 to 100 against a configurable target (e.g. 95% debuff uptime maps to 100). Ignite, Ignite Griefing, and Dispels are informational and do not contribute to this average.

**Preparation (30% weight)**
Average of world buff compliance score, consumable compliance score, and gear enchant score. Frost Resistance is informational and does not contribute.

**Performance (20% weight)**
Average of avoidable deaths score (0 deaths = 100, threshold deaths = 0) and healer overhealing score (0% overheal = 100, threshold overheal = 0). DPS, HPS, and Damage Taken are informational and do not contribute.

**Buffs (15% weight)**
Derived from average raid buff uptime across all tracked reliable buffs, normalised against a target of 95% uptime. Player Buff Coverage is informational and does not contribute.

---

## Weight redistribution

If a category has no data (e.g. all performance services returned unavailable, so the Performance score cannot be computed), the missing weight is redistributed proportionally across the remaining categories. This prevents a 0 score from pulling down the overall result when a service genuinely had no data to analyse.

---

## Score-to-grade mapping

| Score | Grade |
|-------|-------|
| 95 to 100 | Legendary |
| 85 to 94 | Epic |
| 70 to 84 | Rare |
| 50 to 69 | Uncommon |
| 25 to 49 | Common |
| 0 to 24 | Poor |

These thresholds use WoW's own item quality vocabulary as the grading language. A Legendary overall score means the raid performed at an exceptional level across all measured categories. A Poor score indicates significant and addressable problems in multiple areas.

---

## Category weights

| Category | Weight | Rationale |
|----------|--------|-----------|
| Execution | 35% | Mechanic compliance is the highest-leverage category in Classic. Dropped debuffs and missed interrupts have direct and compounding damage consequences. |
| Preparation | 30% | In Vanilla Classic, world buffs and consumables represent a meaningful fraction of total raid DPS output. Preparation is controllable and repeatable. |
| Performance | 20% | DPS output is a downstream result of Execution and Preparation, not an independent input. Avoidable deaths are scored here. |
| Buffs | 15% | Buff uptime in organised guilds tends to be high and has lower variance than other categories. Its weight reflects its relative importance, not its difficulty. |

Weights are configurable per expansion. What matters in Vanilla Classic differs from what matters in Mists of Pandaria, where buff infrastructure and interrupts may carry different relative importance.

---

## Implementation notes

The Report Card service reads from the database rather than operating on in-memory data. This allows it to access results from all previous services regardless of whether they ran in the current job execution or in a prior checkpoint. The pipeline marks each service as checkpointed after completion, so a job that fails mid-run can resume from where it left off and the Report Card service will have access to all previously persisted results.
