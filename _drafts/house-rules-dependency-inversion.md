---
layout: post
title: "House Rules: D"
series: house-rules
tags: [architecture, solid, opinion]
status: seed
---

## Pre-draft seed — not a post yet

### The hook

This is the shortest post in the series. On purpose.

The Dependency Inversion Principle says: depend on abstractions, not concrete classes. High-level modules shouldn't depend on low-level modules. Both should depend on interfaces.

I already wrote this one. Four times.

- [Roll Your Own DI](/2026/04/14/bad-advice-roll-your-own-di/) — build a container, wire everything through interfaces
- [Delete Your Singletons](/2026/04/14/bad-advice-delete-your-singletons/) — the purest possible violation of DI, and why the `AppContainer` already solved it
- [Roll Your Own Fakes](/2026/04/14/bad-advice-roll-your-own-fakes/) — if you need Mockito, your code isn't behind interfaces
- [Wrap Your Vendors](/2026/04/15/bad-advice-wrap-your-vendors/) — every third-party SDK behind an interface you own

The entire Bad Advice arc was the D in SOLID dressed up in bar-stool clothes. I just didn't call it that until the S post.

Go read those. I'll wait.

### The running tally (the punchline to the whole series)

S: interfaces. O: interfaces. L: interfaces. I: interfaces. D: interfaces. I wrote four entire posts about it before I even started this series.

Five letters. One answer. It's always been interfaces.

### Why this post should be short

The joke IS that it's short. The series builds — S is a full post, O is a full post, L and I are full posts. The reader clicks on D expecting the grand finale and gets "I already wrote this one." The brevity is the punchline to both the House Rules series and the Bad Advice arc.

Don't pad it. Don't add examples. Don't re-explain DI. The four Bad Advice posts are the examples. This post is the bow on top.

### The closing line (the mic drop)

After the tally: "Nine posts. One answer. Different behavior every time. I guess that's polymorphism."

Then: *I didn't write 'em, but those are the rules.*

### Potential title ideas

- House Rules: D (just the letter — the shortest title for the shortest post)
- House Rules: I Already Wrote This One
- House Rules: See Previous
