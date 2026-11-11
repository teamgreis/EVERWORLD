# EVERWORLD — Master Plan & Game Design

**Status:** Binding product baseline  
**Version:** 1.0  
**Product type:** Persistent browser-based multiplayer fantasy life simulation / RPG

## 1. Product vision

EVERWORLD is a shared, persistent fantasy world in which every player builds a life and a story. It blends cozy social and home-building gameplay with exploration, progression, quests, dungeons and tabletop-inspired fantasy combat.

Players arrive as ordinary residents, develop skills and professions, earn currency, build and decorate homes, make friends, trade, join guilds, explore dangerous regions and choose whether to participate in PvP.

The world remains persistent while players are offline. Selected timed systems continue according to trusted server timestamps and defined limits.

**Core promise:** A player opens a URL on a phone, tablet or computer, signs in using an account created by the owner, and enters the same shared world as every other player.

## 2. Binding product decisions

- Visual style: 2D isometric, cozy, colorful, readable and expressive.
- World structure: one shared world containing multiple connected cities and wilderness regions.
- Time model: hybrid persistence. Crops, crafting orders and explicitly designed production timers may progress while a player is offline.
- Account policy: public registration is disabled. The owner/admin creates every account.
- Platform: browser-only. No native app, game launcher or player-side installation.
- Multiplayer: shared world, visible nearby players, social systems, group content and controlled PvP.
- Persistence: player, inventory, housing, economy and world state are stored server-side.

## 3. Core gameplay loop

1. Sign in and enter the shared world.
2. Explore the current city and meet other players.
3. Gather resources, accept quests or work a profession.
4. Earn currency, experience and materials.
5. Improve the character, equipment, home and property.
6. Trade, socialize, form groups and join a guild.
7. Explore new regions, dungeons and events.
8. Participate in consensual or zone-restricted PvP.
9. Return later to collect completed offline production and continue progression.

Combat must not be the only efficient route to progression. Non-combat professions and social play must remain meaningful.

## 4. World and regions

The world is divided into connected cities, wilderness regions and dungeon instances. Each region is composed of server-managed zones. A client receives only the state required for its current zone and nearby interactions.

Initial working names, subject to later lore review:

| Region | Role | Planned content |
|---|---|---|
| Eldoria | Starting capital | Welcome area, town square, services, starter housing |
| Thornhaven | Rural town | Farming, gathering, food and trade |
| Ironfall | Mountain settlement | Mining, smelting, blacksmithing |
| Moonveil | Arcane city | Magic, alchemy, libraries and research |
| Wildlands | Wilderness | Gathering, creatures, exploration |
| Ashen Frontier | High-risk frontier | Dangerous encounters and opt-in PvP |

Names are placeholders until the world lore and visual identity are finalized.

### City services

- Town square and public meeting areas
- NPC vendors and service providers
- Player market and trading interfaces
- Housing and property access
- Crafting stations
- Quest boards
- Inns and social spaces
- Transport points connecting regions

The browser client displays an isometric local view and a separate world/region map. The server validates travel and zone transitions.

## 5. Characters and progression

Each account initially owns one primary character. Multiple-character support is not part of the first release.

Character profile:
- Unique account username and display name
- Character appearance and cosmetic outfit
- Level and experience
- Attributes and derived combat values
- Equipment and inventory
- Profession progression
- Quest state
- Housing and property references
- Social relationships and guild membership

### Initial attributes

| Attribute | Intended effect |
|---|---|
| Strength | Melee power and selected physical checks |
| Defense | Damage mitigation |
| Dexterity | Evasion, initiative and selected gathering actions |
| Intelligence | Magic, knowledge and selected crafting |
| Luck | Carefully balanced discovery and reward modifiers |
| Endurance | Stamina and selected activity limits |

Exact formulas, caps and progression curves are tuned through playtesting. Attributes are not sold for real money.

Character development should support flexible paths such as warrior, mage, ranger, artisan, farmer, merchant and hybrids. A permanent class lock at creation is not required.

Progression sources include quests, exploration, crafting, gathering, social events and combat. Repetitive exploits need diminishing returns or sensible limits. Cosmetic progression should not create unfair combat advantages.

## 6. Housing, city life and construction

Housing is a central feature, not a decorative afterthought.

Players can acquire an available plot or home, expand it, place furniture and decorations, maintain a garden, use storage and crafting stations, invite friends according to permissions, and display achievements or crafted items.

Property rules:
- Every property has an owner and server-side access permissions.
- Visitors cannot edit, remove or take objects without permission.
- Plot availability and ownership changes are transactional.
- Ordinary PvP cannot permanently destroy or steal a player's home.
- Housing state is saved independently of the browser session.

## 7. Economy, professions and crafting

Gold is the primary in-game currency. The economy must have transparent sources and sinks.

Sources may include quests, contracts, profession work, player-to-player sales, events and approved exploration rewards.

Sinks may include property and upgrades, crafting fees and materials, travel and services, cosmetics and decoration, and repairs if later introduced.

Initial profession candidates:
- Farming
- Gathering
- Mining
- Woodcutting
- Fishing
- Cooking
- Alchemy
- Blacksmithing
- Carpentry
- Trading

Professions are introduced incrementally; the first playable version does not need every profession.

Crafting recipes define required inputs, duration, output and skill requirements. Crafting orders are stored as timed server-side actions. A completed order is claimed once only; repeated requests must not duplicate output.

Player trades require confirmation from both parties. The server validates inventories and currency, and commits the exchange atomically in a database transaction. Trade history is retained for support and moderation.

## 8. Offline and hybrid time model

Offline progression is calculated from trusted server timestamps, never client clocks.

Each timed action stores action type, owner/player identifier, start timestamp, completion timestamp or duration, input reservation, expected output, status and claim timestamp.

Lifecycle:
1. Validate player, resources and requirements.
2. Reserve or consume required inputs atomically.
3. Create the action with start and completion timestamps.
4. On claim or scheduled processing, calculate elapsed time from server time.
5. Enforce caps and completion rules.
6. Apply output exactly once and mark the action complete in the same transaction.

Initial candidates are crop growth, crafting orders and approved building production. Timed expeditions may be added later.

Rules:
- No client-provided completion time is trusted.
- Inputs are reserved or consumed atomically at action start.
- Completion can be claimed only once.
- Production has explicit caps and balancing rules.
- Offline progression cannot allow unbounded resource generation.
- Clock changes, downtime and interrupted actions need defined behavior.

## 9. Quests, NPCs and narrative

Quests provide structured goals and world storytelling.

Quest types include main story, city and faction quests, daily/weekly tasks, profession contracts, exploration discoveries, group/dungeon quests and seasonal events.

NPCs should have recognizable roles, dialogue and quest hooks. A later relationship system may track friendship, reputation and recurring interactions. NPC behavior must remain deterministic and manageable for the initial server footprint.

## 10. Dungeons and cooperative play

Dungeons are instanced or separately managed zones with entry requirements, party support, encounter definitions, boss mechanics, rewards, completion records and optional difficulty tiers.

The server validates party membership, encounter state, damage, rewards and completion. Clients display the encounter and send player intentions.

## 11. Combat and PvP

The initial combat model is turn-based. This reduces synchronization complexity and supports strategic abilities, equipment and readable combat logs.

Combat participants have server-validated attributes, equipment, effects and cooldowns. The client cannot decide damage, hit results, rewards or battle outcomes.

PvP fairness:
- Safe city zones prohibit hostile player attacks.
- Duels require explicit acceptance.
- Dedicated danger/frontier zones may allow opt-in or clearly announced PvP.
- New and low-level players receive protection.
- PvP loss rules are limited and clearly communicated.
- A player cannot lose their home, account or permanent character progression through ordinary defeat.
- Ganking, spawn camping and repeated harassment need prevention and moderation tools.

Guild-versus-guild content is a later feature with scheduled windows, clear consent and defined objectives. It is not part of the first playable milestone.

## 12. Social systems

Planned features:
- Nearby player presence
- Public zone chat
- Direct messages
- Friends list
- Player profiles
- Parties
- Guilds
- Player trading
- Shared quests and events
- Optional relationship and reputation systems

Before broad public access, chat must include mute, block, report and moderator tools. Private messages must respect access controls and data-retention policy.

## 13. Visual and interaction design

Art direction:
- 2D isometric world with readable tile/grid logic
- Cozy fantasy palette, warm lighting and distinctive city identities
- Expressive characters with modular cosmetics
- Clear silhouettes and readable interaction markers
- Lightweight asset sizes suitable for mobile networks
- Consistent UI across desktop, tablet and phone

Browser UX:
- Responsive browser layout
- Keyboard and mouse support
- Touch controls for mobile/tablet
- No plugin, extension or local installation
- Loading states and reconnect behavior
- Accessible contrast and scalable UI
- UI panels for inventory, character, map, quests, chat and housing

Phaser 3 is the planned 2D rendering/gameplay layer. React is used for surrounding application UI such as account, inventory, admin and menus. React should not re-render the entire game canvas for every frame.

## 14. Admin and account policy

The owner creates all accounts. Public registration remains disabled unless the owner explicitly changes this policy in a future product decision.

Roles:
- OWNER: full system and game control
- ADMIN: account and game administration within delegated permissions
- MODERATOR: player reports, chat and moderation
- PLAYER: normal game access

Admin capabilities include account creation, deactivation, role assignment, password reset flow, character inspection, support actions, controlled grants, event operations and audit review.

High-impact actions must be audited. Admin grants and corrections must include a reason and actor identity.

## 15. Persistence and reliability

PostgreSQL is the authoritative persistent store for accounts, characters, inventories, economy, property, quests, guilds, transactions and audit events.

Required properties:
- Database transactions for trades and currency changes
- Unique constraints for ownership and one-time claims
- Idempotent action completion
- Database backups and tested restoration
- Schema migrations with release notes
- Graceful shutdown and reconnect behavior
- Logs that avoid secrets and sensitive credentials

## 16. Monetization and fairness

No pay-to-win design is planned. The initial game is not designed around real-money purchases. Any future monetization requires a separate explicit product decision and legal, age, privacy and payment review.

## 17. Explicit non-goals for the first release

- Native mobile or desktop applications
- Photorealistic 3D
- Seamless massive-MMO scale
- Real-time action combat
- Open-world PvP everywhere
- Public self-registration
- Player-run executable plugins/mods
- Real-money trading
- Full content breadth of a mature commercial MMO

## 18. MVP acceptance criteria

The first playable MVP is accepted when:
1. Owner can create and disable player accounts.
2. Players can sign in from a browser on desktop and mobile.
3. Players can enter the same Eldoria map.
4. Players can move and see nearby connected players.
5. Player position and character identity persist across logout/restart.
6. Zone chat works with basic moderation controls.
7. Server validates movement bounds and zone transitions.
8. Database backup and restore have been tested.
9. The application can be deployed and updated without losing player data.

This master plan is the binding product baseline. Detailed balancing, story content and visual assets may evolve without changing the core decisions above.
