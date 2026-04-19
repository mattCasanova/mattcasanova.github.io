---
layout: post
title: "Flying Motorcycles and Movable Buildings"
tags: [liquidmetal, gamedev, architecture, swift, metal, performance]
---

I added a component system to my game engine last week.

It's not that it's hard. I just never got around to it when I started [LiquidMetal2D](https://github.com/mattcasanova/LiquidMetal2D) back in 2020.

It's also not new. It wasn't new when I wrote about it in [my book in 2017](https://www.amazon.com/Game-Development-Patterns-Best-Practices-ebook/dp/B01MRP7SPA/). It wasn't new when I was teaching it at DigiPen in 2012. Scott Bilas presented the Dungeon Siege version of it at GDC in 2002. Twenty-plus years. Unity and Unreal both ship variants of it. Every engineer who has built a game engine past a certain size has landed on the same answer to the same problem.

The problem has two versions. One is a teaching example. One actually happened to someone I knew.

## The Teaching Example

You're building a game engine. Or a game. Either way. You need something to represent the stuff in the world — the ships, the bullets, the doors. Every game engineering class on earth starts with the same tree:

```
GameObject
└── Vehicle
    ├── Car
    │   └── FlyingCar
    └── Motorcycle
        └── ??? FlyingMotorcycle
```

The tree starts fine. `GameObject` at the top. `Vehicle` adds wheels and a speed. `Car` and `Motorcycle` specialize. `FlyingCar` extends `Car` and adds the flying bits.

Then somebody on your team says: *"I want a flying motorcycle."*

And you stop.

Where do you put the flying logic? It's already in `FlyingCar`. Do you put it in `FlyingMotorcycle` too, and just duplicate the code? Do you hoist it up to `Vehicle` and make *every* vehicle optionally flyable? What about when somebody wants a flying boat next week? Are boats vehicles? Do boats also need the optional-flight branch now?

The right answer is that **the flying logic doesn't belong anywhere on that tree.** "Flying" isn't a kind of vehicle. It's a capability that can get bolted onto a vehicle. Or a character. Or a power-up. Or a random enemy you didn't know you were going to have.

Put it in a `FlyingComponent`. Attach it to whatever needs to fly. Done.

That's the component object model, in one paragraph.[^1]

## The Zynga Version

Here's the version that actually happens in the wild. A friend of mine worked at Zynga during the peak -Ville era. They shipped Farmville. Then Cityville. Then one of the animal-themed ones — Petville or Frontierville or whichever one it was. I'm going off memory and so is he, so forgive the details.

In Farmville, you click on a farm. The farm has resources. The farm waits for time to pass, and then you come back and click it again. `Farm` was a real class. Lots of logic. Years of it.

Then Cityville shipped. A city has buildings. You click on a building. The building has resources. The building waits for time to pass…

You can see where this is going.

`Building` inherited from `Farm`. It had to — the farm had all the logic, all the shipped-and-working code, and nobody was going to rewrite years of battle-tested stuff to change what it was called. So a city building was, under the hood, a farm.

Then they added something that moved. Cars, or carts, or planes — something. It still needed to be clickable. It still needed to hold resources. So they made a `MovableBuilding` that extended `Building`.

Then the animal game came along. Pets. Pets move. Pets hold resources. Pets are clickable.

The pet inherited from the movable building.

The pet inherited from the movable building.

Somewhere, at the bottom of a multi-hour stack trace, a happy little pet in a happy little pet game was, through a chain of inheritance three or four levels deep, structurally a *building.* Not because anybody wanted it to be. Because the code had accreted that way and nobody had the budget to reorganize it.

This is what deep hierarchies do. They don't start absurd. They rot into absurdity one reasonable decision at a time. Year one, `Building extends Farm` is the fastest way to ship. Year three, a pet inherits from a movable building and nobody remembers why.

The fix is the same as the teaching example. The capabilities every one of these things needs — clickable, holds resources, waits on a timer — aren't the kind of thing that should live on a hierarchy. They're things an object *has.* Components. Attach them to whatever needs them.

## The Version in LiquidMetal2D

[LiquidMetal2D](https://github.com/mattcasanova/LiquidMetal2D) had the same shape. Not as bad — it's mine, it's small, it's shallow — but the same shape.

Until last week, extending a `GameObj` meant subclassing:

- Want collision? Subclass `CollisionObj`.
- Want behavior? Subclass `BehaviorObj`.
- Want both? Congratulations, you need multiple inheritance. Swift doesn't have it. Nobody who should have it actually does. So the demo code was full of delegation hacks and runtime type casts:

```swift
for case let obj as BehaviorObj in objects {
    obj.behavior.update(dt)
}
```

That's a runtime cast in a hot loop, running every frame, for every object, to dig out a property the type system couldn't see directly. The [Scott Bilas GDC talk from 2002](https://web.archive.org/web/20040804192606/http://www.drizzle.com/~scottb/gdc/game-objects.ppt) is older than a lot of the people writing Swift today, and it described the fix.

So I wrote the fix. `GameObj` keeps its built-in properties — position, scale, rotation, texture. Everything a 2D object has just by existing. Everything else becomes a component:

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

`add()` puts the component in a dictionary on the `GameObj`. `get()` pulls it back out by type. Components own a reference back to their parent — `unowned`, not weak — because if a component outlives its parent, something has gone very wrong and I'd rather see the crash than debug silent nil behavior.

Deleted `CollisionObj`. Deleted `BehaviorObj`. Deleted `NilCollider` and `NilBehavior`. Net result: **-96 lines of code**, and the collision stress test held steady at 60fps on 4,000 objects. Same performance, less code, more flexibility.

## Extension Without Subclassing

This is the part I like.

Components are still interfaces. `Collider` is a protocol. `Behavior` is a protocol. `CircleCollider` conforms to `Collider`. When you ask a `GameObj` for its collider, you get back something that promises to *be* a `Collider` — a thing that knows how to check collision with another thing. The protocol is the contract. Same as anywhere else I've ever written about in this blog.

But you're not *subclassing* the `GameObj`. You're not modifying it. You're bolting a capability onto it from the side.

That's a cleaner version of what [the OCP post](/2026/04/17/house-rules-whatever-happens-happens/) was circling — open for extension, closed for modification. Subclassing is *one* way to extend. It's not the best way. It builds trees. Trees rot into buildings with pets in them.

Components extend without building trees. Any `GameObj` can have any component. No hierarchy decision about what level "flying" or "heals" or "can be clicked" lives at. Just: does this thing need to fly? Attach the component. Done.

And none of it is new. That's the part that's funny about all of this. I wrote a chapter about it in 2017. We were teaching it in 2012. It was already a solved problem when it got solved. The only reason to re-explain it every few years is that every new engineer building their first toy engine runs into the same tree and figures they can outsmart it.

I thought I could, too. I was wrong a couple of times.

## The AnyHashable Trap

The implementation took three versions. The middle one taught me something about the cost of "nicer" code.

**Version 0.7.0.** First pass. Component dictionary keyed by `ObjectIdentifier` — essentially pointer hashing. Add a `CircleCollider`, fetch it back by `CircleCollider.self`. Works. Fast. Slightly awkward API because callers have to know the concrete type to fetch with.

**Version 0.7.1.** The "nicer" version. I wanted callers to be able to fetch by the base protocol — add a `CircleCollider`, fetch by `Collider.self`. That meant a more flexible key. I swapped `ObjectIdentifier` for `AnyHashable`. Elegant. Clean. Generic.

Tanked the performance from 60fps to 15fps at 4,000 objects.

| Approach                                  | 4,000 objects FPS | Why                                    |
|-------------------------------------------|-------------------|----------------------------------------|
| `ObjectIdentifier` (v0.7.0)               | 60 fps            | Pointer hash — essentially free        |
| `AnyHashable` (v0.7.1)                    | 15 fps            | Type erasure + boxing every lookup     |
| `ObjectIdentifier` with shared ids (0.7.2) | 60 fps            | Same speed as 0.7.0, and the nice API  |

`AnyHashable` is fine. It's in the standard library. It works correctly. But every time you use one as a dictionary key, it boxes the value into a heap allocation and type-erases it through an existential container. In a cold path — scene registration, factory lookups — that overhead is invisible. In a hot path — per-pair collision checks at 60fps with thousands of pairs per frame — it's catastrophic.

**Version 0.7.2.** The fix. Go back to `ObjectIdentifier`, but let protocols override the `id` so all subtypes share one slot:

```swift
protocol Component: AnyObject {
    static var id: ObjectIdentifier { get }
}

extension Component {
    static var id: ObjectIdentifier { ObjectIdentifier(Self.self) }
}

protocol Collider: Component { ... }
extension Collider {
    static var id: ObjectIdentifier { ObjectIdentifier((any Collider).self) }
}
```

`CircleCollider.id` now returns `ObjectIdentifier(Collider.self)`. Add a `CircleCollider`, fetch by `Collider.self`, same key, same dictionary entry. The ergonomics of 0.7.1 with the performance of 0.7.0.

Back to 60fps.

The three-version progression is a pattern I've been seeing a lot lately:

1. **Make it work** — correct, fast, slightly ugly API.
2. **Make it nice** — elegant API, quietly slow.
3. **Make it right** — correct, fast, and nice.

The temptation is to skip step 1 and start at step 2. Don't. You don't find the hot paths until they're hot. You don't see the allocation cost until it's already the bottleneck. Build the functional version first, profile it, then improve the API without regressing. You cannot optimize what you haven't measured.[^2]

## What's Next

While writing [the OCP post](/2026/04/17/house-rules-whatever-happens-happens/) this week, I noticed something. I'd built `GameObj.toUniform()` so any game object could pack its own GPU data — the architecture was set up for multiple shaders. But the renderer itself only ever ran one render pass. The extension point existed. The rest of the pipeline wasn't using it.

Components fix that too.

With a component system, a `GameObj` can have a `GraphicsComponent` — or *more than one.* An `AlphaBlendGraphicsComponent` that knows how to pack data for the textured shader. A `WireframeGraphicsComponent` that knows how to pack data for a collision-bounds debug shader. The renderer can walk the objects twice — once for each component type — and do two render passes on the same object. Solid shaded ship on the first pass. Collision wireframe on top of it on the second.

That's the next piece. I don't have it yet.[^3] But the foundation is there, which is the whole point of the component system — you can plug a new thing in without rebuilding the thing next to it.

Flying motorcycle. Movable building. Same problem. Same answer.

Attach a component.

[^1]: Any unit-based strategy game has a version of this problem. In the original Starcraft, the Medic healed. Starcraft 2 took healing *off* the Medic, bolted it onto the Dropship, and called the combined unit the Medivac. "Can heal" isn't a kind of unit — it's a capability. Bolt it on whichever unit is supposed to heal this patch.
[^2]: This is the second time in three weeks I've been bitten by the "nicer API" trap. There is probably a post to be written about how much of my career has been apologizing in code for decisions I made because they read well.
[^3]: It's 11:49pm and I'm writing this from Vegas, where [the Nightcap](/2026/04/17/nightcap-fifteen-posts-deep/) correctly predicted I'd be blogging. Apparently this is who I am now.
