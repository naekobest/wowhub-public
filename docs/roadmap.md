# Roadmap

## Expansion Support

Classic progression servers move forward over time and WoW Hub is designed to keep up. Each expansion requires its own set of analysis services, spell IDs, thresholds, and enabled checks.

| Expansion | Priority | Status |
|-----------|----------|--------|
| WoW Classic (Vanilla) | 1 | Beta |
| Season of Discovery | 2 | Planned |
| The Burning Crusade Classic | 3 | Planned |
| Wrath of the Lich King Classic | 4 | Planned |
| Cataclysm Classic | 5 | Planned |
| Mists of Pandaria Classic | 6 | Planned |
| Retail | 7 | Planned |

## Feature Roadmap

### Near Term (Beta to v1.0)

- **Character Profiles**: per character history across all analyzed raids, including DPS/HPS trends, Mechanic Score, Preparation Score, and role aware benchmarks
- **Mobile responsive layout**: the current UI is desktop first
- **Discord bot**: post an analysis summary directly to your guild's Discord channel

### Medium Term

- **Expanded service coverage**: additional Execution checks (kiting, soak assignments) and additional Preparation checks (raid composition analysis)
- **Comparative reports**: a "your guild vs. last week" diff view
- **Public leaderboards**: opt in raid scoring by server and faction

### Long Term / Expansion Dependent

- **Retail support**: Mythic+ analysis, affixes, dungeon specific mechanics
- **API access**: for guild management tools and third party integrations

## Suggesting a Feature

Open an issue with the `feature-request` label. For new analysis services, include:

- Which expansion it applies to
- Which WarcraftLogs data type captures it (damage events, buff events, etc.)
- What good and bad execution looks like (threshold rationale)

See [CONTRIBUTING.md](../CONTRIBUTING.md) for details.
