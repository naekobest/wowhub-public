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

WarcraftLogs does not expose the number of interruptible cast windows per fight in its aggregated data, so an opportunity rate (interrupts landed / interrupts possible) cannot be computed. Instead, the service normalises the total raid interrupt count against a configurable per-expansion target.

Meeting or exceeding the target scores 100. Lower counts scale linearly toward 0.

---

## Why it matters

Some bosses have spells that must be interrupted to prevent wipes or significant healing strain. In those encounters, interrupt discipline is as important as damage output. Identifying players who never interrupt — or who do not use their interrupt ability at all — is the first step toward fixing assignment gaps. The leaderboard makes it easy to see whether interrupt responsibility is distributed across the raid or concentrated on one or two players.

---

## WCL data used

Interrupt events (Tier 3). The event table records the source player, the ability used to interrupt, and the target spell that was interrupted.

---

## Implementation notes

Ability-level tracking records which specific interrupt ability each player used (Counterspell, Pummel, Shield Bash, Kick, etc.). This is useful for verifying that players are using their primary interrupt rather than an off-GCD ability, and for identifying when players in the same fight are overloading the same interrupt rotation while others have unused cooldowns.

Results are not scored per player — only the raid-wide aggregate feeds the category score. Per-player data is informational.
