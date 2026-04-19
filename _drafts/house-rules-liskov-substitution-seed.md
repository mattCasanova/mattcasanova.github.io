---
layout: post
title: "House Rules: L (Liskov Substitution)"
series: house-rules
tags: [architecture, solid, opinion]
status: seed
---

## Pre-draft seed — not a post yet

### The hook

Years ago, before I got my current job, I interviewed at another big company. At some point in the interview, the guy asked me to name the SOLID principles. I couldn't tell exactly what his full intentions were, but it just seemed like a gotcha question. He didn't expect me to know. The tone you use when you don't expect people to know all five off the top of their head.

I rattled them off. Again. He didn't seem thrilled about that.

Here's the thing — Liskov Substitution sounds scary and it isn't. It just has a person's name on it instead of a description. "Single Responsibility" tells you what it is. "Open-Closed" tells you what it is. "Liskov Substitution" tells you that someone named Liskov had an opinion about something. That's why interviewers weaponize it. It sounds academic. It sounds like textbook trivia. It's not.

### The honest admission (right after interview hook)

This one was hard to write.

Not because it's hard to understand — it's not. It was hard to write because I don't have any LSP violations in my code. The whole point of the principle is avoiding the bug I'm about to show you, and I've already avoided it. It's hard to find an example of a thing that isn't there.

That isn't to say there's no bugs in my code — there are plenty. It's just that none of them are LSP violations. Which is great for my app and terrible for this blog post.

So I had to think up examples. And that's when I realized something: the reason I don't have violations isn't because I'm disciplined about LSP specifically. It's because if you follow the other four principles, LSP mostly takes care of itself. More on that later.

But first — the violations that DO happen, even in clean codebases, go way beyond just interfaces. We've all seen bugs where the code normally behaves one way and then in this one specific path it does something completely different. Unexpected. Not documented. Not clear why. Somebody comes along later and has to fix it.

You don't categorize those in your head as "LSP violations." You categorize them as bugs. If I say "null pointer exception," you can probably think of three times in your life you hit one. But this is more general and way more common: the code always behaves this way, except when it goes down this one path, and then it does this other thing, and nobody expected it.

We've all seen these. *"What the hell is this doing here? Why would they do X in this path but Y in this path? It doesn't make any sense."* That's LSP. You just never called it that.

### The principle (reframed)

The formal definition: **If S is a subtype of T, then objects of type T should be replaceable with objects of type S without breaking the program.**

The real definition: **don't be astonishing.** This is basically the Principle of Least Astonishment applied to your implementations. Does this thing behave the way everything else in the system expects it to? If yes, you're good. If no, you have a bug that the compiler can't catch.

### Core insight: the inverse of the other four

The other four principles are about keeping different things separate. LSP is about not pretending different things are the same.

With S, O, I, and D, you minimize blast radius when things change outward — new features, new requirements, new code. LSP is about blast radius when things change inward — a new edge case, a new exception, a new "well actually this one is different."

The trap is premature minimization. You see an existing interface and think "this new thing is close enough, I'll just shove it in here." Less code. Less ceremony. Feels clean. But you're optimizing for the author (less to write today) at the expense of the maintainer (a surprise to debug in six months). Same tradeoff from the Bad Advice arc — "the author pays a minute, the maintainer pays a month." Echo that quote and link back to the Fakes post where we first said it.

The fake similarity is what blows up later. Claiming two things are the same to avoid the cost of admitting they're different — that's the LSP violation. The fix is admitting they're different and giving them different interfaces, even if it costs you more code today.

### The thesis: LSP is a canary, not a principle you apply

LSP isn't a principle you sit down and apply like the other four. It's a **symptom you watch for.** When you see an LSP violation, it usually means one of the other principles is broken upstream. Deep inheritance tree? Probably an SRP or OCP violation. A subclass that surprises you? Probably missing an interface (ISP). An implementation that lies about what it does? Probably a dependency that should have been inverted (DIP).

If you follow S, O, I, and D, LSP mostly takes care of itself. It's the canary in the coal mine.

### The 90s version (the textbook one)

Deep inheritance trees. The stuff they warn you about in the books. This is where the classic violations live.

**The Zynga story (secondhand, from a buddy):** Zynga built Farmville, then Cityville, then whatever-ville. The codebase had deep inheritance trees where things like a zebra inherited from movable + building or something equally insane. A zebra inheriting from a building. That's astonishing. That's LSP violated at a structural level — the subclass IS NOT the thing it claims to be.

This is why LSP sounds academic — the textbook examples are all about inheritance hierarchies from the 90s and 2000s. Squares and rectangles. Birds that can't fly. Nobody writes code like that anymore (I hope). So people hear LSP and think "doesn't apply to me."

### The modern version (the one everyone hits)

You don't use deep inheritance anymore. You use interfaces. Shallow hierarchies. Composition. So you think you don't hit LSP.

But the spirit of LSP — **don't lie about what your code does** — is the most common bug in software. Nobody calls it an LSP violation. They just call it a bug.

- **The "hidden side effect"**: `saveData(data)` looks the same everywhere. Path A saves to a database. Path B saves to a file. Path C saves to a database AND triggers a massive email notification blast. A developer uses Path C thinking it's Path A. System breaks.

- **The "except for Monday" bug**: `ProcessPayment()` works for every provider. Except PayPal, where you have to pass a dummy value. The moment you write `if (provider == 'PayPal')` before calling the interface, the abstraction has lied to you.

- **The buffer lie (LiquidMetal2D)**: The renderer trusts that `uniform.size` tells it how many bytes `setBuffer()` will write. If a subclass lies about `size`, the renderer writes past the buffer boundary. GPU corruption. Crash. Types check out. Protocol satisfied. Behavior wrong.

### The honest part: the real world is full of edge cases

The reason these violations happen isn't because someone was dumb. It's because the business has legitimate edge cases. PayPal really does work differently from Stripe. The free tier really does have different limits than premium. The V2 device really does have a speaker the V1 doesn't.

The violation isn't having edge cases — the violation is hiding them behind an interface that pretends they don't exist. That's the lie. The fix isn't eliminating edge cases — it's recognizing them and stopping pretending they fit the same interface when they don't. If they behave differently, they shouldn't share the same contract.

### Why this letter is different (transition into the example)

It's easy to show an example of doing it correctly for the other principles. Here's SRP in my code. Here's OCP in my game engine. But if I show you an example of LSP done right, it just looks like normal code. There's no trick. You're just doing the thing everyone expects. That's the whole point — and that's why there's nothing to show.

So the example here is contrived. But it's contrived to show the point, hopefully very obviously.

### Concrete example: The Ghost Enemy (from the game engine, not square/rectangle)

A `GameObj` subclass where someone enforces a game rule at the wrong layer:

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

The base class promises: set `tintColor`, it shows up on screen. This subclass silently breaks that promise. Someone on the team writes a power-up that makes ghosts visible — `ghost.tintColor = Vec4(1, 0, 0, 1)`. Nothing happens. The ghost stays translucent white. No crash. No error. The property was set. It just doesn't work on this subclass.

Not because someone forgot — because someone *chose* to enforce a game rule inside the uniform packing. That's what makes it LSP and not just a bug. It was a deliberate decision that violated the contract.

The fix: the uniform should just pack whatever it's given. Ghosts being translucent white is a game rule. Enforce it where game rules live — in the scene's `update()`, in the ghost's initialization, in whatever spawns the ghost. Not in `toUniform()`. The uniform's job is packing data. The game's job is deciding what the data should be.

**The lesson: don't enforce business rules by breaking the contract at the wrong layer.**

### Real-world story (TBD)

Still looking for a personal "it worked in path one and two, why does path three do this wildly different thing" story. Will come. Don't force it.

### Real-world story (TBD)

Still need a specific personal story. "We all expected the code to work this way. It works in path one and two. Why does path three do this wildly different thing?" These are the most common bugs — broken expectations. The story will come during a debugging session. Don't force it.

### Why I don't see classic LSP much

Because I've used shallow inheritance for a long time. That's not a gap — that's the point. If you follow modern patterns (interfaces over inheritance, composition over inheritance), you naturally avoid the structural LSP violations. The behavioral ones — hidden side effects, lying implementations, edge cases that break expectations — those are the modern LSP, and everyone hits them.

### Structure idea

1. **Interview hook** — the gotcha question
2. **The scary name, the simple idea** — Liskov just means "don't be astonishing"
3. **The 90s version** — deep inheritance, Zynga zebra-from-building, the classic violations. This is what the textbooks talk about. This is why it sounds academic.
4. **The modern version** — you don't use deep inheritance anymore, so you think you don't hit LSP. But you do. Hidden side effects. The "except for Monday" bug. The buffer that claims to be 64 bytes and writes 96. The compiler says it's fine. It's not fine.
5. **The canary** — LSP isn't a principle you apply. It's a symptom you watch for. When you see it violated, something else went wrong upstream. If you follow S, O, I, and D, LSP mostly handles itself.

### Connection to the arc

- S post: "one reason to change" — the foundation
- O post: "whatever happens, happens" — new code, not changed code
- **L post: "don't be astonishing" — the canary in the coal mine**
- I post: "no client should depend on methods it doesn't use" — the god menu / Detekt suppression story
- D post: already covered in the Bad Advice arc

### Potential title ideas

- House Rules: The Compiler Won't Save You
- House Rules: It Compiled. It Lied.
- House Rules: Don't Be Astonishing
- House Rules: The Canary in the Coal Mine
- House Rules: The L That Sounds Scarier Than It Is
- House Rules: Liskov Isn't a Gotcha
- House Rules: Every Bug You've Ever Seen
