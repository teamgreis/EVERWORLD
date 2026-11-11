# EVERWORLD — World, Zones, Chunks and Instances

**Status:** Binding world-runtime design baseline  
**Scope:** World partitioning, client streaming, zone transitions and instanced content  
**Primary constraint:** Keep the world feeling connected without loading or simulating the entire world at once.

## 1. Core decision

EVERWORLD is one persistent logical world, not one giant map or one process-wide broadcast room.

The world is composed of:
1. **Regions** — lore and navigation areas such as Eldoria, Thornhaven and the Wildlands.
2. **Zones** — server-managed multiplayer spaces with clear simulation and visibility boundaries.
3. **Map chunks** — small client-loading units for terrain, decoration and static map data.
4. **Instances** — private or party-scoped copies of a zone, primarily for dungeons and story encounters.

A region can contain many zones. A zone can be rendered from multiple map chunks. A dungeon instance uses the same zone runtime but has its own isolated state and membership.

The player should experience doors, paths, gates, lifts, boats or short transition effects—not a technical loading screen every few steps.

## 2. World coordinate model

Use stable integer tile coordinates as the canonical world position:
- `region_id`
- `zone_id`
- `x` and `y` tile coordinates
- optional facing/direction

The client may interpolate positions in pixels, but the server stores and validates tile-space or fixed-point coordinates. Avoid floating-point coordinates as the sole source of truth.

Each zone defines:
- dimensions and collision boundaries
- walkable cells and movement rules
- exits and their destination spawn coordinates
- static object and interaction definitions
- permitted gameplay rules (safe, social, combat, PvP, etc.)
- asset and map-chunk manifest

A transition must never be inferred from a client-supplied destination alone. The server checks that the player is at a valid exit and that the destination is available.

## 3. Zone sizing and map chunks

Keep **zone boundaries** separate from **rendering chunk boundaries**.

Initial tuning defaults (not hard capacity promises):
- Map chunk: 32 × 32 tiles.
- A normal outdoor zone: start around 64 × 64 tiles, split into 2 × 2 chunks.
- Larger cities are assembled from several connected zones instead of one enormous zone.
- Interiors and small encounter spaces can be much smaller.

These values are configuration, not hard-coded assumptions. Profile actual map complexity, collision density, active entities and client devices before increasing them.

### Client chunk loading
- Load the current chunk plus the immediately adjacent chunks needed for a smooth camera view.
- Preload the next chunk in the direction of travel when the player approaches a chunk edge.
- Unload distant chunks and dispose of their textures/objects when no longer needed.
- Keep a small bounded LRU cache for recently visited static chunks.
- Never load every region's tiles or sprites when the player signs in.

Static map data and asset manifests are versioned and cacheable. Mutable objects, player positions and interaction state come from the server and are not treated as immutable assets.

## 4. Server-side interest management

The server maintains a spatial index for active entities in each zone. Start with a simple uniform grid; do not introduce a spatial database or a complex geometry engine for the MVP.

Suggested initial grid cell: 8 × 8 tiles. This is a tunable implementation detail.

When a player moves:
1. Determine the player's current spatial cell.
2. Find nearby cells within the configured visibility radius.
3. Compare the result with the player's current visible-entity set.
4. Send enter/leave events only for entities added to or removed from that set.
5. Send compact state deltas for entities that remain visible.

The server must not broadcast every movement to every player in a zone. A zone room is a transport grouping, not permission to disclose all zone state to all clients.

Visibility rules are server-owned and may account for distance, walls, stealth, party membership and special abilities. Chat audience and entity visibility are separate concepts.

## 5. Movement and network updates

Movement is server-authoritative:
- The client sends bounded movement intent (direction/input), not a trusted final position.
- The server validates speed, collision, restrictions and zone exits.
- The server emits accepted state updates to affected observers.
- The client interpolates between accepted snapshots for smooth rendering.
- Superseded input is coalesced; stale movement packets are discarded.
- Reconnect uses the last valid server position.

Do not send a packet every render frame. Network update cadence is independent of the client's 60 FPS render loop. Begin with a modest configurable update rate and tune it using latency, event-loop lag and bandwidth measurements.

Persist position on meaningful checkpoints (clean disconnect, zone transition and bounded periodic checkpoints), not every frame or every tiny movement step.

## 6. Zone lifecycle and handoff

A zone has a lifecycle:
- **Dormant:** no active players; static definitions remain in PostgreSQL/filesystem/cache.
- **Active:** at least one player or required world event is present; runtime state is in memory.
- **Draining:** no new joins during shutdown or controlled migration; existing players are moved or disconnected safely.
- **Stopped:** ephemeral runtime state is flushed; durable state remains persisted.

Do not create a timer or worker loop for every player. Use shared, bounded schedulers only for genuinely time-driven world events. Most offline progression is computed from server timestamps when queried or claimed.

### Transition sequence
1. Client approaches or activates a server-defined exit.
2. Server validates position, exit, destination access and any party/story requirements.
3. Server prepares the destination spawn and required zone snapshot.
4. Server acknowledges the transition with a short-lived transition ID.
5. Client loads destination assets while retaining a transition overlay.
6. Server atomically changes session membership from old zone to new zone and emits the initial destination snapshot.
7. Client confirms rendering readiness; the overlay is removed.
8. Old-zone visibility subscriptions are removed.

The transition ID makes retries idempotent. A failed load or dropped connection must not duplicate the character, duplicate rewards or leave two active memberships. On reconnect, the server resolves the player's single authoritative zone and position.

## 7. Instances: dungeons, interiors and story scenes

Instances are isolated runtime contexts, not separate microservices.

An instance record contains:
- instance ID and template ID
- owner scope (solo player, party, guild/event where appropriate)
- member list and access rules
- creation and expiry timestamps
- lifecycle status
- instance-specific quest/encounter state
- durable reward/completion references

### Instance rules
- Dungeon instances are party-scoped by default.
- Private homes/interiors use property permissions and may share a template while keeping owner-specific state.
- Story scenes can be player- or group-scoped when choices must not leak across the world.
- Global events use shared public zones rather than cloning the world unnecessarily.
- Rewards and completion records are written transactionally and are idempotent.
- An instance can be reclaimed after a server restart from durable state where required.

### Cleanup
- Empty temporary instances enter a grace period.
- Before deletion, persist all required progress and reward state.
- Expired disposable instances are cleaned by one shared scheduler/batch job.
- Never delete a player-owned home or durable quest progress as temporary instance data.

## 8. Population and capacity controls

Do not promise a fixed number of players per zone before profiling.

Each zone can define a configurable soft population threshold and a hard safety ceiling. When the soft threshold is reached:
- show the zone as busy where useful
- route new arrivals to another suitable public copy only if the design allows it
- offer a nearby alternative channel/instance for optional activities
- preserve friends/party joining where possible

The hard ceiling is a safety mechanism, not a scaling strategy. Do not silently split a shared city into copies if that would break social expectations.

The first performance validation target remains 20 concurrent clients in one active zone on the Raspberry Pi 5. It is a test target only; actual capacity depends on map complexity, entity count, message rate, database activity and client hardware.

## 9. Persistent and transient state

**Persist in PostgreSQL:**
- account and character identity
- last valid region/zone/position
- property and durable object state
- inventory, currency and transaction ledger
- quest and story flags
- instance progress that must survive restart
- durable world events and audit records

**Keep transient in memory where safe:**
- connected socket/session routing
- current visible-entity sets
- interpolation snapshots
- short-lived movement intents
- temporary proximity indexes
- ephemeral effects that can be reconstructed

The database is authoritative for valuable state. In-memory state is a runtime optimization, never the only copy of currency, items, ownership or quest completion.

## 10. Asset and data delivery

- Serve hashed, versioned static assets with long-lived cache headers.
- Use a small manifest per region/zone, not one global asset manifest containing every asset.
- Lazy-load Phaser and game assets after authentication and when entering the world.
- Prefer texture atlases and reuse shared sprites.
- Compress map data; avoid verbose per-tile JSON where a compact encoding is practical.
- Keep map collision data compact and server-readable.
- Separate static definitions from player-mutated state.
- Preload only likely next destinations; do not eagerly download every city.

## 11. Failure handling

- On socket loss, mark the session temporarily disconnected and retain a bounded reconnect window.
- On reconnect, authenticate again and restore one authoritative zone membership.
- On process restart, rebuild transient indexes from active sessions as clients reconnect.
- Persist valuable state before acknowledging irreversible actions.
- Zone handoff and reward claims must be idempotent.
- If a zone cannot be restored, return the player to a safe persisted spawn rather than inventing a position.

## 12. MVP implementation order

1. Define region, zone, exit and chunk schemas.
2. Build one small Eldoria zone with collision and two exits.
3. Implement server-authoritative movement and position persistence.
4. Implement zone membership and nearby-player interest management.
5. Add client chunk loading/unloading and a bounded cache.
6. Implement a reliable zone transition with reconnect recovery.
7. Add one small private dungeon instance.
8. Profile on the target Raspberry Pi and tune chunk size, visibility radius, update cadence and population thresholds.

Do not build dynamic sharding, cross-process zone migration, Redis pub/sub or a multi-region cluster for the MVP. Revisit only when measurements show the single-process design has reached a real limit.
