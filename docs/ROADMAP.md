# EVERWORLD — Development Roadmap

Roadmap items are ordered by dependency. Features may be re-estimated after each working milestone. The goal is a small, tested playable slice before adding breadth.

## Phase 0 — Product and repository foundation

- [x] Confirm browser-only delivery
- [x] Confirm 2D isometric art direction
- [x] Confirm one shared world with multiple cities
- [x] Confirm hybrid offline progression
- [x] Confirm owner-created accounts only
- [x] Define architecture and game design baseline
- [ ] Add CI checks and repository templates

**Exit criteria:** decisions and repository conventions are documented.

## Phase 1 — Secure application foundation

- [ ] TypeScript project structure
- [ ] Docker Compose for app and PostgreSQL
- [ ] Database schema and migrations
- [ ] Owner bootstrap procedure
- [ ] Login, logout and session handling
- [ ] Role-based authorization
- [ ] Admin-created accounts
- [ ] Disable/reactivate accounts
- [ ] Password reset flow
- [ ] Audit log
- [ ] Admin dashboard
- [ ] Health/version endpoint
- [ ] Automated tests for account and role flows
- [ ] Backup and restore procedure

**Exit criteria:** owner can securely create and manage player accounts; no public registration exists; data survives container replacement.

## Phase 2 — First walkable world: Eldoria

- [ ] Browser client shell
- [ ] Phaser 3 canvas integration
- [ ] Region/zone/exit/chunk data model
- [ ] Isometric tilemap and collision layer
- [ ] Initial Eldoria map split into small server-managed zones
- [ ] Lazy chunk loading, adjacent-chunk preloading and bounded cache
- [ ] Character sprite and animation
- [ ] Keyboard movement
- [ ] Touch movement controls
- [ ] Camera follow and zoom
- [ ] Server-validated movement
- [ ] Persisted player position
- [ ] Reconnect and session recovery

**Exit criteria:** one authenticated player can enter Eldoria, move around, leave and return at the saved location.

## Phase 3 — Real multiplayer presence

- [ ] Socket.IO authentication
- [ ] Zone join/leave lifecycle and idempotent handoff
- [ ] Reconnect recovery to one authoritative zone
- [ ] Spatial-grid interest management
- [ ] Nearby player state
- [ ] Movement synchronization
- [ ] Disconnect timeout and cleanup
- [ ] Zone chat
- [ ] Mute/block/report foundations
- [ ] Basic load and abuse tests

**Exit criteria:** several browsers can occupy the same zone and see each other move and communicate.

## Phase 4 — Character and inventory

- [ ] Character profile
- [ ] Attributes and level
- [ ] Experience and progression service
- [ ] Item definitions and item instances
- [ ] Inventory UI
- [ ] Equipment slots
- [ ] Server-authoritative item transfers
- [ ] Basic NPC interactions

**Exit criteria:** player progression and inventory persist and cannot be changed by client-side tampering.

## Phase 5 — City life and economy

- [ ] Eldoria shops and services
- [ ] Gold wallet and ledger
- [ ] Player-to-player trade
- [ ] Starter housing plots
- [ ] House interior/decoration prototype
- [ ] Storage
- [ ] Gathering resources
- [ ] First profession and crafting recipe

**Exit criteria:** players can earn, spend and trade safely; housing ownership and decoration persist.

## Phase 6 — Hybrid offline systems

- [ ] Timed action model
- [ ] Crafting orders
- [ ] Crop growth
- [ ] Completion and claim idempotency
- [ ] Offline production limits
- [ ] Clock/downtime tests

**Exit criteria:** selected activities progress while a player is offline and cannot be claimed twice.

## Phase 7 — RPG adventure

- [ ] Quest framework
- [ ] NPC dialogue
- [ ] Basic monsters
- [ ] Turn-based combat
- [ ] Equipment and abilities
- [ ] First dungeon
- [ ] Instance lifecycle, access rules and cleanup
- [ ] Group/party system

**Exit criteria:** a small party can complete a dungeon and receive server-validated rewards.

## Phase 8 — Social world and expansion

- [ ] Friends
- [ ] Guilds
- [ ] Direct messages
- [ ] More professions
- [ ] Thornhaven
- [ ] Ironfall
- [ ] Moonveil
- [ ] Travel between cities
- [ ] World events

**Exit criteria:** the shared world supports multiple connected cities and meaningful social/economic play.

## Phase 9 — Controlled PvP and live operations

- [ ] Duel consent
- [ ] PvP arena
- [ ] Danger-zone rules
- [ ] New-player protection
- [ ] Abuse and griefing controls
- [ ] Moderation workflow
- [ ] Scheduled events
- [ ] Operational dashboards and alerting

**Exit criteria:** PvP is fair, bounded and enforceable; moderation and support processes are operational.

## Release gates

No milestone is complete based only on a UI mockup. It must include working implementation, automated tests for critical logic, security and permission checks, persistence/restart verification, documentation updates and deployment verification on the target environment where practical.

## Scope control

Do not build every city, profession, item or dungeon before the first multiplayer slice works. Prioritize a coherent Eldoria experience with a small set of polished interactions.
