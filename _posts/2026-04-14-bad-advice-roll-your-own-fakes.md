---
layout: post
title: "A Confident Dose of Bad Advice: Roll Your Own Fakes"
series: bad-advice
tags: [testing, architecture, opinion, mobile, java, kotlin]
---

In the last two posts I told you to [roll your own DI](/2026/04/14/bad-advice-roll-your-own-di/) and to [delete your singletons](/2026/04/14/bad-advice-delete-your-singletons/). This post is what happens if you ignore both of them.

**If you need Mockito to test your code, your code isn't testable. Fix the architecture, not the tests.**

That's the whole take. The rest of this post is me explaining why, and why the fix is almost always *write an interface and a hand-rolled fake,* not *install a framework that lets you pretend the interface exists.*

## I'm not picking on Mockito specifically

Before anybody emails me: this isn't really a post about Mockito. I've used Mockito at multiple companies. I've used [MockK](https://mockk.io) on Kotlin-first projects. Both are well-designed, powerful libraries that deliver exactly what they promise. I'm not calling out any one tool.

The argument I'm about to make applies equally to **MockK** (Kotlin), **Mockingbird** and **Cuckoo** (Swift), **OCMock** (Objective-C), **PowerMock** and **JMockit** (Java), and whatever class-level mocking framework has shipped since I last looked. They're all variants of the same category: *a framework that lets you mock concrete classes, static methods, singletons, and other things you shouldn't need to mock in the first place.*

I'm going to say *"Mockito"* throughout this post because it's the name everybody recognizes. Substitute whatever framework you actually reach for. The argument is the same.

## The core argument

Mockito lets you mock things. Specifically, it lets you mock:

- **Concrete classes** — not just interfaces.
- **Singletons and static classes** — the ones you can't instantiate normally.
- **Static methods and functions** — the ones that don't belong to any instance.
- **Final classes** — with the `mockito-inline` extension.
- **Private methods** — with PowerMock or similar.

That list reads like a feature list. It's actually a confession. Every single item on it is a thing that is *hard to mock for a reason,* and the reason is always the same:

**The production code is tightly coupled to something it shouldn't be.**

- You should never need to mock a concrete class, because your code should depend on an interface, not a class.
- You should never need to mock a singleton, because your code should depend on an interface passed through a constructor, not a static method reached through a global.
- You should never need to mock a static function, because your code should depend on an interface with an instance method, not a free-floating function.
- You should never need to mock a private method because — *seriously, WTF.*

Mockito's whole feature matrix is a list of *"things the framework will help you mock even though the reason they're hard to mock is that they're architectural mistakes."* And the framework's job — whether it meant to or not — is to **remove the pain signal that was supposed to push you toward a better design.**

That's the trap. It's a framework that papers over architectural decisions you should have made differently, and it's so good at papering them over that you stop noticing you made them.

## Mockito lets you skip the architecture work

If your classes programmed against interfaces the way [the DI post](/2026/04/14/bad-advice-roll-your-own-di/) argues they should — interfaces on every service, dependencies passed through constructors, everything wired by an `AppContainer` — **you would never need to mock a concrete class.** You'd just pass a fake implementation of the interface into the constructor. One line. Done.

Mockito removes that pressure. It lets you inject concrete classes, then mock them in tests, then call it a day. The code ships. The tests pass. But the architecture stays bad, because the feedback loop that was supposed to tell you *"wait, I can't test this"* has been short-circuited by the framework. *"I can test it. Mockito says so."*

This is actually a really clever trap if you think about it. Mockito is *doing work.* It's helping. It's solving the problem you asked it to solve. It's just solving the wrong problem — it's solving "how do I test this bad code" instead of "why is the code bad."

## The tests you write with Mockito are usually bad

The typical Mockito test looks something like this:

```kotlin
@Test
fun `loading a user updates the view model`() {
    val mockRepo = mock<UserRepository>()
    whenever(mockRepo.getUser(42)).thenReturn(User(42, "Matt"))

    val viewModel = UserViewModel(mockRepo)
    viewModel.loadUser(42)

    verify(mockRepo).getUser(42)
    assertEquals("Matt", viewModel.userName.value)
}
```

What is this test actually asserting? Two things: *"the view model called `getUser(42)`"* and *"the name came out right."*

The first assertion is the problem. It's testing an **implementation detail.** It's saying *"this code path makes this specific call in this specific way,"* which means any refactor that changes *how* you fetch the user — batching, caching, a different method name, a different signature — breaks the test, even if the behavior is unchanged. The test isn't asserting a contract. It's asserting a paragraph of the current implementation.

You end up with a test suite where every behavioral refactor triggers a cascade of test edits, because half the tests are really asserting *"these methods got called with these arguments in this order,"* not *"this output was correct."* The test suite stops being a safety net and starts being a tax on every change.

This isn't Mockito's fault, strictly. You can write bad tests with any framework. Or without one — you're free to write bad tests all on your own. But Mockito makes *this particular kind of bad test* the natural thing to write. The whole API is designed around `whenever` and `verify`. The ergonomics push you toward testing call patterns instead of behavior, and most teams follow the ergonomics.

## It creates false confidence

You see 90% coverage. You see green checkmarks. You ship. Then production breaks at runtime because the real `AuthService` had a subtle bug that your mocked `AuthService` never had — because the mock returns whatever canned response the test told it to, and nobody was auditing whether the canned response matched the real service's actual behavior.

Mockito's convenience makes you lazy about integration tests, because the unit tests *feel* like they cover everything. They don't. They cover exactly what you told them to cover, which is almost always *"the code path I was thinking about when I wrote the test."* The stuff that breaks in production is the stuff you weren't thinking about.

## It metastasizes

Once a codebase has Mockito in it, every new test uses Mockito because it's what's already there. Every new engineer onboards into the existing pattern and reaches for `whenever` and `verify` because that's what the surrounding tests do. The rational move — *"let's not add Mockito to new modules"* — requires a discipline big codebases rarely maintain. So you don't stop. You add one test. Then five. Then a hundred. The framework starts as one cell in one test file and quietly becomes half your suite.

This is the same shape as the singleton trap from [the last post](/2026/04/14/bad-advice-delete-your-singletons/): once it's load-bearing, you can't rip it out without touching everything. The blast radius grows quietly until the day you need to change it, and then you find out how big it got.

## The Kotlin / Swift split (an aside worth making)

I've worked on a lot more Kotlin/Android projects with heavy Mockito usage than Swift/iOS projects with heavy OCMock or Mockingbird usage. That's not because Swift engineers are smarter or Kotlin engineers are lazier — it's **framework inertia.**

The Swift community has leaned hard on protocol-oriented programming and delegation since the early 2010s. Apple's own sample code uses protocols everywhere. UIKit and SwiftUI are built on delegate patterns. When a Swift engineer wants to do something differently, the default reflex is *"make a protocol, inject an implementation, pass it through the constructor."* That reflex is already pushing you toward testable code without you having to think about it. The testing story kind of comes along for the ride.

Android historically did the opposite. Every `setOnClickListener { ... }` is an anonymous class. Every Retrofit tutorial shows `RetrofitClient.getInstance()` as a concrete singleton. The default idioms shaped the code. And when someone wanted to unit-test a `LoginViewModel` that called `RetrofitClient.getInstance().login(...)`, the answer wasn't *"refactor to take an interface through the constructor"* — it was *"install Mockito, mock the singleton, move on."*

Same engineers, same intelligence, different default idioms. Android got Mockito-heavy because the framework defaults pushed the code toward concrete coupling, and Mockito was the relief valve. Swift got protocol-oriented because UIKit pushed the code toward protocols from day one, and the relief valve was mostly unnecessary.

None of this is a Kotlin-bashing aside. It's the same observation I've been making in this series: **the default path the tooling gives you becomes the code you write.** Fighting the tooling is expensive. Going with it is cheap. The job of a careful engineer is to know which defaults to accept and which to override.

Mockito is a default to override.

## The alternative (it's just DI again)

The pattern I prefer is the same one from [the DI post](/2026/04/14/bad-advice-roll-your-own-di/) and [the singletons post](/2026/04/14/bad-advice-delete-your-singletons/):

1. Every service is behind an **interface.**
2. Your production code ships a `DefaultThing` implementation.
3. Your test code ships a `FakeThing` implementation that you write by hand.
4. You pass the fake into the constructor. No mocking framework required. No reflection. No bytecode manipulation. Just two classes of the same interface.

Here's what a fake looks like:

```kotlin
class FakeAuthService : AuthService {
    var loginCallCount = 0
    var lastLoginEmail: String? = null
    var loginResult: User = User(id = 1, name = "Fake")
    private var _currentUser: User? = null

    override val currentUser: User? get() = _currentUser

    override suspend fun login(email: String, password: String): User {
        loginCallCount++
        lastLoginEmail = email
        _currentUser = loginResult
        return loginResult
    }

    override suspend fun logout() {
        _currentUser = null
    }
}
```

That's the whole file. Maybe 20 lines. It's a real Kotlin class. It implements the real interface. It has observable state the test can assert against directly — `fake.loginCallCount` instead of `verify(mockSession).login(...)`. And when you refactor `AuthService.login` to add a parameter, the compiler tells you the fake needs updating too, because the fake is a real implementation of a real interface and the compiler treats it like one.

Your test becomes:

```kotlin
@Test
fun `login updates current user`() {
    val fake = FakeAuthService()
    fake.loginResult = User(id = 42, name = "Matt")
    val viewModel = LoginViewModel(authService = fake)

    viewModel.login("matt@example.com", "password")

    assertEquals(1, fake.loginCallCount)
    assertEquals("matt@example.com", fake.lastLoginEmail)
    assertEquals("Matt", viewModel.currentUserName.value)
}
```

No `verify`. No `whenever`. No reflection. No `@Mock` annotations. No framework dependency to install. No *"wait, how do I mock a final class again?"* No *"oh right, I forgot to reset the static mock at the end of each test."* Just two implementations of the same interface, passed through a constructor, asserted against by reading fields.

And here's the part I want to land hard: **this is easier than the Mockito version, not harder.** The Mockito version requires you to remember the DSL, install the right artifact, wire the right Gradle plugin, and sometimes debug why the mocks aren't being called in the order you expected.

The fake version requires you to write 20 lines of Kotlin that a first-year engineer could read or write without a tutorial.

## "But writing fakes by hand is tedious"

Yes. There's some boilerplate. Each interface needs a corresponding fake. You write it once.

And here's the part where the calculus has changed in the last two years: **you don't even write it anymore.** You tell Claude *"make me a fake implementation of `AuthService` with call counters for every method and settable return values"* and Claude produces it in two seconds. Same deal with Cursor, Copilot, whatever you're using. Mechanical, pattern-driven code generation is exactly the kind of task AI coding tools are already extremely good at.

I'm not making a lazy joke about this being easier. I'm saying the last remaining excuse for Mockito — *"but hand-writing fakes is tedious boilerplate"* — has collapsed to zero. It was never a strong excuse. It's now a non-excuse.

In the AI-assisted coding era, mechanical boilerplate is free. Architectural decisions are still expensive, because you still have to *know* which decisions to make. The cost-benefit of Mockito has completely inverted: the cost (coupling, brittle tests, refactoring tax, framework lock-in, weird bytecode bugs at runtime) is unchanged, and the benefit (*"saves me from writing fakes"*) has vanished.

Mockito, you're out of business. We didn't need you in the first place. Make like a tree and get out of here.

## The counter-argument I'll barely acknowledge

I'll grant you one thing, grudgingly: **genuine legacy code, in the short term.** If you're working on a 15-year-old Java app where ripping out concrete dependencies would take a year, Mockito is a pragmatic rescue tool for adding testability to code that wasn't designed for it. I've done this. It's fine. It is also the *only* case I'm willing to acknowledge, and even this one has an expiration date.

The other excuses I hear don't hold up:

- *"Third-party libraries."* You shouldn't be unit-testing third-party libraries in the first place. You should be wrapping them behind an interface you own, and your unit tests should talk to the interface, not the library. I'll have a whole post on this next in the series.
- *"Framework integration points."* Why are you unit-testing the iOS framework? Or Sentry? Or Firebase? You're not testing the framework. You're testing *your code that calls the framework,* and that code should be behind an interface like anything else.

So the counter-argument really reduces to one thing: **legacy code that is genuinely expensive to refactor right now.** That's fair. I don't want to pretend otherwise. But *right now* is the operative phrase, and *right now* is changing fast.

Here's the update I'd add to the legacy-code counter-argument in 2026: **with AI agents, "take a year to refactor" has become "take a week" for a lot of cases.** Not all cases. Not every codebase. But for a meaningful fraction of the work I'd have classified as *"too expensive to refactor"* three years ago, the calculus has genuinely changed. Ripping out concrete dependencies, replacing them with interfaces, and hand-writing fakes is exactly the kind of mechanical, pattern-driven work agents are good at. You kick off a refactor PR on Monday, review it on Wednesday, ship it on Friday.

So even the *"it's legacy, we can't"* exception is weaker than it used to be. If you're carrying a Mockito dependency into a new module in 2026, ask yourself: *is this really a legacy constraint, or is it habit?* Most of the time, it's habit.

## The deeper connection

This is the third beat of a three-post argument that all points at the same underlying mistake from different angles:

- **[Roll Your Own DI](/2026/04/14/bad-advice-roll-your-own-di/)** — skipping the interface boundary at wiring time.
- **[Delete Your Singletons](/2026/04/14/bad-advice-delete-your-singletons/)** — baking the missing interface into the architecture so deeply you can't reach it anymore.
- **Roll Your Own Fakes** (this post) — papering over both of the above at test time.

All three are the same mistake: **you're optimizing for time in the wrong direction. The author pays a minute, the maintainer pays a month.** The fix is the same for all three: program against interfaces, pass dependencies through constructors, and the "convenience" problems you were trying to solve never come up.

**Mockito is where the bill comes due.** If you followed the advice in the first two posts — every service behind an interface, wired through constructors, no `getInstance()` anywhere — you *genuinely never need Mockito.* The whole reason you'd reach for it is that you took shortcuts earlier and now need a way around them at test time. The framework is treating the symptom of architectural decisions you made one and two steps back.

And while we're here — a confession from writing this whole series. At some point I started to suspect that **there's really only one design pattern, and everything else is a naming convention.**

- What's the strategy pattern? Interfaces for behavior.
- What's the command pattern? Interfaces for delayed actions.
- What's dependency injection? Interfaces as construction parameters.
- What's the observer pattern? Interfaces with callbacks.
- What's the adapter pattern? Interfaces that translate.
- What's the decorator pattern? Interfaces that wrap.

More than half the Gang of Four book is *"program against an abstraction."* The rest is a list of clever names for specific situations where programming against an abstraction is useful.

If you're not programming against interfaces, you're not writing bad code in some theoretical sense.

You're writing bad code *full stop,* because you are violating the single underlying principle that every other design pattern is trying to give you a nicknamed version of.

Harsh, I know. **The truth hurts.**

## The Vibe Check

If you need Mockito to test a class, ask yourself: *what would it take to not need Mockito?* Nine times out of ten the answer is one interface and a 20-line fake. Write the interface. Write the fake. Delete the Mockito dependency from your `build.gradle`. Watch your tests get shorter, your refactors get faster, and your failure modes get easier to debug.

And if you're on a team where Mockito is already load-bearing, that's fine — start with *"no new Mockito in new modules"* and let the existing stuff decay on its own. You don't have to rip it all out at once. The codebase is allowed to grow up.

Truth hurts. Buy me a drink, I'll tell you another one.
