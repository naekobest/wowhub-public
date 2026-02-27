# Ignite Griefing

**Service key:** `ignite_griefing`
**Category:** Execution
**Expansions:** Vanilla Classic, Season of Discovery
**Contributes to scoring:** Informational — does not currently feed into the Execution score

---

## What it measures

Instances where a fire mage cast a spell that reset the shared Ignite debuff with a lower value than what was already on the target. Because Ignite is a single shared debuff, any fire crit from any mage replaces the existing stack value with the new one. If the new value is lower, the raid loses damage. This is called "griefing" the Ignite — usually unintentional, but detectable from log data.

Results record per boss:
- Which players caused grief events
- How many grief events per player
- For each event: the spell used, damage dealt, amount resisted, fight offset in seconds, and phase type (build or maintenance)

A raid summary ranks players by total grief count across the full clear.

---

## Algorithm

The service uses a stack-tracking state machine to model the Ignite state per boss fight.

**Phase tracking:** Events within an active Ignite window are processed chronologically. The first fire crit in the window creates the Ignite (stack count 1) and is never flagged. Subsequent crits from any source increment the stack (capped at 5).

**Build phase (stacks 1 through 4):** Any Scorch, Fire Blast, Blast Wave, or Flamestrike hit during stack accumulation is flagged as a grief — the cast decision was wrong regardless of whether it crit. The correct play during the build phase is to cast Fireball only.

**Maintenance phase (5 stacks):** Fireball is correct. Fire Blast, Blast Wave, and Flamestrike are flagged unconditionally.

**Fire Blast save exemption:** Fire Blast in maintenance phase is exempted when the gap since the last fire crit exceeds 2,500 ms. This threshold is derived from the 4,000 ms Ignite duration minus Scorch's 1,500 ms cast time. If a Scorch could not have landed before Ignite expired, a Fire Blast to save the chain is the correct play and is not flagged.

---

## Why it matters

In a 5-mage fire group with an active rolling Ignite, the difference between a 5-stack Fireball Ignite and a 1-stack Scorch reset can be 10,000 to 40,000 damage per tick depending on the mage's spell power. A raid where two mages are consistently resetting each other's chains during the build phase can lose a meaningful fraction of its total fire damage on every boss. The service makes this visible and attributable without requiring manual frame-by-frame log scrubbing.

This is an audit tool, not a blame system. Many resets are situationally correct (saving a dying chain, casting under latency). The data shows whether resets are concentrated on one player or distributed evenly, which is the first question a mage officer needs to answer.

---

## WCL data used

- Debuff aura table for Ignite windows (Tier 2) — time bands when Ignite was active on the target.
- DamageDone events filtered to all tracked fire spell IDs (Tier 3) — individual hits with timestamps, source player, spell ID, damage, and hit type (crit vs. hit).

---

## Implementation notes

The tracked spell set is configurable per expansion: Pyroblast, Fireball, Scorch, Fire Blast, Blast Wave, and Flamestrike. Each spell ID is classified into a category. The spell map is built lazily on first use and cached for the lifetime of the service instance.

Scorch partial resists are recorded in the `resisted` field on each grief entry. Resisted damage reduces the effective Ignite value placed on the target, making Scorch resets potentially even more damaging than the raw damage figure suggests.
