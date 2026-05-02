---
layout: post
title: "Nightcap: Twenty-One Posts Deep"
series: nightcap
tags: [meta, nightcap, friday]
---

L was giving me trouble.

[Last Friday, anyway](/2026/04/17/nightcap-fifteen-posts-deep/). Tonight L is done. Multiple drafts deep, a Vegas flight in the middle, and a closer that flips the interviewer into the LSP violator. *[House Rules: It Compiled. It Lied.](/2026/04/22/house-rules-it-compiled-it-lied/)* shipped this just minutes ago. The third letter is out the door.

It's 11:40 on a Saturday. I have a drink. The Friday Nightcap is one day late. Saturday counts.

### The Ledger

**Blog.** Six posts since [last Friday](/2026/04/17/nightcap-fifteen-posts-deep/). [Flying Motorcycles and Movable Buildings](/2026/04/18/flying-motorcycles-movable-buildings/) the morning after. Then the Vegas-flight weekend — [Render Pass Strikes Back](/2026/04/19/render-pass-strikes-back/), [The Five-Year Particle System](/2026/04/19/five-year-particle-system/), and [No Map No Magic Prompt](/2026/04/19/no-map-no-magic-prompt/) on the same day. Today, two more: [Patience, Then Persistence](/2026/04/22/patience-then-persistence/) and the LSP one above. Twenty-one posts deep. Liskov was the hard one — multiple drafts, the Vegas flight in the middle, and the cached-client framing was the unlock. L is done. House Rules is three deep.

**LiquidMetal2D.** Big week. The component system from [Part One](/2026/04/18/flying-motorcycles-movable-buildings/) paid off twice. Once for [multi-shader support](/2026/04/19/render-pass-strikes-back/) — `GameObj` is `final` now, custom GPU data lives on components, composition or bust. Once again for the new [particle system](/2026/04/19/five-year-particle-system/) — emitter pool, alpha + additive blends, color variation, scale-over-lifetime, two demos. The plan from here is an iPad config tool to drive the particles. Which means I needed [persistence](/2026/04/22/patience-then-persistence/), so I shipped that too — `BlobStore` for app-owned files, `DocumentIO` for user-owned ones, tiered by who owns the file.

**Flux.** Backseat this week. Terminal-emulator-plus-Rust is a double-novelty tax — every session is also a language lesson, even with AI in the loop. AI helps me learn. AI does *not* get to write Flux for me. I don't want Flux to be AI slop. So Flux goes slower on purpose. I read every line. The primitives hold up; the pace is the cost of caring. Hitting it hard tomorrow.

**Apps.** Making good progress. Architecture's rocking. More to say later.

### One more thing

The pattern that keeps showing up: writing the post pushes the code forward.

The [OCP post](/2026/04/17/house-rules-whatever-happens-happens/) caught a real coupling bug in the engine last week and got me halfway to making `GameObj` `final`. This week's [LSP post](/2026/04/22/house-rules-it-compiled-it-lied/) finished the job — by the time I'd written it, the inheritance escape hatch I was hedging about in the prose had already been closed in the engine.

It's not just the engine, either. The upcoming Interface Segregation post is going to be a direct write-up of a fix I made in one of the apps. Some of the [Bad Advice](/series/bad-advice/) rants caught real issues there too. The pattern is bigger than any one project. **The blog is making the code better.**

I think it's because writing publicly forces me to confront places I'm being lazy. Privately I can hand-wave a sloppy decision. Out loud, I have to either justify it or fix it — and most of the time the justification doesn't survive contact with the page.

That's not a workflow I planned. It's a side effect of writing about the code I'm shipping the same week. I'll keep doing it as long as it keeps working.

Also: I rewatched Firefly this week. One season, one movie. Still pissed they cancelled it. Glad it is coming back.

Alright, it's past 11:44pm on Saturday night. I've been programming and blogging for hours today. The drink is for closing time.

No power in the verse can stop me.
