---
layout: post
title: "A Confident Dose of Bad Advice: Don't Use Mockito"
series: bad-advice
tags: [testing, architecture, opinion, mobile, java, kotlin]
---

> **PRE-DRAFT SEED — not ready to publish.** Placeholder for a future "A Confident Dose of Bad Advice" series entry about why Mockito (and similar class-level mocking libraries) is usually a sign your architecture is wrong.

---

## The seed

The take: **If you need Mockito to test your code, your code isn't testable. Fix the architecture, not the tests.**

Mockito lets you mock concrete classes, not just interfaces. That *sounds* like a feature — "now you can test anything!" — but it's actually the thing that enables people to write non-modular code and then patch testability onto it afterward. The whole reason you'd need to mock a concrete class is that you didn't put it behind an interface. The framework is papering over the architectural decision.

## Things to expand on

### The core argument

1. **Mockito lets you skip the architecture work.** If your classes programmed against interfaces (the way the [DI post](/2026/04/14/bad-advice-roll-your-own-di/) argues they should), you'd never need to mock a concrete class — you'd just pass a fake implementation of the interface. Mockito removes that pressure. It lets you inject concrete classes, then mock them in tests, then call it a day. The code ships. The tests pass. But the architecture stays bad.
2. **The tests you write with Mockito are usually bad.** They test "this method was called with these arguments" instead of "the behavior is correct." You end up with a test suite that asserts implementation details instead of contracts, which means every refactor breaks the tests even when the behavior hasn't changed.
3. **It creates false confidence.** You see 90% coverage, you see green checkmarks, you ship. Then you find out at runtime that the real `AuthService` had a bug your mocked `AuthService` never had, because nobody wrote an integration test. Mockito's convenience made you lazy about integration tests, because unit tests *felt* like they covered everything.
4. **It's addictive.** Once a codebase has Mockito in it, every new test uses Mockito because it's what's already there. Refactoring to remove it becomes a massive project. The rational move — "let's not add Mockito to the new modules" — requires discipline that big codebases rarely maintain.

### The alternative

The pattern I prefer is the same one I argued for in the [DI post](/2026/04/14/bad-advice-roll-your-own-di/):

1. Every service is behind an **interface**.
2. Your production code ships the `DefaultThing` implementation.
3. Your test code ships a `FakeThing` implementation that you pass in directly.
4. No mocking framework required. No reflection. No class-level magic. Just two implementations of the same interface and a constructor.

```kotlin
// Production
val viewModel = LoginViewModel(sessionService = DefaultSessionService(...))

// Test
val fakeSession = FakeSessionService(expectedLogin = someUser)
val viewModel = LoginViewModel(sessionService = fakeSession)
// Now assert against fakeSession's recorded calls or state
```

That's it. The fake has whatever behavior you need. It's code you wrote. It's not magic. The tests are readable because the fakes are readable. And — critically — **the fakes force you to think about the interface's behavior contract**, because you have to implement it.

### The counter-argument I'll acknowledge

The one real case for Mockito is when you're testing code you can't change — legacy codebases, third-party libraries, framework integration points. If you're working on a 15-year-old Java app where ripping out concrete dependencies would take a year, Mockito is a pragmatic choice for *rescuing* testability from code that wasn't designed for it. I've done this. It's fine. But it should be the exception, not the default.

The problem is that Mockito is usually the default, not the exception. Most teams reach for it on day one of a new project because it's what they learned in their last job, and then the architecture never gets the interface-first treatment it needed.

### The Meta angle

My current employer has a general internal guideline against Mockito-style class mocking for new code. Some engineers push back on it, usually because they learned testing that way and the guideline feels pedantic. But the guideline is right, for exactly the reason I've been describing: **it forces you to write testable architecture instead of patching testability on afterward.** The short-term pain of "I can't mock this" becomes the long-term win of "I never had to because it was already testable."

(Note to self: check internal comms policy before using the Meta angle verbatim. Safer version: "some larger engineering orgs have adopted this guideline formally, and I've seen it work.")

### The deeper connection

This is actually the same argument as the [DI post](/2026/04/14/bad-advice-roll-your-own-di/) — and it's the same argument as the next post in this series will be, about singletons. All three are pointing at the same underlying mistake from different angles:

- **Dagger/Hilt + concrete class injection** (the DI post) — skips the interface boundary at wiring time.
- **Mockito** (this post) — papers over the missing interface at test time.
- **Singletons** (the next post) — bakes the missing interface into the architecture so deeply you can't reach it anymore.

All three are the same mistake: **optimizing for the convenience of the person writing code today at the cost of the person maintaining it later.** The fix is the same for all three: program against interfaces, pass fakes in tests, and the "convenience" problems you were trying to solve disappear.

I'll get to singletons next, because they're the root cause of most of the times people reach for Mockito in the first place. When the framework tells you "you can mock that concrete class," that's usually because the class in question is a singleton that the test code can't construct or replace. The mocking framework is treating a symptom. The next post is about the disease.

## Possible closing hook

> If you need Mockito to test a class, ask yourself: what would it take to not need Mockito? Nine times out of ten, the answer is one interface. Write the interface. Delete the Mockito dependency. Watch your tests get shorter.

## Notes

- **Series order:** This is post #2 in the informal arc. Post #1 is [Just Roll Your Own DI](/2026/04/14/bad-advice-roll-your-own-di/), already shipped. Post #3 is the Singletons seed (`bad-advice-singletons.md`), which forward-references as "the next post in this series" from the deeper-connection section above.
- Cross-link to the singletons post as the next in the series when that post ships. Update the forward reference ("I'll get to singletons next") to a hyperlink.
- Back-link from the DI post? Optional. The DI post already works standalone. Might add a "see also" footer to it after the Mockito post goes live.
- Maybe title variants:
  - "Don't Use Mockito" — direct
  - "Mockito Is a Crutch" — punchier
  - "The Mockito Trap" — framing
  - "If You Need Mockito, Your Code Isn't Testable Yet" — thesis as title
- Mention that this applies equally to iOS equivalents (OCMock, Mockingbird, Cuckoo, etc.) — same argument, different language.
- **Drop the Meta internal guideline reference entirely.** Per the blog's current-employer rule, don't mention Meta as a source of anecdotes. The Mockito argument stands without it — it's a general industry guideline at a lot of places, not unique to any one company.
