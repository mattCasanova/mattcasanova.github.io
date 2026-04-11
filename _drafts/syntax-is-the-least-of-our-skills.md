---
layout: post
title: "Syntax Is the Least of Our Skills"
tags: [ai, claude-code, architecture, software-engineering, flux]
---

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Syntax is the least of our skills, and one that we will be well rid of. Without it we'll be able to focus on the more important aspects of software like function and module structure, testing and test coverage, component cohesion, coupling, and collaboration, and architecture.</p>&mdash; Uncle Bob Martin (@unclebobmartin) <a href="https://twitter.com/unclebobmartin/status/2042938176021393645?ref_src=twsrc%5Etfw">April 11, 2026</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Uncle Bob is right. And I have receipts.

## The Skill That Actually Matters

Senior engineers and staff engineers don't get paid more because they type faster. They get paid more because they can architect systems — define module boundaries, manage coupling and cohesion, make decisions about dependency injection, know when to build versus reuse.

This has always been the hierarchy. A junior engineer fresh out of college might struggle with syntax, which makes it hard to even think about architecture. But as you gain experience, syntax fades into the background. You jump between languages because they're all fundamentally the same — the syntax is slightly different, but the patterns, the architecture, the problem-solving? That transfers.

I've written C, C++, Swift, Kotlin, JavaScript, and now Rust. The semicolons move around. The ownership models have different names. But the questions are the same: where do the module boundaries go? How do services get passed around? What's the testing strategy? Is this coupling going to hurt us in six months?

That's what AI is accelerating. Not the thinking — the typing.

## What I'm Actually Seeing

I'm running three projects right now with Claude Code, and each one proves this differently.

### Mobile Apps — Where Domain Knowledge Pays Off Most

I've been doing mobile development since 2011 — first iOS game, then Android starting around 2014 just before Android Studio launched. I know these platforms deeply. The classes, the patterns, the gotchas.

When I started a new mobile project recently, I spent a full day just on the plan. Not writing code — explaining architecture. How I wanted the project organized. Why it needed to be modular from the start. Specifics like image caching strategies, using interfaces for service boundaries, whether to use Dagger/Hilt for dependency injection or roll our own (we rolled our own — it's actually easy for what we needed).

Then I had Claude generate the scaffolding. The code generation was fast. But here's the thing — Claude still made design choices I wouldn't have. The structure wasn't quite modular enough. Some service boundaries were wrong. So I went through and said: *this needs to be fixed, this needs to be fixed, this needs to be fixed.*

The code generation saved me days. But it was the planning and review — the architecture work — that made the result actually good. Without that, I'd have a working app built on a foundation that would fight me on every future feature.

### Flux Terminal — Learning a New Domain in Real Time

[Flux](https://github.com/mattcasanova/flux) is the opposite scenario. I have high-level experience building apps and games, and years of C++ — but I've never written Rust and I've never built a terminal emulator.

The first code Claude generated worked. But it wasn't great. It built in a 60fps rendering loop — which you need in a game engine but absolutely don't need in a terminal. I caught that and had it stripped out. Then I noticed macOS and Linux-specific code baked directly into the main app class. I had to flag it: *no, this needs to be dynamic based on the operating system. It's going to be a pain to rip this out later if we don't define our platform boundaries now.*

Those are architectural calls. Claude doesn't make them — I do, because I've spent years learning where boundaries need to go. The language is new but the skill isn't.

And here's what's interesting: I run two Claude instances in parallel. One is writing code for the terminal. The other is my learning session — I go through almost line by line asking "what does this do? Why is it like this?" I learned about Rust's `Box<T>` (the equivalent of C++'s `unique_ptr`), how moving from stack to heap is a `memcpy`, and how the compiler enforces that you can't have dangling references after a move. Edge cases I would have asked about in C++ too — Rust just handles them at compile time instead of letting you segfault at runtime.

I keep these separate deliberately. I don't want to pollute the coding context with my learning questions, and I don't want to pollute my learning with implementation details.

### LiquidMetal2D — The Velocity Proof

My 2D game engine in Swift + Metal is the project where this all comes together, because I have deep domain expertise in both the language and the problem space.

I hadn't touched it in five years. In one month with Claude Code, I shipped [25+ improvements](https://github.com/users/mattCasanova/projects/3): spatial grid collision detection, a composition-based component system, renderer API overhaul, a stress test running 7,000 objects at 30fps on a phone.

That's the real time savings — when you already know the architecture you want and the language you're working in, AI handles the manual part at a speed that would have been unthinkable before. Work that would have taken months of weekend sessions took weeks.

And this velocity is exactly what gave me the confidence to start the terminal project. A month ago, "build a GPU terminal emulator" would have sounded too ambitious for a side project. After shipping 25 tasks in 30 days, ambitious just means more nights.

## The Part People Get Wrong

There's this fear that AI is going to replace developers. I don't see it. Here's what I actually see:

**Anyone can get something working.** Someone who's never coded before can probably use AI to build a basic app. It'll run. It might even look good. But the question is the same question it's always been: *was it built right from the start?*

Because features compound. Scale compounds. Technical debt compounds. And the problems that emerge at scale — the coupling that makes a feature impossible to add, the missing abstraction that forces you to rewrite a module, the dependency that should have been injected but wasn't — those are the problems that require experience to solve. Or even to see coming.

Claude can help debug. It's actually good at it. But someone with years of experience can look at the code, get a hint from the error, and suggest a direction that Claude alone wouldn't find. That's the collaboration: human experience sets the direction, AI handles the execution.

## What AI Actually Changes

AI isn't replacing the senior engineer. It's making the senior engineer more dangerous.

The manual skills — remembering syntax, looking up library APIs, writing boilerplate, converting between languages — those are getting automated. And good riddance. A senior engineer was never paid for those skills anyway.

What's left is the stuff that was always the hard part: architecture, system design, knowing where the boundaries should go, knowing which abstractions will hold up under scale. The stuff Uncle Bob is talking about.

Syntax is the least of our skills. It always was. Now we can finally stop pretending otherwise.
