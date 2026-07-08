---
layout: post
title: "A Confident Dose of Bad Advice: There Is No Spoon"
series: bad-advice
tags: [architecture, design-patterns, learning, opinion]
---

Here's tonight's bad advice: **stop memorizing design patterns.**

Not *stop studying* them. Study them — all twenty-three, especially if patterns and architecture are still new to you. You need them. What you don't need is the flashcard version, the one where you can recite the Factory class diagram but freeze when the code in front of you doesn't match the picture in the book. Study everything. Memorize nothing. The gap between those two is the whole post.

I'm allowed to say this. I taught this stuff — game programming, university level, seven years, freshmen and sophomores — and I [co-wrote a patterns book](https://www.amazon.com/Game-Development-Patterns-Best-Practices-ebook/dp/B01MRP7SPA) in 2016. I work at Meta now. I have spent more hours than any human should explaining the difference between Strategy and State to a room of twenty-year-olds. So when I tell you those differences mostly evaporate, it isn't because I never learned them. It's because I learned them and kept going.

I just wrote the [polite version of this](/2026/06/19/house-rules-two-drink-minimum/) — the House Rules post on the Strategy pattern. Straight, sober (mostly), careful, here's-how-it-differs-from-State. Every word of it is true. This is the post where I tell you what I actually think.

There is no spoon.

There are no patterns.

## Actually...There's just one pattern

You remember the kid in *The Matrix*.

Bald.

Robes.

Bending a spoon with his fuckin mind.

 *Do not try and bend the spoon — that's impossible. Instead, only try to realize the truth: there is no spoon. Then you'll see, it is not the spoon that bends, it is only yourself.*

That's design patterns.

**Don't sit down and reach for the Strategy pattern.**

Don't stare at your code and ask "do I need Strategy here, or State, or a Factory?"

That's trying to bend the spoon.

Realize the truth instead: *everything changes.* You put a thing behind an interface because the thing behind it is going to move — swap, grow, get a second version you can't see yet. Someone walks by later, points, and says "ah — Strategy." You didn't reach for a pattern. You planned for change, and the pattern is just the name somebody stuck on what that looked like.

So there's really just the one. Put a seam where the change is coming, and program to it. That's the pattern — and everything else in the catalog is that same move wearing a different name.

## So what *is* the difference between Strategy and State?

This is the question I'd make every student sit with until it hurt.

Structurally? Nothing. Same class diagram. An interface, a few implementations, a thing that holds one of them. The *only* difference is who pulls the swap — you, from outside (Strategy), or the object, from inside, on a transition (State). That's the entire distinction the textbook spends two chapters on. One bit of information: who flips the switch.

And once you see that, you see the bigger thing. **Strategy is everything.** Unless you've got exactly one thing that will never, ever change — and be honest, in a real codebase, *does that ever actually happen?* — then everything is a strategy that might need to change. Every interface you write is a bet that the thing behind it moves. And even the thing you swear *won't* move — you still want it behind an interface and injected in, because the day you write a test you'll need to swap a fake in its place. "Never changes" and "never needs mocking" are different promises, and the second one comes due the first time you try to test around it. You're not *choosing* Strategy. You're admitting the code is alive.

## A Factory is just how you avoid the switch

Same trick, one level up. What's a Factory, really? It's how you get your strategy without hardcoding a `switch` statement in forty different places. That's the job. Avoid the switch — or, more honestly, *don't paste it across the codebase* so that adding one case turns into a treasure hunt through every file that ever cared.

And half the time you don't even want a switch. If you've got an enum keying which thing to build, reach for a map — enum to object — instead of a `switch` you'll forget to update in three of the four spots. (Fine: if it's four cases and it's *always* going to be four, a plain `switch` or enum is fine. Don't gold-plate it.) The pattern police will call that the Factory pattern. It's a hash map and a good habit.

Notice what just happened: Factory turned out to be "a tidy way to hand you a Strategy." They're not two ideas. They're the same idea, and one of them carries the other to the table.

## The standard library knew before you were born

Open the C++ STL. You've got a `list`, a `vector`, an `array` — genuinely different animals at the low level, different costs, different invalidation rules, different everything. And the library hides all of it behind **iterators**, so `sort`, `find`, `for_each` don't care which container they're walking. The container is a strategy. The algorithm is a strategy. The iterator is the seam. The whole STL is this one idea, industrialized — in a language that didn't even *have* an `interface` keyword. It did it with templates and got the same result.

When you're nineteen, linked-list-versus-array is a midterm you can fail. Ten years in, it's "a container, I'll iterate it." Same idea, lower stakes. The wall turned out to be a curtain.

## Even the hardware does it

Look at the [shaders from the polite post](/2026/06/19/house-rules-two-drink-minimum/), one level deeper. The interface between my renderer and a shader is a Swift protocol — fine, you can see the keyword. Now drop below it. The interface between the shader and the *hardware* isn't a protocol at all. A vertex shader's contract is "take vertices, output a position." A fragment shader's is "output a color." That contract is enforced by the GPU pipeline — by silicon — with no `interface`, no class, no inheritance anywhere near it. Still the same move: swappable behavior behind a contract, picked from outside, and the thing running it has no idea which one it got.

The spoon bends at the hardware. It bends everywhere, because there was never a spoon — just behavior on the far side of a seam, all the way down to the metal.

## And the languages, while we're here

C++ and Python feel like different planets when you start. Different syntax, different rules, one yells about semicolons and the other about whitespace. Ten years in they're the same handful of ideas in different costumes, and the only fork that genuinely matters is *do I free the memory or does the runtime do it for me.* Everything else is which side of the comma the type goes on. Patterns are that same joke — a pile of names for "behavior behind a seam," plus or minus who pulls the trigger.

## Where this advice puts you in a ditch

Because it's still *bad* advice. It's true 90% of the time and a cliff the other 10%, which is the shape of every dose in this series.

The first way it bites: you hear "memorize nothing" as "learn nothing." You can't. **You don't get to skip the names.** The reason I can tell you Strategy and State are the same shape is that I built both, the long way, with the full ceremony, enough times that the ceremony fell off on its own. The veteran who says "it's all one idea" earned that sentence. The junior who says it is just skipping the reading. *Same words, opposite meaning.* Study the twenty-three. Then earn the right to stop counting them.

The second way: the abstraction is a beautiful lie right up until it leaks — and the day it leaks, the difference you waved off as "just a costume" is the only thing in the room. You picked `list` because *it's just a container* and now you're doing random access in a hot loop, the cache is thrashing, the frame budget's gone, and linked-list-versus-array is the entire bug. You called State and Strategy the same thing, and they are — until the night your object needs to transition *itself* and you built it to wait for a caller that's never coming. The one real difference is exactly the thing that bit you: who pulls the swap.

## Last call

So study them all. Don't memorize them — internalize the one thing every last one of them is actually teaching: **your code is going to change, and your job is to be ready for it.** Abstract it. Put it behind an interface. Don't hardcode the same switch in five places. Think one level above the name.

Do that long enough and the names stop mattering. You stop asking "is this Strategy or State." You see change coming and you build the seam, and you let someone else name it later.

There is no spoon. There's a seam, and a thing on the far side that's going to change — because it always changes. Everything is a strategy for handling that. The rest is vocabulary.

Realize the truth, and you were never choosing a pattern at all.

*There is no spoon.*
