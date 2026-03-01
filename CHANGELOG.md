# Changelog

## 2026-03-01

### Ignite Rewrite

The Ignite analysis service has been completely rewritten. The previous implementation reconstructed "combo sequences" by tracking consecutive Ignite ticks within a 2,150 ms window per source mage. The new implementation uses the WCL debuff aura band data directly: each continuous Ignite debuff period on the boss is one Ignite instance, and contributing spells are matched to each instance by timestamp overlap.

This is a more accurate model. Ignite in Vanilla Classic is a shared debuff, so tracking it per source mage was misleading. The new approach tracks the debuff itself and attributes all fire crits that landed during each Ignite window, regardless of which mage cast them.

Per-instance data now includes:
- Duration and time range within the fight
- Maximum tick damage
- Contributing spells with caster name and individual damage
- Total contributing damage across all casters

### Ignite Grief Integration

Grief detection results are now embedded directly into the Ignite view. Each Ignite instance card shows a warning badge with the grief count and an expandable detail section listing every grief event: the spell cast, the responsible player, the phase (build or maintenance), and the damage dealt.

Backend changes:
- `IgniteGriefingService` now produces a `perIgnite` field in its output, indexing griefs by debuff band number (matching the Ignite service's band indices). Each entry contains a grief count and a list of grief events with spell name, player name, damage, resisted amount, fight offset, and phase type.
- The frontend component tree gained an `allSectionResults` prop that passes all results within a report section to each service component. The Ignite component uses this to read grief data from the `ignite_griefing` service without tight coupling between the two services.

The standalone Ignite Griefing tab remains as a raid-wide overview showing total griefs per player and per-boss breakdowns. Per-instance details are now in the Ignite tab.

### UI

- **Collapsible accordion pattern** applied to all result components. Cards that previously rendered all detail content inline now use a collapsed header with a chevron toggle. Expanding a card reveals the detail section with a smooth CSS grid animation. This reduces visual noise on initial load while keeping all data accessible.

### Documentation

- Service documentation for Ignite updated to reflect the debuff-band-based approach. The "How it works" explanation in both the standalone page and the in-report help sheet now covers uptime tracking, per-instance breakdowns, and grief detection in a single view.
- Ignite Griefing documentation updated to describe the raid-wide overview role and cross-references the Ignite tab for per-instance grief details.

## 2026-02-28

### UI

- **Boss / Trash / Total scope switcher** — all analysis sections (Execution, Performance, Buffs) now have a scope toggle. Switching between Boss, Trash, and Total filters the displayed results without a page reload. The toggle is hidden for sections that only produce boss-scoped data.

### Analysis Services

All services that previously reported only boss-scoped results now produce three named segments — `boss`, `trash`, and `total` — so the scope switcher has data to work with across the entire report:

- **DeathAnalysisService** — trash avoidable deaths are always zero by design (unavoidable by nature); this is now explicitly documented in the result structure.
- **DebuffUptimeService** — total segment is computed via `array_merge` to correctly aggregate overlapping boss and trash windows.
- **InterruptTrackingService** — full boss/trash/total breakdown added.
- **DispelTrackingService** — full boss/trash/total breakdown added.
- **RaidBuffUptimeService** — full boss/trash/total breakdown added.

### Pipeline

- Trash fight windows are now fetched for all services that declare `needsTrashData`, including Deaths (previously the flag was hardcoded to `false` for that fetch).
- Buff event data is now built for trash fight windows in addition to boss fights, making buff-based services fully scope-aware.

### Fixed

- `needsTrashData` was ignored during the Deaths event fetch — deaths in trash pulls were missing from results.
- `ScopeToggle` visibility is now guarded explicitly; the previous `?? false` cast could produce incorrect visibility in edge cases.

## 2026-02-27

### New Analysis Services

- **DispelTrackingService** — tracks successful dispels per boss fight, broken down by Magic, Poison, Disease, and Curse. Class-aware classification covers Paladin, Priest, Druid, Mage, and Shaman using per-expansion spell ID configuration. Produces a per-player leaderboard per type and a raid-wide summary. Available as informational data in the Execution section; does not affect the Execution score.

### Improvements

- **Ignite Griefing: stack-tracking algorithm** — replaced the previous event-window scan with a full Ignite stack state machine. Two grief phases are now distinguished: build phase (stacks 1 through 4, where any Scorch, Fire Blast, Blast Wave, or Flamestrike hit is flagged) and maintenance phase (5 stacks, where Fire Blast, Blast Wave, and Flamestrike are flagged). Fire Blast is exempted from maintenance flagging when the gap since the last fire crit exceeds 2,500 ms, meaning Scorch could not have landed in time to refresh the chain. Each grief entry records the spell, damage, resisted amount, fight offset in seconds, and phase type.
- **Armor Debuff** — the raid summary now includes zone-grouped uptime and casts-per-minute for multi-zone raids. A bug in the WCL effective cast filter was also corrected; it had silently returned zero casts on some log formats.
- **Query Planner** — services can now declare `needsReportWideEvents` to fetch events once for the full log rather than per boss. Trash debuff data is fetched in a separate pass for services that declare `needsTrashData`.

### Infrastructure

- **`WclAura` value object** — centralises normalisation of WarcraftLogs' dual-key aura field structure. The WCL aura table returns spell IDs under either `guid` or `abilityGameID` and uptime under either `uptimeMs` or `total` depending on context. `WclAura::guid()` and `WclAura::uptimeMs()` resolve the correct field transparently, replacing per-service null-check patterns across all buff and debuff services.
- **`PlayerRole` enum** — formal type for tank, healer, and DPS role mapping with a `wclGroupKey()` method that resolves each role to its WCL `playerDetails` group key. Replaces hardcoded string arrays throughout the pipeline context hydration and analysis service layer.

### Fixed

- Dispel classification uses flat WCL event fields (`abilityGameID`, `extraAbilityGameID`) rather than the nested `ability.guid` object, which is absent from raw events fetched via the events API.
- Config key corrected from `player_buff_uptime` to `player_buff_coverage` in `enabled_services`.
- Pipeline error logging now includes the exception class and service key, making log-driven debugging faster.

### Documentation

- **Analysis Services** (`docs/services.md`) fully rewritten. The previous table-based overview is replaced with detailed per-service explanations: what each service measures, why the mechanic matters, what WarcraftLogs data it consumes, and how its output feeds into scoring. The Report Card section now documents the scoring formula for each category (Execution, Preparation, Performance, Buffs), how missing data is handled via weight redistribution, and the full score-to-grade mapping table. Category weight rationale is also included.
- Added per-service documentation files in `docs/services/` with technical analysis, scoring formulas, WCL data dependencies, and implementation notes for each analysis service.

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
