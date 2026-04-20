---
layout: post
title: "The Five-Year Particle System"
tags: [liquidmetal, gamedev, metal, swift]
---

In [the OCP post](/2026/04/17/house-rules-whatever-happens-happens/) two days ago, I said *"I still don't have a particle system in LiquidMetal2D. I'll get to it when the bones are ready."*

In [Flying Motorcycles](/2026/04/18/flying-motorcycles-movable-buildings/) last night, I said *"particles can wait; I've been saying that for five years, they'll keep."*

Particles did not, in fact, keep.

## What Actually Landed

I started LiquidMetal2D in 2020. *"Add a particle system"* has been sitting on the TODO since before the engine had a name. Every few months I'd look at it, decide the foundation wasn't ready, and go do something else — broadphase collision, the component system, [the multi-shader refactor](/2026/04/19/render-pass-strikes-back/). Always something more foundational.

Last night after shipping Render Pass Strikes Back, I finally ran out of excuses.

The first cut was a `ParticleShader`, a `ParticleEmitterComponent`, a `Particle` struct, and a procedural soft-circle texture built into the engine. Additive blending only. It worked. It looked like fire.

From there, each pass added one thing and kept going. An alpha-blended variant for smoke. Per-particle scale animation so smoke could puff out instead of stamping as constant-sized dots. Per-particle color variation — each particle picks its own point on a gradient lane. An optional toggle for chaos-flavored uncorrelated variation, in case you want sparks instead of fire.

Everything additive. No breaking changes. Existing scenes keep working untouched.

## The Fire Demo

<iframe width="560" height="315" src="https://www.youtube.com/embed/haYbecn2AX0" title="LiquidMetal2D Particle Demo (Fire)" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Drag the emitter anywhere on screen. Seven sliders on the side: emission rate, speed, scale, lifetime, spread, gravity. Color toggle flips between *Fire* (orange → red) and *Neon* (pink → purple). There's a Burst button for one-shot puffs and a Pause that keeps existing particles alive while stopping new spawns.

What you're looking at is additive blending — `sourceAlpha * one`. Overlap *brightens.* That's why fire looks like fire. Two embers on top of each other don't blend, they add, and the overlap becomes hot. Order-independent — addition is commutative, so there's no z-sort needed. Fast.

Here's the minimum emitter setup:

```swift
obj.add(ParticleEmitterComponent(
    parent: obj,
    maxParticles: 400,
    textureID: renderer.defaultParticleTextureId,
    emissionRate: 140,
    lifetimeRange: 0.8...1.6,
    speedRange: 6...14,
    angleRange: (.pi / 2 - 0.25)...(.pi / 2 + 0.25),
    scaleRange: 4...8,
    startColor: Vec4(1.0, 0.55, 0.15, 0.7),
    startColorVariation: Vec4(1.0, 0.85, 0.20, 0.7),
    endColor: Vec4(0.9, 0.1, 0.0, 0.0),
    endColorVariation: Vec4(0.5, 0.0, 0.0, 0.0),
    gravity: Vec2(0, 1)
))
```

That's it. One component, attached to any `GameObj`. The shader walks the scene, finds emitters, ticks their particles, writes the uniforms.

## Same Shader, Different Blend Mode

Fire worked on day one. Smoke did not.

Smoke isn't fire with different colors. Two gray puffs overlapping shouldn't *brighten* — they should composite. Stacked soft smoke is still soft smoke, not a hotspot. That's alpha blending: `sourceAlpha * oneMinusSourceAlpha`. Order-dependent. You have to sort particles back-to-front per frame or the compositing goes wrong.

So `ParticleShader` took a `BlendMode` parameter. Same shader class. Same fragment function. Same particle struct. Different pipeline-state blend factors, plus a z-sort on the `.alpha` path.

Here's the realization that surprised me mid-session: **the texture's alpha channel is not the alpha mode.** A PNG with transparency is just data. The *blend mode* decides what the GPU does with that data. The same star-shaped PNG, rendered additive, gives you glowing stars that brighten on overlap. Rendered alpha, it gives you crisp stamped stars that composite. Same pixels, same shader, different pipeline state.

That's why it's one shader class with an enum, not two shaders that share 95% of their code. The blend mode is a configuration, not an architecture.

## The Smoke Demo

<iframe width="560" height="315" src="https://www.youtube.com/embed/ZELSZznkrKI" title="LiquidMetal2D Particle Demo (Smoke)" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Same drag-to-move pattern. Eight sliders this time — the extra two are *Start Scale* and *End Scale*, because real smoke puffs *out.* That was the other realization. Constant-size additive particles look fine as fire because overlap fills in the growth visually. Constant-size alpha particles look like tiny stamped coins. Doesn't read as smoke at all.

Fix: `endScaleRange: ClosedRange<Float>?` on the emitter. Nil keeps the old behavior. Non-nil picks a random end scale per particle, and the shader lerps `startScale → endScale` by `age / lifetime`. Same mirror of how color animation was already working (`startColor → endColor`). Nice symmetry.

## Gradient Lanes, Not Palettes

*Color variation* went through a design call worth naming. I didn't want a palette — a discrete list of N colors to pick from. That's a separate feature (filed for later). What I wanted was a **gradient lane**: each particle picks a point along a line between two colors and sticks with it for its lifetime.

API is one extra optional field:

```swift
startColor: Vec4(1.0, 0.55, 0.15, 0.7),           // primary orange
startColorVariation: Vec4(1.0, 0.85, 0.20, 0.7),  // golden yellow
```

Per particle, roll `t = random(0...1)`, blend `t%` of the way from start-color to the variation-color. The second tag (0.10.2) added a `correlatedColorVariation: Bool` toggle — when `true` (the default), the same `t` is used for the start/end pair, so a particle that starts golden-ish also ends on the warm side of its red range. When `false`, the two rolls are independent, so you get chaos-flavored random pairings. Good for magic / sparks / effects that should read as unstable.

Small feature. Changes the visual density of a scene a lot.

## They're a Component, Of Course

The reason the whole thing took one evening is that the component system [from two days ago](/2026/04/18/flying-motorcycles-movable-buildings/) and the multi-shader refactor [from last night](/2026/04/19/render-pass-strikes-back/) were already doing the heavy lifting.

`ParticleEmitterComponent` is a component, same as every other component. Attach it to any `GameObj`. Scene update calls `emitter.update(dt:)`. Shader reads from it. `ParticleShader` is another `Shader` conformer — register it on the renderer, and it gets the same `beginFrame` / `bind` / `submit` / `flush` lifecycle every other shader gets. No new axis of architecture. No renderer changes. Just another component and another shader that slot into the same model that already handled alpha-blend, wireframe, and ripple.

Compound interest on primitives you got right. You don't really notice it until the next feature costs you three hours instead of three weekends.

## Next: A Tool, Not a Shader

Five minutes into tweaking fire with the sliders, it stopped feeling like programming and started feeling like *design.* Drag the emission rate up. Shorten the lifetime. Flip to neon. Iterate. The engine handed me a dial and I kept turning it.

Which made the next move obvious. The next thing LiquidMetal2D needs isn't another shader. It's a **particle configuration tool** — sliders, color pickers, live preview, save to JSON, ship the preset into your game. Pure-code engine with an actual visual editor beside it. That's the next post.

---

*Five years of *"particles can wait,"* one evening to ship them, another evening's worth of ideas queued up behind. Issues [#111](https://github.com/mattCasanova/LiquidMetal2D/issues) (palette-based variation), [#112](https://github.com/mattCasanova/LiquidMetal2D/issues) (save/load plumbing), and [#113](https://github.com/mattCasanova/LiquidMetal2D/issues) (emitter spawn shapes — line, box, circle) are already filed.*

*Best trip to Vegas I've ever had. Didn't win a dime.*
