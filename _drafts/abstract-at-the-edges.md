---
layout: post
title: "Abstract at the Edges, Concrete in the Middle"
tags: [architecture, performance, opinion]
---

> **PRE-DRAFT SEED — not ready to publish.** Captured after the Roll Your Own Fakes post landed on *"there's really only one design pattern: interfaces."* This post is the honest follow-up — *yes, but not everywhere, and here's where the rule stops applying.*

---

## The seed

The bad-advice series told you to program against interfaces. Every service behind an abstraction. Every dependency through a constructor. Every piece of shared state reachable only through a container. That's all true — **for architecture.** It's the foundation and structure of a program. It's how the big blocks fit together.

But there's another kind of code that doesn't play by those rules: **low-level implementation.** The internals of a renderer. A collision detection loop. A physics integrator. A hot path that runs 60 times a second. Inside those, every extra virtual call, every layer of indirection, every interface vtable lookup — it all costs frame budget. You absolutely can (and should) write concrete code at the leaves, with direct dispatch, no interfaces, no polymorphism, nothing clever.

The rule is:

> **Abstract at the edges. Concrete in the middle.**

Architecture boundaries — where one subsystem talks to another, where user input meets your app, where your app talks to the network or the database or the filesystem — those should be abstractions. Interfaces. Protocols. Things you can swap.

But the *inside* of a subsystem — especially a performance-critical one — is allowed to be concrete. The inside of your renderer doesn't need a `ShapeInterface` with a virtual `draw()` method. It needs a struct with coordinates and a function that calls your GPU API directly. The inside of your physics loop doesn't need an `IntegratorStrategy`. It needs inline code in a tight loop.

This isn't a contradiction of the bad-advice series. It's a boundary condition on it. **The rule is true where the rule applies.** The rule applies at architecture boundaries. It stops applying when you're inside a hot loop where the cost of an interface is measurable and the benefit (swappability for tests or flexibility) isn't actually needed.

## Things to expand on

### The distinction: architecture vs implementation

- **Architecture decisions:** where subsystems meet, how they talk, what they depend on. Services, repositories, data stores, network clients, platform APIs. All interfaces, all injected, all swappable.
- **Implementation decisions:** how a specific algorithm does its work, how memory is laid out, how a hot loop is structured. Usually concrete, usually direct, usually no abstraction overhead.
- These are two different layers of the same program, and the rules that apply to one don't automatically apply to the other.

### Concrete examples from LiquidMetal2D (my 2D game engine in Swift + Metal)

- **Scenes** use the [builder pattern](https://example.com) with a type-parametric approach — I use Swift's class-method-on-protocol feature to let `T.build()` construct the right scene type. That's an abstraction at the architecture level.
- **Renderer** uses direct calls to concrete types. Vertex buffers, draw commands, uniform updates — all concrete. Virtual dispatch on every draw call would kill the frame budget. A 60fps game with thousands of draw calls per frame can't afford vtable lookups.
- **Collision detection** is a hot loop. It's inlined concrete code. No `CollisionShape` interface with a virtual `intersects(other: CollisionShape)` method. It's concrete `Circle`, concrete `AABB`, and a switch on the pair of types.
- **Scheduler / update loop** is concrete. The dispatch happens at the scene layer (abstract), but inside each system's update function, it's tight, direct code.

### Where the line lives

The line between architecture and implementation isn't always obvious. Some heuristics:

- **If you need to swap it in tests, it's architecture.** Put it behind an interface.
- **If you need to swap it at runtime (strategy, feature flag, A/B test), it's architecture.** Interface.
- **If it runs in a hot loop at 60fps and you can measure the interface cost, it's implementation.** Concrete.
- **If you could replace the whole module with a different one, the module boundary is architecture. Its internals are implementation.**
- **If an intern could inline the interface and gain measurable performance without losing correctness, the interface was premature.** Concrete.

### Why this isn't a contradiction

The bad-advice series is about *architectural decisions.* Every post in it is about services, containers, wiring, testability — the foundation and structure. None of it is claiming that every line of code in a renderer or a physics engine needs to be polymorphic. That would be absurd. It'd be performance suicide, and anyone who's shipped a game engine knows it.

But the series doesn't say that explicitly, which is why this post needs to exist — to mark the boundary where the rule ends. Without this post, a reader could walk away thinking *"oh, so I should put every function in my renderer behind a protocol"* and then come back in six months asking why their game runs at 10fps. **The rule was never meant to apply to the hot path.**

### The game-dev angle

Game engines are the canonical example because the framerate constraint is visible and measurable. But it applies to any hot-loop code: audio DSP, image processing, real-time data pipelines, anything where microseconds per iteration × millions of iterations becomes a bottleneck. You can't afford polymorphism there, and you shouldn't pretend you can.

### The "when I violate my own rule" framing

This could be written as a self-own — *"I just spent three posts telling you to use interfaces, and here's where I don't."* It doesn't undermine the series; it strengthens it, because it shows I understand where the rule applies and where it doesn't. A rule with known boundaries is more trustworthy than a rule without them.

## Open questions

- **Is this a bad-advice post or a standalone Neon & Noise essay?** I'm not sure. The voice of the bad-advice series is rant-y. This post is more measured — it's *"here's the nuance"* not *"here's the harsh truth."* It might fit better as a standalone architecture essay. Or it could be in the series as the one entry that pulls back on the others. Decide later.
- **Should it link back to all three bad-advice posts?** Probably yes. It's the honest caveat to the whole arc.
- **How much real code should I show?** Probably a before/after: one example with an interface (the wrong place), one example concrete (the right place), and a discussion of the performance difference.

## Possible titles

- **Abstract at the Edges, Concrete in the Middle** (current working title — memorable shape claim)
- *The Architecture Line*
- *Interfaces Are for Architecture, Not Hot Loops*
- *When Abstractions Hurt*
- *A Confident Dose of Bad Advice: Sometimes You Should Skip the Interface* (if included in the series)
- *A Confident Dose of Bad Advice: Violating My Own Rules* (self-own framing)

## Notes

- This seed was captured during the editing pass on *Roll Your Own Fakes.* The user wrestled with whether to add a caveat to that post's *"there's only one design pattern: interfaces"* claim and decided to save the caveat for its own post instead.
- **Do not add caveats to the existing bad-advice posts** — that dilutes their voice. This post exists to hold the nuance without polluting the rant.
- The LiquidMetal2D examples are real — Matt's 2D game engine in Swift + Metal for iOS has genuine instances of both "abstract at the edges" and "concrete in the middle." Use actual code from the engine when expanding.
- Could also be cross-referenced from a future game-engine post.
