# Death Analysis

**Service key:** `deaths`
**Category:** Performance
**Expansions:** All
**Contributes to scoring:** Yes — avoidable deaths are the primary input to the Performance category score

---

## What it measures

Player deaths per boss fight, classified as avoidable or unavoidable based on the killing blow ability ID.

- **Avoidable:** Deaths from abilities with predictable timing or player-controlled avoidance — standing in fire, failing to move out of a cleave, taking a soakable ability without designated responsibility.
- **Unavoidable:** Deaths from tank mechanics, lethal raid-wide damage events, or abilities where player action cannot change the outcome.

Per-boss results record the full death roster and each death's classification. Per-player totals accumulate across the full clear.

---

## Scoring

Players with zero avoidable deaths over the full clear receive a score of 100. Each avoidable death reduces the score. The zero-score threshold is configurable per expansion (the count at which the score reaches 0). Unavoidable deaths do not affect the score.

The Performance category score is a weighted average of the avoidable deaths score and the healer overhealing score.

---

## Why it matters

Each avoidable death costs the raid a combat resurrection (a limited, long-cooldown resource), removes one DPS or healing slot for the duration of the fight, and potentially triggers a wipe if the dead player was a tank or critical healer. An avoidable death is direct evidence that a specific player failed a personal mechanic — the killing blow ability ID names the mechanic.

---

## WCL data used

Death table (Tier 2). Each death event records the player, the killing blow ability, and the timestamp within the fight.

---

## Implementation notes

The avoidable vs. unavoidable classification is maintained as a per-expansion list of ability IDs. Abilities not in the list default to unavoidable — the classification is conservative. Adding a new boss ability to the avoidable list requires a config update.

The score formula is linear: avoidable_deaths / threshold maps 0 deaths to 100 and threshold deaths to 0. Deaths above the threshold all score 0 — the score is not negative.
