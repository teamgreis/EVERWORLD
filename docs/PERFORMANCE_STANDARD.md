# EVERWORLD — Performance & Lean Runtime Standard

**Status:** Binding engineering standard  
**Priority:** Fast startup, low memory, low bandwidth and smooth play on modest devices and the Raspberry Pi 5 host.

## 1. Non-negotiable principles

1. Measure before adding infrastructure.
2. Keep the runtime small: one application service and PostgreSQL for the initial deployment.
3. No Redis, Kubernetes, microservices, service mesh or separate worker fleet in MVP.
4. Load only the current zone and nearby entities.
5. Use event-driven updates; never poll every player or run per-player timer loops.
6. Prefer simple data structures and bounded queues.
7. Defer nonessential assets and UI modules until needed.
8. Make performance budgets part of pull-request review.

## 2. Runtime topology

Initial production topology:
- One Node.js TypeScript process for API, WebSocket gateway and lightweight scheduled maintenance.
- One PostgreSQL instance.
- YunoHost reverse proxy for HTTPS and WebSocket forwarding.
- Static frontend assets served efficiently with compression and long-lived immutable caching.
- No Redis until measurements demonstrate a specific bottleneck that cannot be solved simply.

Avoid running separate development watchers, duplicate API processes or heavyweight observability stacks in production.

## 3. Browser payload and rendering budgets

Initial targets (to be measured on representative devices):
- Initial compressed JS: target <= 250 KB; hard review threshold 350 KB.
- Initial CSS: target <= 50 KB compressed.
- Initial critical route data: target <= 100 KB.
- Lazy-load Phaser/game scene after login or when entering the world.
- Lazy-load city maps, spritesheets, audio and optional panels by zone/feature.
- Use WebP/AVIF where supported, sprite atlases and shared textures.
- Avoid large runtime-generated shadows, filters and excessive particles.
- Cap device pixel ratio for the game canvas where needed.
- Target stable 60 FPS on capable desktop devices and 30 FPS on modest mobile devices; adapt effects and render resolution dynamically.

These are engineering targets, not promises until profiled.

## 4. Frontend architecture

- Keep React for menus, forms, inventory, dialogs and administrative UI.
- Use Phaser only for the isometric world view; do not duplicate UI frameworks inside the canvas.
- Keep world rendering isolated from business logic.
- Use a small scene lifecycle: boot, preload, world.
- Reuse sprite objects and textures; avoid per-frame allocations.
- Update animations and interpolation with the render loop, but send network state only when meaningful.
- Use spatial culling and camera bounds; do not render off-screen entities.
- Avoid large state updates for every movement frame.
- Respect reduced-motion preferences and low-power devices.

## 5. Network protocol

- Send movement intents at a bounded rate; coalesce superseded input.
- Broadcast only changed state to players in the same zone and nearby interest area.
- Use compact payloads and avoid repeated verbose static metadata.
- Send snapshots on zone entry/reconnect, then incremental deltas.
- Batch low-priority updates; prioritize movement and interaction feedback.
- Apply per-event size and rate limits.
- Reconnect with exponential backoff and restore a validated snapshot.
- Never broadcast every player's inventory or the full world map.

## 6. Database efficiency

- Use indexes for measured access patterns: character, zone, owner, status and timestamps.
- Paginate large inventories, messages, market results and audit history.
- Avoid N+1 queries; fetch required relations in bounded queries.
- Use transactions for inventory, currency, crafting and quest reward mutations.
- Keep frequently changing presence and movement ephemeral; persist position on zone transition and periodic checkpoints, not every frame.
- Use database constraints for uniqueness and referential integrity.
- Use bounded connection pools appropriate for Raspberry Pi memory.
- Cache immutable item definitions, recipes and map metadata in process; invalidate on content version change.
- Use explain/analyze and query timing before adding indexes or caching layers.

## 7. Background and offline work

Do not schedule one timer per player. Store start/end timestamps and compute due results on interaction, login or a small bounded batch processor. Use idempotent transactions and indexes on due status/time. Batch work with a strict maximum duration and row count.

## 8. Raspberry Pi 5 operating envelope

The Pi 5 is the initial target, not an excuse to overbuild:
- Keep PostgreSQL memory and connection counts bounded.
- Set Node heap and process restart policy deliberately after measuring.
- Use ARM64-compatible, pinned container images if containers are used.
- Keep logs rotated and structured; do not log every movement event.
- Monitor CPU, RAM, database size, active sockets, event-loop lag and request latency.
- Run backups outside peak gameplay and verify restoration periodically.

## 9. Performance testing gates

Every milestone should include:
- Frontend bundle-size report.
- Cold-load and repeat-load timings.
- FPS and memory profiling on desktop and mobile.
- API latency and database query timings.
- Concurrent connection and zone-load tests.
- Reconnect and packet-loss tests.
- Long-session memory-leak test.
- ARM64 deployment smoke test.

Initial MVP test target: 20 concurrent connected clients in a single active zone on the Pi 5, with measured CPU/RAM/latency recorded. This is a test target, not a guaranteed capacity claim.

## 10. When to add complexity

Add Redis only when cross-process ephemeral coordination or measured cache pressure requires it. Add a worker process only when background work measurably affects request latency. Add a CDN only when asset delivery or geography justifies it. Split services only when independent scaling or fault isolation has a demonstrated need.

**Rule:** The fastest architecture is usually the smallest architecture that meets the measured workload.


## 11. Graphics quality and adaptive rendering

The binding graphics profiles and visual acceptance criteria are defined in `docs/GRAPHICS_QUALITY.md`.

- Auto, Low, High and Ultra profiles are supported as client-side rendering settings.
- Auto adapts optional effects and render scale from sustained frame-time measurements.
- Phaser renders the world; React renders application UI.
- Graphics quality never changes authoritative gameplay rules or server simulation.
- Advanced effects are optional and must be budgeted, measured and gracefully disabled.
- The Raspberry Pi hosts the backend; player devices render the game.
