---
layout: post
title: "Nightcap: No Fireworks"
series: nightcap
tags: [meta, nightcap, ai]
---

It's late on a Monday, which makes this a nightcap on a technicality. The nightcap doesn't check the clock, and apparently it doesn't check the calendar either. Drink's poured.

Long weekend — Friday, Saturday, Sunday. The day shift and the night shift blurred into one long one: same chair, same coffee shop, the same job I do Monday through Friday, just pointed at my own stack. And since the sky over the water spent Saturday exploding on purpose, the theme sorted itself out: everyone else was making fireworks. I spent three days making sure I never have any.

Seventeen days since the last post, too — a new record for quiet around here, and not one I'm proud of. Here's where things stand.

### The Ledger

**Day job.** Holiday. Nothing to report, which is the report.

**Apps + website.** Everything. A brain-frying amount of everything.

Friday and Saturday were the deploy train. The billing rail I [wired up in June](/2026/06/13/nightcap-five-days-of-fable/) is now **live in production** — fresh web box, real product catalog, sales tax collecting — sitting dark behind a feature flag. One smoke test stands between that flag and real money. The video catalog finished its cutover to the new delivery pipeline: twenty-one videos, one canonical source, YouTube out of the serving path entirely. The mobile apps got their video players, rethemed to match. And a confession from the same push: a header bug meant the mobile kill switch I *thought* was armed, wasn't. It is now.

The admin panel got gutted and rebuilt too — I had Claude rewrite the whole thing so I can run everything from my local machine. One-click deploy. One-click database backup. Two-click restore — technically one, but a restore *should* ask if you're sure.

About that feature flag: I built the flag system myself, a while back, before billing ever shipped. Small admin tool, nothing fancy. Here's the thing, though. I've worked at big companies and small ones, and at the big ones you *use* tools like that every day — but somebody else built them. Years ago, rolling your own would've been a decent-sized chunk of work, and if the exciting work was somewhere else, you know exactly which one you'd build. Now I don't have to choose. The boring tools get built in parallel while I work on the parts I actually care about.

Same trick with tests. Coverage tools all spit out HTML, so I pointed them at a folder my admin panel reads — coverage reports live in my own dashboard now. Then I had Claude grinding out tests in parallel all weekend: roughly **90% coverage** across the website, the Vue front end, and both mobile apps. UI tests excluded — those come later, along with an idea I'm chewing on for heartbeat tests against the APIs. Coverage isn't quality. But it buys confidence, and confidence is what lets you deploy on a Saturday.

Sunday was audit day. [The Night Auditor](/2026/06/07/the-night-auditor/) rides again — fresh agent, honest eyes, the whole infrastructure this time. What came out the other side: the web and database boxes each hold their own keys now instead of sharing one ring; every storage bucket keeps version history, so a fat-fingered delete stopped being a permanent event; the video masters are migrating into their own locked vault; health alerts and billing alerts are up, and if the site ever goes down, my phone buzzes. I tested that one. It buzzed. Also prepaid three years of compute — about eight hundred bucks up front to save closer to a thousand over the term. The night shift does accounting too.

Best find of the audit: my "zero-downtime" deploy was a three-to-eight-minute outage. Every deploy. The maintenance flag went up before the build instead of around the migrations, so the live site served 503s while npm did its thing. Fixing that exposed a second bug that had been hiding behind the first — a shared cache poisoning the live release for twelve seconds of errors on every single deploy, masked all this time by the maintenance window. Both fixed. Final probe run: twenty-five requests fired during a live deploy, twenty-five 200s. Deploys are boring now — no migrations, no downtime, nothing to see. Boring was the whole point.

There was paper, too: a plan to ship the database's logs off the box every few minutes — a crash used to cost up to an hour of data, the plan gets that to about five — and a plan to give the site a blog of its own. Flat markdown files, no database, no CMS. Write file, push, done. Where *would* I have gotten that idea.

**Flux / LiquidMetal2D.** Zeros again, and I'll stop pretending it's a scheduling problem. I still want Flux. I'll get back to it. But the apps-and-website project is the one my wife and I are building together, and it's the one that has me sitting in the same chair for three days grinning at deploy logs. Excitement decides where the spare cycles go. Flux and the engine are waiting, and they know I know.

**Blog.** I owe you a spoon. [Two-Drink Minimum](/2026/06/19/house-rules-two-drink-minimum/) promised *Bad Advice: There Is No Spoon* as "already poured." It's still poured. Nobody's touched it. It's next — and you know my record on "next," so I'll just point at the Builder post and note that late is not the same as never.

### One more thing: the fable came back

Last month I had [five days with a borrowed model](/2026/06/13/nightcap-five-days-of-fable/) — Anthropic's Fable 5, early access that evaporated mid-week. It drew maps and vanished, and I built from the maps with the everyday model.

It came back. A taste on Thursday, and as far as I can tell I've got it through tomorrow. So the weekend plan wrote itself: burn every Fable token they'd give me before it disappears again. Three days of it planning, auditing, reviewing, writing tests — everything above has its fingerprints on it somewhere. I didn't quite max it out, and not for lack of trying: I ran out of work I could run in parallel before I ran out of Fable.

It wore the Night Auditor badge this time, too — read every provisioning script, every runbook, every plan doc, and handed me the ranked list that ate my Sunday. Its verdict on a solo operator's infrastructure, quote: *"genuinely well-run."* I'll take that from the machine. It has no reason to flatter me and every opportunity not to.

### Tuition

Here's the thing about this weekend: almost none of it was new to me. That's *why* it moved fast.

Back before [V-Shred](/2026/04/14/bad-advice-roll-your-own-di/) I'd done nothing but mobile for five years. Backend was web-dev territory; I'd barely touched SQL. So I started teaching myself Laravel on DigitalOcean droplets, and provisioning a server meant half a day of typing commands over SSH — for a default project with literally nothing in it. So I scripted a piece of it. Next Laravel version came out, I re-provisioned, the script got a little better. A deploy script here, a config there — years of that, never knowing what any of it was *for*. No product. No plan. Just: better to have the knowledge than not.

Then the tools changed. Grok first, then Claude, and that pile of half-scripts grew into a real provisioning system — web box, database box, AWS launch templates that pull my scripts straight from GitHub with deploy keys. The half-day of typing is the one-click deploy up above. Years of iterating on a thing I never needed, and now I need it every weekend.

So why spend real money — the Claude subscription, the AWS bill — on a project that doesn't make any? Same reason I'd rather run cheap servers and do the deployment work myself with Claude than pay for managed everything: the money isn't buying product, it's buying learning. That's tuition. I paid it on droplets back then and I'm paying it on agents now, and I don't know where it goes any more than I did the first time. That has never once made it a bad investment.

[The barrier dropped, the bar rose.](/2026/04/19/no-map-no-magic-prompt/) It has never been easier to get code live. It is exactly as hard as it's always been to make it *good* — that part's still your responsibility. School's open. I don't know why anyone would skip class.

---

So: billing live and dark, catalog cut over, players shipped, coverage at ninety, keys split, buckets versioned, deploys boring, phone armed, fable burning down to the last day. Saturday night the sky did its thing and I watched with a drink in my hand, because that's where fireworks belong — in the sky, not in the pager.

One bang all weekend, and it's this one.

Bang.
