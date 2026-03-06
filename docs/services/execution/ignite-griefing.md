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

Grief data is indexed by Ignite debuff band number via the `perIgnite` field. Each Ignite instance card in the Ignite view displays a warning badge with the grief count and an expandable section showing every grief event in context alongside the contributing spells.

---

## Algorithm

The service uses a stack-tracking state machine to model the Ignite state per boss fight.

### Phase classification

Ignite windows are classified into two phases before grief detection begins:

**Opener windows** occur before Fire Vulnerability (Fire Vulnerability, spell 22959) reaches 5 stacks on the target. The 5-stack threshold is detected by counting Scorch hits — each Scorch hit applies one FV stack. Grief detection is skipped entirely for opener windows. The cast sequence during FV ramp-up is governed by different rules and should not be flagged.

**Real windows** occur after FV reaches 5 stacks. Only real windows are subject to grief analysis.

### Build phase (stacks 1 through 4)

Any Scorch, Fire Blast, Blast Wave, or Flamestrike hit during stack accumulation is flagged as a grief — the cast decision was wrong regardless of whether it crit. The correct play during the build phase is to cast only Fireball or Pyroblast.

### Maintenance phase (5 stacks)

**Fireball and Pyroblast** are only flagged when they occur more than 3 seconds after the 5-stack was reached within that window. An early Fireball to continue the chain immediately after reaching 5 stacks is still valid play and is not flagged.

**Fire Blast** is never flagged at 5 stacks. Fire Blast is always an acceptable maintenance helper or panic save at full stacks — it cannot reduce Ignite value when stacks are already maxed.

**Blast Wave and Flamestrike** are flagged unconditionally because they can only reduce Ignite damage at full stacks.

**Fire Blast save exemption (build phase):** During the build phase, Fire Blast in maintenance phase is additionally exempted when the gap since the last fire crit exceeds 2,500 ms. If a Scorch could not have landed before Ignite expired, a Fire Blast to save the chain is correct play.

### CD tracking

Each contributing spell in the Ignite tab displays colored status dots indicating which cooldowns were active at the time of the crit: Combustion, spell power trinkets (Warmth of Forgiveness / Mind Quickening Gem), or Power Infusion. This gives mage officers full context for evaluating the value of each Ignite window and whether CDs were stacked correctly.

### Talent check

The service reads CombatantInfo data to detect whether a fire mage has the Incinerate talent. A missing-talent warning is surfaced in the result for fire mages who lack it, since Incinerate is required for optimal Scorch sequence length.

---

## Why it matters

In a 5-mage fire group with an active rolling Ignite, the difference between a 5-stack Fireball Ignite and a 1-stack Scorch reset can be 10,000 to 40,000 damage per tick depending on the mage's spell power. A raid where two mages are consistently casting Scorch during the build phase can lose a meaningful fraction of its total fire damage on every boss.

The per-instance integration with the Ignite view provides full context: guild leaders and mage officers can see not just that a grief happened, but exactly which Ignite instance it affected, what phase it occurred in, and how the grief related to the contributing spells from other mages.

This is an audit tool, not a blame system. Many casts are situationally correct (saving a dying chain, casting under latency). The data shows whether griefs are concentrated on one player or distributed evenly, which is the first question a mage officer needs to answer.

---

## WCL data used

- Debuff aura table for Ignite windows (Tier 2) — time bands when Ignite was active on the target. Each band is processed independently.
- DamageDone events filtered to all tracked fire spell IDs (Tier 3) — individual hits with timestamps, source player, spell ID, damage, and hit type (crit vs. hit).
- Buff aura data for CD detection (Tier 2) — active Combustion, trinket, and Power Infusion buffs per player.
- CombatantInfo for talent detection (Tier 2) — equipped talents per player at fight start.

---

## Implementation notes

The tracked spell set is configurable per expansion: Pyroblast, Fireball, Scorch, Fire Blast, Blast Wave, and Flamestrike. Each spell ID is classified into a category. The spell map is built lazily on first use and cached for the lifetime of the service instance.

The `perIgnite` field indexes griefs by the zero-based debuff band number, matching the Ignite service's band order. Each entry contains `griefCount` and a `griefs` array with spell name, player name, damage, resisted amount, fight offset, and phase type. Internal fields used during processing are stripped before output to keep the public API clean.

Scorch partial resists are recorded in the `resisted` field on each grief entry. Resisted damage reduces the effective Ignite value placed on the target, making Scorch resets potentially even more damaging than the raw damage figure suggests.

Fire Vulnerability stacks are detected by counting Scorch DamageDone hits rather than debuff stack events, because WCL does not produce `applydebuffstack` events for Fire Vulnerability (spell 22959) in all log formats.