# Interrupts

**Service key:** `interrupts`
**Category:** Execution
**Expansions:** All
**Contributes to scoring:** Yes — weighted input to the Execution category score

---

## What it measures

Total successful spell interrupts per boss fight, broken down by player and by ability. The service produces a leaderboard showing who interrupted, what ability they used, and how many times. Across the full clear, players are ranked by total interrupt count.

---

## Scoring

The service uses two scoring modes depending on data availability:

**Demand-tracked encounters** (boss fights with configured interruptible spell IDs): the service fetches Cast events from WCL for each configured encounter and counts interruptible casts (interrupted + completed). The response rate is `interrupted / interruptible_casts * 100`. Meeting or exceeding the target response rate scores 100.

**Untracked encounters** (trash pulls and bosses without configured spell IDs): the service falls back to normalizing the total raid interrupt count against a configurable per-expansion target. Meeting or exceeding the target scores 100. Lower counts scale linearly toward 0.

Each expansion config includes a mapping of encounter IDs to interruptible spell IDs. The `demandInterrupts` counter is tracked separately from `totalInterrupts` to prevent count overflow when mixing boss and trash scopes in aggregate views.

---

## Why it matters

Some bosses have spells that must be interrupted to prevent wipes or significant healing strain. In those encounters, interrupt discipline is as important as damage output. Identifying players who never interrupt — or who do not use their interrupt ability at all — is the first step toward fixing assignment gaps. The leaderboard makes it easy to see whether interrupt responsibility is distributed across the raid or concentrated on one or two players.

---

## WCL data used

- Interrupt events (Tier 3) — records the source player, the ability used to interrupt, and the target spell that was interrupted.
- Cast events (Tier 3) — for demand-tracked encounters, fetches interruptible cast windows to calculate response rate.

---

## Implementation notes

Ability-level tracking records which specific interrupt ability each player used (Counterspell, Pummel, Shield Bash, Kick, etc.). This is useful for verifying that players are using their primary interrupt rather than an off-GCD ability, and for identifying when players in the same fight are overloading the same interrupt rotation while others have unused cooldowns.

Results are not scored per player — only the raid-wide aggregate feeds the category score. Per-player data is informational.

The frontend shows a response rate bar visualization on boss-scoped results where all interrupts are demand-tracked. Mixed scopes (boss + trash) fall back to displaying the total interrupt count.
