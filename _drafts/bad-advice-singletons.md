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

1. **Testability is dead.** A singleton can't be mocked, replaced, or reset between tests. If you have a singleton inside a view model — say your `LoginViewModel` calls `AuthManager.getInstance().login(...)` — you can't unit test that view model without actually invoking the real `AuthManager`. The real auth manager hits a real network, reads real secure storage, and talks to real services, because it's the only copy of itself that exists. There's nothing to swap for a fake. Every test that touches code that touches the singleton ends up with order-dependent state, because the singleton is shared across tests. Your test suite becomes flaky. You start adding `@Before` methods that reach into the singleton's private state to reset it. That's the smell — the moment you're manually resetting singleton internals in tests, the architecture has already failed.
2. **Flexibility is dead.** A singleton means *there is exactly one implementation*, forever. You can't swap it. You can't have two. You can't test it against a fake. You can't extend it without modifying it. That's exactly the shape of code that the **dependency inversion principle** (the *D* in SOLID) tells you not to write — "depend on abstractions, not concretions." A singleton is the purest possible violation: every caller depends on the concrete class by name. Contrast that with the service-behind-an-interface pattern from the [DI post](/2026/04/14/bad-advice-roll-your-own-di/): callers depend on `AuthService` (the interface), not `DefaultAuthService` (the implementation), which means you can swap implementations, write fakes, run two variants side-by-side, or change your mind about how auth actually works without touching every file that needs it.
3. **Thread safety is an afterthought.** Most singletons aren't written with thread safety in mind. Then the app grows, the codebase gets threaded, and you find out your singleton is sharing state in two places at once.
4. **Architectural lock-in.** The worst one. Once a singleton is load-bearing — once it's being called from 200 files — you can't replace it without touching every one of those files. The singleton doesn't just leak into your architecture, it *becomes* your architecture.
5. **It hides dependencies.** A class that calls `AuthManager.getInstance().getCurrentUser()` doesn't advertise its dependency on the auth system. It just reaches through the singleton. The class's constructor looks clean. The dependency graph looks clean. Until you try to test it — then you discover the class is secretly coupled to three other systems it never mentioned in its constructor.

### The refactor story (hypothetical, but it rhymes with a thousand real codebases)

Picture a social mobile app — pick any one you use daily, it doesn't matter which. The app launches in year one with one clear concept: **one user, one account, one session.** You sign in, the app knows who you are, everything from the feed to the DMs to the notifications assumes that "you" is a single person represented by a single account.

The engineering team builds around that assumption. They write a `CurrentUser` singleton. It's one line to access from anywhere: `CurrentUser.shared.id`, `CurrentUser.shared.profile`, `CurrentUser.shared.preferences`. Every screen, every view model, every repository reaches for it. The feed pulls posts for `CurrentUser.shared.id`. The DMs sync for `CurrentUser.shared.id`. The notification badge reads unread counts for `CurrentUser.shared.id`. It works. It ships. It's beautiful in its simplicity.

For about two years, this is fine. The product grows, features stack up, but "one user per app" remains true. Maybe 300 files end up calling `CurrentUser.shared` directly. Nobody worries about it. The pattern is idiomatic, every new engineer writes the same code because that's what the surrounding code already does, and the app runs in production for millions of users with no visible issue.

Then product decides the app should support **multiple accounts**. Work account, personal account, side-project account, maybe a burner. Tap the avatar, switch users, see a different feed. It's a common feature add — almost every serious social or communication app eventually ships this. Gmail has it. Twitter has it. Discord has it in the form of servers. The product decision is sensible; the engineering decision on day one wasn't visibly wrong.

And then someone has to build it.

Suddenly "the current user" isn't a coherent concept anymore. What does `CurrentUser.shared` even *mean* when two accounts are logged in at once? The feed screen needs one user. The DMs screen might be showing a different user. The notification center wants to pull unread counts for all of them simultaneously. A singleton can represent "the one thing." It cannot represent "whichever of these things this screen is currently showing."

Every one of those 300 call sites becomes a question. Which user does this code path actually care about? The answer is almost always "the one that's active in the context of this screen or request," but the singleton has been smuggling that context in for years and nobody has written it down. Now someone has to go read every file and figure it out. Most of it is doable. Some of it reveals legitimate bugs where the original code *assumed* one user and would silently break under two. A few lines are genuinely scary because nobody remembers why the original engineer reached for `CurrentUser.shared` instead of passing the user through.

Ripping it out is a multi-month project. Every class that previously called `CurrentUser.shared` now takes a `User` through its constructor or a context object. Tests have to be rewritten because you can no longer count on "the singleton is the one thing." Every feature team in the org has to coordinate because nobody can ship new code against the old pattern once the migration is in progress. There are war rooms. There are 1am rollbacks. There are engineers who have to stop doing their day job and become full-time migration surgeons for a quarter.

It gets done. The app supports multi-account eventually. But the month-three code review of the refactor PR is always the same conversation: *"Why was this a singleton to begin with?"*

And the answer is always the same: **because two years ago, it wasn't wrong.**

Here's the careful part: **this is not a story about bad engineers.** The person who wrote the original `CurrentUser` singleton made a reasonable call at the time. "One user per app" was the actual shape of the product. The singleton matched the shape. Everyone who maintained it in the years after knew singletons had problems in theory, but by then it was load-bearing — you can't rip out a singleton that's called from 300 files while also shipping the next feature on deadline. The people who could see it was wrong couldn't afford to fix it. The people who could have avoided it originally had no way to know the product would eventually support two, then three, then four accounts.

That's the real shape of the lesson: **singletons bake themselves in.** You make a simple, quick decision in year one to save a week of thinking about service boundaries. The decision works. It keeps working. Then two years later the world shifts — the roadmap expands, the product grows, the business model changes, the customer use case broadens — and you discover the simple quick decision is now load-bearing architecture you can't replace without a multi-month project. At no point was anyone "wrong" in a way that would have been obvious at the time. The singleton was just a ticking time bomb with a long fuse, and the fuse was the roadmap nobody could predict in year one.

This exact shape of refactor has happened at every mobile org I know about. Sometimes the singleton is "current user." Sometimes it's "current device." Sometimes it's "current tenant" in a B2B app, or "current environment" in a client SDK, or "current locale" in an internationalized product. The *object* changes. The pattern is identical: **a singleton that matches the product's shape today becomes impossible to extend when the product's shape changes tomorrow.**

### The generalized lesson

**Architectural decisions that feel like they save time on day one often cost 100x that much time on day 1000.** Singletons are the canonical example. "Just call this from anywhere" becomes "we can't extend this because it's *called from everywhere*."

The ones that hurt the most aren't the obviously-wrong decisions — those get caught in code review and fixed early. The ones that hurt are the *reasonable* decisions that match the shape of the product today and don't match the shape of the product tomorrow. The shape of the product changes more often than anyone plans for. Building with the assumption that it won't is how ticking time bombs get shipped.

Prefer the thing that's slightly more work today in exchange for the flexibility tomorrow. Services behind interfaces. Dependencies through constructors. Facades over singletons. These cost you maybe a day of thinking at the start of a project, and in exchange you buy yourself the ability to absorb a roadmap change without a multi-month rescue operation.

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

- **Series order (revised):** This is now post **#2** in the informal arc — moved up from #3.
  - Post #1: [Just Roll Your Own DI](/2026/04/14/bad-advice-roll-your-own-di/) (shipped) — "put services in your AppContainer, wire them through constructors"
  - Post #2: **this post** — "don't use singletons. You don't need them. The DI post already gave you a better version."
  - Post #3: Mockito (see `_drafts/bad-advice-dont-use-mockito.md`) — "Mockito exists to paper over the problems singletons create"
  - Post #4: Third-party wrapping (see `_drafts/bad-advice-wrap-third-parties.md`) — same principle applied to external dependencies
- **The suggested opener** is a forward reference from the DI post, not a back-reference from Mockito: *"In the last post I walked through how to roll your own dependency injection — services on an AppContainer, passed through constructors, everything testable because everything is behind an interface. This post is the flip side of that argument: **don't use singletons.** If you've built the AppContainer pattern, you literally don't need them."*
- **Core move:** frame singletons as "the thing you reach for when you haven't built a service container yet." The DI post gave you the container. This post says: now that you have it, singletons are redundant *and* dangerous.
- **Anecdote is hypothetical.** The `CurrentUser` / social app multi-account story is a hypothetical composite, not a specific employer story. Per the blog's current-employer rule, keep it that way. Do not name Meta or any internal product.
- **Possible alt titles:**
  - "A Confident Dose of Bad Advice: Stop Using Singletons" — direct
  - "A Confident Dose of Bad Advice: The `getInstance()` Trap" — technical
  - "A Confident Dose of Bad Advice: The Singleton Is a Lie" — dramatic
  - "A Confident Dose of Bad Advice: Everything Is a Singleton Until It Isn't" — the punchline framing
- **The blast-radius framing** is the unifying thread across all the series entries: every architectural pattern in these posts is about minimizing the blast radius when something changes. Default parameters on functions, interfaces on services, DI over singletons, wrappers over third parties — all the same principle. "Minimize the blast radius when something changes" could be the tagline for the whole series. Consider opening the post with it.
- **Hit the blast-radius point hard.** The real cost of a singleton isn't that it's ugly today — it's that when you *do* need to change it, you're potentially touching hundreds of files. That's the lesson worth hammering, even harder than the testability point. Call sites pile up for years against the convenience of "just import the singleton," and then the day comes when you need to swap it or add a second variant, and you discover you own 300 files of coupled code.
- Cross-link forward to the Mockito and third-party wrapping posts when they exist.
- **Possible concrete example:** the old William Hill logger story (Sentry, Airship, Firebase, Facebook Analytics — all accessed as third-party singletons, all had to be ripped out or swapped when requirements changed). That belongs in the third-party wrapping post, but can be cross-referenced from this one as "here's the pattern I've seen repeatedly — see the next post for more on this specifically."
