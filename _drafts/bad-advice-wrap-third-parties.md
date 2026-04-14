---
layout: post
title: "A Confident Dose of Bad Advice: Always Wrap Your Third Parties"
series: bad-advice
tags: [architecture, opinion, testing, mobile, third-party, logging]
---

> **PRE-DRAFT SEED — not ready to publish.** Placeholder for a future "A Confident Dose of Bad Advice" series entry about why every third-party SDK and every storage concept should be wrapped behind an interface you own, so the blast radius of any change is one file instead of a hundred.

---

## The seed

The take: **Any third-party SDK you import — analytics, logging, crash reporting, feature flags, storage, networking — should be wrapped behind an interface you control. Period. No exceptions.** The third party gives you their API. You wrap it in your own service interface. Your app talks to your interface, not to the third party. When the third party changes, or you swap it out, or you need to add a second one alongside, you change one file instead of a hundred.

Same principle as the [DI post](/2026/04/14/bad-advice-roll-your-own-di/) and the singletons post: *minimize the blast radius when something changes.* This is the same argument, applied to stuff that isn't yours.

## Things to expand on

### The William Hill story

Years ago I worked at William Hill on a mobile app that had, at various times, **multiple** third-party loggers and analytics SDKs all active at once. Off the top of my head I remember:

- **Sentry** — for crash and error reporting.
- **Airship** — for push notifications and user engagement tracking.
- **Firebase Analytics** — for event analytics.
- **Facebook Analytics** — for the ad-attribution side of the house.

None of them were hidden behind interfaces. Each SDK was imported directly into the files that needed it, and each SDK exposed itself as a singleton — `SomeThing.getInstance()` or `SomeThing.shared` — that you just called wherever. This is the default shape of almost every mobile SDK. They ship that way because it's easy to adopt in 30 seconds, but the cost is hidden until the day you need to change something.

Over my time at the company, we ended up:

1. **Swapping one analytics provider for another** because a business decision or a pricing change or a privacy requirement shifted. Every call site had to be rewritten. Not complicated work, just *tedious* — you'd grep for `FBAnalytics.log(...)`, you'd find 80 places, you'd rewrite every one.
2. **Adding a *second* analytics provider alongside the first** (because some team wanted different metrics in a different tool). Now every event needed to be logged twice. You either go update all 80 sites *again*, or you wrap it in a helper, at which point you might as well have wrapped it in an interface to begin with.
3. **Removing one we no longer wanted.** Same tedious pattern.

By the time I left, the pattern was obvious: every time a third-party SDK touched more than a dozen files, *changing anything about it was a week of tedious grunt work.*

### The V-Shred lesson

When I went to V-Shred in 2019 and got the greenfield Android project, this was lesson #1 I brought with me. Before I wrote a single feature, I built a `LoggingService` interface and routed every logger call in the app through it. On the inside, the service fanned out to Firebase and Facebook Analytics (and later other tools), but on the outside, every view model, every repository, every error handler just called `logging.event(...)` or `logging.error(...)` and that was it.

The result: when we added a new analytics dependency, I updated *one file.* When we changed how errors were tagged with the current user, I updated *one file.* When we wanted to route different severity levels to different backends, I updated *one file.* Every feature team could add instrumentation without knowing or caring what was on the other side of the interface. That small investment — maybe a half-day of work at the start of the project — paid for itself the first month.

This wasn't a full DI container yet (that came on the current project), but the principle was there: **the logger is yours, not the third party's.** Everything the third party provides is an implementation detail of your wrapper.

### The current project: wrapping storage

On the mobile app I'm working on now, the same principle applies to storage. I have a `StorageService` interface that looks something like:

```kotlin
interface StorageService {
    fun put(key: String, value: String)
    fun get(key: String): String?
    fun remove(key: String)
    // ... plus typed helpers for ints, bools, objects via serialization
}
```

The implementation today writes to **SharedPreferences** on Android and **UserDefaults** on iOS. Both are the obvious, built-in, "here's where you put key-value stuff" APIs for their respective platforms. They're fine for what I need today.

But I know they won't be fine forever. Room (Android) or Core Data (iOS) might be the right call once the schemas get complicated. Or I might want to swap in something like SQLite directly. Or encrypted storage for some fields. The point is: **the day I make that call, I want it to be a one-file change.** I want my `StorageService` interface to stay the same, and the implementation to swap out behind it, and zero view models or repositories to know or care.

Same principle. Same payoff. Your app talks to `StorageService`. `StorageService` talks to whatever is actually right today. When right-today changes, the contract with the rest of your app doesn't break.

### The generalized principle: minimize the blast radius

This is the thread that runs through the whole "A Confident Dose of Bad Advice" series:

- [**Roll your own DI**](/2026/04/14/bad-advice-roll-your-own-di/): minimize the blast radius of how services get wired together.
- **Don't use singletons**: minimize the blast radius when a service's shape has to change.
- **Don't use Mockito**: minimize the blast radius of bad architectural decisions by not letting tests paper over them.
- **Wrap third parties** (this post): minimize the blast radius when someone else's code changes.

Every design pattern in software is fundamentally about one thing: **making change cheap.** That's it. That's the whole game. If you can change X in one file instead of a hundred, your code is good. If you can't, it isn't. You can judge any architectural decision by asking: *when this needs to change, how many files will I touch?* The answer should almost always be a small number.

The function default parameter is the tiniest example of this principle. You have a function with five parameters. You want to add a sixth. You give it a sensible default. Every existing call site keeps working. Zero-file change for the callers. That's the ideal shape, extended to everything you build. Every service, every SDK, every storage mechanism — if you can change it without touching the callers, you win.

### Why this is "bad advice"

Because purists will tell you this is over-engineering. They'll say "don't add an abstraction until you need one," "YAGNI," "you're just making your code more complicated for no immediate benefit." Those arguments aren't wrong in general. Premature abstraction is real, and I've seen it bite.

But for **third-party SDKs specifically**, the abstraction is free. The interface is one file. It takes 30 minutes to write. The `LoggingService` interface in my current project is 20 lines. The `StorageService` interface is 15. Neither of them "abstracted something I didn't need to abstract yet" — they both paid for themselves immediately by making the implementation swappable in tests.

The half-day you spend wrapping third parties today is buying you a one-file migration path when the third party inevitably changes. Third parties *always* eventually change. They update their SDKs in breaking ways. They get deprecated. They get bought by a bigger company and the pricing triples. They get blocked by app store policy changes. They add features you don't want or remove features you depend on. Something, eventually, makes you want to replace them.

When that day comes — and it comes for every app that ships for more than 18 months — you want to change one file, not a hundred.

## Possible closing hook

> Every third party you adopt is temporary. The interface you wrap it in is permanent. Build accordingly.

Or:

> You don't own the third party. You own the wrapper. Act like it.

## Notes

- **Series order:** This is post **#4** in the informal arc.
  - Post #1: [Just Roll Your Own DI](/2026/04/14/bad-advice-roll-your-own-di/) (shipped)
  - Post #2: Singletons (`_drafts/bad-advice-singletons.md`)
  - Post #3: Mockito (`_drafts/bad-advice-dont-use-mockito.md`)
  - Post #4: **this post**
- **Series tagline:** "minimize the blast radius when something changes." This could be the unifying phrase across all four posts. Consider adding it as a callback to the series home page or the first paragraph of each future post.
- **Tool names confirmed:** Sentry (errors), Airship (push), Firebase Analytics (events), Facebook Analytics (ad attribution). If memory surfaces additional tools from that era, add them when expanding the draft.
- **William Hill is past-employer territory** — fair to name per the blog's current-employer rule. V-Shred same.
- **Do not name Meta or any current-project stack in detail.** Per the current-employer rule.
- **Drop Meta entirely from the `StorageService` section** — the current project example can be described as "a mobile app I'm working on" without attribution. Reader doesn't need to know it's for my day job or a side project.
- **Possible alt titles:**
  - "A Confident Dose of Bad Advice: Always Wrap Your Third Parties" — direct
  - "A Confident Dose of Bad Advice: Never Import an SDK Directly" — stronger thesis
  - "A Confident Dose of Bad Advice: You Don't Own That Logger" — the ownership framing
  - "A Confident Dose of Bad Advice: Wrap It Before You Ship It" — punchy
- **The blast-radius framing should close the post.** Tie it back to the rest of the series. This is where the "minimize the blast radius" tagline gets its cleanest articulation.
- Consider a concrete numbers callout if I can find one — e.g. "swapping analytics tools at William Hill would have touched roughly 80 files; if it had been behind a wrapper, it would have touched 1."
