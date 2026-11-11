# EVERWORLD — Inventory, Items, Loot, Trading & Economy

**Status:** Game-design specification  
**Principle:** Every item has a purpose, clear ownership and a predictable lifecycle.

## 1. Item families

- Materials: wood, stone, ore, fibers, herbs and magical components.
- Consumables: food, potions, medicine and temporary boosters.
- Equipment: weapons, armor, tools and accessories.
- Quest objects: evidence, keys, story artifacts and delivery goods.
- Placeables: furniture, decorations, crops and building components.
- Collectibles: cosmetics, trophies, pets and discoveries.
- Currency instruments: coins and explicitly designed vouchers.

Items are data-driven definitions. Individual durable items may have instances with durability, quality, owner binding, metadata and provenance. Stackable materials use compact quantity records rather than one database row per unit.

## 2. Inventory model

- Character inventory with configurable slot and weight rules.
- Equipment slots and quick-use slots.
- Personal storage and protected home storage.
- Shared guild storage with explicit permissions and audit trail.
- Market listings and escrow storage.
- Temporary dungeon/party loot distribution.
- Mail or delivery storage with expiry and claim rules.

The server is authoritative. Clients submit intents such as move, split, equip, consume or deposit; the server validates ownership, capacity, restrictions and transaction state.

## 3. Item lifecycle and integrity

All transfers use database transactions. A transfer must debit one location and credit another atomically. Unique constraints and idempotency keys prevent duplicate item creation or claims.

Important item states include available, equipped, reserved, listed, escrowed, consumed, destroyed and bound. An item cannot be simultaneously available in two locations.

High-value actions create an immutable transaction record with actor, source, destination, item, quantity, timestamp and reason. Administrative corrections use explicit compensating transactions; ordinary history is not silently rewritten.

## 4. Loot

Loot tables are versioned data, selected by encounter, region, event and difficulty. Randomness is generated server-side. Reward rolls are resolved once and persisted before clients are told the result.

Loot modes:
- Personal rewards for story-critical progression.
- Round-robin or contribution-aware party distribution.
- Public drops for low-value shared resources.
- Owner-configured event rewards with published rules.

Avoid pay-to-win reward structures. Rare items should have useful alternative acquisition routes over time.

## 5. Crafting and production

Recipes define inputs, outputs, required station, profession knowledge, minimum skill, duration and optional quality factors. Starting a job reserves inputs transactionally. Completion is calculated from server timestamps; claiming is idempotent.

Crafting quality can depend on profession mastery, tools, station and optional cooperative contributions. Do not require a rare role for basic progression.

## 6. Trading and markets

- Direct player trade uses a two-sided confirmation screen.
- Both players lock their offer before final confirmation.
- Any change invalidates both confirmations.
- Market listings use escrow; listed goods cannot be moved elsewhere.
- Listing fees, taxes and cancellation rules are visible before confirmation.
- NPC vendors provide price anchors, not unlimited currency creation.
- Market history and price summaries are aggregated rather than recalculated from every historical transaction on each page load.

## 7. Economy health

Track currency creation and sinks, item supply, market concentration, price movement, high-value transfers and suspicious duplication patterns.

Currency sources: quests, NPC work, approved events and bounded rewards.  
Currency sinks: repairs, crafting fees, property upkeep where appropriate, market taxes, travel and cosmetic services.

Avoid punitive upkeep that makes casual players lose homes simply for being offline. Owner-issued grants and event rewards are audited and separately labeled.

## 8. Theft, ownership and recovery

Theft is a gameplay action only where zone and property rules explicitly allow it. Personal home storage is protected by default. Stealing from an online or offline player must respect the same server-side permissions and safe-zone policy.

Evidence, witnesses, alarms, guards, reports, stolen-item flags and recovery procedures support consequences. Quest-critical and account-bound items cannot be stolen. Moderation tools can freeze a disputed item while an investigation is active.

## 9. First-release scope

MVP: stackable materials, consumables, equipment, character inventory, protected home storage, simple recipes, NPC vendor transactions, direct trade and server-authoritative rewards.

Later: player market, auctions, guild storage, item durability, advanced quality, mail, complex taxation, player-run shops and sophisticated economic analytics.

## 10. Acceptance criteria

- No item duplication through concurrent requests, retries or reconnects.
- Failed transactions leave both inventories unchanged.
- Every high-value transfer is auditable.
- Client manipulation cannot create items, currency or rewards.
- Inventory views load only the required page/category.
- Common item definitions are cached; mutable ownership and quantity remain authoritative in the database.
