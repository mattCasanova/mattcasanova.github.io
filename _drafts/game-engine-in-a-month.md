---
layout: post
title: "25 Tasks in 30 Days"
tags: [liquidmetal, swift, claude-code, ai, gamedev]
---

I hadn't touched my game engine in five years. In one month, I shipped more improvements than I had in the previous two years combined. Here's what changed.

## The Engine

[LiquidMetal2D](https://github.com/mattCasanova/LiquidMetal2D) is a 2D game engine I've been building in Swift + Metal for iOS. It does instanced rendering, has a scene system, a scheduler, collision detection, the works. I started it years ago, partly as a teaching tool — I taught C/C++ and game development as a professor for seven years and [co-authored a book](https://www.amazon.com/Game-Development-Patterns-Best-Practices-ebook/dp/B01MRP7SPA/) on game development patterns.

The engine already worked. It used SIMD, had decent performance, rendered sprites via Metal. But it had accumulated tech debt the way side projects do: naive collision detection, an inheritance-heavy object model, boilerplate-heavy scene registration, inconsistent APIs. The kind of stuff you know needs fixing but never gets prioritized over features.

Then I started using [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## What Changed

Claude Code runs in your terminal. It reads your files, understands your architecture, and writes code that fits into your existing patterns. That last part is what matters — it's not generating code in a vacuum. It's reading my renderer, my scene system, my collision code, and making changes that are consistent with how the engine is already structured.

In about 30 days, working nights and weekends, I closed [25+ tasks](https://github.com/users/mattCasanova/projects/3) on the project board:

### Broadphase Collision Detection

The biggest feature. Replaced O(n²) brute force with a uniform grid spatial partitioning system using half-neighbor traversal. The first implementation was actually slower than brute force due to hidden allocation and lookup costs — finding and fixing those was [its own story](/drafts/spatial-grid-collision/). Final result: 7,000 objects with full collision at 30fps on a phone.

### Component System

Replaced the inheritance-based object model (`CollisionObj`, `BehaviorObj`) with a composition-based component system. Three iterations to get right — the middle version used `AnyHashable` dictionary keys and [tanked performance from 59fps to 15fps](/drafts/component-system/). The final version uses `ObjectIdentifier` with shared protocol ids. Net result: -96 lines of code, zero overhead, no more subclass explosion.

### Scene System Overhaul

Eliminated all the boilerplate in scene registration:

```swift
// Before — repeat for every scene
sceneFactory.addScene(type: SceneTypes.menu, builder: TSceneBuilder<MenuScene>())

// After
sceneFactory.addScenes([MenuScene.self, GameplayScene.self])
```

Deleted `SceneBuilder` and `TSceneBuilder` entirely. Each scene declares its own type via `static var sceneType`, and `DefaultScene.build()` uses `Self()` so subclasses inherit it automatically.

### Renderer API Cleanup

- `setDefaultPerspective()` no longer sets the camera — projection and camera are independent concerns now
- Renamed `getWorldBoundsFromCamera` to `getVisibleBounds` — because it returns the camera's visible area, not the whole world
- Replaced 12 identical 5-line `setPerspective(fov:aspect:nearZ:farZ:)` calls with one-line `setDefaultPerspective()`
- Added convenience methods for common cases (`setOrthographic(width:height:)`, no-arg `setCamera()`)

### Bug Fixes

A sweep of real bugs that had been lurking:

- **Semaphore blocking main thread** — could hang indefinitely
- **CADisplayLink never invalidated** — retain cycle, memory leak
- **Texture pixel format mismatch** — RGBA vs native BGRA
- **Scheduler timer drift** — resetting to 0 discarded overshoot
- **Scheduler mutation during iteration** — classic collection modification bug
- **NotificationCenter observer cleanup** — observers never removed
- **RenderPass fatalError on missing drawable** — crashed when app went to background
- **Delta time edge cases** — spikes after backgrounding
- **Unowned reference crashes on colliders** — lifetime contract violation

### Testing

Added test coverage for areas that had none: spatial grid, point colliders, world bounds calculations, projection/unprojection round-trips, scheduler edge cases.

### Demos

Rewrote the camera rotation demo to actually demonstrate what it's named for. Added a collision stress test — 4,000 ships bouncing around with live stats showing pairs checked, collisions found, and FPS. Toggle between spatial grid and brute force for a dramatic side-by-side comparison.

## What This Actually Looks Like

I want to be specific about how the collaboration works, because "I used AI" can mean a lot of things.

**I drive the architecture.** I decided to use a uniform grid over a quadtree. I decided components should use `unowned` references. I decided the renderer API was inconsistent. Claude doesn't make these calls — I've been thinking about game engine architecture for 20 years.

**Claude drives the implementation velocity.** Once I've decided *what* to do, Claude helps me do it faster. It reads the existing code, understands the patterns, and produces implementations that are consistent with the engine's style. When I say "replace the brute force collision with a spatial grid," it reads my `Collider` protocol, my `GameObj` class, and my existing collision code, and produces something that plugs in cleanly.

**The bugs are the best example.** That list of fixes above? I didn't find most of those by reading code. Claude found them while working on features. "Hey, while I was implementing the spatial grid, I noticed your CADisplayLink is never invalidated — that's a retain cycle." That kind of incidental discovery is incredibly valuable on a solo project where there's no one else to code review.

**It's not magic.** Claude wrote code that tanked performance from 59fps to 15fps (the `AnyHashable` thing). It doesn't always get the architecture right on the first try. I still have to review everything, benchmark, and make judgment calls. But the cycle time from "I want to do X" to "X is done and tested" went from weeks to hours.

## The Confidence Effect

Here's the part that surprised me: the velocity on LiquidMetal2D is what gave me the confidence to start [Flux](https://github.com/mattcasanova/flux) — an open-source GPU terminal emulator. A month ago, I would have said a terminal emulator is too ambitious for a side project. But after shipping 25+ improvements to a game engine in 30 days, "ambitious side project" just means "more nights."

That's the real story here. It's not that AI makes you faster at the thing you're already doing. It's that the increased velocity changes what you're willing to attempt.
