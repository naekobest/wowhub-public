# Cooldown Usage

**Service key:** `cooldown_usage`
**Category:** Execution
**Expansions:** All
**Contributes to scoring:** Yes — throughput cooldowns feed into the Execution category score

---

## What it measures

Tracks class-specific major cooldowns per player per boss fight. Each cooldown is classified into one of three types:

| Type | Behavior | Examples |
|------|----------|----------|
| Throughput | Scored | Death Wish, Adrenaline Rush, Arcane Power, Rapid Fire |
| Utility | Tracked with cast targets | Power Infusion, Innervate, Lay on Hands |
| Defensive | Informational only | Shield Wall, Last Stand, Divine Shield |

For each boss fight, the service records how many times each player used each of their class cooldowns, compared against the expected number of uses based on fight duration and cooldown duration.

---

## Scoring

Throughput cooldowns are scored using count-based comparison: `min(actual_casts / expected_casts, 1.0) * 100`. Expected casts are derived from `floor(fight_duration / cooldown_duration) + 1` (accounting for the initial cast at fight start).

Utility and defensive cooldowns are tracked but not scored. Utility cooldowns record cast targets with self-cast detection, so raid leaders can see which player received each Power Infusion or Innervate.

Talent-required cooldowns are auto-detected via a first-pass scan of the log. A cooldown is only scored if the player used it at least once during the raid. This prevents false penalization of Warriors who did not talent into Death Wish or Rogues who chose Combat over Assassination.

---

## Why it matters

Major throughput cooldowns are a significant fraction of a player's damage output on boss encounters. A Warrior who never uses Death Wish on a 3-minute fight is leaving measurable damage on the table. The per-player, per-boss breakdown makes it easy to identify players who are not using their cooldowns on pull or who forget them on shorter encounters.

Utility cooldown tracking answers questions that are otherwise invisible in aggregate data: which healer received Innervate, whether Power Infusion went to the right target, and whether defensive cooldowns were used proactively or reactively.

---

## WCL data used

Cast events (Tier 3). The event table records the source player, the ability cast, and the target (for utility cooldowns).

---

## Implementation notes

The cooldown configuration is defined per expansion and per class. Each entry specifies the spell ID, cooldown duration, cooldown type, and whether the ability requires a specific talent. The first-pass scan checks whether any cast of a talent-gated cooldown exists in the log before including it in expected counts.

PI, Innervate, and Lay on Hands track cast targets using the `targetID` field from WCL cast events. Self-casts (where source equals target) are flagged in the output so raid leaders can distinguish external utility from self-use.
