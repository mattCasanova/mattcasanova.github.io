---
layout: post
title: "Nightcap: Back of House"
series: nightcap
tags: [meta, nightcap, testing, ai]
summary: "Two weeks of foundation and nothing you can screenshot. Features got cheap, the foundation got slow, and the slowest part of the slow part is me, pressing yes."
---

It's 9:42 on a Thursday and the drink is poured.

The day shift clocked out at five. The night shift has been running ever since, same as most nights this stretch. I already put two posts on the blog today, [the spoon](/2026/07/23/bad-advice-there-is-no-spoon/) and [the breaker](/2026/07/23/kill-the-lights/), so this is the third thing I've written tonight, and it's the one where I just talk.

Here's what I keep circling.

For about two weeks now I have built almost no features.

### The fun part got slow

I've said it on this blog more than once: the foundation is the fun part. Build the admin page before the feature. Wire the brakes before the car moves. I meant it, and I still do.

But something flipped and I only just caught the shape of it.

Features are cheap now. A real feature, the thing a user actually touches, I can stand most of one up in an evening and spend the rest just turning dials. That part moves.

The foundation is what eats the week. Hardening the lint rules. Dragging Larastan or PHPStan up another level. Building out the E2E and the UI suites so I can run them locally or [let CI run them](/2026/07/22/the-rate-went-up/). All of it for good reason. All of it slow.

And it's a different kind of fun than I'm used to. I'm genuinely lit up about the progress. I'm also two weeks into work where the report never changes.

### There's nothing to see

That's the part.

I finish a night, and the deliverable is: the tests are green, and there are more of them than there were yesterday.

That's it. That's the screenshot. A number went up and a bar stayed green.

It's real progress. I trust the apps more every night, and that confidence is the entire point of the ladder. But you can't post a picture of confidence. Nobody watches you count the walk-in. The customer sees the drink, never the inventory that made sure the drink could exist.

Two weeks of inventory. Books balanced. No new drinks on the menu.

### The bottleneck of the bottleneck

Here's the frustrating math.

The foundation is holding up everything else. It's the bottleneck right now, by choice.

And I'm the bottleneck of the foundation.

Because I don't let the agents run loose. It's build-this, then yes, yes, yes, yes, while I watch. Then hand the diff to a [reviewer](/2026/07/07/blade-runners/), read the briefing, pass it back. The machine is fast. The gate is me, on purpose, and a gate you can hurry isn't a gate.

I mitigate it by running web, mobile, and a side project all at once, three tables going so no one agent is ever waiting on me for long. It helps. It's still slow.

So the slowest part of the slowest part is a guy with a drink pressing yes.

### The empty object that wasn't

Then it got worse, in the good way, which is that the tests caught something real.

A few of the mobile runs choked on the API. Laravel handed back an empty array where the client wanted an empty object. Classic PHP tell: an empty map and an empty list serialize to the same `[]`, and a strict Swift or Kotlin decoder gags on it.

I can harden the client, and I probably will. But the real fix is on the server, and that's where the problem starts, because the server already has foundation work in flight. There's a half-built branch of exactly the slow, sprawling hardening this whole post is about. The last thing I want is a quick bug fix stitched into the middle of that, so the two land together and can't be reviewed, reverted, or reasoned about apart.

So the bug fix waits. And mobile, which surfaced it, gets set to the side too, because there's no clean client fix until the server decides what it's handing back.

Which is how I ended up reading about git worktrees at nine at night. I've barely scratched them. I don't even know yet what the loop feels like, how you hop between them, how you run the tests in each. That's homework. But "a clean branch for the bug that doesn't touch the branch I'm already deep in" is exactly the itch they're supposed to scratch, so I'm going to go find out.

### Side quests

[Flux](/2026/07/13/nightcap-lets-crank-it-up/) is still flickering awake in the corner. Slow, two minutes at a time, but not dead.

And a new one snuck in. mattcasanova.com, a domain I bought years ago and never did a thing with, is about to become a little running tracker. I've been running, I built myself a small calendar to log the day-to-day, and now I get to go stand up AWS for it.

Somewhere in all this I became an AWS guy. Nobody asked me to. It happened one provisioning script at a time.

### Last pour

Two weeks of foundation, a number that keeps climbing, and a bug that's about to teach me a new trick. Nothing you can see from outside. All of it the reason the outside holds.

The drink's for the inventory.

See you, space cowboy.
