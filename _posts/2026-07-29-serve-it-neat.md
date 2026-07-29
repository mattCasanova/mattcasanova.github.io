---
layout: post
title: "Serve It Neat"
tags: [laravel, php, blog, workflow, ai, opinion]
summary: "A blog post about building a blog, which I admit is ridiculous. The part underneath isn't: the machine kept leaving me gaps, and the gaps built a second website. Then the CMS turned out to be a folder of markdown with nothing added to it."
---

This is a blog post about building a blog.

I know how that sounds. Give me a second, because the blog isn't the interesting part.

The interesting part is why it exists at all, and the answer is that I wasn't doing anything.

### The gaps

Everybody's story about building with AI is the speed. Mine too. I've written about it... more than once.

Here's the part that doesn't make the highlight reel: a lot of that speed is spent sitting there (Or could be spent sitting there), waiting for the Agent to finish.

First, don't do that shit (I'll talk about this more in another post).

Second, this usually isn't features. Features, Especially features you're iterating on, come back fast. And there will be a new page to view or button to press

What takes long is the heavy stuff. [Tearing out how errors move through two apps](/2026/07/11/carry-that-weight/). [Dragging Larastan up another level, ratcheting every linter I own](/2026/07/22/the-rate-went-up/). Those runs go long. And there's nothing for me to do while they go, because the whole point is that I read the diff and run the new tests at the end, not the middle.

So I get a gap. Too short to open something real. Too long to stare at a progress line.

Which sounds like it contradicts [what I said last week](/2026/07/23/nightcap-back-of-house/), where I was very clear that I'm the bottleneck. The slowest part of the slow part, a guy pressing yes.

Both are true, and I don't think that's a contradiction so much as a queue. The work arrives in bursts. When web, mobile, and this site all come back at once, I'm the jam. When all three are still running, I'm a guy standing at the rail with nothing in front of him.

Running three at once was supposed to be the fix. It mostly is. It just doesn't get rid of the gap, it makes the gap unpredictable.

You can't schedule around that. You can only keep something small in your pocket for when it happens.

### What was in my pocket

I've been running. To keep that honest I had a plan written down: what the next run is, what the week looks like, whether I actually did it.

Grok built the first version. Then Claude rebuilt it in HTML so it didn't look like a spreadsheet's ugly cousin. It lived in my Dropbox as a file I opened every morning to see what I was supposed to do that day.

That was fine. It worked for months.

Then I had a gap, and the gap turned into: I own [mattcasanova.com](https://mattcasanova.com/). Bought it years ago. Never put a single thing on it.

I said [six days ago](/2026/07/23/nightcap-back-of-house/) that it was about to become a little running tracker. It's live now. It's a Laravel app on one small box in AWS, and the whole thing is fourteen days old.

The tracker is genuinely just a workout log. Nothing clever.

The part I want to talk about is the thing I built next, in the next gap.

### Everything on the shelf was too big

I wanted [a blog](https://mattcasanova.com/blog) on it.

The options are all fine. WordPress on Bluehost, five minutes, done, and now I'm running WordPress, which is a large amount of machine for the job of showing people some paragraphs. I looked at Ghost, which is genuinely nice, and it wants to live on its own subdomain because it's a whole separate app. Reasonable. Also a second thing to run, patch, and worry about, sitting next to the app I already had.

But none of that was really the problem, because I wasn't shopping for a CMS. I was shopping for a workflow, and I already knew exactly which one, because I'm using it right now to write this.

I ramble. Claude finds the shape. I edit until it sounds like me. It comes out as a markdown file, the file goes in a folder, the folder goes to git.

There is no admin panel in that story. There's no login. There's no editor with a toolbar. There's no fucking spam bots giving you garbage or porn links in the comments.

It's just a folder.

That workflow is Jekyll, and Jekyll is Ruby, so it was never coming to a Laravel app. Is there a Jekyll for Laravel? Sort of. Kind of. There are packages. Mostly what I found was: **Laravel can parse markdown.**

Which reads like a non-answer right up until you stop wanting a product.

### A folder and a for-loop

One service, one controller, some Blade. It landed in a day, and I added no new dependencies to do it, because Laravel already parses both markdown and YAML.

There is no build step, no cache, and nothing in the database. Not "a lightweight schema." Nothing. The `blog` table does not exist, because there's no reason for it to. Publishing is `git mv` and a deploy.

One thing I did on purpose: the blog is served as plain Blade, deliberately **outside** Inertia. The rest of the site is Inertia and Vue and I like it there. But a crawler doesn't run JavaScript, and a blog's second job is to be read by things that aren't people. And maybe get traffic.

So the blog is a full HTML document, server-rendered, with the meta tags in the markup where a bot can see them without executing anything. Same domain, same Tailwind tokens, different renderer.

It even runs the same Tokyo Night palette as this site. Different face on the type, same room.

### Last call

So: I built a blog. There's no framework in it and no database behind it, and the honest summary is that a folder of markdown was always enough.

That's the whole thing I learned, and it's older than I am. The CMS was never load-bearing. It was the file system with a login page bolted on.

No mixer, no garnish, nothing underneath it pretending to be more than it is.

Serve it neat.

The other reason I'm writing this down is that it's about to get used twice. The site I don't talk about here yet needs a blog too, and this one is small enough that porting it is mostly a copy. That's the second time it pays, which is [what a dividend actually looks like](/2026/07/22/the-rate-went-up/).

And none of it would exist if the machine were slower.

Not because it made me fast. Because it kept walking away, and left me standing at the rail with a gap to fill and a domain I'd never used.

See you, space cowboy.
