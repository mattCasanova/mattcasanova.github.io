---
layout: post
title: "Nightcap: Ten to One"
series: nightcap
tags: [meta, nightcap, testing, ci, ios, android]
summary: "The UI test backlog is finally closed on both apps. Android runs the suite up in the cloud. iOS never will, because Apple minutes bill at ten to one, so an eight-year-old iMac is about to get a job."
---

It's a little before ten on a Friday and this one's short.

The UI test backlog is closed.

Both apps. iOS and Android. Done.

### The tab I ran up on myself

I've been writing about [the foundation](/2026/07/23/nightcap-back-of-house/) for weeks, so I'll spare you the rerun. The part worth saying tonight is why it took so long.

It took so long because I waited.

I built features for months without the UI suites, and every one of those features quietly wrote a test I still owed. By the time I sat down to pay it, it wasn't a task anymore. It was a backlog.

So I ran an agent on iOS and an agent on Android at the same time, side by side, all week. That's the whole trick, and it still took about a week. UI tests are slow to write and slower to prove, and running two at once doesn't move the clock much when you're the one reading both diffs.

Anyway. Paid.

The website's in decent shape too. Playwright's running up on Actions, and I got the last of that squared away earlier tonight.

### What's standing now

Build-time tests, linters, and formatters on both mobile apps. Compiler flags cranked up on both. Pre-commit hooks. Pre-push hooks. Then CI on top of all of it, catching the slow stuff after the fact, because after the fact still beats never.

Android runs the whole instrumented suite up on GitHub Actions on push. Automatically. No hands.

iOS doesn't. And it isn't going to.

### Ten to one

Here's the math nobody warns you about.

A free private account gets you around 2,000 Actions minutes a month. GitHub Pro, the cheap tier, gets you 3,000.

macOS runners bill at ten to one.

So the 3,000 minutes I pay for is 300 minutes of iOS. That isn't a CI budget. That's a demo.

Which means the answer was never in the cloud. It's the 2018 iMac sitting in my house with 64 gigs of RAM in it, doing nothing.

launchd is the Mac's version of cron. Point it at the repo, pull, run the suite, go back to sleep. That machine is eight years old and it has exactly one job left in it, and it's a good one.

Not running yet. This week.

### Why any of this

None of it is interesting on its own. Nobody wants to read about a pre-push hook.

But it's the argument I keep circling, and I think it's the argument right now.

And I'm not the only one making it.

Zuckerberg's been saying a version of the same thing. We've got all this compute, so what should anyone actually be doing with it? The answer isn't only go faster. It's build out the platforms that can hold it when you do. I don't think he used the word foundation. That's the word for it.

I'm not talking him up because of where I work.

He takes more incoming than just about anybody. I think some of that is that he's quiet, and if he talked in public more he'd get less of it. This is a decent example of why.

We all have more compute than we know what to do with. So the question stopped being how fast can you go. Everybody's fast.

The question is what holds when you do.

You don't get to run agents in parallel because you're brave. You get to run them in parallel because there are enough checks in the way that a bad one can't get far. The gates are what buy the speed.

And the gates got cheap. That's the genuinely new part. Another test, another linter, another reviewer standing at the door: all of that used to cost real hours. Now it costs a prompt and a read.

So build the platform, not just the thing sitting on top of it.

### Last pour

I'm not done. There's refactoring and hardening left in every repo I own, and while we're still building out content the balance can keep leaning that way a while longer.

But I can start writing features again. I think I will.

Maybe [Flux](/2026/07/13/nightcap-lets-crank-it-up/) gets a look this week. It's been sitting there since I got sidetracked [building a run tracker](/2026/07/29/serve-it-neat/) on a domain I'd owned for years and never used. I'm blogging over there now too, though that one's about losing the weight, not shipping the code.

Big day tomorrow. Going to bed, getting up early.

See you, space cowboy.
