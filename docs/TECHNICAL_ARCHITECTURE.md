# EVERWORLD — Technical Architecture

**Status:** Binding architecture baseline  
**Target host:** YunoHost on Raspberry Pi 5, 8 GB RAM  
**Delivery:** Browser-only web application

## 1. Architecture goals

- Run the complete game through a browser URL.
- Keep authoritative game logic on the server.
- Support real-time presence and communication.
- Persist valuable game state in PostgreSQL.
- Keep the initial footprint suitable for the Raspberry Pi 5.
- Deploy reproducibly from GitHub.
- Separate game rendering, application UI, API and domain logic.

## 2. High-level architecture

    Player Browser
      React application shell (account, menus, panels, admin)
      Phaser 3 canvas (2D isometric world)
              | HTTPS / WebSocket
              v
    YunoHost reverse proxy + TLS
              |
              v
    Node.js application
      Express REST API
      Socket.IO gateway
      Authentication and authorization
      World/zone services
      Character and inventory services
      Economy and transaction services
      Timed/offline action processor
      Moderation and audit services
              |
              v
    PostgreSQL

Redis is optional and deferred until measured load requires shared ephemeral state, caching or multi-process coordination.

## 3. Frontend

### React + TypeScript

Responsible for login and account flows, character creation, application shell, inventory and character panels, chat/social UI, map/navigation UI, admin dashboard, responsive layouts and accessibility.

### Phaser 3

Responsible for isometric map rendering, tile and object layers, character sprites and animation, camera and zoom, pointer/touch interaction, local presentation of movement and effects, and rendering nearby players and NPCs.

The Phaser scene is a presentation/input layer. It does not own authoritative inventory, currency, combat or persistence.

### Browser compatibility

Target current evergreen versions of Safari, Chrome, Firefox and Edge. Mobile Safari and Android Chrome are first-class targets. No browser plugin or local installation is required.

## 4. Backend

Node.js with TypeScript.

- Express provides versioned REST endpoints.
- Socket.IO provides real-time player presence, zone events and chat.
- Domain services contain game rules and database operations.
- Validation is applied at every API and socket boundary.
- Authorization is checked server-side for every protected action.

Keep domain logic independent from Express and Socket.IO handlers so it can be tested directly.

## 5. Database

PostgreSQL is the source of truth.

Planned core entities:
- users
- characters
- character_attributes
- sessions
- worlds
- cities
- zones
- zone_objects
- properties
- buildings
- inventories
- items
- item_instances
- equipment
- currencies
- transactions
- professions
- recipes
- crafting_orders
- crops
- quests
- quest_progress
- dungeons
- parties
- guilds
- friendships
- messages
- moderation_reports
- timed_actions
- audit_logs

Use migrations for schema evolution. Add indexes based on query patterns and measured performance. Avoid storing transient per-frame position updates as an unbounded event log; persist position at appropriate intervals and on clean zone/session transitions.

## 6. Multiplayer and zone model

The world is logically shared but partitioned into zones.

A connected client subscribes to the zone it currently occupies and receives only nearby player/NPC state. The server tracks connected sessions and validates zone membership.

Socket event categories:
- connection/authentication
- zone join/leave
- player movement intent
- player state updates
- chat messages
- party and guild events
- interaction intents

All incoming events require schema validation, authorization, rate limits and server-side checks.

Movement:
- Client sends intended movement/input, not an authoritative final position.
- Server validates speed, collision bounds, zone boundaries and movement rate.
- Server broadcasts accepted state to relevant nearby clients.
- Reconnect restores the last valid persisted position.

Do not broadcast the entire world to every connected player.

## 7. Persistence and offline actions

Timed actions are records with server-owned timestamps and status transitions.

Recommended lifecycle:
1. Validate player, resources and requirements.
2. Reserve/consume required inputs in a database transaction.
3. Create action with start and completion timestamps.
4. On claim or scheduled processing, calculate elapsed time from server time.
5. Enforce caps and completion rules.
6. Apply output exactly once and mark action complete in the same transaction.

A unique/idempotency mechanism is required to prevent duplicate claims. Offline actions are calculated on demand where practical; do not run a per-player timer loop.

## 8. Security boundaries

- Passwords are hashed with Argon2id.
- Sessions use secure, HttpOnly, SameSite cookies.
- HTTPS is terminated by YunoHost.
- CSRF protection is required for cookie-authenticated state-changing HTTP requests.
- Socket connections authenticate using the same trusted session mechanism.
- Server-side role checks protect admin endpoints.
- Rate limits apply to authentication, chat and sensitive actions.
- Client-supplied currency, damage, XP, item ownership and timestamps are never trusted.
- Secrets are supplied through deployment secrets/environment, never committed.
- Logs must not contain passwords, tokens or session cookies.

## 9. Deployment topology

Initial topology:
- One web/game application container.
- One PostgreSQL container with a persistent volume.
- YunoHost reverse proxy routes HTTPS traffic to a loopback-bound application port.
- No public database port.
- Automated backups stored outside the database container.
- CI builds an immutable versioned image and publishes it to GitHub Container Registry.

Redis, separate workers and multiple game-server processes are deferred until measured need.

## 10. Raspberry Pi 5 performance strategy

- Start with a small number of active zones.
- Load only visible map chunks and nearby entities.
- Compress and cache static assets.
- Optimize sprite atlases and textures.
- Batch database writes where safe.
- Avoid polling when WebSockets can notify clients.
- Use bounded message payloads and rate limits.
- Monitor CPU, RAM, database size, active sessions and WebSocket counts.
- Load-test before increasing advertised concurrent-player capacity.

No fixed concurrent-player guarantee is made before profiling the actual game and host.

## 11. Observability and operations

Required: structured application logs, health endpoint, container health checks, database backup schedule, restore test procedure, release/version endpoint, deployment logs, resource monitoring and error reporting without secrets.

## 12. Testing strategy

- Unit tests for game rules and calculations.
- Integration tests for database transactions and API authorization.
- Socket tests for zone membership, chat and movement validation.
- Browser tests for login, navigation and core gameplay.
- Load tests for target zone population.
- Backup/restore tests.
- Deployment rollback test.

## 13. Architectural decision records

### ADR-001: Browser-only client

Decision: all players use a browser URL. Reason: no installation friction and one deployable client.

### ADR-002: Isometric 2D renderer

Decision: Phaser 3 for the world canvas. Reason: suitable 2D rendering and input handling without building a renderer from scratch.

### ADR-003: PostgreSQL as source of truth

Decision: PostgreSQL for persistent game data. Reason: transactional economy, relational integrity and mature backup tooling.

### ADR-004: Server-authoritative game state

Decision: server validates all valuable state changes. Reason: prevent client tampering and maintain a fair shared world.

### ADR-005: Zone-based multiplayer

Decision: clients receive only their current zone and relevant nearby state. Reason: bounded bandwidth and memory usage.

### ADR-006: GitHub-driven deployment

Decision: GitHub Actions builds and publishes the app image; server deploys the published image. Reason: reproducible builds and automatic updates without manually copying source files.
