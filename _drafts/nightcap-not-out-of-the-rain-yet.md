---
layout: post
title: "Nightcap: Not Out of the Rain Yet"
series: nightcap
tags: [meta, nightcap, ai]
---

It's a little after 3 on a Saturday. Not Friday, not night, no drink poured yet — but the nightcap doesn't check the clock, and one's coming soon.

Two weeks since [the last one](/2026/05/22/nightcap-both-shifts-off/). Here's where things stand.

### The Ledger

**Day job.** The storm is passing. Early May we had to make a call at work — the kind you don't get to take back — and the weather rolled in behind it. The last couple weeks were still busy, but for the first time it feels like the front is moving off. Not out of the rain yet. But there's air to breathe again, and that air is going to the night shift.

I took that vacation on the 22nd like I said. Spent most of it checking my phone, waiting for an emergency that — thankfully — never came. A weekend off that I spent bracing for it to not be off. Still counts.

**Apps + website.** No code in about a month. The website and the app behind it have been frozen since the storms started. But I'm not idle on them this weekend — I'm doing an **audit**: no new features, just taking honest stock of where all the code actually is. Sometimes the most useful thing you can do with a stalled project is reread it.

**Flux / LiquidMetal2D.** Still backseat. Soon.

**Blog.** I owe you a *House Rules: Builder* post. I [promised it "Sunday"](/2026/05/22/nightcap-both-shifts-off/) two weeks ago and that Sunday came and went. But the pattern's not theoretical anymore — I did a real builder refactor at work this stretch. I don't talk about the work, but I'll say this: it came out clean. Properly clean. That's the post. It's coming.

### The golden age, revisited

Last time I joked that *we lived in the golden age of AI and it lasted about three and a half months.* The couple of weeks before that, the models had gotten genuinely bad at code — basic stuff, the kind of thing they'd been nailing in Vegas a month earlier. I never figured out where the degradation lived. Throttling somewhere? Me? The model? No idea.

It seems to have lifted a little. Opus 4.8 is out. Honestly it still doesn't feel like 4.7 at its peak to me — but I've barely touched my *personal* projects, so that read is unfair and incomplete. At work, it's been slightly better.

Here's the part worth keeping. I've always said: **never trust AI output 100%, always check it.** I think I'd quietly gotten lazy about my own rule — the stuff was so good for a while that I started skimming instead of reading. The bad stretch was a wake-up call. Not "the model slipped," but "you forgot you can't trust it, *ever*, no matter how good it looks." I liked it too much and let my guard down. Lesson re-learned. Re-filed under things I already knew.

### The Mac Studio lasted a day

Last post I said I was buying a Mac Studio — 128 GB of RAM — to run open-source models locally and chase 90% of Claude with no meter running.

I bought it. Set it up like any new dev box, a full evening of it. Installed OpenCode — an open-source coding agent you can point at any model, paid or not — and used Ollama to pull down a couple of open-source models for it to run on: Qwen and Llama. The theory was that a 72B model sitting entirely in RAM would at least be *fast*.

It could not tell me how many episodes are in season one of *Invincible*. (It's eight.) Basic questions took *minutes* and still came back wrong. I don't think I got a single useful answer out of it, never mind anything I'd let near a codebase or an automation loop.

Maybe I did it wrong — wrong harness, wrong model, wrong quantization, wrong expectations. Probably some of all four. But I'd used the machine for exactly one day, I wasn't confident I'd close the gap, and I wasn't going to gamble that money on "maybe." So I returned it. Perfect condition, back in the box.

Local models, for my use, right now? I'll let Marty take it — the way he put it to a 1955 gymnasium that wasn't ready for "Johnny B. Goode": *I guess you guys aren't ready for that yet. But your kids are gonna love it.*

---

So: storm passing, projects thawing, builder post loading, one Mac Studio richer in store credit and lessons. A quieter two weeks than the last few, which is exactly what I needed.

Until next signal.
