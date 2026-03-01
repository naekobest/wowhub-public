# Ignite Griefing

**Service key:** `ignite_griefing`
**Category:** Execution
**Expansions:** Vanilla Classic, Season of Discovery
**Contributes to scoring:** Informational — does not currently feed into the Execution score

---

## What it measures

Instances where a fire mage cast a wrong spell during an active Ignite window, reducing total Ignite damage. Because Ignite is a single shared debuff that stacks up to 5 times, casting the wrong spell at the wrong time wastes a stack slot (during the build phase) or can only reduce Ignite value (during maintenance).

The service produces two output views:

### Raid-wide overview (Ignite Griefing tab)

A summary showing total grief count per player across all bosses, with per-boss breakdowns. Each grief entry records the spell used, damage dealt, amount resisted, fight offset in seconds, and phase type (build or maintenance).

### Per-instance detail (embedded in Ignite tab)

Since March 2026, grief data is indexed by Ignite debuff band number via the `perIgnite` field. Each Ignite instance card in the Ignite view displays a warning badge with the grief count and an expandable section showing every grief event in context alongside the contributing spells.

---

## Algorithm

The service uses a stack-tracking state machine to model the Ignite state per boss fight.

**Phase tracking:** Events within an active Ignite debuff band are processed chronologically. The first fire crit in the band creates the Ignite (stack count 1) and is never flagged. Subsequent crits from any source increment the stack (capped at 5).

**Build phase (stacks 1 through 4):** Any Scorch, Fire Blast, Blast Wave, or Flamestrike hit during stack accumulation is flagged as a grief — the cast decision was wrong regardless of whether it crit. The correct play during the build phase is to cast only Fireball or Pyroblast.

**Maintenance phase (5 stacks):** Fireball and Scorch are correct maintenance spells. Fire Blast, Blast Wave, and Flamestrike are flagged unconditionally because they can only reduce Ignite damage at full stacks.

**Fire Blast save exemption:** Fire Blast in maintenance phase is exempted when the gap since the last fire crit exceeds 2,500 ms. This threshold is derived from the 4,000 ms Ignite duration minus Scorch's 1,500 ms cast time. If a Scorch could not have landed before Ignite expired, a Fire Blast to save the chain is the correct play and is not flagged.

---

## Why it matters

In a 5-mage fire group with an active rolling Ignite, the difference between a 5-stack Fireball Ignite and a 1-stack Scorch reset can be 10,000 to 40,000 damage per tick depending on the mage's spell power. A raid where two mages are consistently casting Scorch during the build phase can lose a meaningful fraction of its total fire damage on every boss.

The per-instance integration with the Ignite view provides full context: guild leaders and mage officers can see not just that a grief happened, but exactly which Ignite instance it affected, what phase it occurred in, and how the grief related to the contributing spells from other mages.

This is an audit tool, not a blame system. Many casts are situationally correct (saving a dying chain, casting under latency). The data shows whether griefs are concentrated on one player or distributed evenly, which is the first question a mage officer needs to answer.

---

## WCL data used

- Debuff aura table for Ignite windows (Tier 2) — time bands when Ignite was active on the target. Each band is processed independently.
- DamageDone events filtered to all tracked fire spell IDs (Tier 3) — individual hits with timestamps, source player, spell ID, damage, and hit type (crit vs. hit).

---

## Implementation notes

The tracked spell set is configurable per expansion: Pyroblast, Fireball, Scorch, Fire Blast, Blast Wave, and Flamestrike. Each spell ID is classified into a category. The spell map is built lazily on first use and cached for the lifetime of the service instance.

The `perIgnite` field indexes griefs by the zero-based debuff band number, matching the Ignite service's band order. Each entry contains `griefCount` and a `griefs` array with spell name, player name, damage, resisted amount, fight offset, and phase type. Internal fields used during processing (`bandIndex`, `playerName` on the player-level grief list) are stripped before output to keep the public API clean.

Scorch partial resists are recorded in the `resisted` field on each grief entry. Resisted damage reduces the effective Ignite value placed on the target, making Scorch resets potentially even more damaging than the raw damage figure suggests.
