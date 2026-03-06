# Changelog

## 2026-03-06

### App Name

WarcraftPulse is now the official app name. The name is read from `VITE_APP_NAME` instead of being hardcoded throughout the frontend — self-hosted instances can configure the name without code changes.

## 2026-03-05

### Changelog

A public changelog is now available at `/changelog`. Published entries are listed in a two-panel index view. Each entry has its own detail page with GitHub-style Markdown rendering, including table support.

Admin UI: create, edit, and publish changelog entries. A publish action fans out notifications to all users. Notification links point directly to the relevant entry. Published entries can now be edited without unpublishing first.

A Changelog link has been added to the sidebar navigation.

### Public Profile URLs: Sequential Numbers

Public profile URLs have changed from `/u/{username}` to `/u/{number}`. Each user is assigned a sequential profile number at account creation. Existing users were backfilled in signup order. The new format is opaque — it does not expose usernames in the URL.

### Gear Check: Boss-Context Warnings

The Gear Enchant service now detects items that are counterproductive for a specific boss encounter. The initial implementation targets undead-slaying items: weapons, temporary enchants, and permanent enchants that only deal bonus damage to undead targets. These are detected on Grand Widow Faerlina and Maexxna, which are not undead and receive no benefit from these items.

Per-boss context warnings appear in the Gear Enchant section when a player has an undead-slaying item equipped during a fight where it provides no bonus.

### Frost Resistance: Enchants, Per-Slot Breakdown, Role Thresholds, Kel'Thuzad

The Frost Resistance service now tracks enchant-based resistance alongside item stats. The per-slot breakdown in the result shows item and enchant contributions separately, so it is clear whether a player's FR comes from dedicated resist gear or enchants.

Resistance targets are now role-based: melee classes target 80 FR, ranged classes (Mage, Warlock, Hunter, Priest) target 100 FR. The previous flat target of 120 over-penalized melee and under-penalized ranged.

Kel'Thuzad has been added as a resistance check encounter alongside Sapphiron.

### Ignite Griefing: Phase Classification, CD Tracking, Talent Check

The Ignite Griefing service has been substantially overhauled.

**Phase classification:** Ignite windows are now classified as opener (before Fire Vulnerability reaches 5 stacks) or real (after 5 stacks). Grief detection is skipped for opener windows — the early cast sequence during FV ramp-up follows different rules and should not generate false grief flags.

**Revised maintenance rules:** Fire Blast is no longer flagged as a grief at 5 stacks. At full stacks, Fire Blast is always a valid maintenance helper or panic save. Fireball and Pyroblast are only flagged when they occur more than 3 seconds after the 5-stack was reached within that window.

**CD tracking:** Each contributing spell in the Ignite tab now shows colored status dots indicating whether Combustion, a spell power trinket (Warmth of Forgiveness, MQG), or Power Infusion was active at the time of the crit.

**Talent check:** The service reads CombatantInfo data to detect whether a mage has the Incinerate talent. A missing-talent warning is surfaced in the result for fire mages who lack it.

### Trinket Usage Tracking

New analysis service: `trinket_usage` in the Performance category. Tracks on-use trinket and racial ability activations per player per boss.

Detection uses two data sources: `applybuff` events for trinkets and cast events for racial abilities. CombatantInfo gear slots 12 and 13 are checked to detect trinkets that were equipped but never used — these appear with 0% efficiency and correctly reduce the player's score.

Scoring: `actual_uses / expected_uses`, capped at 100%. Expected uses are derived from fight duration divided by the trinket cooldown. Racials are tracked but not scored.

Configured trinkets for Vanilla Classic include major DPS trinkets (Zandalarian Hero Charm, Earthstrike, Badge of the Swarmguard, Jom Gabbar, Kiss of the Spider), healer and tank trinkets, and racial abilities (Blood Fury, Berserking).

## 2026-03-04

### Expose Armor Drop Detection: Refined Gap Analysis

The Expose Armor drop detection introduced on 2026-03-03 has been refined with more accurate gap classification and scoring.

**Gap filtering**: Gaps under 2 seconds are now filtered as WCL timing artifacts. Short transitions between casts produce sub-2s debuff gaps that do not represent actual Expose Armor coverage failures.

**Revised severity tiers**:

| Threshold | Severity | Timeline color |
|-----------|----------|---------------|
| 2–5 seconds | Short | Yellow |
| 5–15 seconds | Medium | Orange |
| Over 15 seconds | Long | Red |

**Expose score**: The raid summary now includes an `exposeScore` that penalises raids proportionally to the number, duration, and severity of Expose Armor gaps across all boss fights.

**"Not maintained" state**: Bosses where Expose Armor uptime is below 20% and no gap data exists are flagged as "Not maintained" rather than showing misleading gap counts.

**Gap duration display**: Per-boss fallback rows now show the duration of each gap inline.

## 2026-03-03

### Major Cooldown Usage Tracking

New analysis service: `cooldown_usage` in the Execution category. Tracks class-specific major cooldowns per player per boss using WCL Cast events.

Three cooldown types are distinguished:

| Type | Behavior | Examples |
|------|----------|----------|
| Throughput | Scored (actual/expected casts * 100) | Death Wish, Adrenaline Rush, Arcane Power |
| Utility | Tracked with cast targets | Power Infusion, Innervate, Lay on Hands |
| Defensive | Informational only | Shield Wall, Last Stand |

Talent-required cooldowns are auto-detected via a first-pass scan of the log. A cooldown is only scored if the player used it at least once during the raid, preventing false penalization of players who did not talent into it. Utility cooldowns like Power Infusion and Innervate track cast targets with self-cast detection, so raid leaders can see which player received each external cooldown.

Expected cast count is derived from fight duration divided by cooldown duration. The score is `min(actual / expected, 1.0) * 100`.

### Interrupt Response Rate

The interrupt service now fetches interruptible casts from the WCL Cast events table for configured boss encounters. Each expansion config includes a mapping of encounter IDs to interruptible spell IDs.

Two new metrics are available per boss:

- **Interruptible casts**: how many casts were available to interrupt (interrupted + completed)
- **Response rate**: `interrupted / interruptible_casts` as a percentage

The frontend shows a response rate bar on boss-scoped results where all interrupts are demand-tracked. Mixed scopes (boss + trash) fall back to total interrupt count since trash interrupt windows are not configured.

`demandInterrupts` is tracked separately from `totalInterrupts` to prevent count overflow when mixing boss and trash scopes.

### Per-Player Damage Taken Scoring

Damage Taken now contributes to the Performance score. Each non-tank player is scored relative to the raid average boss damage taken. Players who took significantly less damage than average score higher. Tanks are excluded from scoring (score = null) since their damage taken is role-inherent.

The raid-wide damage taken score is the average of all non-tank player scores.

### Expose Armor Drop Detection

The Armor Debuff service now detects coverage gaps (drops) per boss. When Expose Armor uptime falls below 100%, the service calculates the number of gaps, total gap duration, and longest single gap per boss. The raid summary aggregates drop totals across all bosses. Trash fights are excluded from raid summary drop totals to avoid noise from pull transitions.

A segmented timeline bar in the frontend visualizes Expose Armor coverage per boss, highlighting gaps in red against green uptime bands.

### Major Cooldown Result Component

New frontend result component for the cooldown usage service. Each player's cooldown list is displayed with actual vs. expected cast counts, a usage percentage bar, and WoW quality tier score badges. Utility cooldowns show cast target names. Defensive cooldowns are listed without scores.

### Admin Log Viewer

Application logs are now viewable in the admin panel. The `LogReaderService` parses PSR-3 formatted log files with support for multi-line stack traces. The admin interface provides filtering by log level, date range, and free-text search with cursor-based pagination.

An Application Logs summary card on the admin index page shows recent error and warning counts.

### UI

- **Ctrl+J keyboard shortcut** opens the Analyze modal from anywhere in the app. The shortcut is suppressed when another dialog is open or an input element is focused. A `Ctrl+J` kbd badge is shown on the Analyze Report sidebar item.
- **Notification bell redesign**: visible container with border and background matching the search bar style. Larger badge with soft glow animation replacing the infinite ping. Per-type colored icon backgrounds. Stronger unread/read distinction (opacity and background color). Better empty state with a BellOff icon. Hidden for guests.
- **Early Adopter badge** now uses the achievement system instead of a hardcoded date comparison. The tier badge (Member/Premium/Premium Pro) remains independent and always visible.
- **Onboarding fix**: step 3 "Full documentation" link no longer traps users on the onboarding page. The link now completes the onboarding step before navigating. Onboarding progress persists in sessionStorage so page navigation does not reset to step 1.

### Infrastructure

- **Dependabot** configured for npm and Composer dependency updates with weekly schedule.
- Security dependency bumps: minimatch, ajv, rollup.

## 2026-03-02

### Performance Scoring

DPS and Healing services now contribute to the Performance category score. Both use a class-based median comparison model: for each player, their output (DPS or HPS) is compared against the median for their class within the same raid. This produces a 0 to 100 score that reflects how well a player performed relative to what their class is expected to deliver in that specific raid environment.

The class median approach avoids punishing players for playing lower-DPS specs and avoids rewarding players for simply being in a raid with lower overall output. A Warrior doing 300 DPS in a raid where the Warrior median is 280 scores higher than a Warrior doing 350 in a raid where the Warrior median is 400.

Both services declare `needsPlayers()` in their data requirements so the pipeline provides role-segmented rosters automatically. Healers and tanks remain excluded from DPS rankings. Only healers are included in Healing scoring.

Pets and NPCs are now filtered from DPS and Damage Taken results using the `knownPlayerNames` set from the WCL roster. Previously, hunter pets and warlock pets could appear as separate entries in the player rankings.

The Performance category in the Report Card has been updated:

| Component | Weight within Performance |
|-----------|--------------------------|
| Avoidable deaths | Primary |
| Overhealing | Secondary |
| DPS (class median) | Tertiary |
| Healing (class median) | Tertiary |

DPS and Healing scoring thresholds are configurable per expansion via `dps_median_score` and `healing_median_score` in the expansion config.

### Player Breakdown

All analysis result components have been redesigned with a focus on per-player visibility and deeper drill-down capability.

**Per-player score badges** now appear on every result card that produces player-level data. Each badge uses the same WoW quality tier color scale (Poor through Legendary) derived from the player's individual score for that metric. Score badges are visible on DPS, Healing, Deaths, Interrupts, Dispels, and Armor Debuff result cards.

**Deaths** — redesigned with a per-boss accordion that is always visible (no toggle needed). Each boss section shows the full death roster with the killing ability displayed inline. The "Avoidable" section is hidden when there are zero avoidable deaths for that boss.

**Damage Taken** — added a scope toggle (raid-wide vs. per-zone) and a proportional damage bar visualization. Top damage sources per boss are shown in a dedicated breakdown. Per-player rows link directly to the WCL player detail page.

**Armor Debuff** — split into Sunder/Expose sub-tabs. The player leaderboard is always visible (no longer collapsed by default). Per-player uptime and score badges are shown. Fixed an issue where the CPM header was duplicated per row and zero-uptime bars were hidden.

**Interrupts** — added a response rate bar visualization and interruptible cast demand tracking. The player leaderboard is always visible with a per-boss accordion breakdown.

**Dispels** — same pattern as Interrupts: response rate bars, always-visible leaderboard, per-boss accordion.

**DPS and Healing** — per-player score badges using the class median scoring system. Dual display of efficiency (relative to class median) and absolute values.

### Achievements

WarcraftPulse now has a hybrid achievement system inspired by WoW's own achievement UI. Achievements are tiered badges (not a point system) that unlock as users interact with the platform.

**45 achievements** across 8 categories:

**Upload and Analysis** (tiered I through IV) — Raid Reporter, Boss Slayer, Expansion Explorer, Raid Enthusiast, Death Chronicler, Wipe Watcher, Headcount.

**Score and Quality** (tiered I through IV plus one-time legendary) — Consistent Performer, Execution Expert, Preparation Master, Performance Powerhouse, Buff Commander, Comeback King. Legendary one-time achievements: Rising Star, Purple Reign, Touched by Greatness, Perfectionist, Flawless, Unchallenged, Upward Trend.

**Preparation and Dedication** (tiered I through IV) — Always Prepared, Buff Hoarder, Character Collector, Class Historian, Deathless.

**Class** (tiered I through III plus one-time legendary) — Healer Soul, Tank Instinct, DPS Devotee, Spec Collector. Legendary: Purity of Purpose.

**Community and Social** (tiered) — Recruiter, Guild Liaison, Realm Hopper.

**Meta and Rare** (tiered I through III plus one-time legendary) — Streak, Dedicated Member. Legendary: First Blood, Early Adopter, Night Owl, Weekend Warrior, Nostalgia Trip, Zero to Hero, The Completionist.

Achievement progress is synced daily via a scheduled job. Each achievement checker runs as a SQL predicate, so the sync is efficient even across large user bases. Users can pin up to 3 achievements to their public profile as a showcase. Achievement visibility can be toggled globally in appearance settings.

Tier colors match WoW's item quality scale: Common, Uncommon, Rare, Epic, Legendary.

### Public User Profiles

Users now have a public profile page at `/u/{number}`. The profile displays the user's pinned achievement showcase (up to 3), their visible achievements grouped by category, and basic account information. Profile privacy and achievement visibility are configurable in settings.

### Onboarding

A guided onboarding checklist helps new users through account setup: connecting their WarcraftLogs account, submitting their first report, claiming a character, and configuring their profile. The checklist persists across sessions and dismisses automatically once all steps are completed.

### UI

- Deferred prop loading with skeleton states on category tab content. Switching between analysis categories (Execution, Preparation, Performance, Buffs) shows a pulsing skeleton while data loads rather than a blank page.

## 2026-03-01

### Ignite Rewrite

The Ignite analysis service has been completely rewritten. The previous implementation reconstructed "combo sequences" by tracking consecutive Ignite ticks within a 2,150 ms window per source mage. The new implementation uses the WCL debuff aura band data directly: each continuous Ignite debuff period on the boss is one Ignite instance, and contributing spells are matched to each instance by timestamp overlap.

This is a more accurate model. Ignite in Vanilla Classic is a single shared debuff across all fire mages. Every fire crit from every mage feeds into the same Ignite debuff, stacking up to 5 times. A well-coordinated mage group keeps Ignite rolling continuously with high-value Fireball crits while using Scorch only for maintenance refreshes at full stacks.

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