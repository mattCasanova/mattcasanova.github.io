---
layout: post
title: "House Rules: Two-Drink Minimum"
series: house-rules
tags: [architecture, design-patterns, strategy, metal, swift, opinion]
summary: "The simplest, most boring pattern in the book, played completely straight. Strategy, with the shader system out of my game engine as the example."
---

*Design-patterns arc: [Builder](/2026/06/07/house-rules-built-not-shaken/) → [Factory](/2026/06/08/house-rules-who-loads-the-jukebox/) → Strategy. SOLID's [in the can](/2026/05/06/house-rules-d/).*

This is the short one.

Builder earned a cocktail. Factory earned a jukebox. Strategy doesn't get a metaphor, because it doesn't need one — it's the simplest, most boring pattern in the book, and I'm going to play it completely straight. I've got a louder opinion about *how* simple it is, but that one's getting its own post and it isn't a House Rules. Tonight I'm just pouring.

## The whole pattern, in one breath

Strategy is a set of interchangeable algorithms behind one interface, picked and swapped from outside the thing that uses it. One job, more than one way to do it, and the *way* is an object you hand in.

The example everyone already knows: a sort that takes a comparator. Same sort routine, you hand it the order — by name, by date, reversed — and the sort never changes. You swapped the algorithm without touching the thing running it. That's the entire idea.

Now the one I actually reach for, because it's the cleanest Strategy I own.

## One renderer, four ways to draw

My 2D engine, LiquidMetal2D, draws everything through a single interface. Trimmed to the parts that matter:

```swift
@MainActor
public protocol Shader: AnyObject {
    func beginFrame() -> Bool
    func bind(pass: RenderPass, projectionBuffer: MTLBuffer)
    func submit(objects: [GameObj])
    func flush(pass: RenderPass)
}
```

Four concrete shaders implement it: `AlphaBlendShader` (the default, plain textured sprites), `RippleShader` (UV-distortion water), `ParticleShader` (instanced particles), `WireframeShader`. Each one owns its own GPU pipeline state, its own buffer, its own batching logic. They share nothing but the four-method shape above.

Here's the part that made me want to use this as the example. The doc comment I wrote on that protocol — months ago, with no plan to ever blog it — says, verbatim:

> *Each shader implements its own sort/batching strategy internally.*

I named the pattern in the code without thinking about it. That's how baseline this one is.

The renderer holds whichever shader is active and delegates to it:

```swift
renderer.useShader(rippleShader)
renderer.submit(objects: water)      // RippleShader decides how these draw

renderer.useShader(particleShader)
renderer.submit(objects: emitters)   // ParticleShader decides how these draw
```

`useShader(_:)` is the whole pattern in one call. From the renderer's own docs:

> *Makes `shader` the active shader. Flushes the previous active shader's pending draws first, then binds the new pipeline + resources onto the current pass.*

The renderer never names a concrete shader type. It holds a `Shader`, calls `bind`, `submit`, `flush` on it, and has no idea whether it's drawing rippling water or additive-blended sparks. Swap the shader, same renderer, completely different pixels. I author the algorithm, I hand it in, I switch it whenever I want — and the engine doesn't care. Strategy, top to bottom.

## The two lines people blur

I always end up drawing these, because Strategy sits right next to two patterns that look identical on a class diagram.

**State.** The renderer sets the active shader *from outside*. A shader never decides, mid-frame, to become a different shader. Set-from-outside-and-it-stays is Strategy. If the thing swapped *itself* on some internal transition — drew a few frames, then quietly promoted itself to the next behavior — that would be State. Same shape, different question: *who pulls the swap.* Outside is Strategy. Itself is State.

**Factory.** The shader *is* the drawing behavior — it's the thing that does the work. A factory would be the bit that decides *which* shader to build from, say, a material name. I have construction like that too, but the factory isn't the Strategy; the shader it produces is. **Factory chooses. Strategy does.**

## What is *not* a Strategy

This is the line I'd put on the door, because it's the one that survives every "well, isn't *everything* a Strategy?" argument:

**A Strategy is a thing that *is* behavior. Anything whose job is to *select* or *build* behavior is not one** — it's the thing that hands you one. The factory that picks the shader, the DI container that injects it, the `switch` that maps a key to it: none of those are strategies. They move strategies around. And an interface with exactly one implementation you'll never swap isn't a Strategy either — it's a class wearing a hat.

Everything that genuinely *is* swappable behavior behind a seam qualifies. Everything that *delivers* it doesn't.

## Where it doesn't fit — and what it costs

**One implementation, forever.** If there's exactly one way to do the job and there always will be, skip the interface and call the type. The seam is ceremony you'll never spend. Same engine, the opposite call: it's Metal-only. There's no `MetalRenderer` sitting behind a graphics-API strategy next to a `VulkanRenderer` and a `DirectXRenderer` — there's just *the* renderer, because there's only ever going to be one. I'm on iOS; I'm never swapping graphics APIs, least of all at runtime. So the Metal types get used directly, by name, right inside it. A cross-platform engine that has to speak Vulkan on Linux and DirectX on Windows might earn that strategy. Mine never will, so building it would be paying rent on a swap that can't happen. (There *is* a `Renderer` protocol in the engine — but that seam exists so someone using it can swap the *whole* renderer for their own, and so the game can stay blind to the engine in a test. It's a boundary, not a graphics-API switch. Different seam, different reason.)

**The indirection tax.** Every swap point is a place a bug can hide. Something throws, you're holding a `Shader`, and now the questions start — *which* concrete is this, and who set it as active? You traded a type you could click on for one more hop up the stack. Real cost. Cheap rent at two-plus implementations, a waste at one.

## Last call

The renderer got easy to use the day it stopped knowing what a `RippleShader` was. That's the win — the thing running the algorithm and the algorithm itself stopped needing to know about each other, and either side can change without the other noticing.

Hard to use, easy to break, becomes easy to use, hard to break — and this time it didn't even take a cocktail to explain. One interface, more than one implementation, swapped from outside.

I taught this one to sophomores for years. It's boring. There's nothing special about Strategy, and that's the honest review.

Which is exactly why I've got a louder opinion about it — that it's barely a pattern at all, that half the book is secretly this same move wearing a different coat. But that's a rant, not a house rule, so it's getting its own post: *A Confident Dose of Bad Advice: There Is No Spoon.* It's next, and it's already poured. Get ready.

---

*I didn't write 'em, but those are the rules.*
