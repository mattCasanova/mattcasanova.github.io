---
layout: post
title: "Warp Open-Sourced. So What."
tags: [flux, warp, open-source, terminal]
---

I was already wondering if I'd bitten off more than I could chew. Then Warp open-sourced their terminal.

A few weeks ago, in [Standing on the Shoulders of xterm](/2026/04/13/standing-on-the-shoulders-of-xterm/), I wrote that the open-source version of a Warp-shaped terminal could only live in open source — because a VC-backed company can't ship one. Their investors don't want them to.

The other night Warp did the thing I said couldn't happen.

For about an hour I sat with the obvious question. *Am I just wasting my time?* I didn't think they'd do this. I've been blogging about it for a few weeks, and now they've rolled out exactly what I wanted. I'm a guy making a terminal at night. They're Sequoia-backed. I'm nowhere near done. So what am I even doing here?

I came pretty close to closing the laptop.

## What They Actually Shipped

Then I read the license. Then the README. Then the pricing page.

The client codebase is **AGPL-3.0**.[^1] The `warpui` and `warpui_core` crates — their UI framework — are MIT. Everything else, the terminal core, the command system, the editor, the AI bits, is AGPL. The server side, which they call Oz, stays closed-source.

The pricing page is what made the rest of it click. Free at $0. Build at $18/month. Max at $180/month. Business at $45 a seat. Enterprise on request. Every tier is gated on **AI credits, cloud agent runs, and codebase indexing**. The terminal itself has always been free. It's still free. Open-sourcing it doesn't change that, because the terminal was never the business.

Oz is the business. The terminal is how Oz finds you.

That's why the AGPL is on the client. They don't mind individual users running it. They don't mind contributors filing bug reports through the issue-to-PR pipeline they built. What they mind is somebody else forking the client, bolting on a different agent backend, and undercutting Oz. AGPL forecloses exactly that move.

Honest. Internally consistent. Perfectly designed. Just not what I'm building.

## They Kept the Parts I Don't Want to Build

The version of Warp they put on GitHub is not the version I'm building.

They kept the AI. They kept the cloud. They kept the telemetry. They kept the account. They kept the codebase indexing, the conversation storage, the cloud agent runs. All [the things](/2026/04/13/standing-on-the-shoulders-of-xterm/) that keep me from using Warp at work are still in the open-source release, because they're not bugs. They're the product.

I want a terminal that doesn't phone home, doesn't need an account, doesn't try to be an IDE, and doesn't run a cloud agent platform behind every keystroke. I want a fast, local, GPU-rendered, block-based terminal that gets out of your way. Open source. MIT or close to it. No telemetry. No login screen.

That's a different terminal. I just thought, for two weeks, that we were converging on the same one.

We're not. So I'm going to keep going.

## Still Going

Maybe I am biting off more than I can chew. Probably. [Define 'Interesting'](/2026/04/12/define-interesting/) is still true. Vim is still half-broken. The roadmap is still a year long. None of that changed the other night.

The thing I want to exist still doesn't exist.

[This is still heavy](/2026/04/10/building-a-gpu-terminal-in-rust/).

[^1]: AGPLv3 exists because at some point a lawyer looked at GPLv3 and asked *"but what if they just run it as a service?"* and a different lawyer answered *"oh god."*
