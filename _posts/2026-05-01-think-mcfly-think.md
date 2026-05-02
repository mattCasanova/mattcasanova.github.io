---
layout: post
title: "If Your AI Output Is Slop, the Problem Isn't AI"
tags: [career, ai, taste, claude-code, linchpin]
---

This post was originally going to be about education versus certification. About a mentor I worked with. About centaur chess. About a tutor in your pocket. There were seven drafts in seven different shapes — too aggressive in one, too academic in another, too scattered in the next.

Maybe I'll get to all that in other posts. They're real ideas. They just weren't this post.

What's left, eight drafts in, is the only thing that mattered: shipping something I care about that I also think is good. Caring is one axis. Thinking it's good — actually noticing — is the other.

That's the post.

Two weeks ago I started the first draft of *[No Map. No Magic Prompt.](/2026/04/19/no-map-no-magic-prompt/)* — a piece that argued mechanics got cheap with AI and taste got valuable, and that the only defensible position now is integrating the two. That post pulls on Seth Godin's *[Linchpin](https://www.amazon.com/dp/B003NX762Y)* for the broad thesis: do art — taste is the part AI can't do for you.

This post is about the diagram in that book.

---

## The Diagram

I've mentioned *Linchpin* on this blog before. It's one of my top five favorite books. I've read it — and listened to the audiobook — a couple dozen times at this point. The thesis is roughly this: *do unique art instead of mediocre obedience.* Make yourself the indispensable one in the room because of what only you can bring, not because of what you were told to do.

Nobody's making you care. Nobody's making you notice. In a normal job, maybe that doesn't matter to anyone but you. But when layoffs come, you'd better hope your boss thinks you do. And if you're starting your own thing, your customers had better think you do too.

Toward the end of the book Godin draws a 2x2. Two axes. Two questions:

> *Do you care?*
>
> *Do you notice?*

The bottom-left is the long-suffering middle of every org chart — didn't notice, didn't care, kept getting paid. The top-left is the cynic with taste who never ships. The bottom-right is the energetic doer who builds the wrong thing fast. Top-right is the *Linchpin* — the one who notices what's worth doing and cares enough to do it.

If you don't care, you're replaceable. If you don't notice, you're replaceable by someone who does.

Caring is whether you'll show up. Whether you'll come back to the blog, the GitHub account, the thing you're making — day after day, week after week. It's not just office work. It's the thing you keep returning to. The cost of caring just got dramatically cheaper, because AI handles the mechanical parts. *No Map* covers that half.

Noticing is whether you can tell. Some code is better than other code. Some terminals are better than other terminals. Some drafts are better than other drafts.

That part didn't get cheaper. It might never.

---

## Do I Care?

Yes. That's why I keep coming back. I have apps to make, games to make, terminals to make, blogs to make.

The cost of executing the technical part collapsed. What used to take me a weekend takes a night. What used to take a month takes a weekend. So I can run a portfolio of things instead of having to choose one. That's the only reason this blog exists — twenty-some posts in two weeks, all dictated, all edited fast.

(This week was shit for blog posts, by the way. That's how work goes sometimes.)

I care because I spend hours explaining architecture to AI before a single line of code gets written. If you didn't care, you could get an approximate version of an app or a website in a couple of hours. It wouldn't be good.

I care because I'll yell at AI to do the architecture I know is better — because I've been doing this for a long time. I want the plan right first. Then I look at what AI gives back, and I redirect.

I care because I wrote four posts of [*Bad Advice*](/series/bad-advice/) and I'm working through the SOLID principles one letter at a time over in [*House Rules*](/series/house-rules/). Clearly I care, because I think that shit matters.

That doesn't mean I'm good. That just means I care.

---

## Do I Notice?

Yeah, I think so. That's why this is the eighth draft, and why you won't see the other seven.

Each of them looked like a blog. Each had some good lines. Each came from my words run through an AI agent and back as well-edited prose. And every single one, as I read it, felt like shit. I'd give notes. Update the structure. Ask for the voice to tone down so I could say *fuck* more. Ask for the voice to come back up. Ask to start over from the seed.

Eight drafts. This is the one that lives because I threw seven away and just started talking.

I have enough taste to know V1 was slop. That doesn't mean I have enough to know V8 isn't.

But it isn't just the drafts. I notice my stuff is shit, generally. I notice I could constantly do better. I notice — quietly, without calling anyone out — that other people are struggling too. I don't say anything because everyone is under some kind of constraint. Maybe other people notice my stuff sucks and don't tell me, for the same reason. That's life.

Code, after all this time, I can mostly tell. I worked at a job once where our funnel logging was producing bad attribution data across web and mobile apps. We couldn't reliably tell where users were entering the product, which is the whole point of having funnel logic in the first place. I went looking. The entry-point enum was being passed around as nullable across ten function-call layers, all the way down to a logger that did a null check at the bottom and wrote "unknown" when it failed.

There were two ways to fix it. The lazy fix was to keep the parameter nullable and add better fallbacks at the logger. I'd done that fix before, somewhere else. It works the day you ship it, and then a new caller gets added without passing the parameter, and your data quietly drifts.

The real fix was to trace the nullability back to its source — a deep link parser that returned null when the URL didn't match — and make the parameter required at every layer in between. Multiple days of work. Multiple call sites at every layer.

A lot of people had worked on that code before me. They didn't notice, or didn't care, or both. The growth team was making product decisions on numbers we knew were wrong. I made changes like this in multiple places across the codebase, trying to clean up the numbers. I wanted to be different.

Design is a different story. I can't shoot pictures worth shit. I can't do angles or rule of thirds worth shit. I can't shoot video worth shit. I'm trying to study marketing. I'm trying to study video.

I notice that I suck. That's a start.

---

## If Your AI Output Is Slop, the Problem Isn't AI

All of this is to say: AI slop isn't a reason not to use a new tool.

The pushback I hear from some stubborn engineers goes: *"AI gives me slop. So I won't use AI."*

The reason any of us notice AI slop is that we have *taste.* We can tell. The first version of a bunch of stuff is going to suck — that's not AI's fault. That's your taste, working.

Here's the part you need to understand, and nobody likes hearing it.

If your AI built shitty architecture, the problem wasn't the AI. The problem was you. If your AI built a shitty skeleton of a blog post and you shipped that shitty skeleton, the problem is you.

That's it.

The problem isn't AI. The problem is you have bad taste.

This post is the demonstration of the alternative. (Maybe. It actually probably still sucks.) AI did most of the typing across the eight drafts. I did the noticing. If the noticing axis still fails on V8, I'll find out from a reader who catches what I missed, and V9 is the next thing on the disk.

That's the loop. Care enough to ship the thing. Notice the parts that suck. Care enough to fix them. Run it again until both axes agree.

---

## How Do You Like Them Apples?

The first seven drafts of this post had a *Good Will Hunting* opening. The kernel was good. The execution was shit.

Here's the kernel. Will cared enough about learning to do it on his own — without the credentials, without the $150K, without the lecture halls.

Late charges at the public library, a buck fifty, all the same education. The line still hits because the bottleneck wasn't access. The bottleneck was whether he was willing to walk in.

It's not about access anymore. You have access — to AI, to the docs, to all of it. The only question left is whether you'll show up.

The newbie who treats AI as a magic-eight-ball — type a wish, accept the answer, ship the slop — loses, because they can't tell the slop. The newbie who treats AI as a relentless tutor — *why did you do that, what if I'd done this, write me a worse version and tell me why it's worse* — that's the one who eats the world.

Same applies to the pro. The pro who treats AI as a magic-eight-ball is the friend who hasn't worked in nine months. The pro who keeps asking and keeps noticing is the one shipping at midnight on a Tuesday.

Is this AI slop? Probably. I'm shipping it anyway.
