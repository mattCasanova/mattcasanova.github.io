---
layout: post
title: "Blade Runners"
tags: [ai, agents, workflow, claude-code, opinion]
---

Fable 5 turns back into a pumpkin at midnight tonight.[^1]

I got a second taste of it this week — a limited window on the Max plan, a slice of the weekly budget carved out just for it, twice the model at roughly twice the price. And after I [finished the blog last night](/2026/07/06/nightcap-no-fireworks/) I still had about a quarter of that Fable budget sitting in the tank, burning a hole.

I wasn't going to spend the good stuff on nothing. So I spent it building a crew.

Here's the thing I keep landing on: the model that draws the best maps is the one you have for a few days. The tooling those maps become is yours forever. Given a window on a better model, the highest-leverage thing you can do is not ship a feature — it's build the machine that ships features after the window closes. So that's what tonight's post is about. Not what I built. What I built to build.

### Who audits the auditor?

The uncomfortable truth of working this way is that AI writes a *lot* of code, fast, and you are still the one whose name is on it. That gap — code volume up, human attention flat — is the whole problem to solve.

Using AI to check AI is the obvious move, and I'll say up front it isn't a clever one — everybody's doing some version of it now. One agent writes, another agent reads it cold and tries to poke holes. It's turtles all the way down: an AI auditing an AI's work, and who audits *that* one? Me. I'm the ground the turtles stand on.

What I actually built tonight is bigger than the trick, though. Seven skills by the time I ran dry, in two families — and the difference between the families is the whole point.

A **reviewer** looks at the change in front of it — the thing I'm about to commit — through a fixed set of lenses, and asks whether *this* fits. Per-change, day-to-day work. It's what most people mean when they say "AI code review."

An **auditor** does something a reviewer structurally can't. It doesn't look at a commit. It looks at the whole project *as it stands right now* and asks how the thing holds up — not "is this diff good" but "is this system still sound." A single review of a single change can be perfectly fine — I thought it worked, it passed — and still miss that for one kind of user, in one edge case, the feature quietly falls over. Or that an architecture I picked three months ago was the right call then and a bad one now, after seven more layers landed on top of it. Reviews catch mistakes as they go in. Audits catch the ones that only *became* mistakes later.

Uncle Bob's end of the [spectrum](/2026/04/11/syntax-is-the-least-of-our-skills/) says don't read every line; own the architecture and let the details be details. I'm not all the way there — I want to stay in the quality. But I also can't personally re-read every diff across web, iOS, and Android in a single night without becoming the bottleneck the AI was supposed to remove. So I built the machine that reads them, and I read the machine. The rule hasn't changed since I [re-learned it the hard way](/2026/06/06/nightcap-not-out-of-the-rain-yet/): you never hand it the keys. You just make it show its work to something else before it shows it to you.

That's the whole idea behind a blade runner. Build a thing whose entire job is to hunt the flaws in another thing's work. And the good joke — the one *Blade Runner 2049* already told — is that the hunter is the same species as the hunted. My auditors are AI. My reviewers are AI. More human than `human`. I just decide who gets retired.

### The Night Auditor, standardized

[The Night Auditor](/2026/06/07/the-night-auditor/) was the prototype. That weekend I ran a handful of one-off audits — website here, mobile drift there, Grok on the infra — all ad hoc, all improvised, none of it repeatable. It worked, and then it evaporated, because a thing you did once by hand isn't a system. It's a memory.

This week I turned it into one. There's now an auditor per repo: the Laravel-plus-Vue website, iOS, Android, and the deploy scripts. Each knows its own turf. The Laravel auditor runs Laravel-shaped lenses — is Composer current, is npm current, are the routes locked down, do the APIs throttle, is any of this actually reusable or did I copy-paste my way into a corner. The mobile ones run mobile-shaped lenses. They all start the same way, though: read the previous audits, read the open issues, *then* go deep. A run isn't a blank slate; it picks up where the last one left off.

The best part is that the auditors can see each other. Because Claude reads my whole workspace, the Laravel auditor knows what the deploy scripts say the server can take, and the mobile auditors can read the actual API — the real response shapes, the real headers — instead of guessing at the contract from the client side. So the audits cross-flag. The iOS pass can catch a place where the app assumes something the backend never promised. That's a lens I couldn't hand a human reviewer without weeks of ramp across three codebases; here it's free, because it's all one context.

Two gears. There's a deep pass that kicks off a dynamic workflow and spawns a whole swarm of sub-agents — thorough, and honestly kind of expensive in tokens, which is exactly what a temporary window on a good model is *for*. And there's a single pass, lighter and cheaper, the one I can afford to run without thinking about it. Deep pass to find everything once; single pass to make sure nothing new crept in since.

The output is the part I actually care about. It ranks what it finds — high, medium, low — and it argues with itself on the way there, which is a feature, not a bug. Then it does two things: writes a dated report into an `audits/` folder, and files issues for the real findings. Because the failure mode of this whole way of working isn't bad code. It's an agent telling you about a bug at 11pm, you nodding, and then never writing it down — so it waits two months and bites you when you've forgotten it ever spoke. A finding that isn't an issue is a rumor.

And when it flags something I did *on purpose* — a trade-off, not a mistake — that doesn't get "fixed." It gets filed as a known gap: written down, parked, and pulled back up later if something trips the trigger or enough time passes that the decision deserves a fresh look. The agent still can't tell a deliberate choice from a screwup — they look identical in the code. I can. That judgment is still the job.

It found real things, too. A couple of spots around logout I should've been more careful with, and some billing edge cases I just hadn't fully tested yet. Nothing on fire — but exactly the kind of quiet, boring, easy-to-miss gap that a cold read catches and a busy author never will.

### Cells. Interlinked.

Audits are for code that already shipped. The other family is for code on its way in — and for that I built the reviewers, same DNA, different moment.

My standing rule is that nothing commits without my say-so. I keep bug fixes small on purpose so I can actually read them, but a feature is a feature, and sometimes it genuinely takes a big change across a lot of files. Do that on iOS and Android and the web in the same day and I *will* miss something. I'm not being modest; that's arithmetic.

So now, when a build agent finishes a feature — often on both platforms at once — I send in fresh reviewers. A separate agent reviews the iOS side, another the Android side, each with no stake in the code it's reading, each looking through its fixed lenses at how the change fits the rest of the project. They compare notes across platforms: here's how iOS solved it, here's how Android did, here's where they drifted. And before I read a single line myself, they hand me a briefing — what got done, how, and where the red flags are. It's the baseline test from *2049*: after every job, the runner comes back and gets checked against a standard to make sure it hasn't drifted. Cells. Interlinked. I walk into my own review already knowing where to look hardest.

That sets the rhythm. Day to day, the reviewers run on whatever I'm shipping. About once a week, a single-pass audit sweeps for whatever the week's features stirred up. Once a month, once a quarter, the deep pass does the full backward look at the whole thing. Reviewers keep the new work honest; audits keep the old work from rotting. Getting two AIs to push on each other doesn't remove me from any of it. It aims me.

### What gets built next

All of this created a new, good problem: the backlog exploded. Every audit filed issues, and now I've got a pile of them stacked across four repos and a finite number of nights.

So the next thing I build is the coordinator — call it a sprint planner that happens to have read all my code. Something that reads the current state and the open issues across every repo and helps me answer the only question that actually matters on a given night: what's the one thing that, if I do it, unblocks the most other things — and lets the most work run in parallel after. Not to boss me around. If something's fun and I'm lit up about it, I'm building that first, full stop; the machine doesn't get a vote on joy. But when I don't have a gut call, I'd rather have a straight answer than a guess.

That's the real shape of all of it. The auditors, the reviewers, the coordinator that's coming — none of it writes the exciting code. It's the assistant work: the remembering, the sorting, the second opinion, the "did you write that down." I hand the machine the parts that are easy to drop on the floor, so the part I keep is the part I actually like.

I ran the deploy, web, and iOS audits on Fable and hit zero partway through Android — finished that one on plain Opus 4.8. For the record, this is what *spent it wisely* looks like:

![Anthropic weekly-limits panel: All models at 54% used, Fable at 100% used, both resetting Saturday 3:00 PM](/assets/images/2026-07-07-fable-100-used.png)

*One hundred percent. I don't leave a good model in the bottle.*

The tank's dry until Saturday's reset — which, thanks to that extension, buys me one last run before it closes Sunday for good. Not that it changes tonight; I'd have spent this round exactly the same way regardless. The crew it helped me build clocks in tomorrow on the regular model, and every weekend after that.

That's the trade I'd take every time.

See you, space cowboy.

[^1]: Reader, it did not. Somewhere around the third audit an email landed pushing the window to Sunday night. By then I had — in the grand tradition of anyone ever handed a deadline — already set the whole allotment on fire to beat it. The deadline moved; the ashes stayed put.
