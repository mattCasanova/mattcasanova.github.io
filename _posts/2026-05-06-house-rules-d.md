---
layout: post
title: "House Rules: D"
date: 2026-05-06
series: house-rules
tags: [architecture, solid, opinion]
---

*SOLID series: [S](/2026/04/15/house-rules-the-other-four-letters/) → [O](/2026/04/17/house-rules-whatever-happens-happens/) → [L](/2026/04/22/house-rules-it-compiled-it-lied/) → [I](/2026/05/05/house-rules-i-suppressed-the-linter/) → **D***

This is the shortest post in the series. On purpose.

The Dependency Inversion Principle says: depend on abstractions, not concrete classes. High-level modules shouldn't depend on low-level modules. Both should depend on interfaces.

I already wrote this one.

Four times.

- [Roll Your Own DI](/2026/04/14/bad-advice-roll-your-own-di/) — build a container, wire everything through interfaces.
- [Delete Your Singletons](/2026/04/14/bad-advice-delete-your-singletons/) — the purest possible violation of dependency inversion, and why the `AppContainer` already solved it.
- [Roll Your Own Fakes](/2026/04/14/bad-advice-roll-your-own-fakes/) — if you need Mockito, your code isn't behind interfaces.
- [Wrap Your Vendors](/2026/04/15/bad-advice-wrap-your-vendors/) — every third-party SDK behind an interface you own.

The entire **Bad Advice** arc was the D in SOLID dressed up in bar-stool clothes. I just didn't call it that until the S post.

Go read those. I'll wait.

## The running tally

S: interfaces. O: interfaces. L: interfaces. I: interfaces. D: interfaces.

Five letters. One answer. It's always been interfaces.

I wrote four entire posts about it before I even started this series.

Nine posts. One answer. Different behavior every time.

I guess that's polymorphism.

---

*I didn't write 'em, but those are the rules.*
