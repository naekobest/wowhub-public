# Ignite

**Service key:** `ignite`
**Category:** Execution
**Expansions:** Vanilla Classic, Season of Discovery
**Contributes to scoring:** Informational — does not currently feed into the Execution score

---

## What it measures

Two related dimensions of Ignite mechanics for fire mages.

### Uptime

Ignite uptime as a percentage of each boss fight's duration. This measures whether the mage group kept a fire crit landing on the target continuously enough to maintain the Ignite debuff.

### Combo sequences

Consecutive Ignite ticks attributed to the same source player within the 2,150 ms Ignite refresh window constitute a combo. The service reconstructs these sequences from raw DamageDone events by tracking crit timestamps. For each player and each boss, the service records:

- Total number of combos
- Longest combo duration in seconds
- Maximum tick value in the longest combo
- Fight offset where the longest combo started

A per-player summary across the full clear reports the best combo seen overall.

Ticks below a minimum damage threshold are excluded to filter out noise from DoT clipping artifacts.

---

## Why it matters

In Vanilla Classic, Ignite is a shared debuff across all fire mages. All fire crits from all mages stack into the same Ignite debuff on the target. When a large Ignite has accumulated from several Fireball crits, maintaining that stack with precisely timed Scorch casts can sustain significantly higher damage than letting the debuff drop and restart from zero. This technique — Ignite rolling — is one of the highest-leverage mechanical skills available to a fire mage group.

Showing mages their personal best combo and the precise fight offset where their chains dropped provides concrete feedback that generalised logs cannot offer.

---

## WCL data used

- Debuff aura table for uptime (Tier 2) — bands where Ignite was active on the target.
- DamageDone events filtered to the Ignite spell ID (Tier 3) — individual tick events with timestamps, source player, and damage values.

---

## Implementation notes

The 2,150 ms window is derived from the Vanilla game engine: Ignite ticks every 2 seconds, and a new fire crit must land before the 2-second window plus a small buffer. A strict window without the buffer would incorrectly split combos due to server-side tick timing variance.

Combo reconstruction processes events chronologically per fight. The service does not cross fight boundaries — each boss pull is analysed independently.
