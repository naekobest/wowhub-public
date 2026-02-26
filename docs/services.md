# Analysis Services

Full list of implemented analysis services. Each service produces one type of scored result that feeds into the overall raid score via `ReportCardService`.

Expansion abbreviations: **V** = WoW Classic (Vanilla), **SoD** = Season of Discovery, **All** = every supported expansion.

## Execution

Mechanics that directly affect raid performance if missed.

| Service | Key | Expansions | Description |
|---------|-----|------------|-------------|
| InterruptTrackingService | `interrupts` | All | Tracks interrupt attempts per player relative to available cast windows. Players with low interrupt rates on interruptible targets are flagged. |
| ArmorDebuffService | `armor_debuff` | All | Measures Sunder Armor and Expose Armor uptime per boss fight. Tracks stack count over time and per-player cast contributions. |
| DebuffUptimeService | `debuff_uptime` | All | Tracks uptime of key raid debuffs (Curse of Recklessness, Curse of Elements, Faerie Fire, etc.) per boss fight. Spell IDs and uptime thresholds are configured per expansion. |
| IgniteService | `ignite` | V, SoD | Detects dropped Ignite rolling combos for fire mages. Tracks consecutive Ignite ticks per source player and flags chains that ended before the 2.15 s window elapsed. |
| IgniteGriefingService | `ignite_griefing` | V, SoD | Detects which mages reset another mage's active Ignite by casting their own fire spell mid-chain. Results are broken down per boss with the offending player, triggering spell, and reset count. Non-mages and untracked spells are excluded. |

## Preparation

What players bring to the raid before the first pull.

| Service | Key | Expansions | Description |
|---------|-----|------------|-------------|
| WorldBuffService | `world_buffs` | All | Records which world buffs each player had active at the first boss pull. Compliance rate is calculated per buff and per boss. |
| ConsumableService | `consumables` | All | Checks flasks, elixirs, food buffs, and role-relevant consumables per player at pull time. |
| GearEnchantService | `gear_enchants` | All | Validates that all enchantable slots carry a permanent enchant. Slots are configured per expansion. Two-handed weapons suppress the offhand enchant check. Slot-specific `enchantable_item_ids` filters prevent false positives for off-hand items that cannot receive weapon enchants. |
| FrostResistanceService | `frost_resistance` | V, SoD | Confirms that players wear the required frost resistance set for resistance check fights (e.g. Sapphiron, Ahune). Compares equipped item IDs against a known resistance set defined per fight. |

## Performance

How effectively players use their toolkit during combat.

| Service | Key | Expansions | Description |
|---------|-----|------------|-------------|
| DpsService | `dps` | All | Tracks damage per second for DPS players across three segments: trash pulls, boss fights, and full clear total. Healers and tanks are excluded from rankings. |
| HealingMetricsService | `healing` | All | Tracks healing per second per healer across the same three segments (trash, boss, total). |
| DamageTakenService | `damage_taken` | All | Records total damage taken per player broken down by trash, boss, and full clear. Avoidable vs. unavoidable classification is planned for a future update. |
| DeathAnalysisService | `deaths` | All | Counts player deaths per boss fight. Flagged as avoidable or unavoidable based on the killing blow event type. |

## Buffs

Raid-wide buff infrastructure and uptime.

| Service | Key | Expansions | Description |
|---------|-----|------------|-------------|
| RaidBuffUptimeService | `buff_uptime` | All | Measures uptime of persistent raid buffs (Blessing of Kings, Mark of the Wild, etc.) across the raid for each boss fight. |
| PlayerBuffCoverageService | `player_buff_coverage` | All | Records which players received each tracked buff and for how long. Useful for identifying gaps in buff distribution. |

## Overview

| Service | Key | Description |
|---------|-----|-------------|
| ReportCardService | — | Runs last. Reads all other services' persisted results, calculates a weighted 0–100% score per category, and derives the overall raid score. Category weights are configurable per expansion. |
