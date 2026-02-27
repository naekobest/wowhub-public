# Player Buff Coverage

**Service key:** `player_buff_coverage`
**Category:** Buffs
**Expansions:** All
**Contributes to scoring:** Informational — does not currently feed into the Buffs score

---

## What it measures

Which players received each tracked buff and which were missing it, at the individual recipient level. Where `RaidBuffUptimeService` measures aggregate uptime of a buff across the raid, this service identifies coverage gaps per player.

For each boss fight and each tracked buff, the service identifies which players appear in the aura data for that spell and which do not. Players missing a tracked buff are listed per buff. The summary reports total coverage across all tracked buffs and all players.

---

## Why it matters

A Paladin may have maintained Blessing of Kings on 20 of 25 players but forgotten the five at the back of the group. The uptime metric on that buff would look fine — it was active for 100% of the fight on the players who had it. This service surfaces the individual gap: five players went the entire fight without a stat buff they should have had.

This is the complement to uptime tracking. Uptime answers "was the buff present?"; coverage answers "who specifically was missing it?"

---

## WCL data used

Buff aura table with per-player recipient data (Tier 2). The aura table records which player IDs received each tracked buff and the time bands of their coverage.

---

## Data quality limitations

WarcraftLogs' aggregated aura table does not reliably expose per-player uptime for buff recipients in all API versions. In some cases the recipient list is available but uptime duration per recipient is not. The service therefore records presence or absence per player rather than uptime percentage per player. This is a known limitation of the WCL table API for buff auras.

---

## Implementation notes

The tracked buff list is shared with `RaidBuffUptimeService`. Both services read from the same fetched aura table data, so no additional WCL API calls are required. Coverage data is produced as a secondary pass over the same aura structure.
