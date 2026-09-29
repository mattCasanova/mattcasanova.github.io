---
layout: post
title: "Phoning It In"
tags: [ai, workflow, claude-code, agents]
summary: "Thirty days with no day shift, a bus between Daegu and Busan, a phone with a data cap, and four agents on a Mac mini back home that never sleeps. What gets done from a phone, and the one thing that doesn't."
---

I'm on a bus from Daegu to Busan, and three agents are working.

One is building website features. One is building a HIIT timer for the mobile apps. The third one is taking dictation, because I'm writing this into my phone, on a bus, with no real phone plan.

Phoning it in. Literally, for once.

### The day shift is dark

Meta gives you a recharge after five years: thirty days of PTO, all at once. Mine is now. We came to Korea to see family.

So there's no day shift for a month. [Day shift is the job, night shift is the side projects](/2026/05/22/nightcap-both-shifts-off/), and right now it's all night shift. Of course I brought my laptop.

But I knew going in that a lot of this trip would be time I couldn't use it. Walking around with my wife. A couple of hours on a bus between cities with whatever Wi-Fi the bus feels like sharing. We don't have a phone plan here. T-Mobile gives us something like five gigs a month,[^cap] which is enough to talk to an agent and not enough for much else.

That's normally where side projects die. [I've watched it happen](/2026/05/22/nightcap-both-shifts-off/) every time the calendar filled up. This time I planned for it.

### The house never sleeps

Before we left I set the Mac mini at home to never sleep and to wake on LAN. Then I opened four Claude Code sessions and left them there: one called free web, one called free mobile, and two called free workspace, which just means open, no job yet.[^free] Each one sits in its own repo with the plans and the house rules already loaded.

Then I turned on remote control, the Claude Code feature that lets your phone talk to a session running somewhere else. The sessions are on a desk in Seattle. I'm on a bus seat in Korea.

Hopefully they stay open the whole month. If that machine restarts, the plan goes with it, and I'll find out from a phone that can't reach anything. That's the risk. I took it.

Nobody's home. Everybody's working.

I didn't invent this. Remote control is a feature somebody else shipped, and wake on LAN is older than my career. But I don't know anybody else doing it, either. Four agents on a Mac mini nobody's sitting at, driven from a bus on the other side of the planet. It's not common. I'll take the head start.

### What fits through a phone

Here's what a day of this looks like. We've been here one, so this is day one talking.

I rant a feature into the phone the same way I'd [rant it into Superwhisper at my desk](/2026/04/19/no-map-no-magic-prompt/). The agent plans it. I read the plan on the phone, which is fine, a plan is just prose. I push back. It builds.

When it's done, it commits to a feature branch. Not main. Never main from here.

Then I send in a second agent to [review it](/2026/07/07/blade-runners/), the same reviewer I'd use at home. It hands me a briefing, and I read that on the phone too. A couple of cycles of that and the branch is in decent shape.

Then we get off the bus, or go for a walk, and the phone goes back in my pocket. Planning, deciding, having the bots review each other: all of that fits through a phone. It's just words.

### What doesn't

Code doesn't.

It's pretty hard to read a diff on a phone. GitHub on a five-inch screen tells you a file changed. It doesn't tell you much about whether it should have. And the one rule that hasn't moved on this blog is that [nothing merges without me reading it](/2026/07/07/blade-runners/). [The reviewers aim me. They don't replace me.](/2026/07/22/the-rate-went-up/)

So the branch waits. When we land somewhere with a table and real Wi-Fi, I open the laptop, check out the branch, read it the way I'd read it at home, and make my tweaks. For the mobile side I install the build on this same phone and actually use the timer, because [I'm my own QA](/2026/08/02/nobody-points-at-the-blade/) and a feature that reads fine in a briefing can still feel wrong in your hand.

That's the honest version. From a phone I can get a feature planned, built, reviewed, and parked on a branch. I can't get it merged, and I shouldn't be able to. That part waits until I've got a laptop in front of me.

So the bottleneck moved. It didn't shrink. At home it was me at midnight. Here it's me at the next hotel with a desk.

### Last call

Back in April I told you [there's no map](/2026/04/19/no-map-no-magic-prompt/) for working this way, and that you draw your own. This week mine is a phone, a bus, and a box at home that won't go to sleep. Next month it'll look different. It always does, and finding the next version is the part of this job I'm enjoying most right now.

The bus is slowing down. Busan's coming up. Three sessions are still going, and one of them is about to save this file to a machine I won't touch for a month.

Phoning it in was supposed to be the insult. Turns out it's the plan.

See you, space cowboy.

[^cap]: Five gigs a month, I think. Whatever the number is, dictation fits under it and diffs don't.

[^free]: Free as in no job yet. Not free as in beer.
