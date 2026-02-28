# Analysis Services

Each analysis service is an independent class that declares what WarcraftLogs data it needs, runs its logic against a `ReportContext`, and persists structured results. Services never talk to the WCL API directly. The `QueryPlanner` merges all service data requirements before the raid is fetched, so each analysis job makes the minimum number of API calls regardless of how many services are active.

Services are registered per expansion in config. Adding a new mechanic means implementing one class and pointing the config at it. Everything else (query planning, result storage, report card aggregation) happens automatically.

Expansion abbreviations used in this document: **V** = WoW Classic (Vanilla) and Fresh Classic, **All** = every currently supported expansion.

For per-service technical documentation including scoring formulas, WCL data dependencies, and implementation notes, see [services/index.md](services/index.md).

---

## Execution

Execution measures whether players performed correctly during combat. These are mechanics where failure has a direct, measurable cost to the raid, not just an indirect one. Poor execution scores mean the raid left damage or survival on the table due to missed rotations, dropped debuffs, or ignored interrupts.

### Debuff Uptime

**Key:** `debuff_uptime` | **Expansions:** All

Tracks the uptime of key raid debuffs on the primary boss target. In WoW Classic the relevant debuffs are Faerie Fire, Curse of Recklessness, Curse of Elements, and Curse of Shadow. These buffs amplify the damage the entire raid deals and are worth significantly more than any individual player's DPS contribution when they drop.

The service is composition-aware. Faerie Fire is only tracked when at least one Druid is in the raid. Curses scale with warlock count: one warlock unlocks Curse of Recklessness, two unlocks Curse of Elements, three unlocks Curse of Shadow. Debuffs that require a class the raid does not have are silently excluded rather than scored as zero.

Each debuff is measured as a percentage of the fight duration it was active on the target. A gap classification (minor, major, critical) is derived from configurable thresholds. Results are segmented into boss, trash, and total scopes. Per-boss results are aggregated to a raid-wide average per scope.

**Why it matters:** A raid with 50% Curse of Elements uptime is playing at a fraction of its potential damage output for fire and frost mages. Even a 10 second gap on a boss with a 3 minute enrage timer matters.

**WCL data used:** Debuff aura table (Tier 2).

### Armor Debuff

**Key:** `armor_debuff` | **Expansions:** All

Measures Sunder Armor and Expose Armor uptime per boss. Armor debuffs reduce the target's physical damage reduction, amplifying melee and physical DPS. In Classic raid content, full Sunder stacks on the main target can represent a 15 to 30% damage increase for physical attackers depending on the boss armor value.

The service tracks both debuffs independently, calculates uptime as a fraction of fight duration, and detects coverage gaps classified by severity. A gap of under 5 seconds is minor; over 10 seconds is critical. Per-player cast data is also collected: how many effective casts each tank landed and at what rate per minute, so a guild leader can see whether the coverage gap came from a single player dropping the ball or from a broader coordination failure.

**Why it matters:** Warriors spend global cooldowns on Sunder. If the debuff still falls off regularly, either the tank rotation is wrong or the assignment is unclear. This service gives the data to have that conversation.

**WCL data used:** Debuff aura table (Tier 2), cast events for Sunder and Expose Armor (Tier 3).

### Interrupts

**Key:** `interrupts` | **Expansions:** All

Tracks the total number of successful spell interrupts per boss fight and the breakdown by player. The service does not calculate an opportunity rate because WarcraftLogs does not expose the number of interruptible cast windows in aggregated form. Instead it normalizes the total interrupt count against a configurable target per expansion and produces a leaderboard of who is carrying interrupt responsibility.

Results show which players interrupted and with which abilities (Counterspell, Pummel, Shield Bash, etc.), segmented by boss, trash, and total scope. The summary across all bosses ranks players by total interrupts.

**Why it matters:** Some bosses have spells that must be interrupted to prevent wipes or significant healing strain. Identifying players who never interrupt is the first step toward fixing assignment coverage.

**WCL data used:** Interrupt events (Tier 3).

### Ignite

**Key:** `ignite` | **Expansions:** V

Ignite is the Mage talent that causes fire crits to apply a stacking DoT on the target. In Vanilla Classic, the game engine processes Ignite as a single shared debuff across all fire mages. This creates a mechanic called Ignite rolling: fire mages want to keep the debuff refreshed continuously with small spells like Scorch, allowing Fireball crits from multiple mages to stack into very large ticks without the debuff resetting the accumulation. Dropping the chain before the 2.15 second window resets the stacking value.

The service measures two things. First, it calculates Ignite uptime as a fraction of boss fight duration using the debuff aura data from WCL. Second, it reconstructs Ignite combo sequences from raw damage events: consecutive ticks from the same mage within the 2.15 second window constitute one combo. The service records the longest combo per player per boss (duration in seconds, maximum tick value, fight offset of where it started) and aggregates to a best-combo summary across the clear.

Ticks below a minimum damage threshold are excluded to filter out noise from DoT clipping artifacts.

**Why it matters:** A well-maintained Ignite chain is one of the highest-leverage mechanics available to a fire mage group in Vanilla. Being able to show mages their personal best combo and where chains dropped gives concrete feedback.

**WCL data used:** Debuff aura table for uptime (Tier 2), DamageDone events filtered to Ignite spell ID (Tier 3).

### Dispels

**Key:** `dispels` | **Expansions:** All

Tracks successful dispels per boss fight, broken down by dispel type: Magic, Poison, Disease, and Curse. The service is class-aware — only players with a dispel-capable class (Paladin, Priest, Druid, Mage, Shaman) are counted. Unknown or pet-sourced dispel events are excluded.

Classification uses per-class logic keyed on WCL event fields:

- **Mage**: always Curse (Remove Curse is the only dispel available).
- **Priest**: defaults to Magic; classified as Disease when the caster spell ID appears in the per-expansion `priest_disease_spells` config list.
- **Paladin**: Cleanse removes Magic, Poison, and Disease. WCL dispel events expose the removed debuff's spell ID (`extraAbilityGameID`), not its school. Classification checks the removed debuff ID against per-expansion `paladin_poison_debuffs` and `paladin_disease_debuffs` lists. Unmatched debuffs default to Magic.
- **Druid**: uses caster spell ID (`abilityGameID`) checked against the `druid_poison` config list; defaults to Curse when the list is empty or the spell is not matched (covers expansions where Remove Corruption handles both types from a single spell ID).
- **Shaman**: uses caster spell ID matched against `shaman_poison_spells` and `shaman_disease_spells`; returns null for unrecognised spells (e.g. WotLK Cleanse Spirit, which cannot be classified from the event data alone).

Per-boss results list each dispel type's total count and a sorted leaderboard of contributing players, broken down by boss, trash, and total scope. The raid summary aggregates totals and player counts across all bosses per scope.

Dispels are currently informational and do not feed into the Execution score.

**Why it matters:** Uncleared debuffs cause damage, crowd-control, or healing drain. In fights with a high dispel requirement, identifying who is not dispelling (or which type is going unhandled) tells the raid leader whether the problem is assignment or awareness.

**WCL data used:** Dispel events (Tier 3).

### Ignite Griefing

**Key:** `ignite_griefing` | **Expansions:** V

The counterpart to Ignite tracking. Because Ignite is a shared debuff, any fire mage who lands a spell hit while another mage's Ignite is active will reset the debuff timer with their own value. If their Ignite value is lower than the existing stack, the raid loses damage. This is known as "griefing" the Ignite.

The service detects Scorch and Fire Blast hits that landed while an Ignite window was active on the boss. These hits are the spells most likely to cause unintentional griefing when mages do not coordinate. Each instance records the spell, the damage value, the resist amount (Scorch partial resists reduce damage further), and the fight timestamp. Results are sorted by offender and summarized across the full clear.

The service is an audit tool, not a punishment system. Many resets are intentional (a mage refreshing a dying chain). The data lets the fire group identify whether resets are concentrated on a specific player or distributed evenly, and whether the timing suggests coordination issues.

**Why it matters:** A raid with five fire mages where two are consistently resetting each other's large Fireball Ignites can lose tens of thousands of damage per boss. The service makes this visible without requiring manual log scrubbing.

**WCL data used:** Debuff aura table for Ignite windows (Tier 2), DamageDone events filtered to Scorch and Fire Blast spell IDs (Tier 3).

---

## Preparation

Preparation measures what players brought to the raid before combat began. It is evaluated from the snapshot WarcraftLogs captures at the start of each fight: buffs, gear, and aura state. Poor preparation scores reflect players who arrived underequipped or underbuffed rather than any mistake made during the fight itself.

### World Buffs

**Key:** `world_buffs` | **Expansions:** All

Records which world buffs each player had active at the first boss pull. World buffs in WoW Classic are powerful, consumable-strength effects obtained in the open world before the raid. Examples include Rallying Cry of the Dragonslayer, Warchief's Blessing, Songflower Serenade, and Darkmoon Faire buffs. A fully world-buffed player can deal 20 to 40% more damage than the same player without them.

The service reads the buff aura list from the WarcraftLogs combatant info event, which captures the state at the start of each fight. For each player, it records which expected world buffs were present and which were missing, then calculates a per-player compliance percentage. A raid-wide average compliance rate is the primary metric.

Darkmoon Faire buffs are only evaluated when the Faire is actually open on the raid date. The service calculates DMF availability from the real-world calendar (the Faire opens the Monday after the first Friday of each month and runs for seven days), so reports from non-DMF weeks do not penalize players for not having a buff that was unavailable.

**Why it matters:** World buffs are a significant multiplier on raid performance in Vanilla. A guild that consistently raids with 70% world buff compliance is operating at a meaningful disadvantage relative to one at 95%.

**WCL data used:** Combatant info (Tier 2).

### Consumables

**Key:** `consumables` | **Expansions:** All

Checks flasks, elixirs, food buffs, and role-relevant consumables per player at pull time. The consumable list is configurable per expansion and supports class-specific requirements: a Warrior tank needs different consumables than a Shadow Priest.

Consumable categories are split into required and optional. Missing a required category (e.g. a strength elixir for a physical DPS) directly reduces the player's score. Optional categories (engineering items, lesser potions) contribute a smaller bonus weight if present. This prevents players from being penalized for skipping genuinely minor consumables while still rewarding thoroughness.

A raid summary tracks how many players had at least one missing required category and how many were missing the majority of their expected consumables across bosses. The per-category breakdown shows which consumable type was most commonly skipped across the roster.

**Why it matters:** Consumable compliance is one of the clearest indicators of raid culture and individual preparation discipline. The data is objective.

**WCL data used:** Combatant info (Tier 2).

### Gear Enchants

**Key:** `gear_enchants` | **Expansions:** All

Validates that all enchantable gear slots carry a permanent enchant. Each slot is configured per expansion with a severity level (missing a weapon enchant is treated more harshly than missing boots). The service also maintains a list of known bad enchants per slot so players with incorrect enchants (e.g. a lower rank enchant or a PvP enchant on a raid slot) are flagged rather than scored as if they had no enchant at all.

Two-handed weapon handling is built in. When the mainhand item ID is recognized as a two-handed weapon, the offhand slot check is skipped entirely. Offhand items that cannot receive weapon enchants (held-in-offhand books, off-hand frills) are also excluded via a per-slot filter so legitimate gear choices do not trigger false positives.

The per-player score is the fraction of enchantable slots that carry a valid enchant, expressed as a percentage. The raid summary reports the average enchant score and the most common missing or bad enchant slot across the roster.

**Why it matters:** Unenchanted gear is a constant and recurring gap in guild performance at all experience levels. The enchant service provides an automated audit that would otherwise require manually checking armory links before every raid.

**WCL data used:** Combatant info, specifically gear slot data with permanentEnchant IDs (Tier 2).

### Frost Resistance

**Key:** `frost_resistance` | **Expansions:** V

Checks that players have sufficient frost resistance gear equipped for encounters where resistance is required, most notably Sapphiron in Naxxramas and Ahune. The service reads gear from the combatant info snapshot, sums the frost resistance values of all equipped items (excluding shirt and ranged/tabard slots per reference implementation), and compares the total to a configurable target.

The service only runs on fights whose encounter ID is listed in the relevant encounter config. For encounters where resistance gear is not required, the service produces no output and does not affect scoring.

**Why it matters:** A player who enters a resistance check fight without the required gear will take substantially more damage and may die repeatedly, costing the raid time and healer mana. The service makes it trivial to identify who showed up undergeared before the attempt begins.

**WCL data used:** Combatant info gear data (Tier 2).

---

## Performance

Performance measures what happened during combat at the player level. Unlike Execution (which scores mechanic compliance) and Preparation (which scores readiness), Performance reflects how efficiently players used their toolkit over the course of the raid.

DPS and HPS numbers are informational and not currently scored. The scored Performance metrics are deaths and overhealing, both of which have clear correct targets.

### Death Analysis

**Key:** `deaths` | **Expansions:** All

Counts player deaths per boss fight and classifies each death as avoidable or unavoidable based on the killing blow ability ID. Avoidable deaths are from abilities with predictable timing or player-controlled avoidance (standing in fire, failing to move out of a cleave). Unavoidable deaths come from tank mechanics or raid-wide unavoidable damage.

The service tracks the full death roster per boss and accumulates totals per player across the clear, segmented by boss, trash, and total scope. Avoidable deaths on trash pulls are always zero by design — trash deaths are classified as unavoidable since avoidable trash mechanics are not tracked. Players with zero deaths receive a score of 100. Each avoidable death reduces the score. The zero-score threshold (where the score hits 0) is configurable per expansion.

Performance for the overall category is scored from avoidable deaths: 0 avoidable deaths = 100, at the threshold count the score reaches 0.

**Why it matters:** Avoidable deaths are direct evidence of players not executing their personal mechanics correctly. One avoidable death costs the raid a combat rez, one dps slot, and potentially the boss.

**WCL data used:** Death table (Tier 2).

### DPS

**Key:** `dps` | **Expansions:** All

Tracks damage per second for DPS players across three segments: trash pulls, boss fights, and full clear total. Healers and tanks are excluded from the ranking entirely. Role exclusions come from the WCL player roster, which assigns each actor a role based on their talent specialization.

Each segment computes DPS per player as total damage divided by total active fight time for that segment. The trash segment aggregates all non-boss combat, the boss segment aggregates boss pulls only, and the total segment combines both. Per-boss results are also available showing DPS for each specific encounter.

DPS is not currently scored (it does not feed into the Performance category score). It is provided as reference data alongside scored metrics.

**WCL data used:** DamageDone table (Tier 2), with a separate trash pull pass when `needsTrashData` is declared.

### Healing Metrics

**Key:** `healing` | **Expansions:** All

Tracks healing per second per healer with the same trash, boss, and total segmentation as DPS. Only players with the healer role assignment are included. The service also tracks overhealing percentage per healer: raw healing minus effective healing expressed as a fraction of raw.

The per-fight analysis records effective HPS, raw HPS, and overhealing percentage. The raid summary aggregates this across the full clear for each segment. Overhealing percentage across all healers is the primary input to Performance scoring: a lower percentage means healers were casting efficient, reactive heals rather than blanketing the raid with unnecessary output.

**Why it matters:** Overhealing in Classic raids is often a leading indicator of mana issues in longer fights. Healers who consistently overheal by 60% run out of mana faster and may be the reason a boss enrages.

**WCL data used:** Healing table (Tier 2), with a trash data pass.

### Damage Taken

**Key:** `damage_taken` | **Expansions:** All

Records total damage taken per player, broken down by trash, boss, and full clear. Also aggregates the top damage sources across the raid so guild leaders can see which abilities dealt the most total damage raid-wide.

Damage taken is currently informational. It is not scored and does not feed into the Performance category. Avoidable vs. unavoidable classification per damage source is planned for a future update.

**WCL data used:** DamageTaken table (Tier 2), with a trash data pass.

---

## Buffs

Buffs measures the raid's buff infrastructure: which persistent buffs were present, how reliably they were maintained across boss fights, and whether all players received their expected coverage.

### Raid Buff Uptime

**Key:** `buff_uptime` | **Expansions:** All

Measures the uptime of persistent raid buffs across all boss fights. Tracked buffs are split into two groups:

**Reliable buffs** are spells the raid should maintain at close to 100% uptime regardless of class composition, such as Blessing of Kings, Mark of the Wild, Fortitude, and Arcane Intellect. These are tracked and scored.

**Class buffs** are buffs provided by specific classes that may not always be present. Their uptime is tracked for informational purposes but not penalized when the relevant class is absent from the raid.

For each tracked buff, the service calculates uptime as a fraction of fight duration and the number of players affected, segmented by boss, trash, and total scope. Per-boss results aggregate to a raid-wide average uptime for each buff per scope. The Buffs category score is derived from the boss-scope average.

**Why it matters:** Letting Blessing of Kings fall off for 30 seconds per boss fight is a preventable mistake. Buff uptime data makes it visible and attributable.

**WCL data used:** Buff aura table (Tier 2).

### Player Buff Coverage

**Key:** `player_buff_coverage` | **Expansions:** All

Records which players received each tracked buff and which were missing it. Where `RaidBuffUptimeService` measures the aggregate uptime of a buff across the raid, this service identifies gaps at the individual recipient level.

For each boss fight, the service finds which players appear in the buff aura data for each tracked spell and which do not. Players missing a tracked buff are listed per buff. The summary counts total coverage across all tracked buffs and all players.

**Why it matters:** A Paladin may have maintained Blessing of Kings on 20 of 25 players but forgotten the five sitting at the back of the group. The uptime metric would look fine. This service surfaces the coverage gap.

**WCL data used:** Buff aura table with per-player recipient data (Tier 2).

---

## Report Card

**Key:** `report_card` | **Expansions:** All | **Runs last**

`ReportCardService` runs after all other services have persisted their results. It reads the raid-scoped results from each category, computes a 0 to 100 score per category, and derives an overall raid score from a weighted average. It does not make any WCL API calls.

Each category score is calculated from the metrics that service produced:

**Execution:** average of debuff uptime score, armor debuff uptime score, and interrupt coverage score. Each metric is normalized to 0 to 100 against a configurable target (e.g. 95% debuff uptime = 100 score).

**Preparation:** average of world buff compliance score, consumable compliance score, and gear enchant score.

**Performance:** average of avoidable deaths score (inverted: 0 deaths = 100) and healer overhealing score (inverted: 0% overheal = 100).

**Buffs:** derived from average raid buff uptime across all tracked buffs, normalized against a target of 95% uptime.

If a category has no data (e.g. no deaths occurred so the deaths metric is unavailable), the missing weight is redistributed proportionally across the remaining categories.

The final score per category maps to WoW's item quality grades:

| Score | Grade |
|-------|-------|
| 95 to 100 | Legendary |
| 85 to 94 | Epic |
| 70 to 84 | Rare |
| 50 to 69 | Uncommon |
| 25 to 49 | Common |
| 0 to 24 | Poor |

---

## Category Weights

Default weights used to combine category scores into the overall raid score. Weights are configurable per expansion in the expansion config file.

| Category | Default Weight | Rationale |
|----------|---------------|-----------|
| Execution | 35% | Mechanics are the highest-leverage category in Classic. Dropped debuffs and missed interrupts have direct damage consequences. |
| Preparation | 30% | In Vanilla Classic, world buffs and consumables are a meaningful fraction of raid DPS output. |
| Performance | 20% | DPS output is a result of Execution and Preparation, not a separate input. Deaths are scored here but are already reflected somewhat in boss progress. |
| Buffs | 15% | Buff uptime is usually high in organized guilds and has a lower variance than the other categories. |
