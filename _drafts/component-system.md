---
layout: post
title: "The AnyHashable Trap"
tags: [liquidmetal, swift, metal, gamedev, performance, architecture]
---

I replaced inheritance with composition in my game engine. The first version worked. The second version — a "nicer" API — tanked performance from 59fps to 15fps. The third version fixed it. This is what happened.

## The Inheritance Problem

[LiquidMetal2D](https://github.com/mattCasanova/LiquidMetal2D) is a 2D game engine in Swift + Metal. Until recently, extending a game object meant subclassing:

- Want a collider? Subclass `CollisionObj`.
- Want a behavior? Subclass `BehaviorObj`.
- Want both? You need multiple inheritance — which Swift doesn't have.

So the demo code was full of workarounds: delegation patterns, runtime type casting in hot loops (`for case let obj as BehaviorObj in objects`), awkward class hierarchies. The classic inheritance trap.

Game developers solved this 20+ years ago. Scott Bilas presented "A Data-Driven Game Object System" at GDC 2002, describing how Dungeon Siege used components instead of inheritance to manage 7,300+ unique object types. We called it the **Component Object Model** back then — years before Unity popularized "Entity Component System." The core insight hasn't changed: composition over inheritance.

## Version 0.7.0 — It Works

The plan: keep `GameObj` with its built-in properties (position, velocity, scale, rotation, textureID) since everything in a 2D engine draws. Add a component dictionary for everything else.

```swift
// Before (inheritance):
class CollisionObj: GameObj {
    var collider: Collider = NilCollider()
}

// After (composition):
let ship = GameObj()
ship.add(CircleCollider(obj: ship, radius: 1))
ship.add(FindAndGoBehavior(obj: ship, bounds: bounds))

// Access:
obj.get(Collider.self)?.doesCollideWith(...)
```

Components use `unowned let parent: GameObj` — not weak, not optional. If a component outlives its parent, it crashes intentionally. That's better than silent nil behavior where your game appears to work but things stop colliding with no explanation. The ownership model is clear: scene owns objects, objects own components.

The component dictionary uses `ObjectIdentifier` as keys — essentially pointer hashing. Functionally free.

Deleted `CollisionObj`, `BehaviorObj`, `NilCollider`, `NilBehavior`. Net result: -96 lines of code. Ran the collision stress test: **59fps at 4,000 objects.** Same as before. Zero overhead.

## Version 0.7.1 — The "Nicer" API

There was one ergonomic issue. Components were keyed by their concrete type:

```swift
ship.add(CircleCollider(...))    // stored under CircleCollider's id
ship.get(Collider.self)          // nil — wrong key!
ship.get(CircleCollider.self)    // works, but callers shouldn't need to know the concrete type
```

I wanted to add a `CircleCollider` and fetch it as `Collider`. The base protocol should be the lookup key.

My solution: switch from `ObjectIdentifier` to `AnyHashable` enum keys. Each component declares its key type, base protocols override it, subtypes inherit the override. Clean API.

Ran the stress test: **15fps.**

From 59fps to 15fps. By changing the dictionary key type.

## What AnyHashable Actually Does

`AnyHashable` is Swift's type-erased hashable wrapper. Every time you use one as a dictionary key, it:

1. **Boxes** the value into a heap-allocated wrapper
2. **Type-erases** it through an existential container
3. Does this on **every lookup**, not just insertion

In a cold path — scene registration, factory lookups, one-time setup — this overhead is invisible. In a hot path — per-pair collision checks at 60fps with thousands of pairs per frame — it's catastrophic.

| Approach | 4,000 objects FPS | Why |
|----------|------------------|-----|
| `ObjectIdentifier` (v0.7.0) | 59 fps | Pointer hash — essentially free |
| `AnyHashable` (v0.7.1) | 15 fps | Type erasure + boxing every lookup |

A 4x performance regression from changing one type.

## Version 0.7.2 — The Fix

The fix was simple: go back to `ObjectIdentifier`, but have base protocols override the `id` so all subtypes share the same key.

```swift
protocol Component: AnyObject {
    static var id: ObjectIdentifier { get }
}

// Default: each type gets its own slot
extension Component {
    static var id: ObjectIdentifier { ObjectIdentifier(Self.self) }
}

// Collider overrides: all collider types share one slot
protocol Collider: Component { ... }
extension Collider {
    static var id: ObjectIdentifier { ObjectIdentifier((any Collider).self) }
}
```

Now `CircleCollider.id` returns `ObjectIdentifier(Collider.self)`. Add a `CircleCollider`, fetch by `Collider.self` — same key. The ergonomics of v0.7.1 with the performance of v0.7.0.

**59fps.** Back to zero overhead.

## Design Decisions

**One collider per object** — all collider types share a slot. Makes sense because `Collider` has a useful base interface (`doesCollideWith`).

**Behaviors: shared by default, overridable** — one behavior per object covers simple games. If you need multiple behaviors (combat AI + movement AI + patrol), each can override `id` to get its own slot.

**Custom components get their own slot automatically** — game-specific components like `ZombieDemoComponent` use `ObjectIdentifier(Self.self)` by default. No conflicts with engine components.

## The Lesson

**Know the cost of your abstractions.**

`AnyHashable` is a fine tool. It's in the standard library. It works correctly. But it has a hidden per-call cost that's designed for convenience, not performance. Using it in a dictionary that gets hit thousands of times per frame is like using `NSLog` in a render loop — technically correct, practically catastrophic.

The three-version progression is a pattern I see a lot:

1. **Make it work** (ObjectIdentifier — correct, fast, slightly awkward API)
2. **Make it nice** (AnyHashable — elegant API, 4x slower)
3. **Make it right** (ObjectIdentifier with shared ids — correct, fast, AND nice)

The temptation is to skip step 1 and go straight to the "nice" version. Don't. Get it working first, measure, then improve the API without regressing performance. You can't optimize what you haven't profiled.
