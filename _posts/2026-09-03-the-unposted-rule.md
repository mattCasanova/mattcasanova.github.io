---
layout: post
title: "The Unposted Rule"
tags: [career, product, ai, opinion]
summary: "Twenty years of loving the technical side, and the ladder was quietly measuring something else. The path runs from engineer to product, it always has, and nobody says it out loud. This is me saying it out loud."
---

I've been at Meta for almost six years, and the whole time I've been kicking around the question every engineer eventually kicks around: what actually gets somebody to the next level?

I've always had my answer ready: be more technical. Architecture has been my thing for twenty years, and I still believe it's fundamental. [The last post](/2026/08/14/tip-the-kitchen/) was a couple thousand words of me believing it.

The realization I've come to lately is simpler and worse: I may have been wrong. Not about the architecture. About it being the answer.

Here's the observation, and it's an observation, not a complaint. Last post I said the customer product gets the love and the code product keeps the lights on. The part I didn't say: the promotions follow the love. I know I do important work. My managers know I do important work. But there's no sexiness to it; keeping the lights on isn't sexy. Project impact is the currency, and keeping the lights on doesn't mint much of it. It gets filed somewhere else, under a heading like better engineering.

Real category. Smaller currency.

Nobody talks about the lights until they're off.

And nobody's wrong here; I'm not knocking anyone. Features are the job. But growing as an engineer turns out to mean something nobody ever put on a ladder doc: you can't just be technical. You have to be product-focused too.

I used to think my half of this was a communication problem. Write a tighter doc, tell a better story. Now I think I was missing a rule. Every bar posts its house rules where you can read them. The rules that actually run the place never get posted; the regulars just know. This post is about the one that took me twenty years to read.

I could be wrong about it; that's part of the story. But here it is:

**The path runs from engineer to product, and the ladder has been bending that way the whole time.**

Nobody told me.

Sometimes I am just slow.

### Team Carmack

At DigiPen we read *Masters of Doom*, and I connected with Carmack more than Romero from the first chapter. Building the engine that drove the game always seemed more interesting than building the levels. Twenty years later it still shows: I build game engines more than I build games. I don't have a strong desire to ship a game. But if I can add more particles to a smoke effect and hold 60 frames a second?

That's the good shit.

I was always the engine guy. Freshman year we wrote a text-based game, and I could feel something wrong with my "engine" before I had words for it: every room in the dungeon hard-coded, wired together with function calls. Then it clicked that if the engine just read text files, the dungeon was effectively infinite, no recompile. That hit me harder than any level design ever did. So I went deep on the unglamorous layers: memory management, message systems, the glue that makes everything else run smooth. Plenty of classmates wanted graphics or physics.

Almost nobody wanted the glue.

Somebody in school once asked the room whether we'd be okay working on something other than games. It was a whole series of questions like that: accounting software, flight navigation systems, whatever.

My hand was up the whole time. I didn't care what the product was if the problems were cool. Which also explains the seven years of teaching: a job that was literally studying technical stuff all day and then talking about it. (Look at me now.)

And in fairness to younger me, technical depth *was* the differentiator. It's why the interviews at the big companies are brutal. Deep skill got you the better job, and it happened to be the thing I loved. Nothing about that felt like a wrong turn.

It carried straight into industry, too. [My first non-teaching job](/2026/05/06/bad-advice-prove-them-wrong/) was business apps at a registered-agent company in Vegas, and in my first weeks I found every API call in the mobile app hand-built as an XML string, pasted inline into each screen. I didn't know SOAP from REST yet. Didn't matter; I could see the shape was wrong. I pulled it into a reusable network layer and fixed a bug on the way through. Later, when we moved to Ionic, I wrote the scripts that stamped out multiple client-branded apps from one codebase.

Notice the division of labor in that story, because I didn't. Designers and owners decided what to build. I made building it cheap. I thought the technical half was the career.

It was a job description.

### The Roll Call

This year I've been reading the product shelf, for reasons I'll get to, and a pattern showed up that I cannot unsee. I gave it one line in the last post. It deserves the full roll call.

Ben Horowitz, whose *The Hard Thing About Hard Things* I've reread more than any business book I own: engineer at Silicon Graphics, then a product manager at Netscape, then CEO of Loudcloud, then Opsware, then a16z. Marc Andreessen wrote Mosaic before he ran anything. Gates wrote the BASIC. Zuckerberg wrote the first Facebook himself. Boz wrote News Feed years before he was a CTO, and a CTO at that altitude isn't grinding code; he's steering product-shaped decisions all day.

Every technical founder you admire made the same walk. Engineer first. Product forever after. And the reason is structural, not stylistic: a company steers by product decisions, and influence follows the steering wheel.

It's not just the famous ones. The owner of that company in Vegas knew how to program; that's how the company existed at all. During a presentation once, they told me flat out that they don't care about the code. That's why they were paying me. And they were right to say it, which is the uncomfortable part. Caring about the code is the job you hire.

Deciding the product is the chair you keep.

### Same Gravity

The obvious out: that's founders and managers. Stay on the IC track, stay technical, problem solved.

Except look closer, because I finally did. My engineering managers, everywhere I've worked, big companies and small, do not give technical lectures. Not day to day, not week to week. They talk product, priorities, dates. In retrospect it's so obvious it's embarrassing: the transition into engineering management is already the transition into a product job. It just keeps the old costume.

And above senior on the IC side, same gravity. When I mentor people, the advice that actually moves them is never "know your data structures better." It's initiative: see the bug nobody's going to fix, and fix it. It's planning: every half, when there is no map, draw the map. A staff engineer once told me directly that the job is influence and direction. The principals I've watched are genuinely technical, but the technical is the entry fee, not the differentiator.

There's a structural reason here too, and it isn't a conspiracy: genuinely hard engineering problems are rare. Most of the job, honestly counted, is building APIs, building UI, or both, on top of an architecture that already exists. A career ladder can't pay out on rare events, so the everyday currency above senior becomes judgment, planning, direction. Product-shaped skills. On both branches.

I've felt the exchange rate personally. Last year, the people shipping a shiny new feature on a flagship app got talked about everywhere; the behind-the-scenes work I was proudest of was, for lack of a better term, keeping the lights on. I fix the critical bugs. I write the technical docs: here's the class of failure, here's the path forward. When nobody reads them I shrink them, then shrink them again. It doesn't sell, and I no longer think that's mainly a writing problem. Counterfactual wins don't trend.

Twenty years focused on the technical, and the pattern was invisible until it wasn't. Maybe everyone else already knew. Maybe it was only unspoken to me, because I was busy studying the thing I loved and not the game around it.

Either way, the rule was readable the whole time. I just wasn't reading it.

### The Barrier and the Bar

Here's what turned an observation into a plan.

AI made everyone's job harder. I mean that precisely: the barrier to entry has never been lower, and the bar of quality has never been higher. A product person with zero technical skill can now have Claude build them an iPhone app. No mockup tools, no contractors, just describe it. It's an open question whether that app survives three years or three weeks before the code rots out from under it. But still... There is a flood of shitty apps coming, built by people who know exactly what to build and not at all how.

So which side does Claude conquer first, code or product?

Code. Obviously code. [The typing is already gone](/2026/04/11/syntax-is-the-least-of-our-skills/). Architecture is where an engineer's judgment still earns its keep, and I think the way my apps are built is close to the best-practices version of everything I know. But let me be honest about what that knowledge is: not genius, not secrets. Years of experience that I could write down in twenty blog posts. Some of it I already have. And anything that can be written down, the machine reads. It's reading my repos right now, and everyone else's.

Which leaves the question I keep turning over: is it easier to teach an engineer product, or a product manager the deep end of engineering? My bet is engineer to product, the shorter walk. I'm aware I would think that. I'm blind on exactly one side of that comparison, and that's fair.

It's still the bet I'm placing.

### The Experiment

First, the disclaimer, and it's not a throwaway: this whole post is a guess.

I've never been fired. But I've watched people be completely confident they got fired for the wrong reason, and the useful move was never the confidence. It was going back to look at the firing anyway, in case there was something in it to learn. Same discipline here. Maybe the ladder is right in a way I can't see yet. Maybe I'm not as technical as I think I am. Maybe I'm a worse communicator than I want to admit (almost certainly), and the docs don't sell because the docs aren't good. Maybe none of this has anything to do with product focus at all.

Every one of those could be true.

Actually, they're all probably true at varying degrees.

Last post I joked that I've personally been very smart in rooms where the second product lost anyway. There's a version of this story where the room was right.

So the trajectory isn't a verdict. It's the experiment I'm running on myself, out loud, and I'll report the results here either way.

Which brings me to what I'm actually doing with the early mornings, late nights and the margins.

I'm reading the product shelf for real: product management, marketing, how to think about a customer instead of a codebase. Arguing with *Escaping the Build Trap* on my runs, [and mostly losing](/2026/08/14/tip-the-kitchen/). My wife and I are building something we intend to sell to actual customers, which makes me the de facto product manager of a real product for the first time in my life, and I am finding out in real time how much I don't know. Nobody hands you this skill. You go get it.

I want to be precise about the landing, because it would be easy to read this as a conversion story. It isn't. I spent the last post yelling that the code is a product too, and I meant every word. The kitchen still matters; with the machine writing code by the pound, it may matter more than ever. The move isn't renouncing the technical side.

The move is refusing the fake choice. Own both products.

The trajectory is real. It was always real. Horowitz walked it. Andreessen walked it. The owner in Vegas walked it. Every engineering manager I've ever had was standing on it while we talked about deadlines. It's the rule that runs the place, and it isn't posted anywhere.

Twenty years in, I'm walking it on purpose. Eyes open.

The rule was never posted. Consider it posted.

See you, space cowboy.
