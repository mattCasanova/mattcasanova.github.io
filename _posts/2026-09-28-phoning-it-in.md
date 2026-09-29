---
layout: post
title: "Phoning It In"
tags: [ai, workflow, claude-code, agents]
summary: "Thirty days with no day shift, a bus between Daegu and Busan, a phone with a data cap, and four agents on a Mac mini back home that never sleeps. What gets done from a phone, and the one thing that doesn't."
---

I'm on a bus from Daegu to Busan, and three agents are working.

One is building website features. One is building a HIIT timer for the mobile apps. The third is taking this down, because I'm dictating it into my phone, on a bus, in a country where my phone plan is a rumor.

Phoning it in. Literally, for once.

### The day shift is dark

Meta gives you a recharge after five years: thirty days of PTO, taken straight, no slicing it up. Mine is now. We came to Korea to see family.

So for a month there is no day shift. [Day shift is the job, night shift is the projects](/2026/05/22/nightcap-both-shifts-off/), and right now the whole clock is night shift. Every hour is side-project hours, and I brought the laptop, because of course I did.

But I knew going in that a lot of those hours would be dead ones. Walking with my wife. Riding between cities with whatever Wi-Fi the bus feels like sharing. T-Mobile caps us at something like five gigs a month here,[^cap] which is enough to dictate and not enough for much else.

Dead hours are where side projects go to die. I've [watched it happen](/2026/05/22/nightcap-both-shifts-off/) every time the calendar filled up. This time I planned for them.

### The house never sleeps

Before we left I set the Mac mini at home to never sleep and to wake on LAN. Then I opened four Claude Code sessions and left them running: one I called free web, one free mobile, and two free workspace, which just means open, no job yet.[^free] Each one is parked in its repo with the plans and the house rules already loaded.

Then I turned on remote control, the Claude Code feature that lets a phone reach a session running somewhere else. So the sessions live on a desk in Seattle, and I talk to them from a bus seat in Korea.

Hopefully they stay open the whole month. If something reboots that box, the plan dies with it, and I'll find out from a phone that can't reach anything. That's the whole risk. I took it.

Nobody's home. Everybody's working.

### What fits through a phone

Here's what a day of this looks like. We've only been here one, so this is day one talking.

I rant a feature into the phone the same way I'd [rant it into Superwhisper at my desk](/2026/04/19/no-map-no-magic-prompt/). The agent plans it. I read the plan on the phone, which is fine, because a plan is prose and a phone is good at prose. I push back. It builds.

When it's done, it commits to a feature branch. Not main. Never main from here.

Then I send in a second agent to [review it](/2026/07/07/blade-runners/): the same fixed lenses, the same platform-shaped reviewer I'd use at home. It hands me a briefing. I read that on the phone too. A couple of cycles of that and the branch is in better shape than plenty of PRs I've approved in offices.

Then we're off the bus, or out on a walk, and the phone goes back in my pocket. Planning, deciding, having the bots argue with each other: all of that fits through a phone. The whole conversation is words, and words are small.

### What doesn't

Code isn't.

I cannot read a diff on a phone. GitHub on a five-inch screen is a way to confirm that a file changed, not a way to know whether it should have. And the rule that has never moved on this blog is that [nothing merges without me reading it](/2026/07/07/blade-runners/). [The reviewers aim me. They don't replace me.](/2026/07/22/the-rate-went-up/)

So the branch waits. When we land somewhere with a table and a real connection, I open the laptop, pull the branch, read it the way I'd read it at home, and make the tweaks. For the mobile side I install the build on this same phone and actually use the timer, because [I'm my own QA](/2026/08/02/nobody-points-at-the-blade/) and a timer that reads fine in a briefing can still feel wrong in a hand.

That's the honest shape of it. From a phone I can get the machine started. I can get a feature planned, built, reviewed, and parked on a branch. I cannot get it merged, and I shouldn't be able to. The part that needs my eyes waits for my eyes.

Which means the bottleneck moved and didn't shrink. Back home the bottleneck was me at midnight. Here it's me at the next place with a desk. Same guy. Different chair.

### Last call

This blog's colors are named after this country. Gangnam Night for the background. Last Train Home for the footer. I picked those names in April from a desk in Seattle, and now I'm dictating a post under the real thing.

The bus is slowing down. Busan's coming up. Three sessions are still going, and one of them is about to save this file into a folder on a machine I won't touch for a month.

Phoning it in was supposed to be the insult. Turns out it's the plan.

See you, space cowboy.

[^cap]: Five gigs a month, I think. Whatever the number is, dictation fits under it and diffs don't.

[^free]: Free as in no job yet. Not free as in beer. Nothing about this month is free as in beer.
