---
layout: post
title: "Cheaper and Cheaper"
tags: [liquidmetal, gamedev, ai, performance, product]
summary: "Some big performance wins in my game engine this week, and a bug I audited right past in April. The models are getting better, the engineering keeps getting cheaper, and the interesting part now is making a game that's fun."
---

This will probably be a really short one.

The models are definitely getting better. Opus 5.5 is very good.

This week I got some interesting performance wins in [LiquidMetal2D](https://github.com/mattCasanova/LiquidMetal2D), my game engine: an audit of the math and the hot paths, taking Swift's runtime safety checks out of the frame, and moving the sprite transform onto the GPU. Release builds, iPhone 17 Pro Max simulator, M4 Max:

| What | Before | After |
|---|---|---|
| Particle spawn, full pool (per 60 frames) | 5.95 ms | 0.51 ms (11.7×) |
| Alpha-blend submit, 5,000 sprites | 102 ns/sprite | 29 ns/sprite (3.5×) |
| MassRender frame, 20,000 ships | 9.54 ms | 4.92 ms |
| Most rotating ships at 60 fps | ~40,000 | ~120,000 (3×) |
| Per-frame allocations, steady scenes | several arrays | 0 |

20,000 ships now cost less than 10,000 did before.

### The one I missed

Some of this code is stuff I wrote with AI in April, when I was [in Vegas](/2026/04/19/five-year-particle-system/). And one of the math functions, the circle-to-line-segment test, I audited back then and didn't find the problem.

Here's what it was:

```swift
// before
let lineVector = end - start
let pointLineVector = center - start

let projectedLength = simd_dot(pointLineVector, simd_normalize(lineVector))

let adjustedStartLength = projectedLength + radius
let adjustedEndLength = projectedLength - radius

if adjustedStartLength < 0 ||
    (adjustedEndLength * adjustedEndLength) > simd_length_squared(lineVector) {
    return false
}

let pointLineLengthSquared = simd_length_squared(pointLineVector)
return (pointLineLengthSquared - (projectedLength * projectedLength)) < (radius * radius)
```

It gave the segment square end caps instead of round ones. A circle with radius 10, centered 0.5 behind the start of a 10-unit segment, said miss. A circle with radius 1, centered 1.27 from an endpoint, said hit. The tests all used small circles on long segments and never poked past the ends, so they passed.

The correct version came out 19% slower than the broken one, because the old early exits were part of what made it wrong. Then the rewrite:

```swift
// after
let line = end - start
let toCenter = center - start
let radiusSquared = radius * radius
let along = simd_dot(toCenter, line)

if along <= 0 {
    return simd_length_squared(toCenter) <= radiusSquared
}
let lengthSquared = simd_length_squared(line)
if along >= lengthSquared {
    return simd_length_squared(center - end) <= radiusSquared
}
let cross = line.cross(toCenter)
return cross * cross <= radiusSquared * lengthSquared
```

The cross product of the line and the offset is the line's length times the distance, so `distance ≤ radius` becomes `cross² ≤ radius² × length²`. No square root, no divide. 1.19 ms per million calls against the broken version's 2.30: 1.9× faster, and checked against a closest-point reference on 10,000 random cases.

I didn't find that. This audit did.

### Leaning towards product

The other interesting thing: I'm leaning towards product, because the models are really helping me with the engineering part. I'm still looking at the architecture. But at this point it kind of stops feeling like "I made these awesome improvements." It's definitely AI making the awesome improvements. The engineering pieces are getting cheaper and cheaper.

Skeletal animation is the same thing. I used to teach skeletal animation back in the day, and I didn't put it in this engine until now. So this is less about bragging that I have some cool skeletal animation in my game engine, because less and less am I writing the code. I'm not writing it. The engineering feats are cheaper and cheaper: let's do skeletal animation. Then I talk about how I want it organized, and it gets organized based on my architecture. (It shipped this month, and two-bone IK went in yesterday as 0.17.0.)

It's all organized around the engine I started six years ago, which is interesting. But it's definitely not the same as doing it alone.

The performance wins aren't unimportant. The engineering is good, and I want the game engine to be fast. But it's basically free, so there's less art to it at this point.

I feel like that's the line between engineering and product: "I want to build a good product." Actually building the game will be more important. Making a fun game is going to be the more interesting, artistic part.

So the goal of building all this is a game that hopefully ends up being fun.

See you, space cowboy.
