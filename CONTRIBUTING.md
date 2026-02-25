# Contributing

WoW Hub's source code is closed source. This repository is a public-facing overview.

Community input is genuinely valuable, particularly for:

- **Expansion coverage requests**: which expansion should be prioritized next?
- **Analysis service suggestions**: what mechanics should be tracked?
- **Threshold feedback**: does the scoring feel right for your guild's level of progression?
- **Bug reports**: unexpected scores, missing data, wrong player attribution

## How to Contribute

### Reporting Issues

Open an issue and apply the most relevant label:

- `bug`: something is wrong with the analysis output
- `feature-request`: a new analysis check or feature
- `expansion-request`: support for a new expansion or game version
- `threshold-feedback`: the scoring thresholds feel off

### Suggesting a New Analysis Service

A good analysis service suggestion includes:

1. **Expansion**: which expansion does this apply to? (All? Vanilla only? TBC+?)
2. **Mechanic**: what player behavior are we tracking?
3. **WCL data**: what WarcraftLogs data captures this? (e.g. "Interrupt events for spell ID 2139", "Debuff applications for spell ID 7386")
4. **Threshold rationale**: what does good vs. poor execution look like? (e.g. "Interrupt rate below 60% of available windows = Poor; above 90% = Epic")

You don't need to know how to implement it. A well-described mechanic is enough to work from.

### Expansion Requests

If you want to advocate for a specific expansion, open an issue with the `expansion-request` label and describe:

- Which expansion or game version
- What the most impactful analysis checks would be for that meta
- Any expansion-specific mechanics that don't exist in other versions

Expansions with the most community interest get prioritized.
