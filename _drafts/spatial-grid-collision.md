---
layout: post
title: "The Right Algorithm, Still Slower"
tags: [liquidmetal, swift, metal, gamedev, performance]
---

I had a collision detection problem. The fix was obvious — use a spatial grid. I implemented it, benchmarked it, and it was *slower* than brute force.

This is a story about why the right algorithm isn't enough.

## The Problem

[LiquidMetal2D](https://github.com/mattCasanova/LiquidMetal2D) is a 2D game engine I've been building in Swift + Metal. It does instanced rendering, has a scene system, a scheduler, the works. But for collision detection, I was doing the dumbest thing possible: checking every object against every other object.

With 200 objects, that's ~20,000 pair checks per frame. Fine. With 1,000 objects: 500,000 pairs. With 4,000: 8 million. It's O(n²), and it doesn't scale.

At 2,000 objects, brute force was running at 14fps. At 4,000: roughly 2fps. At 5,000: the game was a slideshow.

## The Obvious Fix

A **spatial grid** — divide the world into fixed-size cells. Each frame, every object registers into the cell that contains it. Instead of checking every pair, you only check objects that share the same cell or adjacent cells.

The algorithm I used is **half-neighbor traversal**: for each cell, pair the objects within it, then pair with 4 "forward" neighbors (right, below-left, below, below-right). This visits each cell-pair exactly once. Zero duplicate pairs, no deduplication overhead.

Algorithmically, this takes collision detection from O(n²) to practically O(n) for evenly distributed objects.

So I built it. Ran the benchmark. And it was slower than brute force.

## What Went Wrong

Two hidden costs, both invisible in the algorithm description:

**Cost #1: Allocation per frame.** My `potentialPairs()` method returned an array of tuples — a fresh `[(Collider, Collider)]` every frame. At 4,000 objects with hundreds of cells, that's thousands of tuple allocations per second. Swift's ARC was doing more work managing memory than the CPU was doing checking collisions.

**Cost #2: Linear lookup per pair.** After getting a pair of colliders, I needed to find which game object each collider belonged to. I was using `firstIndex(where:)` — a linear scan through the object list. O(n) per lookup, called thousands of times per frame. The "fast" spatial query was feeding into a slow lookup that negated all the gains.

## The Fixes

**Fix #1: Kill the allocation.** Replaced `potentialPairs()` with `forEachPotentialPair()` — a callback-based API that iterates inline without ever creating an intermediate collection.

```swift
// Before — allocates every frame
let pairs = spatialGrid.potentialPairs()
for (a, b) in pairs { ... }

// After — zero allocation
spatialGrid.forEachPotentialPair { a, b in ... }
```

Same traversal, same pairs, no allocation. This is the classic game loop pattern: in a hot path running 60 times per second, even small allocations compound. Both APIs still exist in the engine — use the callback in hot paths, the array version when you need to store or filter pairs.

**Fix #2: O(1) lookup.** Replaced the linear scan with an `[ObjectIdentifier: Collider]` dictionary. Object lookups went from O(n) to O(1).

## The Results

<!-- TODO: embed YouTube video of stress test here -->

| Objects | Spatial Grid FPS | Brute Force FPS |
|---------|-----------------|-----------------|
| 2,000 | 60 fps | 14 fps |
| 3,000 | 60 fps | ~2 fps |
| 4,000 | 60 fps | ~2 fps |
| 5,000 | 30-60 fps | ~1 fps |
| 7,000 | 30 fps | unplayable |

7,000 objects with full collision detection at 30fps. On a phone.

## The Lesson

The spatial grid was the right choice. Uniform grids are simpler and faster than quadtrees for 2D games where most objects are similar size. The API (`insert`, `query`, `forEachPotentialPair`) is the same regardless of spatial data structure, so it can always be swapped later if needed.

But the algorithm was never the bottleneck. The constants were. A fresh array allocation every frame and a linear scan per pair were doing more damage than the O(n²) brute force ever did at moderate object counts.

**Algorithmic improvements mean nothing if your constants are terrible.**

This is something you learn from game engines that doesn't always come through in algorithm courses. Big-O tells you the shape of the curve. It doesn't tell you where the curve starts. If your O(n) implementation has a massive constant factor hidden in allocation and lookup overhead, a dumb O(n²) loop with tight memory access can beat it for longer than you'd expect.

Profile first. Then fix what's actually slow.
