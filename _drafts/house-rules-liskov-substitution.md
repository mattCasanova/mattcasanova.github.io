---
layout: post
title: "House Rules: Don't Be Astonishing"
series: house-rules
tags: [architecture, solid, opinion, gamedev, swift]
---

Years ago, before I got my current job, I interviewed at another big company. At some point in the interview, the guy asked me to name the SOLID principles. I couldn't tell exactly what his full intentions were, but it just seemed like a gotcha question. He didn't expect me to know. The tone you use when you don't expect people to know all five off the top of their head.

I rattled them off. Again. He didn't seem thrilled about that.

Here's the thing — Liskov Substitution sounds scary and it isn't. It just has a person's name on it instead of a description. "Single Responsibility" tells you what it is. "Open-Closed" tells you what it is. "Liskov Substitution" tells you that someone named Liskov had an opinion about something. That's why interviewers weaponize it. It sounds academic. It sounds like textbook trivia.

It's not.

## This One Was Hard to Write

Not because it's hard to understand. It was hard to write because I don't have any LSP violations in my code. The whole point of the principle is avoiding the bug I'm about to show you, and I've already avoided it. It's hard to find an example of a thing that isn't there.

That isn't to say there's no bugs in my code — there are plenty. It's just that none of them are LSP violations. Which is great for my app and terrible for this blog post.

So I had to think up examples. And that's when I realized something: the reason I don't have violations isn't because I'm some LSP savant. It's because if you follow the other four principles, this one mostly takes care of itself. More on that later.

But first — here's the formal definition everyone skips past:

> **If S is a subtype of T, then objects of type T should be replaceable with objects of type S without breaking the program.**

Cool. Super helpful. Let me just tattoo that on my forearm so I can reference it during code review.

Here's what it actually means: **don't be astonishing.** Does this thing behave the way everything else in the system expects it to? If yes, you're good. If no, you have a bug that the compiler can't catch. That's the whole principle. The Principle of Least Astonishment, applied to your code.

## You've Already Seen This Bug a Thousand Times

You've seen this bug. I've seen this bug. Everyone's seen this bug. You just didn't call it an LSP violation.

The code normally behaves one way. Path one works. Path two works. Then someone goes down path three and it does something completely different. Unexpected. Not documented. Not clear why. Somebody comes along later and has to fix it.

*"What the hell is this doing here? Why would they do X in this path but Y in this path? It doesn't make any sense."*

That's LSP. You just never called it that.

If I say "null pointer exception," you can probably think of three specific times in your life you hit one. But this is more general and way more common: the code always works this way — except when it doesn't. And when it doesn't, nobody expected it.

## The 90s Version (Why It Sounds Academic)

The textbook version of LSP is all about deep inheritance trees. This is the stuff they warn you about in the design patterns books from the 90s and 2000s.

I never worked at Zynga, but I had a buddy who did. The story goes: they built Farmville, then Cityville, then whatever-ville. The codebase had deep inheritance trees where — and I wish I was making this up — something like a zebra inherited from a combination of movable and building. A zebra. Inheriting from a building. That's astonishing. That's LSP violated at a structural level — the subclass IS NOT the thing it claims to be.

And the classic textbook example is the square and the rectangle. If `Square` extends `Rectangle`, and you call `setHeight()`, the square secretly changes the width too. The caller expected a rectangle. They got a liar.

I'm not going to belabor the square-rectangle thing — that example's been beaten to death by every SOLID blog on the internet. The reason I bring it up at all is to explain why LSP sounds academic: because the examples everyone uses ARE academic. Nobody writes code like that anymore. So people hear "Liskov Substitution" and think "doesn't apply to me."

It does. It just looks different now.

## The Modern Version (The One Everyone Hits)

You don't use deep inheritance trees anymore. You use interfaces. Shallow hierarchies. Composition. So you think you don't hit LSP.

But the spirit of LSP — **don't lie about what your code does** — is the most common bug in software. Nobody calls it an LSP violation. They just call it a bug.

**The "hidden side effect."** `saveData(data)` looks the same everywhere. Path A saves to a database. Path B saves to a file. Path C saves to a database AND triggers a massive email notification blast. A developer uses Path C thinking it's just like Path A. System breaks.

**The "except for Monday" bug.** `ProcessPayment()` works for every provider. Except PayPal, where you have to pass a dummy value or check `if (provider == 'PayPal')` before calling it. The moment you write that `if` check, the abstraction has lied to you. The substitution test fails — PayPal can't be used interchangeably with the others without the calling code knowing too much.

These aren't hypothetical. Every codebase has them. The payment provider that works slightly differently. The storage backend that silently drops data over a certain size. The API endpoint that returns a different shape for one specific edge case. These are LSP violations wearing a trench coat and calling themselves "features."

## The Real World Is Full of Edge Cases

Here's the honest part: these violations don't happen because someone was dumb. They happen because the business has legitimate edge cases.

PayPal really does work differently from Stripe. The free tier really does have different limits than premium. The V2 device really does have a speaker the V1 doesn't.

The violation isn't having edge cases — the violation is hiding them behind an interface that pretends they don't exist. That's the lie.

The fix isn't eliminating edge cases. It's recognizing them and stopping pretending they fit the same interface when they don't. If they behave differently, they shouldn't share the same contract. Uncle Bob's version: if a square behaves differently than a rectangle, they shouldn't share that `setHeight` interface. Stop trying to force them into the same bucket just because they look similar on a napkin.

And here's the tradeoff nobody talks about: when you shove a different thing into an existing interface because "it's close enough," you're optimizing for the author — less code to write today. But you're screwing the maintainer — a surprise to debug in six months. Same thing I wrote in [the Fakes post](/2026/04/14/bad-advice-roll-your-own-fakes/): *the author pays a minute, the maintainer pays a month.*

## The Ghost Enemy

It's easy to show an example of the other principles done right. Here's SRP in my code. Here's OCP in my game engine. But if I show you LSP done right, it just looks like... normal code. There's no trick. You're just doing the thing everyone expects. That's the whole point — and that's why there's nothing to show.

So the example is contrived. But it's contrived to show the point.

In [LiquidMetal2D](https://github.com/mattcasanova/LiquidMetal2D), every `GameObj` has a `tintColor` property and a `toUniform()` method that packs its data for the GPU. The base class honors the contract — set `tintColor`, it shows up on screen:

```swift
// GameObj base class — honors the contract
open func toUniform() -> UniformData {
    let uniform = AlphaBlendUniform()
    uniform.transform.setToTransform2D(
        scale: scale, angle: rotation,
        translate: Vec3(position, zOrder))
    uniform.color = tintColor  // whatever you set, it gets packed
    return uniform
}
```

Now imagine someone writes a `GhostEnemy` subclass. Ghosts are always translucent white — that's the game rule. So the developer hardcodes it:

```swift
class GhostEnemy: GameObj {
    override func toUniform() -> UniformData {
        let uniform = AlphaBlendUniform()
        uniform.transform.setToTransform2D(
            scale: scale, angle: rotation,
            translate: Vec3(position, zOrder))
        // "Ghosts are always translucent white"
        uniform.color = Vec4(1, 1, 1, 0.5)
        return uniform
    }
}
```

Looks reasonable. Ghosts are white. Done.

Then another developer writes a power-up that makes ghosts visible — flash them red when they're vulnerable:

```swift
ghost.tintColor = Vec4(1, 0, 0, 1)
```

Nothing happens. The ghost stays translucent white. No crash. No error. The property was set. It just doesn't work on this particular subclass.

Not because someone forgot. Because someone *chose* to enforce a game rule inside `toUniform()`. The base class made a promise: set `tintColor`, it shows up. The subclass broke that promise.

The fix is simple: the uniform should just pack whatever it's given. Ghosts being translucent white is a game rule. Enforce it where game rules live — in the scene's `update()`, in the ghost's initialization, in whatever spawns the ghost. Not in `toUniform()`. The uniform's job is packing data. The game's job is deciding what the data should be.

**Don't enforce business rules by breaking the contract at the wrong layer.**

## The Inverse of the Other Four

Here's the thing that clicked for me while writing this post.

The other four SOLID principles are about keeping different things separate. SRP: separate your responsibilities. OCP: separate old code from new code. ISP: separate your interfaces. DIP: separate your abstractions from your implementations.

LSP is the inverse. It's about **not pretending different things are the same.**

With S, O, I, and D, you minimize blast radius when things change outward — new features, new requirements, new code. LSP is about blast radius when things change inward — a new edge case, a new exception, a new "well actually this one behaves differently."

The trap is premature minimization. You see an existing interface and think *"this new thing is close enough, I'll just shove it in here."* Less code. Feels clean. But the fake similarity is what blows up later. Claiming two things are the same to avoid the cost of admitting they're different — that is the LSP violation.

## The Canary

LSP isn't a principle you sit down and apply. It's a **symptom you watch for.** When you see an LSP violation — when a subclass surprises you, when an implementation lies about what it does, when path three does something path one and two would never do — it usually means one of the other four principles is broken upstream.

Deep inheritance tree where a zebra extends a building? That's an SRP violation. An implementation that secretly does something different? That's an OCP or ISP issue. A dependency that lies about its behavior? That's a DIP problem.

If you follow S, O, I, and D, LSP mostly takes care of itself. It's the canary. When it drops dead, check the other four.

And that's why this post was hard to write. My engine follows the other four principles. So the canary is still alive and well. I had to make up a ghost to kill it.

---

*This is the third entry in the **House Rules** series. Next up: the I. Interface Segregation Principle. My Detekt linter caught it. I told it to shut up. That was a mistake.*

*So what have we learned so far? S: interfaces. O: interfaces. L: interfaces. Sensing a pattern?*

*I didn't write 'em, but those are the rules.*
