---
layout: post
title: "The Night Auditor"
tags: [ai, agents, workflow]
---

Every hotel has a night auditor — the person who comes in after midnight, when the lobby's empty and nobody's checking in, and reconciles the whole day's books. The work isn't to sell anything. It's to make sure the numbers tell the truth before the next day starts.

That was me this weekend, except the books were my code. I'm back in all my repos — not to ship features, but to audit them.

No new features to show for it. A few real bugs, though, and a bigger idea I want to write down before it stops feeling obvious.

### Fresh eyes used to be expensive

Think about what it takes for a person to audit an app they didn't build. They have to learn the features, read the history, find the landmines, build a mental model — *then* they can tell you what's wrong with the architecture or where the security holes are. That's weeks of ramp before the first useful sentence. Nobody can join your team on Monday and hand you a real architecture review on Tuesday.

So the whole-app audit — the "is this thing actually sound?" pass — is the kind of work that almost never gets done. Not on solo projects, not even on big teams. It's too heavy. There's always a feature more urgent than a review of work that already shipped.

An agent doesn't carry that cost. It doesn't need to know the features. It hops into the codebase with genuinely fresh eyes, reads the code, reads a little of the docs, and tells you what it sees. The ramp is zero. And you can run a bunch of them at once, in parallel, while you do your actual work on the side.

It costs tokens. It's essentially free. The math is so lopsided it's almost embarrassing nobody could do this a year ago.

### What I actually ran

This weekend I had multiple Claude instances going at once:

- One auditing the website end to end — including things like whether the APIs actually throttle, so nobody malicious can just spam an endpoint.
- One that spun up its own sub-agents to audit the iOS and Android apps and **diff them against each other** — looking for drift, places where the two clients quietly disagree.
- And I was testing **Grok** on the deploy and infrastructure scripts — basically a "am I doing anything I shouldn't be?" pass. Are the S3 buckets locked down, is disaster recovery actually set up the way I think it is, are the cron jobs wired correctly. That kind of sanity check.

You don't tell it "fix everything." You tell it "look — architecture, or security, or just this subset of features — and report back." Then you read the report and decide. The agent flags; you decide. Same rule I [re-learned the hard way last week](/2026/06/06/nightcap-not-out-of-the-rain-yet/): you don't hand it the keys, no matter how good it looks.

Reading the report *is* the work, and it's where I stay in charge. Every flag is a question, not a verdict: **did I do this on purpose?** If I did, I probably remember why, and it stays. If I didn't, that's a real issue and now I know about it. The agent can't tell a deliberate trade-off from a mistake — they look identical in the code. I can. That judgment is the whole job, and it's exactly the part you can't delegate.

There's a side benefit I didn't expect, too: the audit refreshes *my* memory. Reading a fresh-eyes pass over a project I haven't touched in weeks is the fastest way back into my own head — what's built, what's half-built, what I was clearly mid-thought on and then dropped. So these audits aren't only for finding bugs. They're for finding bugs, *and* re-loading the project, *and* keeping me the one in charge of it. Multiple uses, one cheap pass.

It found real things. Small, but real — like a super-admin role I'd added where a handful of features never got wired up to actually check for it. Drift you'd never notice from the outside, exactly the kind of thing a fresh-eyes pass exists to catch.

### The docs were lying

Here's the part I didn't expect. The agents kept tripping over my own documentation.

Grok read my deploy-script docs and treated them as truth — except they were out of date. Stale plans, old disaster-recovery notes, architecture docs describing a system that had moved on without them. The audit didn't just find bugs in the code. It found that my *docs* were quietly wrong, and a fresh agent had no way to know.

The root cause is the workflow itself. When you work through an agent instead of opening the whole repo in an IDE, you stop *seeing* the docs that live inside the repo. They rot in folders you never open. Plans, audits, DR notes — out of sight, slowly going false.

So I'm pulling them out. Non-code docs come out of the code repos and into the one folder I actually open every day — my workspace, which is in git like everything else. One known place per project. Living documents I can open and immediately tell whether they're current or stale. The next agent that reads them gets the truth, not last quarter's truth.

### So, the week

No new features. But every repo got a fresh-eyes pass, a few real bugs are fixed, the deploy scripts are getting tightened, and the docs are moving somewhere they'll stop lying.

The audit is the leap, though. In the agent world, the expensive, heavy, always-skipped review is now the cheap thing you run in the background while you do something else. Run more of them than you think you need. They're basically free.

See you, space cowboy.
