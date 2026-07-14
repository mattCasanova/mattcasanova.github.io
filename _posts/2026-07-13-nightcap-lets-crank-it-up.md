---
layout: post
title: "Nightcap: Let's Crank It Up"
series: nightcap
tags: [meta, nightcap, flux, rust, ai]
---

Monday, drink poured, and I'm about to admit something I spent all of last week arguing against.

But first, the reason.

I opened Warp again this week.

Fuck... Warp is Awesome.

### Start with the tab bar

It's vertical. It slides in and out along the side instead of squatting along the bottom. Hover a tab and you get the state of the agent running inside it: still working or finished, what branch it's on, whatever title Claude decided to give the run.

That's just the hover. That's just the tabs.

Then there's the file explorer. Sit in a folder, pull up a real explorer view, open a file off to the side, and read your code next to your terminal output. It isn't an IDE and it isn't pretending to be. But it's an honest file explorer, and it's close to the editor I think we all end up wanting in a couple of years: the terminal is the main event, and the code sits beside it instead of the other way around. There's also something called Warp Drive, which is a badass name, and which I have not touched.

None of that needs a single AI feature switched on.

The one thing they get wrong:

It is ugly.

They have some built-in themes, but you can't customize much after that.

My iTerm2 looks way fucking better than Warp does, and I cannot work out why they won't just let people theme the thing.

And I still can't use it at work. Which is the entire reason [Flux exists](/2026/04/10/building-a-gpu-terminal-in-rust/), and the same wall I hit when [they open-sourced it](/2026/05/05/warp-open-sourced-so-what/).

The terminal I want still doesn't exist.

### They know I know

Not long ago the ledger said [zeros again](/2026/07/06/nightcap-no-fireworks/). Flux and the engine are waiting, I said, and they know I know.

Flux is awake.

The plan's been sitting there since spring. Last thing I shipped was v0.2. The sequence runs 0.2, 0.3, 0.4, on up to 1.0, and 1.0 is the only number I care about, because 1.0 is the one I can use.

Let me be exact about what that does not mean. It does not mean good. It does not mean a Warp replacement. It means usable enough to **dogfood**, which is the only thing that counts right now, because a terminal you can't stand to open is a terminal you never actually test.

Basic navigation already works. And there's a popup in there I'm genuinely happy with.

<iframe width="560" height="315" src="https://www.youtube.com/embed/U3cLIYrSyHY" title="Flux Terminal Demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Bang.

Now the other half of the ledger. It's been missing scrollback. It's been missing copy and paste.

That's not "a little rough."

That's painful.

And painful is precisely why it sat there for months.

I'm cranking too hard on everything else to spend a Tuesday night fighting my own terminal.

Two minutes at a time, until it earns more.

### Two experiments

Here's the admission.

I stopped reading the code.

I just built a crew of [blade runners](/2026/07/07/blade-runners/) and told you the rule out loud: you never hand it the keys, you just make it show its work first. Then I turned around and [moved my rules down into the compiler](/2026/07/11/carry-that-weight/), because a convention in my head is enforced exactly as often as I'm the one typing.

None of that is running on Flux.

No reviewers. No auditors. Just me, a Rust learning curve I'm still climbing, and a plan I had Grok and Claude look over before I started cranking toward 1.0 without reading what comes out the other end.

That isn't backsliding. That's the experiment.

Because the difference between those two codebases isn't the language or the stack. It's who's waiting.

**The apps and the website, I build for other people.** Somebody's card gets charged. Somebody's login has to work at 4 in the morning. That code has to be robust, robust is expensive, and it is worth every minute of the blade runners, the typed errors, and the gates that won't let the machine forget.

**Flux, I build for me.** Nobody is waiting on it. There's no user to disappoint, no card to charge, no 4 in the morning. Which means the one thing Flux genuinely cannot afford is the one thing I keep giving it: time.

I want it to live.

That's the whole goal.

Sprint to 1.0.

I do not care how bad the code is when it gets there.

Because a terminal that doesn't exist has *excellent* architecture.

Perfectly clean.

Zero bugs.

And it's been sitting at v0.2 since April while I told myself I'd read every line before it shipped.

Once the thing is alive, I can worry about the code. I can point the crew at it. I can read it properly and refactor it into something I'd defend. What I cannot do is refactor a project I never finished.

So there are two experiments running in this house now.

On one, I'm being as careful as I know how to be. AI writing, AI reviewing, rules shoved down into the compiler where nobody gets to forget them.

On the other, I'm letting her rip.

And I know exactly what that second one can cost, because [you just watched it cost me](/2026/07/12/nightcap-your-style-is-my-style/) on a codebase where I *was* looking.

We'll see which one I end up regretting.

### Last pour

Fable's still alive, incidentally. It ran out, came back, and now runs out again on the 19th. This model has more lives than Kenny.

Everything else holds. The apps and the website are still the priority. LiquidMetal2D is still parked and still waiting.

And the spoon is still owed. I know. It's been owed a while now, it keeps getting wider instead of done, and at this point it's less a debt than a running bit.

Is any of this a waste of time?

Maybe. Who cares.

Let's crank it up.

See you, space cowboy.
