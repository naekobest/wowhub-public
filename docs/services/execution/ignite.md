# Ignite

**Service key:** `ignite`
**Category:** Execution
**Expansions:** Vanilla Classic, Season of Discovery
**Contributes to scoring:** Informational — does not currently feed into the Execution score

---

## What it measures

Three dimensions of Ignite mechanics for fire mage groups.

### Uptime

Ignite uptime as a percentage of each boss fight's duration. This measures whether the mage group kept fire crits landing on the target continuously enough to maintain the shared Ignite debuff.

### Per-instance breakdown

Each continuous Ignite debuff period on the boss is one Ignite instance. For each instance the service records:

- Start time and end time within the fight
- Total duration in seconds
- Maximum tick damage observed
- All contributing spells that landed during the Ignite window, attributed to their caster with individual damage values
- Total contributing damage across all casters

### Integrated grief detection

Grief data from the `ignite_griefing` service is displayed inline per Ignite instance. Each instance card shows a warning badge with the grief count and an expandable detail section listing every grief event: the spell cast, the responsible player, the phase (build or maintenance), and the damage dealt. This provides full context without switching tabs.

---

## Why it matters

In Vanilla Classic, Ignite is a shared debuff across all fire mages. Every fire crit from every mage feeds into the same Ignite debuff on the target, stacking up to 5 times. A well-coordinated mage group keeps Ignite rolling continuously with high-value Fireball crits while using Scorch only for maintenance refreshes at 5 stacks.

The per-instance breakdown shows exactly what happened during each Ignite period: which mages contributed, how much damage each spell added, and whether grief events disrupted the stack. This level of detail is not available from raw WarcraftLogs data without manual event-by-event scrubbing.

---

## WCL data used

- Debuff aura table for uptime and Ignite windows (Tier 2) — time bands where the Ignite debuff was active on the target. Each band maps to one Ignite instance.
- DamageDone events filtered to fire spell IDs (Tier 3) — individual hits with timestamps, source player, spell ID, damage, and hit type. Used to attribute contributing spells to each Ignite instance.

---

## Implementation notes

The service uses the WCL debuff band data directly rather than reconstructing Ignite windows from damage events. Each `band` in the WCL aura response represents one continuous period where the Ignite debuff was active. Contributing spells are matched to bands by timestamp overlap.

Hit type detection uses the WCL `hitType` field (1 = crit) to distinguish crits from regular hits. This is important because only crits add stacks to Ignite — non-crit fire hits do not contribute to Ignite and are excluded from the contributing spells list.

The previous implementation (before March 2026) reconstructed "combo sequences" by tracking consecutive Ignite ticks within a 2,150 ms window per source mage. This approach was replaced because it modeled Ignite as a per-player mechanic when it is actually a shared debuff. The band-based approach is both simpler and more accurate.
