---
layout: post
title: "Signal Through the Noise"
date: 2026-04-11
tags: [ai, software-engineering, career, opinion]
---

People are resistant to change. This isn't new.

In 1996, Andy Grove wrote *Only the Paranoid Survive* and wondered what this internet thing was going to become. Maybe someday we'd do banking online. He wasn't against it — he was one of the most forward-thinking CEOs in tech history — but even he wasn't sure. And plenty of people around him were sure it was a fad, or a security nightmare, or both.

Then the web became everything.

I'm sure there were people who said web programmers weren't real programmers. You had to be an application developer on Windows or Linux to be taken seriously. But now web programming is the dominant trade. It drives most of the software industry.

The same thing happened with mobile apps. "Oh, another app." "That's a cutesy little thing, it's not challenging." And sure, apps should lean more toward thin clients — there's nuance there. But now everyone has a smartphone. Apps are everywhere. Mobile development is a serious career.

The same thing happened with Swift vs Objective-C. I worked with a guy who hated Swift when it came out. And honestly, the early versions were rough. But Swift was better, and now it's the standard. I don't even know what that guy's doing anymore.

The same thing happened with Photoshop. Photographers and artists said it wasn't real photography, wasn't real art. Now it's the baseline tool for the entire industry.

Every time a new level of abstraction arrives, there's resistance. And every time, the people who embrace it end up ahead.

AI is the next one.

## Yes, AI Produces Slop

I'm not going to pretend AI is perfect. It produces slop code. It produces slop art — too many fingers, weird architecture decisions, tests that don't test anything. I've seen it firsthand.

And there are real concerns. I have a friend from high school who's a skilled artist, and he hates AI. I get it. AI art is a genuine threat to artists' livelihoods. A lot of people can't afford to pay for a skilled artist, and now they have an alternative that's "good enough." That's a real loss.

But here's the thing — what that artist actually gets paid for isn't brushstrokes. It's taste, experience, and judgment. The same things Photoshop didn't replace. The same things the camera didn't replace before that.

The tools change. The expertise doesn't go away. It gets applied differently.

## Two Scenarios

Here's how I think about it.

**Scenario 1: You embrace it.** You're an expert compiler developer, or a game engine architect, or a terminal emulator builder. You already know the domain deeply. Now you have a tool that handles the manual parts — syntax lookup, boilerplate, API discovery — at 10x speed. Your expertise becomes *more* valuable, not less, because you can apply it faster. You become the 10x engineer that everyone talks about but nobody could actually be before, because the bottleneck was always typing, not thinking.

**Scenario 2: You don't.** You decide you're already good enough. You don't need this. Maybe you're right — today. But somebody is going to come along who is motivated, who is learning, and who is using these tools. They might not be as expert as you. But their ramp-up speed is going to be insane. They'll use AI to learn the domain (like I'm doing with Rust and terminal emulation right now) while simultaneously building real things. And eventually, they'll catch up. And then pass you.

Your bet, if you don't embrace it, is that these tools will never be good enough to matter. That's a risky bet. Nobody writes punch cards anymore. Hardly anyone writes assembly, because compilers write better assembly than humans can. C and C++ are still important for low-level performance work, but Swift is the right tool for iOS, and nobody apologizes for that.

Every generation of tooling raises the floor. The people who thrive are the ones who stay above it.

## I Was Skeptical Too

I remember at the beginning of 2025, Zuckerberg said something about AI coding at the level of a mid-level engineer by the end of the year. I'd been trying to use our internal AI tools at work — they made mistakes, didn't write tests well, produced code I had to heavily edit. My coworkers and I were skeptical. That's a bold prediction. At the time, AI was generating a lot of noise but not much signal.

By September 2025, it was better but still not great. I was using it because I had a lot to get done fast, but it was hit-or-miss. More hit than before, but still enough misses to be frustrating.

Then around February 2026, I started using Claude Code. And I was blown away. Zuckerberg's prediction was off by a month or two.

Part of it was timing — I had just switched teams at Meta after five years. New team, new codebase, different language than what I was used to. I *had* to lean into AI because I needed to get up to speed fast. So I used it to understand the codebase, understand the language nuances, understand why the team made certain architectural decisions. And then I started using it to ship actual work.

I was forced to embrace it. And it turned out to be the best thing that could have happened.

That's the same thing I talked about in my [last post](/2026/04/11/syntax-is-the-least-of-our-skills/) — I use AI to understand, to learn, to ramp up. And then I use my experience to architect, to review, to make the judgment calls that AI gets wrong. It's not one or the other. It's both.

## The Bar Has Been Raised

Here's the thing people miss: AI hasn't lowered the bar for expertise. It's raised it.

The bar to *look like* an expert has never been easier to reach. Anyone can generate a working app. Anyone can generate a website. Anyone can produce code that compiles and runs. The floor has come up dramatically.

Think about it this way. When Facebook launched, the novelty was the idea. But after Facebook existed, anyone could rebuild it — even before AI. The implementation wasn't the hard part. The insight was. Now with AI, it's the same dynamic accelerated to everything. Anyone can build a game. Anyone can build a Mario clone. If you could just type "make me a Halo clone with dinosaurs" and get a working game, then cool — but if *everyone* can do that, then Halo with dinosaurs isn't interesting. Neither is Halo with vampires, or Halo with robots. Nobody cares. It stops being novel.

What *is* interesting is what comes next. The people with taste and judgment who look at these tools and say: now I can build something that wasn't possible before. Now I can push the boundaries. An open-source GPU-rendered terminal with block-based output? That's not a one-sentence prompt. That's architecture, domain knowledge, and a thousand judgment calls about where the module boundaries go. (To be fair, nobody said I had taste — I certainly wouldn't give myself that title. But I'm trying.)

The bar has been raised because we can actually do more. If everyone can build an app, what does it mean to build a *great* app? If everyone can write code, what does it mean to write code that scales, that's maintainable, that doesn't fall apart when you add the fifth feature?

The answer is the same thing it's always been: taste, judgment, experience. The stuff you can't generate. The stuff that comes from years of building things, breaking things, and learning why they broke.

AI slop is real, and there's going to be more of it. But as the expert, you can rise above it. Your experience is the filter. Your taste is the differentiator. And now you have tools that let you apply that taste at a speed that would have been impossible before.

## Just Use It

Companies are pushing their engineers to adopt AI tools, and that's the right call — even if they're sometimes measuring the wrong things. Token usage is not the metric. Lines generated is not the metric. The metric is: are you shipping better work faster?

But the push is right. New things are always uncomfortable. There's always a learning curve. There's always a period where the old way feels more productive because you already know it. That's not a reason to stay.

If your skills are actually valuable — and they are — then these tools should make you five or ten times more valuable. Not by replacing what you know, but by removing the friction between what you know and what you can build.

If you don't use them, someone who does will eventually build what you could have built, in a fraction of the time.

The noise is getting louder. There's more AI slop every day — more generated apps, more generated art, more generated everything. That's not going to stop.

But the signal still matters. Your taste, your judgment, your experience — that's the signal. And now you have tools that amplify it.

Be the signal.
