# Dispels

**Service key:** `dispels`
**Category:** Execution
**Expansions:** All
**Contributes to scoring:** Informational — does not currently feed into the Execution score

---

## What it measures

Successful dispel events per boss fight, grouped into four categories: Magic, Poison, Disease, and Curse. Each category shows the total dispel count for that fight and a sorted per-player leaderboard. The raid summary aggregates totals and player rankings across all bosses.

Only players with a dispel-capable class contribute. The service filters to Paladin, Priest, Druid, Mage, and Shaman. Dispels attributed to pets or unknown sources are excluded.

---

## Classification

WarcraftLogs dispel events expose different fields depending on the class. Classification logic per class:

**Mage** — always Curse. Remove Curse is the only dispel available to Mages.

**Priest** — defaults to Magic (Dispel Magic). Classified as Disease when the caster's ability spell ID (`abilityGameID`) appears in the expansion-configured `priest_disease_spells` list (Abolish Disease, Cure Disease).

**Paladin** — Cleanse removes Magic, Poison, and Disease from the same cast. The removed debuff's spell ID is exposed on the event (`extraAbilityGameID`), not its dispel school. Classification checks the removed debuff ID against per-expansion `paladin_poison_debuffs` and `paladin_disease_debuffs` lists. Debuff IDs in neither list default to Magic.

**Druid** — Cure Poison and Remove Curse have distinct spell IDs in earlier expansions. When the expansion-configured `druid_poison` list is non-empty, the caster spell ID is matched against it; a match classifies the dispel as Poison. When the list is empty (later expansions where Remove Corruption handles all types from a single spell ID), classification from event data alone is not possible and defaults to Curse.

**Shaman** — Cure Poison and Cure Disease have distinct spell IDs per expansion. The caster spell ID is matched against `shaman_poison_spells` and `shaman_disease_spells` lists. Unrecognised Shaman spell IDs (e.g. WotLK Cleanse Spirit, which dispatches all three types from a single spell) return null and the event is excluded from counts.

---

## Why it matters

Uncleared debuffs impose damage, crowd-control, or healing drain on the raid. In fights with a sustained or bursty dispel requirement, identifying which type is going unhandled — and which players are not contributing — tells the raid leader whether the problem is assignment, awareness, or class composition. The type breakdown makes this specific: a raid with 30 total magic dispels and 2 poison dispels when poison debuffs were the fight's primary threat is a very different problem from one where all types are covered.

---

## WCL data used

Dispel events (Tier 3). Each event includes the source player's ID, the ability used to dispel, and the removed debuff's ability ID.

---

## Data quality limitations

For Paladin Cleanse and some Shaman spells, classification relies on per-expansion spell ID lists that must be maintained as new content is added. A Cleanse event targeting a debuff ID not in either config list defaults to Magic, which may inflate Magic counts and under-report Poison or Disease in some encounters.

For Druids and Shamans in expansions where a single spell handles multiple dispel types, the actual type removed cannot be determined from the WCL event alone. Those events are conservatively excluded rather than misclassified.
