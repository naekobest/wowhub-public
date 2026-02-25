# Analysis Pipeline

## Overview

When a report is submitted, an `AnalyzeRaidJob` is dispatched to the appropriate queue tier.
The job runs the **AnalysisPipeline**, which orchestrates all active analysis services for
the detected expansion.

```
AnalyzeRaidJob
  → AnalysisPipeline::run(report)
      → Load active services for this expansion
      → Sort by priority
      → QueryPlanner: merge DataRequirements → minimal GraphQL queries
      → WarcraftLogsClient: execute queries → ReportContext
      → foreach service: analyze(ReportContext) → persist result
      → ReportCardService runs last → overall score
      → Report status → completed
```

---

## QueryPlanner — API Efficiency

Each analysis service declares what WCL data it needs via a `DataRequirements` value object.
The **QueryPlanner** collects requirements from all active services and merges them into the
minimum number of GraphQL queries before any API call is made. No service makes direct API calls.

WCL queries are tiered by cost:

**Tier 1 — Metadata (cheap, 1 request)**
Fight list, actor roster, zone/expansion info, current API budget. Always fetched first.

**Tier 2 — Aggregated Tables (medium, 1–2 requests)**
Pre-computed server-side summaries: damage done, healing, damage taken, deaths, buff uptimes,
player gear/talents. Covers most Performance and Overview services without touching raw events.

**Tier 3 — Raw Events (expensive, N requests)**
Spell-level event streams filtered by ability ID or event type. Only fetched when a service
explicitly requires it. Examples: interrupt events, world buff combatant info, armor debuff
application/removal events.

The budget is re-checked between Tier 2 and Tier 3. If the app budget has been depleted by
other concurrent jobs, Tier 3 queries are deferred.

---

## Analysis Service Interface

Every analysis service implements the same interface:

```php
interface AnalysisServiceInterface
{
    public function serviceKey(): string;
    public function category(): AnalysisCategory;       // Execution|Preparation|Performance|Buffs|Overview
    public function resultScope(): ResultScope;         // Raid|Boss|Player|BossPlayer
    public function supportedExpansions(): array;       // e.g. ['vanilla', 'tbc']
    public function requiredData(): DataRequirements;   // declares needed WCL fields
    public function priority(): int;                    // lower = runs first
    public function analyze(ReportContext $context): array;
}
```

Services are registered per expansion in config. Adding analysis for a new mechanic means
implementing one class and registering it — nothing else changes.

---

## Expansion Configuration

Each expansion has a config file defining:

- Which services are active
- Spell IDs for tracked abilities (sunder armor, expose armor, world buffs, etc.)
- Scoring thresholds (e.g. "interrupt rate below 80% = Poor")
- Difficulty modes (Vanilla = Normal only; Retail = Normal/Heroic/Mythic)

This means the same `InterruptService` works for every expansion — the expansion config supplies
the interrupt spell IDs and the threshold that constitutes "good" interrupt coverage.

---

## Result Storage

Analysis results are stored in four scoped tables:

| Scope | Table | Key |
|-------|-------|-----|
| Raid-wide | `raid_analysis_results` | `raid_id` |
| Per boss | `boss_analysis_results` | `raid_boss_id` |
| Per player (raid) | `player_analysis_results` | `raid_id + player_id` |
| Per boss + player | `boss_player_analysis_results` | `raid_boss_id + player_id` |

Results are written once (`insertOrIgnore`) — re-submitting the same report is a no-op.
The `data` column is JSONB, which allows each service to store whatever structure it needs
without requiring migrations when services evolve.

---

## ReportCard

`ReportCardService` runs last (priority: 999). It reads all other services' persisted results,
calculates a weighted 0–100% score per category, and derives an overall raid score.

Category weights are configurable per expansion. In Vanilla Classic, Preparation (world buffs,
consumables) carries more weight than in later expansions where consumable culture is different.

The final score per category is mapped to WoW's item quality grades:
Poor · Common · Uncommon · Rare · Epic · Legendary
