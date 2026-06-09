---
layout: post
title: '"The Code Is Bad" Is Not an Argument'
tags: [architecture, opinion, tech-debt, career]
---

Every engineer has said it, and every engineer has watched it die in the air: *"the code is bad."*

You say it in a planning meeting. You mean it with your whole chest. And the stakeholder nods, says "noted," and asks why the feature isn't out yet. You walk away convinced they don't get it. They walk away convinced you're stalling. Both of you are a little bit right, and the codebase gets a little bit worse, and next quarter you have the same meeting again.

Here's the thing nobody put on the syllabus: **"the code is bad" is not an argument.** It's a feeling. It might be a *correct* feeling — you might be the smartest person in the building about this exact thing — but a feeling doesn't move a roadmap. And the gap between "I am right" and "I got the resources to fix it" is a skill. It's *our* skill, and most of us are terrible at it, myself absolutely included.

I've had a version of this fight at every job I've ever held. Every single one. The lazy version of this post blames the suits; the other lazy version blames the engineers. Both are wrong, and neither one helps you on Monday. The useful version is about the one lever that's actually yours — but to get there, we have to start with a dumber question: who's even supposed to care?

## It can't just be us

Let's get the obvious out of the way. The customer does not care about your code. Of course they don't. They downloaded the app to control their lights or split a bill or watch a video. They have never once thought about your test coverage, and they never will, and that is *correct*. A customer who cared about your architecture would be a very strange customer.

So when I say the code matters, I'm not saying everyone should care. I'm saying it can't be **only** the programmers.

The people in the middle — call them stakeholders, that's the word my old teams used — sit between the customer who can't see the code and the engineer who can't stop seeing it. It is not their job to maintain it. That's literally why they hired you. But it *is* their job to care when an engineer tells them it's rotting, the same way it's not the CFO's job to fix the truck but it's absolutely their job to care when the mechanic says the fleet won't last the winter.

The problem is the mechanic says "the trucks are bad" and hands over an itemized estimate. We say "the code is bad" and hand over a vibe.

## Say it in money

Watch what happens when you translate. Same complaint, two languages:

- *"The code is bad"* → **"The next feature takes a quarter instead of a week, and shipping it breaks three things we already sold."**
- *"We really need to refactor"* → **"We're paying interest on this every sprint, and the payment is going up."**
- *"It's not testable"* → **"We can't prove a change is safe without a human clicking through the whole app, so every release is a coin flip we do on Fridays."**

Nobody can act on the left column. Everybody can act on the right. The right column isn't dumbed down — it's *translated*, out of engineer-ese and into the only dialect a roadmap actually speaks: time, risk, money.

And the cleanest translation we've got — the one that turns an invisible engineering cost into something a non-engineer can read off a page — is the oldest one in the book. It's right there in the name.

## Beer on credit

We call it tech *debt* because it is debt. Not a metaphor you squint at — run the actual story and it lines right up.

**February:** I get paid, I go to the bar, I spend the whole paycheck on beer. Glorious. Every dollar did present-day work, I owe nothing, I wake up broke and free. That's a clean codebase — the entire budget goes to the thing you actually wanted.

**March:** I get paid, and I get *clever.* I put the beer on a credit card so I can keep the cash for other stuff. End of the month I pay down some of the card — not all of it, I'm busy, it's fine. That's a hack shipped on a deadline, and it can be genuinely smart: I got the beer *and* the cash.

Then April. May. June. Same clever move every month. And by October the card's maxed, I'm making the minimum payment, and the minimum is not cute anymore. So I switch back to cash like a responsible adult — except now I *can't* spend the whole paycheck on tonight's beer, because a fat slice of it has to go service the beer I drank back in *March.* That beer is gone. I don't even remember that beer. It is still taking money out of tonight.

**That's interest** — and it's exactly what the bad-code version charges you. The hack you shipped in spring is still skimming off every sprint in the fall, and the skim grows. Let enough of them pile up and you hit the version nobody climbs out of gracefully: borrowing just to make the payments, every new feature buried under the interest on the old ones.

Code runs on that exact ledger. Clean architecture means a new feature costs **the feature** — your whole budget goes to the customer's actual request. Bad architecture means every feature pays *interest first*: three days fighting the foundation before you write a line of the real thing, every time, forever, until you pay down the principal. The duplicate nobody deduped, the hack on the hack, the test you skipped — each one's a charge on the card, and the card compounds.

That's the version a stakeholder can do something with. Not "the code is bad." *"We're spending forty percent of every sprint servicing debt, and that number climbs every release we don't refinance."* Now it's a budget line. Now it's their problem, in their words, on their spreadsheet.

## Good debt

Here's the part that separates this from a rant, and the part most engineers refuse to say out loud: **sometimes the debt is the right call.**

If the business has six months of runway, you cannot take two years to build the perfect thing. You can't. Somebody else ships the mediocre version in month four and eats your lunch while you're still pouring the foundation. So you borrow — you ship the hack, you skip the tests, you put it on the card — on purpose, with your eyes open, because *getting to month seven alive* is the only architecture that matters when the bank account is the constraint.

I'd know. I built a [game called Sync Ball](https://apps.apple.com/us/app/sync-ball/id356277312) that was hack on hack on hack — every new screen bolted onto a loop that was never designed for it, the whole thing getting more fragile every week. And it was *fine.* It was a six-month, ninety-nine-cent game. The debt never came due because the product's relevance expired before the bill did. I borrowed against a future that was never going to arrive, which is exactly when borrowing is free.

The flip side is the math that should scare you: *if* that thing had taken two months to build right, then six months of runway was never the real constraint — and you ran up the card for nothing. The same hacks that were smart on a disposable game are malpractice on a codebase that has to live ten years. **The variable was never the hack. It's the runway.** And reading the runway — knowing whether you're building a firework or a foundation — is the actual job. Not "always do it right." Borrow deliberately, know the rate, know when you'll refinance.

That's the conversation. Not *whether* to take on debt — you will, everyone does — but *naming it as debt* so somebody with the whole picture can decide if you can afford the payment.

## Tests at review, speed at ship

And yet.

Here is the pattern I have watched at company after company, and I'll keep it general because the specifics aren't the point — the point is how *common* it is. At review season, suddenly everyone upstairs wants to talk about engineering excellence. What's our test coverage? Are we following best practices? It goes in the deck. It goes in your ratings.

Then a feature needs to ship, and the only question in the room is *how fast.* "Do you have bandwidth?" never once means "is this code testable?" It means "can you get it out this week?" Nobody asks for the tests at ship time. They ask for the tests at *review* time, and they ask why it's slow at *every* other time, and the engineer is left holding a contradiction nobody will say out loud:

**Writing testable code is slower up front, because you have to architect the thing so it can be mocked and verified — and that's exactly the time you're being told to hurry.**

And this is where I'd flip the question the whole industry keeps asking. Every company I've been at eventually runs the offsite tech talk: unit versus integration versus end-to-end, the mechanics of a good test. I sit there grinding my teeth, because the mechanics were never the bottleneck. Any engineer past their first year knows *how* to write a test. The real question — the one that isn't a tech talk, it's a whole different conversation — is **why do engineers feel like they can't?**

Because that's the actual shape of it. It's not that we won't, or don't know how, or secretly enjoy the swamp. We're the *least* tight-lipped people in the building about this — we say "the code is bad" every retro, every planning meeting, on the record, loudly. We just say it in the dialect that doesn't move anything, into a structure where "do you have bandwidth?" has only ever meant "can you ship Thursday," and never once meant "is this safe to change a year from now." The tests don't get cut because nobody spoke up. They get cut because *faster* is the only answer that gets rewarded in the room where it's decided.

And skipping never stays a one-time thing — that's the whole trap. You skip the test "just this once" to hit the date. But the next feature bolts straight onto that untested code, with no scaffolding to hang a test on, so you skip that one too. Every skip makes the next test harder to write and the pile of untested code bigger, which makes skipping the obvious call yet again, and again, and again. That's the beer-money interest, in test form: the round you skipped in spring is still making it harder to test in the fall. Compound it for six months, a year, two years, and you've got a codebase with effectively zero tests — and *then* everyone gathers around the coverage dashboard at review time and wonders aloud how it got this bad. I have sat in that exact meeting. The dashboard was never the mystery. The dashboard was the receipt.

## Change is the whole job

Now — why is any of this a real *cost* and not just my taste in code? Because of something hiding in plain sight in the language you write every day. Look at why half its features exist. Default parameters: so you can add a parameter, give it a default that preserves the old behavior, and **not touch the twenty-seven places that already call the function.** Overloads: same job, different spelling. Polymorphism, interfaces, the whole [SOLID](/2026/05/06/house-rules-d/) canon, every [design pattern](/2026/06/08/house-rules-who-loads-the-jukebox/) I've ever written up — all of it is the industry agreeing, in unison, on one assumption: **programs change.** The people who designed your tools built them, deliberately, so that change wouldn't cost you everything. Change isn't the exception they failed to prevent. It's the *premise.* It's the whole job.

Which is exactly where "but code is easy to change, it's not a bridge" falls apart — and people *love* that line, they use it to wave off the foundation entirely.

A bridge holds a known load and you go home. It carries trucks and wind and pedestrians, a load decided in advance, and once the concrete's poured it never wakes up wanting to cross a different river. Your codebase wakes up wanting that *constantly* — point B isn't important anymore, it's point E now, and by the way we need a currency we never integrated. That's the [load you haven't invented yet](https://mattcasanova.github.io/2026/05/05/bad-advice-agile-is-right-you-are-wrong/).

So yes — code is easier to change than a bridge. But "easy to change" is a property of *good* code, not of code. It's something you build in — with exactly those default parameters and interfaces and patterns — and when you skip the work, it evaporates. Now adding one small thing means editing thirty files. Adding a single *test* means first extracting interfaces, wiring up dependency injection so the class can even be mocked, and standing up a whole harness — to verify *one function.* **That's the definition, inside out: good code is easy to change, which is what makes it good; bad code is hard to change, which is what makes it bad.** The resistance isn't a symptom of the bad code — it *is* the bad code. "It's just code, it's easy to change" is the cheapest thing you can say over a clean foundation, and the most expensive thing you can say over a rotten one.

## I wrote the doc. I lost anyway.

I'll give you the version I lived, names filed off. I once spent weeks deep in an important system fixing bugs, and the deeper I dug the more I found. So I didn't just say "the code is bad" — I wrote it up: specific bugs, what each one did to actual users, escalated to my manager and his. Someone said it read as too tactical, so I wrote the strategic version — four classes of bug, the ones that'll bite us, the cost of leaving them. I passed it around, trying to get someone to care. I'm not going to claim I did it perfectly. But I did the translation, in their language, more than once.

It got deprioritized anyway. There were features with dates; the bad code wasn't on fire *yet*; it waited. And that's the genuinely frustrating part of this whole job — knowing there are bugs people are going to care about, watching them sit there, and not being able to get the resources to kill them.

Then, months later, after the feature shipped, the exact bugs I'd classified started showing up in the wild. And the conversation wasn't "how did nobody catch this." It was "this was a known tradeoff, and we chose it." I'd shown it; the people with the budget had decided to push anyway; it was on the record, with a date.

That's the win — and notice it isn't *being right.* Being right is cold comfort; I'd have traded the vindication for the fixes in a heartbeat. The win is that the decision got *made* — on purpose, by the people whose call it was, with the price sitting in front of them — instead of something that "just happened" to a team that never knew it was choosing. You don't control which way they decide. You only control whether it was a decision at all.

## Settling up

So here's where I land, and it's not "stakeholders are dumb" and it's not "engineers are saints." Let me be honest about the imbalance, because I don't think it's fifty-fifty and I won't pretend it is: engineers are not the quiet party here. We raise this every retro, every offsite, every planning meeting — we are the *loudest* people in the building about bad code. The incentive structure just doesn't answer back at the same volume, especially not when the feature's due Thursday.

But the only useful part of any of this is the part you control, and you control exactly one thing: you. You can't make a stakeholder care. You can't vote the runway pressure off the island. So you get two moves. Eat the cost quietly — write it right anyway, work the extra hours — and watch the long-term problem stay broken because nobody with the budget ever learned it was there. Or get good enough at pricing the invisible that ignoring it becomes someone's *informed* choice instead of an accident: *"forty percent of every sprint, climbing, here's the payoff to pay it down, here's why borrowing last spring was smart and borrowing again isn't."*

That second move isn't penance for a silence you're guilty of — you weren't silent. It's leverage, and it's the only leverage the job actually hands you. The hardest skill here was never writing the good code; plenty of people write good code. It's making the cost of the *bad* code legible to the person holding the cash, in their currency, before the bill lands. "The code is bad" is true, and loud, and it moves nothing. The number on the statement moves things.

The bill always comes due. The only choice you ever had was whether anybody got to read the figure before it landed.

Last call. Settle up while you can still afford it.
