---
layout: post
title: "Make It a Good One"
tags: [career, ai, opinion, industry, claude-code]
---


Quick frame before we start. This one's about AI and uncertainty. Before anyone emails me to accuse me of hand-waving the fear from somewhere safe — I'm not. I'm right there with you. I'm on the front lines.

I work at a company that's done layoffs before. I don't know when or if the next round is coming. That's not paranoia — that's the current state of tech jobs.

It's not a secret I work at Meta. I've [mentioned it before](/2026/04/14/the-guy-whose-name-i-dont-remember/). As a rule, I don't talk about where I work.

But...Meta's been in the news again — first a [NY Post report](https://nypost.com/2026/04/17/business/meta-to-cut-8000-jobs-in-major-bloodbath-next-month-report/) about cuts, then a [NYT follow-up](https://www.nytimes.com/2026/04/23/technology/meta-layoffs.html) where supposedly a spokesman confirmed. To be honest, I don't really know. These people have papers to sell and I'm just a lowly engineer who gets news from the news, not Zuckerberg. Maybe the reports are fake. I don't know. Maybe I'll be out of a job in the next month.

And it's not just me, anyway.

My brother got laid off and it took him nine months to find another job. Nine. Months. Comp is compressing. Hiring is brutal. New-grad pipelines are clogged. The fear is rational. I'm not going to spend a single sentence pretending it isn't.

I've been doing this for 20 years and I feel it too.

Here's why I think the math is still in our favor anyway.

## 🔥 This Is Fine. 🔥

I want to make a concession upfront, because most "AI is going to be fine" posts skip this part and they sound naive because of it.

This shift is genuinely different from the past ones.

When the desktop world tipped over into the internet world, you could opt out. The C++ game-engine vet kept doing C++ engine work. Blizzard shipped Warcraft 2 and the Diablo games with Battle.net wired in — most engineers there never touched the networking layer. Skills transferred. Nobody at id Software in 1996 had to learn JavaScript to keep their job. The web grew up next to the old stack, not on top of it.

Same when web tipped into mobile. The web dev kept shipping JavaScript and CSS. The backend dev kept shipping APIs. iOS and Android were *new* careers next to the old ones. You could pick your stack and stay there for a decade.

AI is not that.

> *AI is not just a new technology that sits on the old tool stack. It's a peer to the entire stack. A tool to run the tools.*

Which means there's no neighboring stack to escape into. AI runs across all of them. The C++ vet, the iOS dev, the React engineer, the data scientist, the SRE — they're all using the same set of tools at the editor level now. The shift from desktop to internet was a big shift in industry. It wasn't a big shift in how we worked. AI is.

That's why "not adapting" lands harder this time. There's nowhere to fucking go.

## Mo' Tools, Mo' Programmers

It is different. But it's also similar. There's nuance here.

Here's the thing every panician misses. Every new tool unlock in this field's history has gotten the same panic and created *more* programmers, not less. Assemblers were going to make coders obsolete. The now-millionaire Googlers who started there in the early days would probably disagree.

I wish I was that kind of obsolete.

Compilers, same panic, same outcome. IDEs were going to make real programmers disappear. Stack Overflow was going to turn juniors into copy-paste machines (Okay, that one happened). The cloud was going to kill ops. Frameworks were going to let "anyone build an app." Every wave got the same headlines, and each one created *more* engineers, not less.

I know, because I taught some of them.

Here's a Matt Casanova original:
> **Work expands to fill capabilities.**

It's a cousin of Parkinson's Law, but Parkinson's is really about being lazy — work expanding to fill the time because you screw around.

Mine's about ambition.

It's the whole reason every "this tool will replace us" panic has been wrong. When tools get better, ambition gets bigger. The size of the goal moves up to match what's possible. Do you really think Elon Musk is going to sit around swimming in his money? He isn't Scrooge McDuck. He's going to build more shit.

Take games, since that's where I came from. John Carmack built Doom — a 2D rasterizer hacked into faking 3D. Then he built Quake, a real 3D engine running in software on consumer CPUs. Michael Abrash co-wrote the algorithms and assembly. Both still badasses. Insane work.

Genius work.

Then 3dfx shipped the Voodoo. OpenGL and Direct3D matured into gaming standards. Hardware acceleration went from luxury to baseline.

*Why didn't graphics quality for first-person shooters end with Quake? Why would companies pay for graphics engineers and graphics specialists when the tools made it easier than ever?*

And they did. Because we fucking wanted better games.

Every improvement in tooling didn't *cap* the ambition — it *raised* it. We got Half-Life. We got Halo. We got Call of Duty. We got studios with hundreds of engineers shipping bigger, more complex games every cycle.

I think about this every time I see a tweet that goes *"why would I learn programming, AI does it for me."*

Well, you could argue *"why would you learn graphics? DirectX does it for you."* But you still fucking need graphics knowledge.[^2]

I learned assembly. I taught it for a couple of years. I never programmed in assembly. The compiler optimizes it better than I would and that's fine — I moved [up the stack](/2026/04/11/syntax-is-the-least-of-our-skills/). *Did the assembly programmers vanish?* No. There are probably still some out there, but most of them just switched to solving new problems. Whenever a new tool makes today's hard thing easy, we're back at the edge. New problems to solve.

That's where AI lands.

> *AI is not reducing the number of hard problems. It's solving today's hard problems and exposing a whole bunch more new ones.*

Let me preempt the utopian critic, because they're going to show up in the comments.

Short term, you will not be able to type *"make me a GTA 6 killer with perfect architecture and the most fun gameplay ever"* and have it poof into existence. The hardness moves. It doesn't vanish. What gets hard becomes scope, taste, judgment, the *what* and the *why* of building something.

And even if you *could* one-shot it — what about the guy who says it in twelve sentences? Or spends a weekend prompting it? Or six months?

You don't think that effort makes a better game? Of course it does.

## The Math: Floor Zero, No Ceiling

Here's the math that anchors this whole this.

When AI doubles a company's output — and yeah, *doubles* is a hand-wavey, whatever — the company has two paths. Fire half the engineers and stay flat. Or keep everyone and double the shipped product.

The first path locks in the floor. The second reaches for the ceiling.

For a company, the floor is zero. You can only cut expenses to zero. That's nice, but that's it. That's the bottom. There is no ceiling. There's no cap on revenue. There's no cap on what a useful product can do at scale. And every company I've ever worked at has had more roadmap than engineers.[^1] The bottleneck has always been *let's build it faster,* never *I don't know what to build next.*

So when you frame the choice that way — bounded loss versus unlimited gain — path two wins more often than the panic narrative suggests. *Especially* in a tech industry that's been trained for two decades to chase upside.

Same math, applied to a person:

> **Quit or sprint.**

Floor: quit. Walk away. The bottom of "what could happen to me as a programmer" is zero. You lose your job. You're never a programmer again.

Ceiling: sprint at these tools harder than the person sitting next to you. The person who actually internalizes the new tooling has no upper bound on how productive they can become. There is no ceiling.

Same shape. Two altitudes.

I'll be honest, though. Companies, especially big companies where there could be a lot of waste or redundant work, *will* lay off in the short term. I'm not denying that. We are all living it.

It's also not always the company's fault. We're in tough economic times. Wartime, not peacetime. In peacetime, a big company can absorb projects that aren't hitting and people who aren't pulling their weight. In wartime, that absorption stops. Not every project that worked last year still works. Not every employee who coasted still can. That's not personal. That's the math.

That doesn't mean every layoff is about pulling your weight. Sometimes the project gets cut. Sometimes there's misalignment between you and leadership. Your job is to minimize that misalignment.

The argument isn't that nobody gets cut — it's that long-term, the productive humans get *kept*, not cut. The companies that chose path one (fire half, stay flat) are the ones that stop being interesting to work at, stop shipping, and eventually get out-shipped by the ones that chose path two.

## Hedging in Real Time

So what do I do about it?

I already said. Sprint.

Here's the bet I'm actually staking on myself. I read the news about my employer. I don't know if it lands on me. I might be writing this about a job I no longer have by the time you read it. None of that is in my control.

What is in my control is what I do this month, this week, today.

Here's the way I think about it. There was a moment in the early 2000s when MP3s broke open, and the record labels who refused to adapt — who insisted the future was still going to be CDs, still going to be physical retail, still going to be the same business they'd run for forty years — those labels went down. Not because the music industry died. The music industry is huge. They went down because they bet on the wrong stack.

I don't want to be a record label.

> *I don't want to be the one losing my job because I refused to adopt AI.*

So I sat down a while ago and did the lopsided math.

If AI turns out to be a stupid fad, I lose maybe two months of effort using it. I go back to the thing I've been doing for 20 years and I'm fine. I lose nothing. The downside is bounded. Closely floored.

If AI is *not* a fad — and look, it's not — then I gain knowledge and experience in a brand new field while it's still early enough that "I've been using this seriously for a year" actually means something. I gain a new identity in the market. The upside is unlimited.

Same bounded-loss / unlimited-upside math from the company-level argument, applied to me personally. Same shape. The floor is zero. There is no ceiling.

But here's the truth.

That might not fix it this time. I might still be out of a job.

> *But what's better — out of a job with no AI experience, or out of a job potentially looked at as an AI expert in the next hiring round, with a fresh set of apps, an awesome 2D game engine, an open-source GPU-accelerated terminal emulator in the works, and a blog where I rant about random shit?*

That's the question that resolves the whole thing for me. The bet isn't just about keeping the current job. The bet is about *what I'm worth in the market the day after the current job ends.* The math on that is unambiguous.

I can't control headcount. I can't control comp compressing. I can't control hiring being brutal. I can't control whether my employer is in next week's news cycle.

I can control how much I do.

What that looks like in practice: at work, I'm a heavy AI user. Multiple Claude instances running at once. Internal posts about what I'm building. The leadership push at my company is to use these tools, and I use them — both because I'm getting paid to and because I genuinely don't want to be the holdout.

I've written about [my workflow](/2026/04/19/no-map-no-magic-prompt/) elsewhere — Claude, Superwhisper, multiple sessions in parallel. It's a force multiplier.

At home, same thing. This blog. A website. iOS and Android apps. LiquidMetal2D, the game engine I'm reviving. Flux, the terminal emulator I have no business writing. I have Claude running across all of it.

This blog is part of the bet. 21 posts in 12 days. I'm hedging in real time.

## Trust Me Bro

I want to ground all of this in something that already happened, because the "quit or sprint" math isn't theoretical for me. I've lived a version of it.

I came out of seven years of teaching computer programming with a [published book](https://www.amazon.com/Game-Development-Patterns-Best-Practices-ebook/dp/B01MRP7SPA/) and a stack of side projects. I thought my resume read like *senior engineer with a teaching layer on top.* It read more like *has never had a real job.* Hiring managers don't have time to pick apart the nuance.

So when I moved to Vegas to start an industry career, I got negotiated down to a junior-level offer. Lower pay than I wanted. *Way* lower than what I thought my experience justified.

I took it.

> *I can't make people pay me a 100k a year for my first industry job because, hey, trust me bro, I'm really good. I had to prove it.*

I crushed that job. The other iOS guy quit on a Friday — didn't get fired, but he could feel it coming — and I picked up iOS *and* Android (existing apps, not greenfield) on the next Monday. Started fixing bad patterns immediately. Worked it for 18 months. Asked for a raise. Got nothing, or close to nothing. Then I went and got a much bigger raise from the next company.

That's *quit or sprint* in real life. The floor was zero. I sprinted. The ceiling moved.

I've told the longer version of that story before — [The Guy Whose Name I Don't Remember](/2026/04/14/the-guy-whose-name-i-dont-remember/) — different angle, same biography. The point here is just the math. I've already taken the bet that this post is asking you to take. It works.

## Build It Yourself

Here's the part of the AI argument I haven't seen anyone make crisply.

When everybody can ship a working thing, niche work that wasn't viable before is viable now. Two-person dev shops can realistically service ten or twenty clients each. Custom websites for clients who used to settle for Wix templates can be built from scratch for the same money. The work didn't go away — it just used to be invisible because the unit economics didn't work.

Now they do.

Slots at larger companies may have gone down a little — that's the panic everyone's reading. But the *total* number of people earning a living writing software is going up, not down. The floor for *what's worth building* just dropped, and the ceiling on *what one person can ship* just raised.

If you can't get the W-2 you wanted, the answer isn't necessarily *panic.* Sometimes the answer is *build the thing yourself for the people who'd pay for it.*

This is going to be a massive amount of hard work. Well, welcome to adulthood.

That's what wartime means.

Nobody said war was easy.

## The Holdouts

I have to balance the optimism with a note about the people I see *not* taking the bet. I think this is the actually-load-bearing part of the post, because the math doesn't work unless you do the work.

I have several friends who've been laid off in the past year or two. Some of them are still out of work months later. When I check in with them, I ask the same questions I'd ask anyone. *What side projects are you running? Are you using AI for anything? Have you read about the new stack everyone's pivoting to?*

The answers I get are often dismissive. One reply, in a text, was something like *"use the right tool for the right job"* — said in a way that translated, in context, to *"I'm going to keep using C++ for everything forever."* Which isn't a tool philosophy. It's a refusal.

I have a nephew who recently spent a long time looking for his first programming job. I'd ask his dad — my brother — the same questions. *Side projects? AI?* The answer kept coming back: no, no, no.

I'm not naming names. I love these people. The pattern is across generations and it's going to sound harsh: *the people I know who aren't adapting are the people I know who are out of work.* I don't think that's a coincidence. The hiring market right now will absolutely cut you a check if you can show up with current skills. It will absolutely not call you back if your last serious learning was [*Head First Design Patterns*](https://www.amazon.com/Head-First-Design-Patterns-Brain-Friendly/dp/0596007124/) from 2004.

This is not about being a genius. It's about typing the [fucking question](/2026/05/01/think-mcfly-think/). There has never been a cheaper way to learn. There has never been a more interactive way to learn. If you're not doing it, the only person responsible for that is you.

## Make It a Good One

There's a line at the very end of *Back to the Future Part III* that I think about more than I expected to when I first heard it. Doc Brown, who's just gone back to the future for the last time on his impossible time-train, leans down to Marty and says:

> *Your future is whatever you make it. So make it a good one.*

It's the most engineer-coded line in the trilogy. And I think we're at one of those moments where it lands literally.

I work at a company that does layoffs. I might be next. Comp is down. Hiring is brutal. None of that is in my hands. The shift in what "programming" even means is going to keep accelerating, and I don't get to vote on the speed.

What I get to vote on is how I show up to it. The bet I take. The side project I start. The way I sprint. The question I ask.

Make it a good one.

[^1]: The pushback I expect on the doubling-productivity math is: *"that only works if there's backlog."* If your TAM is exhausted, doubling engineer output doesn't double revenue. Saturated SaaS, mature markets, you name it. The cleanest version of the critique: *the bottleneck wasn't engineers anyway.* Fair. In the textbook sense.

    But in 10 years of working in this industry, I've never seen a company with real backlog discipline. The "bottleneck wasn't engineers" critique assumes a company that knows what it wants, scoped properly, with tradeoffs negotiated up front. None of the companies I've worked at did that. What actually happens: leadership picks a deadline without asking the engineers. The plan changes mid-build. The deadline doesn't. Logging falls off. Tests fall off. Tech debt accrues for years. The refactor that everyone agreed to do "next quarter" never happens. The system slows down until it needs a massive rescue, which sucks, takes forever, and re-creates the same conditions on the other side.

    So even if "double productivity ≠ double output" is true in theory, in practice the gap is narrower than it looks. The hidden-tradeoff tax was already eating the difference. (Counter-counter, in the spirit of fairness: the PMs and leadership also believe AI is magical, so the deadlines are getting *shorter*, not longer. The pressure isn't gone. It's reshaped.)

[^2]: You'd better know when the homogeneous coordinate is 1, or perspective doesn't work. You'd better know an orthonormal matrix's inverse is just its transpose, or your normals get screwed up and your inverse kinematics will crawl. That's the kind of stuff DirectX can't have for you.
