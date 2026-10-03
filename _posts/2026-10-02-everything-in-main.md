---
layout: post
title: "Everything in Main"
tags: [ai, architecture, gamedev, vulkan, opinion]
summary: "I pointed it at two of my own game engines and told it to use them as the example. It put everything in main. I'm still as pro-AI as DHH and Uncle Bob, and this is the part of the argument nobody on either side is holding."
---

The Vulkan port is underway. Dilithium2D, the 2D engine I [said last week](/2026/09/30/rip-off/) I was writing in C++ for fun, next to LiquidMetal2D.

That's the background. The interesting part is separate, and it starts on Twitter.

### The loud part

I follow David Heinemeier Hansson. He gave a talk recently, and the short version is that he's full-on pro AI for building. So is Uncle Bob. Both of them are out there saying you can just build so much now.

That's exactly where I am. I've been saying it since earlier this year. It's why this blog exists.

And there is so much hate.

Some programmers love this stuff. Some really hate it. I don't know why the split is as sharp as it is. Maybe it's fear of losing the job. I genuinely don't know, and I'm not going to pretend I've diagnosed a few thousand strangers.

What I can see is what most of the hate is pointed at, and it's slop.

### Slop is not the argument

Here's the breakdown I keep running into: people treat pro-AI and pro-slop as the same position.

They aren't. The people most associated with "you can build so much now" are also the people telling you to put guardrails around it. Uncle Bob [spent a whole post's worth of replies](/2026/08/02/nobody-points-at-the-blade/) getting lectured for saying the surplus should go into testing. Testing. That's the opposite of ship-whatever-it-gave-you.

I've written my version of this twice. [If your AI output is slop, the problem isn't AI](/2026/05/01/think-mcfly-think/) was the whole argument. The reason you can even see the slop is that you have taste. That's your taste working. And when I say guardrails I don't mean vibes, I mean [the ladder I spent two weeks building](/2026/07/22/the-rate-went-up/): compiler flags, hooks, gates, a heartbeat. A gate you can't talk your way out of at 11pm.

So nobody in the pro-build camp is saying do slop. That's not a thing anyone is arguing for. It's just the thing that's easiest to argue against.

### I wrote the patterns book

I want to be clear about which guy is making this argument, because I don't think I'm the guy people picture.

I read design pattern books. I [wrote a design patterns book](https://www.amazon.com/Game-Development-Patterns-Best-Practices-ebook/dp/B01MRP7SPA/). I care about SOLID enough that I walked [all five letters](/2026/05/06/house-rules-d/) across this blog one at a time and then kept going into the patterns. I taught this stuff to sophomores for seven years.

And I still think what we can do right now is incredibly powerful.

Those two things have never once felt like they were in tension. If anything it's the reverse: **if you're really good without AI, you should be really, really good with AI.** That's the whole position. Not that the fundamentals stopped mattering. That they're the thing that converts.

DHH said something to the effect that the number of programmers who can write code better than AI rounds to zero.[^dhh] That gets more true with every model. Back in April I put it the other way around: if you think you're better than these tools, [maybe you are, today](/2026/04/11/signal-through-the-noise/). Today is doing a lot of work in that sentence, and it has been doing more of it every month since.

Which brings me back to the engine.

### Everything in main

I have two game engines sitting on this machine. LiquidMetal2D, which I started in 2020 and have been writing about all year. And Mach 5, the C++ engine I wrote for the book, which came out of the engine I taught with.

So when I started the Vulkan port, I pointed at both of them. Specifically. By name. Use these as the example, this is what I'm porting, here's the foundation I want.

It basically did not do that.

It put a massive amount of code in main. Code that should be wrapped in a game engine object. It wasn't separating the layers. It wasn't separating ownership. It was just popping a bunch of stuff down and getting it to work.

No architecture. With two working examples of the exact architecture open on the same disk.

And it's a familiar shape. Everything in main is the same mistake as every iOS and Android tutorial that puts the whole app in the view controller or the activity. Business logic, networking, state, all of it in the one file the framework handed you. [I wrote about those 1,200-line view controllers in April](/2026/04/15/house-rules-the-other-four-letters/), and nobody defends that architecture. It's just what you get when the only goal is making the thing run.

Let me be fair to it, because fair matters here. This was a very quick first pass. I don't even have a triangle rendering yet. I caught it early and I'm tweaking it now, which is the system working the way it's supposed to: it goes fast, I read it, I redirect. That's [the loop I described in April](/2026/04/19/no-map-no-magic-prompt/) and it hasn't changed.

But the thing it skipped is the thing I pointed it directly at. Not a subtle call. Not a judgment at the edges. The foundational layer, the part that decides what the next two years of this engine cost.

And in the same month, on the same engine, it found a faster circle-to-line-segment test than the one I'd written, [fixed a bug I'd personally audited past in April](/2026/09/30/cheaper-and-cheaper/), and did the math on it better than I would have. Individual functions, hot paths, the clever trick: genuinely excellent. The shape of the whole thing: not yet.

### Last call

I'm not saying they'll never do architecture. I'd bet against myself on that. The models keep getting better and I've been wrong about the pace in the optimistic direction, which is a weird place to be wrong from.

But right now, today, on my machine, there is still a pile of work in engineering an app, or a game engine, or a terminal. Somebody has to decide what owns what. Somebody has to decide where the seams go. Uncle Bob says he doesn't read the code anymore, that he works off tools that visualize the structure instead,[^bob] and the interesting thing about that position is that it's still a position *about the structure*. He didn't stop caring about the architecture. He stopped reading the lines.

That's the part that moved. The work of putting the lines down is gone, and I've been saying that since the second post on this blog.

The engineering didn't go anywhere.

See you, space cowboy.

[^dhh]: Paraphrased from memory, so don't quote me quoting him. The number he actually used matters less than the direction, and the direction has been one-way all year.

[^bob]: Which I [flagged back in July](/2026/07/07/blade-runners/) as the end of the spectrum I'm not at. I still read the diffs. I just have the bots read them first.
