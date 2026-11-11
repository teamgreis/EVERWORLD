# EVERWORLD

**A living world. Your own story.**

EVERWORLD is a persistent, browser-based multiplayer fantasy life simulation. It combines cozy life simulation, social interaction, city and home building, RPG progression, exploration, dungeons and controlled PvP.

Players need only a modern browser and the game URL. No native client, launcher or player-side installation is required.

## Binding product decisions

| Area | Decision |
|---|---|
| Presentation | 2D isometric, cozy and colorful |
| World | One shared persistent world with multiple connected cities |
| Time model | Hybrid: selected production and timed actions continue offline |
| Accounts | Created and managed exclusively by the owner/admin |
| Client | Responsive browser application for desktop, tablet and mobile |
| Hosting | YunoHost / Raspberry Pi 5, 8 GB RAM |
| Updates | GitHub Actions builds and publishes a container image, then deploys it |
| Persistence | PostgreSQL; player and world data survive application updates |

## Documentation

- [Inventory, items, loot and economy](docs/INVENTORY_ECONOMY.md)
- [Performance and lean runtime standard](docs/PERFORMANCE_STANDARD.md)

- [Master plan and game design](docs/MASTER_PLAN.md)
- [Roles, specializations and professions](docs/ROLES_AND_PROFESSIONS.md)
- [Story, quests, NPCs and factions](docs/STORY_QUESTS_NPCS_FACTIONS.md)
- [Technical architecture](docs/TECHNICAL_ARCHITECTURE.md)
- [Administration and security](docs/ADMIN_AND_SECURITY.md)
- [Development roadmap](docs/ROADMAP.md)
- [Deployment and automatic updates](docs/DEPLOYMENT.md)
- [Development conventions](CONTRIBUTING.md)

## Development status

Planning and repository foundation. Phase 1 is the secure account and administration foundation. The first playable milestone is a small, walkable version of Eldoria with persistent characters and real-time player presence.

## Guiding principles

1. Browser-first: no installation for players.
2. Server-authoritative: the server validates game state, combat, inventory, currency and progression.
3. Persistent: player and world state is stored server-side in PostgreSQL.
4. Modular: cities, zones, items, professions, quests and features can be expanded independently.
5. Fair multiplayer: safe areas, explicit PvP rules and protection against griefing.
6. Lightweight operations: start small enough for a Raspberry Pi 5 and scale only when measured usage requires it.
7. Secure by default: no public registration; owner-controlled accounts and audited administrative actions.
