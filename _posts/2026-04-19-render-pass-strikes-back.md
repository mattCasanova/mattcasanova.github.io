---
layout: post
title: "Component System, Part Two: The Render Pass Strikes Back"
tags: [liquidmetal, gamedev, architecture, swift, metal]
---

[Last night I published Part One](/2026/04/18/flying-motorcycles-movable-buildings/) — the post about a component system I'd added to my engine a few weeks back. The code had been sitting, working fine. The post had been sitting, unpublished, while I was heads-down on Flux and the other side projects. Last night I finally shipped it. I thought shipping it was the end of that story.

Turns out it was the setup.

Here's the plot twist. The component system wasn't just a cleaner way to do colliders and behaviors. It was the solution to a problem I didn't know I had yet — the one quietly hiding in my renderer. And the fix ended up doing more than fix the renderer. Alongside the multi-shader refactor, I finally made the commitment I'd been stopping just short of: `GameObj` is `final` now. The engine itself won't let you subclass it.

## The Thing I Already Wrote About

While drafting [the OCP post](/2026/04/17/house-rules-whatever-happens-happens/) I went to check that my `toUniform()` extension point actually worked — and found it was wired up correctly, but the *rest* of the renderer was hardcoded to the alpha-blend shader anyway. The uniform buffer was sized to `AlphaBlendUniform.typeSize()`. `RenderPass` was subclassed into `AlphaBlendRenderPass`. The `BufferProvider` assumed one stride. OCP in spirit. Not in fact.

I wrote about it at the end of that post. Said something like *"that's the next piece."*

Last night I finally built it.

## Three Shapes of a Plan

I had a flight to Vegas yesterday. Quick flight, but plenty of time to build out the plan. Not the good kind of plan. The *"type out what you think should happen, argue with Claude for an hour, scrap, repeat"* kind of plan.

The plan went through three shapes before something clicked.

**v1 — Subclass `GameObj`.** Make a `TexturedGameObj` that holds `textureID` and `tintColor`. Base stays clean. Alpha-blend-specific state lives on the subclass.

This *works.* It's boring. It's also the thing I specifically told you not to do [last night](/2026/04/18/flying-motorcycles-movable-buildings/). A pet inheriting from a movable building, Swift edition.

**v2 — Use generics.** `func submit<T: GameObj>(objects: [T])` to sidestep Swift's array invariance. `[PlayerShip]` doesn't auto-convert to `[GameObj]`, so you generic over it and move on.

This works too. The shader doesn't actually care about `T`'s concrete type — it's just a workaround for a language quirk. Generics as variance-plumbing is a code smell. I could feel the [AnyHashable post](/2026/04/18/flying-motorcycles-movable-buildings/) about to happen again.

**v3 — Use the component system I had already built.**

Right.

## It Was There the Whole Time

The thing that got me laughing at Claude was this. The component system has been in the engine since 0.7.0. It's the mechanism for colliders (`CircleCollider`, `AABBCollider`), for behaviors (`FindAndGoBehavior`), for game-specific state (`ZombieDemoComponent`). It's the system I *just wrote a whole blog post about* last night.

It was already designed to do exactly this. I just hadn't *used* it for render state.

Once we made that pivot, the design pulls apart clean. Per-object per-shader data — texture ID, tint color, wireframe thickness, ripple time — isn't "GameObj state." It's *component* state. Attach the component, and the shader picks it up. No subclassing. No generics. `submit` takes plain `[GameObj]`. Each shader iterates the list, filters by component presence, and renders.

Here's what a game object looks like after the refactor:

```swift
let ship = GameObj()
ship.add(AlphaBlendComponent(parent: ship, textureID: shipTex,
                              tintColor: Vec4(1, 0, 0, 1)))
ship.add(WireframeComponent(parent: ship, color: .red, thickness: 0.04))
ship.add(CircleCollider(parent: ship, radius: 2))
```

Same ship. Three components. One renders through the alpha-blend shader. One renders through the wireframe shader. One is the collider the wireframe draws around. No hierarchy. No casts. No surprises.

`GameObj` itself got stripped to position, rotation, scale, and lifecycle. That's it. And for the first time, I marked the class `final`. This is no longer a preference or a style guideline — the compiler refuses to let anyone subclass `GameObj`. The engine has opinions now. You compose, or you don't ship. **Composition or bust.**

## The Contract

Here's the `Shader` protocol the whole refactor rests on:

```swift
@MainActor
public protocol Shader: AnyObject {
    var maxObjects: Int { get }

    func beginFrame() -> Bool
    func bind(pass: RenderPass, projectionBuffer: MTLBuffer)
    func submit(objects: [GameObj])
    func flush(pass: RenderPass)

    nonisolated func signalFrameComplete()
}
```

Each shader owns its own pipeline state, its own `BufferProvider` sized to its own uniform stride,[^1] and its own batching strategy. The renderer never looks inside. It calls `bind → submit → flush` on whichever shader is active and moves on.

Swap the active shader mid-pass and the old one flushes automatically. Three shaders in one `beginPass/endPass` becomes a series of `useShader(X) → submit(...)` calls, and each shader only draws the objects that carry its matching component.

That's the whole architecture. Everything else is implementation.

## Five Tags in Two Hours

Five tags in a couple of hours last night:

- **0.8.0** — Breaking refactor. New `Shader` protocol. `AlphaBlendShader` + `AlphaBlendComponent`. `RenderPass` becomes a thin wrapper around the command encoder; `AlphaBlendRenderPass` is deleted. `GameObj` stripped and `final`.
- **0.8.1** — First additive shader: `WireframeShader` + `WireframeComponent`. SDF-on-unit-quad for circle and AABB outlines. Fragment function branches on a shape enum — one pipeline, both primitives.
- **0.8.2** — Fix: Custom shaders weren't getting `beginFrame` called. The renderer was only lifecycling its built-in alpha-blend. Added `register(shader:)` on the `Renderer` protocol so custom shaders opt in.
- **0.8.3** — Fix: Circle wireframes looked 2× thicker than AABBs because the circle SDF was two-sided (a band on both sides of the radius). Switched to one-sided inward so the metric matches. Three-line shader edit.
- **0.8.4** — Third shader, because why not: `RippleShader` + `RippleComponent`. UV-offset distortion via sinusoidal offset in the fragment function. Scene advances `comp.time += dt`, shader reads the clock off the uniform.

Three shaders coexisting. Same architecture handles all of them. Same object can be drawn through all three simultaneously.

## Three Shaders, One Scene

Here it is running:

<iframe width="560" height="315" src="https://www.youtube.com/embed/mDSr-4DkDSU" title="LiquidMetal2D Multi-Shader Demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

One scene. Pairs of ships heading at each other. Left ship has an AABB collider. Right ship has a circle. Each ship carries three components: `AlphaBlendComponent` + `WireframeComponent` + `RippleComponent`.

Three toggles at the bottom:

- **Spawn** — another pair
- **Wire: On/Off** — whether to run the wireframe pass at all
- **Ripple: On/Off** — whether the sprite pass uses the alpha-blend shader or the ripple shader

On collision, the wireframe tint flips from red to green. Per-object state mutation on `WireframeComponent.color` that the shader reads every frame.

The whole render loop is the usual shape:

```
beginPass
usePerspective
  useShader(alphaBlend OR ripple)
  submit(objects)               // each shader filters by its own component
  useShader(wireframe)
  submit(objects)
endPass
```

Each shader walks the same `[GameObj]` list and skips objects that don't have its component. No parallel object lists. No per-shader scenes. No duplicated transforms.

## The Honest Caveat

Before somebody emails me: *you could do all of this in one shader.* Of course you can. Stuff a flag into the uniform, branch in the fragment function, composite the wireframe on top of the sprite in the same pass, done. For a handful of effects that's often the right call.

The refactor isn't an argument that multi-shader is the Correct Way. The point is that the engine can now *support* the choice. When you've got a couple of unrelated effects, you split them. When you've got a few related effects, merge them into one shader and branch. The architecture doesn't dictate either direction anymore.

The place where it actually earns its keep, for me, is debug rendering. The wireframe pass is a real debug tool. Every collider draws its own outline. You can see when a circle overlaps an AABB the exact frame they overlap, because the color changes. I'm probably going to add velocity-direction indicators next, and maybe a pass that visualizes the spatial grid. That kind of stuff is exactly what debug shaders exist for, and now I can add them without touching the production shader.

## The Whole Night in One Session

This whole session lands inside [the workflow I posted yesterday](/2026/04/19/no-map-no-magic-prompt/):

- **Vegas flight.** Iterated the plan with one Claude session — v1 (subclass), v2 (generics), v3 (components). No code yet. Just argument.
- **Vegas room, later last night.** The blogger agent was finishing up [Part One](/2026/04/18/flying-motorcycles-movable-buildings/) and helping me draft [the workflow post](/2026/04/19/no-map-no-magic-prompt/) in parallel while the coder agent worked through the multi-shader plan. I was monitoring the execution and debugging alongside it.
- **The refactor landed.** 0.8.0 tagged and pushed. Then extensions came fast: *"Let's add a wireframe shader."* Did it. Hit the lifecycle bug. Fixed it (0.8.2). Caught the thickness issue. Fixed it (0.8.3). *"Let's add a third shader just to prove the architecture."* Ripple. 0.8.4.
- **Version-bump tax.** Every three-line bug fix shipped as a new tag because the demo depends on the library via tagged release. The coder agent eventually offered to switch the demo to a local path dependency so we could skip the ceremony. Fair point. Didn't take the offer — I wanted to keep testing against actual releases for this session — but it's on the workflow-debt list.
- **Handoff.** Once the session was done, the coder agent dumped its session notes. I handed those notes to the blogger agent (this one). You're reading the output.

A handful of agents across the session. Three posts published. One working engine refactor that nobody complained about. No map. Good primitives.

## Compound Interest

The thing I want to remember from this session is the compound-interest angle.

I added the component system [in Part One](/2026/04/18/flying-motorcycles-movable-buildings/) because I was tired of subclassing. It was supposed to fix colliders and behaviors. That was the entire design intent.

Two days later, when I needed a mechanism for per-object per-shader state, **the component system was already exactly the thing I needed.** Zero changes to the `Component` protocol. Zero changes to `GameObj.add / get`. The three new shaders just grew components of their own and plugged in. The primitive paid off on a problem it wasn't designed for.

That happens a lot with small, well-scoped primitives. That's the whole argument for them.

## The Commit Trail

If you want to walk the actual code, the commit progression is readable as a narrative on its own:

- [0.7.4 → 0.8.0](https://github.com/mattCasanova/LiquidMetal2D/compare/0.7.4...0.8.0) — the breaking refactor (multi-shader support via components)
- [0.8.0 → 0.8.1](https://github.com/mattCasanova/LiquidMetal2D/compare/0.8.0...0.8.1) — WireframeShader lands
- [0.8.1 → 0.8.2](https://github.com/mattCasanova/LiquidMetal2D/compare/0.8.1...0.8.2) — custom shader lifecycle fix
- [0.8.2 → 0.8.3](https://github.com/mattCasanova/LiquidMetal2D/compare/0.8.2...0.8.3) — circle thickness fix
- [0.8.3 → 0.8.4](https://github.com/mattCasanova/LiquidMetal2D/compare/0.8.3...0.8.4) — RippleShader lands

The design plan that drove it lives in the repo: [`multi-shader-pipeline-support.md`](https://github.com/mattCasanova/LiquidMetal2D/blob/main/multi-shader-pipeline-support.md). You can read the three shapes of the plan in there — the same v1 / v2 / v3 story, with more math and less dramatic phrasing.

---

*This is Part Two of the component system arc. [Part One was Flying Motorcycles and Movable Buildings](/2026/04/18/flying-motorcycles-movable-buildings/). There probably won't be a Part Three. The component system isn't done paying off, but the next time a problem shows up that maps onto "per-object metadata a pass needs to read," I'll add another component, not another subclass.*

*Components all the way down.*

[^1]: `maxObjects` is hardcoded per shader — the GPU buffer is allocated once at startup and never grows. I thought about making it dynamic, but resizing buffers mid-frame is genuinely annoying, and the numbers said don't bother. At 10,000 instances across three shaders with triple-buffering, you're spending ~8.8 MB on uniform buffers. A single retina drawable on an iPhone 17 Pro is ~14 MB, and Metal keeps three around. Uniform buffers are a rounding error next to framebuffers and textures. Hardcode. Move on.
