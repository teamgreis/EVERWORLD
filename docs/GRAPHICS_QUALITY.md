# EVERWORLD — Graphics Quality & Rendering Profiles

**Status:** Binding visual-engineering direction  
**Goal:** Premium-looking isometric 2D graphics with smooth, responsive gameplay across phones, tablets and gaming PCs.

## 1. Product decision

EVERWORLD remains a 2D isometric game. High visual quality does not require switching to a heavyweight 3D engine. The chosen direction is a carefully art-directed 2D world with layered environments, expressive character animation, polished UI, selective lighting, atmospheric effects and responsive feedback.

The game offers four graphics modes:
- **Auto (default):** chooses a profile from measured runtime performance and device capabilities, then adapts conservatively.
- **Low:** prioritizes responsiveness, battery life and compatibility on older phones and integrated graphics.
- **High:** the standard intended experience on modern phones, tablets and PCs.
- **Ultra:** maximum supported visual effects for capable desktop/laptop hardware.

Players can override Auto at any time. A manual selection remains active until the player changes it; an optional "return to Auto" action restores adaptive behavior.

## 2. Rendering architecture

- Phaser 3 renders the isometric world in a dedicated canvas.
- React renders menus, HUD panels, inventory, dialogs, maps and settings.
- Prefer the Phaser WebGL renderer when available; retain a compatible fallback path where supported.
- Keep gameplay rules and authoritative state outside the renderer.
- Use texture atlases, shared materials/pipelines and batched draw calls where practical.
- Cull off-screen objects and entities outside the active camera/visibility region.
- Load visual assets by region, zone and feature.
- Keep visual effects independent from network update cadence and server simulation.

The rendering engine must not cause React to re-render the entire world each frame.

## 3. Quality profiles

Profiles are starting configurations, not promises that every effect is supported on every browser/GPU. Capability checks and frame-time measurements decide which options are actually enabled.

| Feature | Low | High | Ultra |
|---|---|---|---|
| Internal render scale | 0.75–0.9 | 0.9–1.0 | Up to 1.0, device-pixel-ratio capped |
| Shadows | baked/simple | soft stylized | richer layered stylized |
| Lighting | baked palette | selective dynamic lights | more simultaneous lights and subtle glow |
| Particles | reduced count | normal budget | expanded bounded budget |
| Weather/ambience | minimal | full standard set | enhanced layered ambience |
| Parallax/background | reduced | standard | richer multi-layer depth |
| Post-processing | off | selective lightweight | optional supported effects |
| Animation detail | essential | full standard | extra cosmetic/ambient detail |
| Texture filtering | performance-oriented | balanced | best supported quality |

The Auto mode may select Low, High or Ultra. It must never reduce input responsiveness or change game rules to achieve a visual target.

## 4. Adaptive quality controller

Auto mode uses a short rolling window of frame-time and dropped-frame measurements, not a single startup benchmark.

Rules:
1. Start with a conservative profile based on browser/GPU capability hints.
2. Measure actual frame time after the scene has warmed up.
3. If frame time remains above the target budget for a sustained window, reduce optional effects or render scale one step at a time.
4. Increase quality only after a longer stable window to avoid oscillation.
5. Apply a cooldown between changes.
6. Never change gameplay simulation speed, movement speed, hit detection or server tick rules.
7. Do not repeatedly allocate or rebuild large resources during profile changes.

Use 16.7 ms as the 60 FPS frame-time reference and 33.3 ms as the 30 FPS reference. The practical target depends on the device and browser. A stable 30 FPS on a modest phone is preferable to unstable 60 FPS with stutter.

Hardware hints such as WebGL renderer information, memory class or core count may be unavailable or privacy-restricted. They are hints only. Real frame-time behavior takes precedence.

## 5. Visual identity and polish

The visual target is premium, readable, warm and distinctive—not photorealistic 3D.

### World
- Hand-authored isometric environment art with consistent tile scale and perspective.
- Distinct material language for stone, wood, water, foliage, metal and magical elements.
- Layered foreground, playable ground and background scenery.
- Carefully composed lighting and color palettes for each region and time of day.
- Subtle ambient animation: foliage, water, flags, lanterns, smoke and environmental details.
- Weather and seasonal variation introduced without excessive simulation cost.

### Characters
- Expressive silhouettes and readable outfits.
- Modular cosmetic layers that share animation rigs/sprite conventions.
- Smooth walk, idle, interaction, emote and combat animations.
- Directional animation where it materially improves readability.
- Clear hit, selection, interaction and status feedback.

### Feedback and interface
- Consistent typography, spacing, icons and transitions.
- Responsive feedback for movement, collecting, crafting, dialogue and combat.
- Short, purposeful animation; respect reduced-motion settings.
- UI must remain crisp at different viewport sizes and pixel densities.
- Avoid visual clutter and effects that obscure nearby players or important gameplay information.

## 6. Asset pipeline and budgets

- Prefer WebP/AVIF for suitable static imagery and optimized PNG/WebP atlases where transparency or pixel fidelity requires it.
- Keep source art separate from generated runtime atlases.
- Build atlases per region or feature; avoid one enormous global atlas.
- Use deterministic asset naming, dimensions and metadata.
- Compress audio and load it on demand.
- Preload only the current zone and likely adjacent chunks.
- Version asset manifests so old caches cannot mix incompatible assets.
- Define per-zone memory and download budgets and measure them on mobile hardware.
- Do not upscale low-resolution artwork and call it high quality; create assets at intended display density and use appropriate variants.

## 7. Accessibility and device behavior

- Support mouse, keyboard and touch.
- Respect browser reduced-motion preferences.
- Provide an option to reduce flashes, screen shake and intense effects.
- Keep text and interaction markers legible at mobile sizes.
- Provide an explicit battery/performance-friendly mode.
- Avoid assuming every device has a discrete GPU or a high refresh-rate display.

## 8. Performance and quality gates

Every graphics milestone must be profiled on:
- a modest mobile device
- a current mid-range phone/tablet
- an integrated-GPU laptop
- a capable gaming PC

Record:
- average and 1% low FPS where tooling allows
- frame-time distribution and long-frame spikes
- GPU/CPU time where browser tooling exposes it
- memory use and texture/atlas footprint
- initial download size and zone transition time
- battery/thermal behavior during longer sessions where test devices allow

Initial targets:
- 60 FPS on capable devices where the scene and browser can sustain it.
- 30 FPS stable on modest supported mobile devices.
- No sustained stutter caused by asset loading, garbage collection or profile switching.
- Visual quality must degrade gracefully rather than block play on weaker devices.

These are validation targets, not measured claims. Do not label a profile "Ultra" on a device merely because it reports a powerful GPU; verify sustained frame-time behavior.

## 9. Raspberry Pi server separation

The Raspberry Pi 5 hosts the authoritative backend, database and multiplayer services. It does not render the players' graphics. The player's own phone, tablet or PC renders the game.

Therefore:
- Client graphics quality must not increase server simulation work.
- Server-side entity updates and network messages must remain bounded regardless of client graphics mode.
- The Pi should serve compressed, cacheable assets and compact world data.
- Asset delivery can later use a CDN if measured bandwidth or geographic latency warrants it; this is not an MVP dependency.

## 10. Implementation order

1. Establish the Phaser renderer and asset-loading lifecycle.
2. Build one polished vertical slice of Eldoria before producing large quantities of art.
3. Add Low/High/Ultra configuration with Auto default.
4. Instrument frame time, memory and loading behavior.
5. Implement conservative adaptive quality changes and manual override.
6. Validate on real phones, tablets and desktop browsers.
7. Add advanced lighting, weather, particles and post-processing only when the baseline remains stable.

**Rule:** Ultra is a visual enhancement, never a requirement for fair or complete gameplay.
