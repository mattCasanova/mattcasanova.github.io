---
layout: post
title: "A Confident Dose of Bad Advice: Stop Using Singletons"
series: bad-advice
tags: [architecture, opinion, testing, mobile]
---

> **PRE-DRAFT SEED — not ready to publish.** Placeholder for a future "A Confident Dose of Bad Advice" series entry about why singletons are almost always the wrong answer, and how the facade-over-service pattern gives you the convenience without the pain.

---

## The seed

The take: **singletons are almost never the right call, and when you think you need one, you probably need a service behind an interface with a thin facade on top.** This is exactly the pattern I already use in the [DI post](/2026/04/14/bad-advice-roll-your-own-di/) for logging and feature flags — `Log` and `FF` look like global statics at the call site, but they delegate to real `LoggingService` and `FeatureService` instances that are configured at app startup. You get the convenience. You don't lose testability.

Singletons, by contrast — I mean the *real* ones, with `getInstance()` and a private constructor and state they mutate internally — are a trap. Here's why:

## Things to expand on

### The core argument

1. **Testability is dead.** A singleton can't be mocked, replaced, or reset between tests. Every test that touches code that touches the singleton now has order-dependent state. Your test suite becomes flaky. You start adding `@Before` methods that reach into the singleton's private state to reset it. That's the smell.
2. **Thread safety is an afterthought.** Most singletons aren't written with thread safety in mind. Then the app grows, the codebase gets threaded, and you find out your singleton is sharing state in two places at once.
3. **Architectural lock-in.** The worst one. Once a singleton is load-bearing — once it's being called from 200 files — you can't replace it without touching every one of those files. The singleton doesn't just leak into your architecture, it *becomes* your architecture.
4. **It hides dependencies.** A class that calls `AuthManager.getInstance().getCurrentUser()` doesn't advertise its dependency on the auth system. It just reaches through the singleton. The class's constructor looks clean. The dependency graph looks clean. Until you try to test it.

### The refactor story (anonymized)

Several years ago I worked on a mobile app for a device-focused product. The app had been built around a singleton that represented "the current device" — one singleton, one device type, one set of capabilities, the whole codebase called into it. It worked great when the product only had one device.

Then the product expanded to support multiple device types. And suddenly "the current device" wasn't one thing anymore. It could be any of several device types, with different capabilities, different settings, different lifecycles. The singleton couldn't represent that.

Ripping it out was a multi-month project. Every call site had to be audited. Every class that had reached into `DeviceManager.getInstance()` had to be refactored to take a device through its constructor or a context object. Tests had to be rewritten because you could no longer count on "the singleton is the one thing." It was painful in exactly the way the singleton had made the early code feel convenient — all the shortcut you took in year one came due in year three with interest.

The lesson: **architectural decisions that feel like they save time on day one often cost 100x that much time on day 1000.** Singletons are the canonical example. What feels like "just call this from anywhere" becomes "we can't extend this because it's everywhere."

### The alternative

Instead of a singleton, use the pattern from the [DI post](/2026/04/14/bad-advice-roll-your-own-di/):

1. Define a **service interface** (`DeviceService`, `AuthService`, etc.).
2. Put one **implementation** of it on your `AppContainer` as a singleton-*by-lifetime* (it lives as long as the app, but it's not globally accessible).
3. If nearly every class needs it, add a thin **facade** (`Device.configure(deviceService)` wired in `AppContainer.init`) so call sites look like `Device.current()`.
4. Critically: the facade delegates to the real service, and the real service is swappable in tests.

You get the ergonomics of a singleton. You get the testability of DI. You get the architectural flexibility to swap implementations later.

### Why people reach for singletons anyway

The honest case for why singletons keep happening:

- **Convenience.** Passing a dependency through five layers of classes is annoying. Singleton is one line.
- **Habit.** Language features like `object` in Kotlin or `@MainActor` classes in Swift push you toward singleton patterns by default.
- **Framework encouragement.** Android in particular has historically embraced singletons in its samples (see every `RetrofitClient.getInstance()` tutorial).
- **It's actually fine for the first year.** The pain is deferred. By the time it matters, the person who wrote the singleton has moved on.

That last one is the real killer. Singletons optimize for the person writing the code today. They penalize the person maintaining the code in three years. If you're both those people, you will pay the cost.

## Possible closing hook

Something like:

> A singleton is a service you're afraid to write an interface for. The moment you put it behind an interface, you've done 80% of the work to not need the singleton anymore. The other 20% is deleting the word `static`.

## Notes

- Anonymize the anecdote fully. No product names, no company names.
- Cross-link to the DI post for the facade pattern description.
- Possible alt title: *"A Confident Dose of Bad Advice: The `getInstance()` Trap."*
- Related post: the Mockito one (see `_drafts/bad-advice-dont-use-mockito.md`). Both are about "the framework that lets you write untestable code anyway."
