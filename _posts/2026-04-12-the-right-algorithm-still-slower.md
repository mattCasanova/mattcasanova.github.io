---
layout: post
title: "The Right Algorithm, Still Slower"
date: 2026-04-12
tags: [liquidmetal, swift, metal, gamedev, performance]
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/9dTq0bG98Gg" title="LiquidMetal2D Collision Stress Test" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

That's 7,000 ships bouncing around a screen with full collision detection, running on a phone, at 30 frames a second.

It took me five years to get around to writing it.

## The Thing I Always Meant to Do

[LiquidMetal2D](https://github.com/mattcasanova/LiquidMetal2D) is a 2D game engine I've been building on and off in Swift + Metal since 2020. It does instanced rendering, has a scene system, a scheduler, a component model — the stuff you'd expect. But the collision detection was always the dumbest possible thing: check every object against every other object, every frame.

O(n²). Brute force.

I knew this was bad. I taught collision detection at the university level for seven years. I taught college students how to implement broadphase systems — uniform grids, quadtrees, sweep-and-prune, the whole catalog. I knew *exactly* what I should be doing. I just... never did it for this engine. The naive loop worked fine for the small demos I had running, I was mostly focused on the renderer, and broadphase was the kind of thing I kept filing under "I'll get to it."

Then I started using Claude Code, picked up LiquidMetal2D after a five-year break, and started blowing through my backlog. Spatial grid was one of the first things I finally sat down and wrote. One evening, and it was done.

Not because it was hard — I could have done it in a weekend five years ago. But because I finally had a collaborator that made "spend an evening wiring up a spatial grid" feel trivial instead of feel like a side quest. That's the real story of this whole blog lately: AI didn't teach me anything new. It removed the friction between "I know how to do this" and "this is done."

## Why Broadphase Matters

Quick recap for anyone who hasn't built a collision system.

Brute force collision checks every object against every other object. That's `n*(n-1)/2` pairs per frame:

- **200 objects** → ~20,000 pair checks
- **1,000 objects** → ~500,000
- **4,000 objects** → ~8 million
- **7,000 objects** → ~24 million

Twenty-four million collision checks per frame. At 60fps, that's 1.4 *billion* checks per second. Not happening on a phone. Honestly, not happening on anything.

A **broadphase** is a spatial partitioning step that cuts this down by only checking pairs that could *plausibly* be close to each other. There are a few common flavors:

- **Uniform grid** — divide the world into fixed-size cells, register each object into the cell that contains it, only check pairs in the same cell or adjacent cells.
- **Quadtree / octree** — recursive subdivision, adapts to uneven density but costs more per operation.
- **Sweep and prune** — sort objects along an axis and check overlapping ranges, great when objects don't move much frame to frame.

For a 2D game where most objects are roughly the same size and distribution is roughly uniform, a uniform grid is the right default. It's simple, cache-friendly, and hard to beat for the common case.

That's what I used. The specific traversal is **half-neighbor**: for each cell, pair the objects inside it, then pair against 4 "forward" neighbors (right, below-left, below, below-right). Every cell-pair gets visited exactly once. Zero duplicate pairs, no deduplication overhead.

This is textbook stuff. I've drawn this on whiteboards. I've graded assignments on it. None of this was new.

## It Was Slower Than Brute Force

Here's the part I did *not* expect.

The first cut of the spatial grid — correct algorithm, correct traversal — was **slower than the brute force loop I was replacing**. Not "about the same." Not "slightly faster but not as dramatically as the big-O suggests." Actually, measurably, embarrassingly slower.

I stared at it for a minute. Then I went looking for what I'd screwed up.

Two things:

**Cost #1: Allocation in the hot path.** My `potentialPairs()` method returned a fresh array of tuples — a new `[(Collider, Collider)]` on every call, every frame. With hundreds of cells and thousands of objects, that's thousands of tuple allocations per second. Swift's ARC was doing more work managing memory than the CPU was doing checking collisions. The spatial grid was saving checks, but the allocator was giving all those savings back.

**Cost #2: Linear lookup per pair.** Once I had a pair of colliders, I still needed to find which `GameObj` each collider belonged to. I was using `firstIndex(where:)` — a linear scan through the object list. O(n), called thousands of times per frame. A fast spatial query feeding into a slow object lookup is a bit like installing a highway that dumps out onto a dirt road.

Two fixes, neither of them algorithmic:

1. Replace `potentialPairs()` with `forEachPotentialPair()` — a callback-based API that iterates inline without ever creating an intermediate collection. Zero allocation. Same traversal.
2. Replace the linear scan with an `[ObjectIdentifier: Collider]` dictionary. Object lookups went from O(n) to O(1).

```swift
// Before — allocates every frame
let pairs = spatialGrid.potentialPairs()
for (a, b) in pairs { check(a, b) }

// After — zero allocation
spatialGrid.forEachPotentialPair { a, b in check(a, b) }
```

Both APIs still exist in the engine. The array version is fine when you need to store, filter, or sort the pairs — just don't use it in a hot loop running 60 times per second. Every allocation in a hot loop compounds into ARC overhead you can't see in the profile until it's already choking you.

After those two fixes, the spatial grid actually behaved like a spatial grid.

## The Numbers

| Objects | Spatial Grid | Brute Force |
|---------|-------------|-------------|
| 2,000 | 60 fps | 14 fps |
| 3,000 | 60 fps | ~2 fps |
| 4,000 | 60 fps | ~2 fps |
| 5,000 | 30-60 fps | ~1 fps |
| 7,000 | 30 fps | unplayable |

7,000 ships. 24 million theoretical pair checks per frame. Full collision detection at 30fps on a phone.

That video up top is the actual stress test in the demo app. You can toggle between brute force and spatial grid at runtime. Brute force crawls to a halt around 2,000 objects and stays there. Spatial grid just keeps going.

## The Real Story

I could wrap this up with "the algorithm isn't the whole fight, mind your constants" — and it's true — but honestly that's not why I wanted to write this post.

I wanted to write it because **I've been meaning to do this for five years**, and one evening with Claude and a stress test later, it was just done. The algorithm wasn't the hard part. The "finally sitting down and doing it" was the hard part.

But the slower-than-brute-force beat matters too, because it points at the way this collaboration actually has to work. I sat down, described what I wanted, and Claude wrote a spatial grid. I then ran it and it was slow. If I'd stopped there — just trusted the output and moved on — I'd have shipped a broadphase that made the game worse. That's the failure mode nobody talks about enough: AI doesn't always produce wrong code, it produces *plausibly correct* code that quietly does the wrong thing. And since it's been trained on mountains of code that looks like yours, it does it confidently.

You have to inspect what Claude writes. You have to run it. When something feels off, you have to actually read the code and go "wait, why are we allocating a fresh array every frame? Why is this a linear scan?" I'm driving the architecture. Claude is filling in the implementation and making best guesses about all the stuff I didn't explicitly specify. Those best guesses are usually fine. Sometimes they're ARC overhead in your hot path.

This is exactly the same story I just told in [88 Miles Per Second](/2026/04/12/88-miles-per-second/), about the terminal renderer. Claude wrote working code. I ran it. It was slow. I read the code and found the problem. Same pattern. Different domain. It's a pair programming loop, not a code generation loop — and the "pair" part is the part you don't get to skip.

It's the same thread I keep pulling in [Syntax Is the Least of Our Skills](/2026/04/11/syntax-is-the-least-of-our-skills/) and [Signal Through the Noise](/2026/04/11/signal-through-the-noise/): the stuff I already know how to do now takes an evening instead of a weekend. The stuff I didn't know how to do at all now takes a weekend instead of a month. And the stuff I would have shipped broken now gets caught because I'm still the one reading the diff.

Five-year backlogs don't survive that kind of friction reduction.
