---
layout: post
title: "Syntax Is the Least of Our Skills"
tags: [ai, claude-code, architecture, software-engineering, flux]
---

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Syntax is the least of our skills, and one that we will be well rid of. Without it we'll be able to focus on the more important aspects of software like function and module structure, testing and test coverage, component cohesion, coupling, and collaboration, and architecture.</p>&mdash; Uncle Bob Martin (@unclebobmartin) <a href="https://twitter.com/unclebobmartin/status/2042938176021393645?ref_src=twsrc%5Etfw">April 11, 2026</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Uncle Bob is right. And I have receipts.

## The Skill That Actually Matters

Experienced engineers don't get paid more because they type faster. They get paid more because they can architect systems — define module boundaries, manage coupling and cohesion, make decisions about dependency injection, know when to build versus reuse.

This has always been true. When you're starting out, syntax is a hurdle — it's hard to think about architecture when you're still fighting the language. But as you gain experience, syntax becomes a toolbox. You stop thinking about where the semicolons go in a for loop and start thinking "I have a tool to iterate." You stop thinking about if/else branches and start thinking "I have a tool to make decisions." And then you gain more experience and you stop thinking about the tools at all — you think in abstractions. Functions, modules, interfaces, boundaries. The for loops and if statements are just automatic. You jump between languages because at that level they're all the same — the syntax is different, but the patterns, the architecture, the problem-solving? That transfers.

I've been programming for 20 years across more languages than I can count. This is nothing to brag about — lots of people have way more experience than me. I taught C++ for seven years. Now I'm working in Rust. The semicolons move around. The ownership models have different names. But the questions are the same: where do the module boundaries go? How do services get passed around? What's the testing strategy? Is this coupling going to hurt us in six months?

That's what AI is accelerating. Not the thinking — the typing.

## What I'm Actually Seeing

In my free time, I'm running three personal projects right now with Claude Code, and each one proves this differently.

### Mobile Apps — Where Domain Knowledge Pays Off Most

I'm goofing off building some mobile apps — they're not really for anything, just side projects. But I've been doing mobile development since 2010 — my first shipped iOS game was [Sync-Ball](https://apps.apple.com/ca/app/sync-ball/id356277312), then Android starting around 2014 just before Android Studio launched. I know these platforms deeply. The classes, the patterns, the gotchas.

When I started a new mobile project recently, I spent a full day just on the plan. Not writing code — explaining architecture. How I wanted the project organized. Why it needed to be modular from the start. Specifics like image caching strategies, using interfaces for service boundaries, whether to use Dagger/Hilt for dependency injection or roll our own (we rolled our own — it's actually easy for what we needed).

Then I had Claude generate the scaffolding. The code generation was fast. But here's the thing — Claude still made design choices I wouldn't have. The structure wasn't quite modular enough. Some service boundaries were wrong.

One example: Claude put the logging service in the app target instead of the shared library. When I asked why, it made a reasonable argument — what if the TV app wants a different logger? Then the library would have an unnecessary dependency. But I know from experience that the trade-off goes the other way. These apps will almost certainly use the same logger. If the service lives in each app target, I have to write it twice. And when I find a bug in the iOS version, I have to remember to fix it in the TV version too. And any future targets. I'd rather the library have one extra dependency it might not need than duplicate a service across every target. That's a judgment call Claude can't make — it doesn't know which future pain is worse.

The code generation saved me days. But it was the planning and review — the architecture work — that made the result actually good. Without that, I'd have a working app built on a foundation that would fight me on every future feature.

### Flux Terminal — Learning a New Domain in Real Time

[Flux](https://github.com/mattcasanova/flux) is the opposite scenario. I have high-level experience building apps and games, and years of C++ — but I've never written Rust and I've never built a terminal emulator.

The first code Claude generated worked. But it wasn't great. It built in a 60fps rendering loop — which you need in a game engine but absolutely don't need in a terminal. I caught that and had it stripped out. Then I noticed macOS and Linux-specific code baked directly into the main app class. I had to flag it: *no, this needs to be dynamic based on the operating system. It's going to be a pain to rip this out later if we don't define our platform boundaries now.*

Those are architectural calls. Claude doesn't make them — I do, because I've spent years learning where boundaries need to go. The language is new but the skill isn't.

And here's what's interesting: I run two Claude instances in parallel. One is writing code for the terminal. The other is my learning session — I go through almost line by line asking "what does this do? Why is it like this?" I learned about Rust's `Box<T>` (the equivalent of C++'s `unique_ptr`), how it moves data from stack to heap (a shallow `memcpy`), and the potential gotchas that could bring. I immediately flagged those gotchas — because moving heap-owning data is exactly the kind of thing that creates dangling references in C++. But in Rust, the compiler enforces that you can't have dangling references after a move. I understood this immediately, because the thing I always taught my students was: *prefer compile time errors to runtime errors.* Rust takes that principle and builds the entire language around it.

I keep these separate deliberately. I don't want to pollute the coding context with my learning questions, and I don't want to pollute my learning with implementation details.

### LiquidMetal2D — The Velocity Proof

[LiquidMetal2D](https://github.com/mattCasanova/LiquidMetal2D) is an open-source 2D game engine I built in Swift + Metal back in 2020. I hadn't touched it in five years. This year I revived it — I can work on it in parallel with the other projects, and it's just a fun thing to do.

None of the improvements are groundbreaking — spatial grid collision, a composition-based component system, renderer API cleanup. These are basic patterns in 2026. I just never put them in before. But I knew I wanted them, and now I can spend 10-15 minutes explaining how I want a system designed and get a first draft of code in minutes. Stuff that would have taken me days or weeks — I'm getting working implementations I can review and tweak almost immediately.

And that velocity is what gave me the confidence to start the terminal project. Before this, "build a GPU terminal emulator" would have sounded too ambitious for a side project.

## The Part People Get Wrong

There's this fear that AI is going to replace developers. I don't see it. Here's what I actually see:

**Anyone can get something working.** Someone who's never coded before can probably use AI to build a basic app. It'll run. It might even look good. But the question is the same question it's always been: *was it built right from the start?*

Because features compound. Scale compounds. Technical debt compounds. And the problems that emerge at scale — the coupling that makes a feature impossible to add, the missing abstraction that forces you to rewrite a module, the dependency that should have been injected but wasn't — those are the problems that require experience to solve. Or even to see coming.

Claude can help debug. It's actually good at it. But someone with years of experience can look at the code, get a hint from the error, and suggest a direction that Claude alone wouldn't find. That's the collaboration: human experience sets the direction, AI handles the execution.

## What AI Actually Changes

AI isn't replacing experienced engineers. It's making them more dangerous. And more valuable.

Here's the secret I used to tell my students: **everybody Googles.** Every semester I'd have a day — not a formal lecture, just a day — on "How to Google Shit." There seems to be this hidden knowledge that experienced developers have, and people learning think it's magic. It's not. If you haven't touched C++ in a while, you're not going to remember every piece of syntax. If you haven't used a specific Metal API recently, you're going to Google it. That's what everyone does. That's what everyone has always done. Before Google, you looked it up in books. Google made that faster. Now AI is making it faster again.

I remember my first real mobile app job — not games, an existing production app. I needed to fix some major bugs in our SOAP API, which I'd never touched because I came from a game development background. So I Googled it. I found three different Stack Overflow answers that were each roughly pointing toward the bug fix I needed. None of them were exactly my problem — they never are. But I was able to pull from all three, copy the relevant pieces, and massage the code until it did what I wanted.

That takes experience. Knowing which three answers to pull from, knowing how to combine them, knowing when the result is right — that's not something a beginner can do. But the mechanical part — finding the code, copying it, adapting the syntax — that was always just overhead.

AI is doing that same thing. It's just faster. It gives me a roughly working first draft that I can then refactor and massage. The same skill I used piecing together Stack Overflow answers in 2017 is the same skill I use reviewing Claude's output in 2026. The manual lookup step just got automated. And good riddance — an experienced engineer was never paid for that part anyway.

But I want to be clear: this isn't a replacement for not knowing stuff. That's the whole point of this post. You have to know things. You can't just let AI generate code you don't understand and hope it works out. That's why I'm learning Rust syntax in parallel with building the terminal — because at some point, if I can't read and reason about the code myself, the project will fall apart.

What's left is the stuff that was always the hard part: architecture, system design, knowing where the boundaries should go, knowing which abstractions will hold up under scale. The stuff Uncle Bob is talking about.

Syntax is the least of our skills. It always was. Now we can finally stop pretending otherwise.
