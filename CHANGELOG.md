# Changelog

## 2026-02-26

### New Analysis Services

- **IgniteGriefingService** — detects per boss which mages reset another mage's active Ignite by casting their own fire spell while a larger chain was in progress. Results include a breakdown by offending mage and triggering spell. Non-mages and untracked spells are excluded from output. Execution category, Vanilla Classic and Season of Discovery.

### Improvements

- **DpsService** is now roster-aware. Healers and tanks are excluded from DPS rankings. The service declares `needsPlayers()` in its data requirements so role assignments from the WCL combatant roster are available during analysis.
- **GearEnchantService** gained two-handed weapon support. When the mainhand slot holds a known two-handed weapon, the offhand enchant check is skipped entirely. A new `enchantable_item_ids` filter per slot restricts enchant requirements to specific item types, so off-hand held items that cannot receive weapon enchants (books, off-hand frills) are no longer flagged as missing enchants.

### Pipeline

- `DataRequirements` gained a `needsPlayers()` flag. Services that need role-segmented roster data declare this flag and receive healer and tank name sets via `ReportContext`.

## 2026-02-25

### New Analysis Services

- **DpsService** — tracks DPS per player across three segments: trash pulls, boss fights, and full clear total. Included in the Performance category for all supported expansions.
- **HealingMetricsService** — tracks HPS per player with the same trash / boss / total breakdown. Previously tracked overall healing only; now segments are available for deeper analysis per fight.
- **DamageTakenService** — records total damage taken per player, broken down by trash, boss, and full clear. Avoidable vs. unavoidable classification is planned for a future update.
- **IgniteService** — detects broken Ignite rolling combos for fire mages. Tracks consecutive Ignite ticks per source player and flags cases where the combo was dropped before the 2.15 s window elapsed. Execution category, Vanilla Classic.
- **FrostResistanceService** — checks that players have the required frost resistance gear equipped for resistance check fights (e.g. Sapphiron, Ahune). Reads combatant info from WarcraftLogs and compares equipped items against a known resist set per expansion. Preparation category.

### Pipeline

- `DataRequirements` gained a `needsTrashData` flag. Services that require trash pull data declare this flag and the QueryPlanner fetches the relevant fight windows alongside boss fight data.
- `ReportContext` exposes `getTrashFights()` alongside the existing `getBossFights()` accessor, giving all services consistent access to both fight types.
- Performance result structures now carry three named segments (`trash`, `boss`, `total`) for DPS, HPS, and damage taken.

### UI

- Report header redesigned: overall score, boss kill count, and zone name now appear as inline KPI chips without a separate summary section.
- Sidebar navigation updated with collapsible per-raid tabs using `NavReport` — switching between Overview, Execution, Preparation, Performance, and Buffs no longer reloads the page.
- Performance tab now renders DPS, HPS, and Damage Taken results with segment switching (trash / boss / total).
- Execution tab now renders Ignite analysis results per player, showing combo count, longest run, and flagged broken sequences.
- Player overview table rebuilt with shadcn Table components. Role tabs (All / Tanks / Healers / DPS) replace the previous flat/grouped toggle. A column visibility dropdown lets users hide Spec and score columns. Sort indicators reset on tab change. Skeleton rows with an amber pulse indicator display while analysis is running.
- WCL API Key setup added to `/settings/api` with a guide and copy buttons for the required OAuth scopes.
