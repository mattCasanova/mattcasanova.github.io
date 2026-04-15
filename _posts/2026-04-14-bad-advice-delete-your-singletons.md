---
layout: post
title: "A Confident Dose of Bad Advice: Delete Your Singletons"
series: bad-advice
tags: [architecture, opinion, testing, mobile]
---

In [the last post](/2026/04/14/bad-advice-roll-your-own-di/) I walked you through how I roll my own dependency injection — an `AppContainer`, services behind interfaces, everything wired through constructors, everything testable because everything is swappable. If you read that post and built the container, this post is the flip side of the same argument:

**Don't use singletons. If you built the container, you don't need them.**

If you didn't build the container — well, now you have two problems.

## What I actually mean by "singleton"

Before anybody yells at me: I'm not saying *"never have one of something."* It can be perfectly reasonable to only have one `APIService` at runtime. Your logger is probably the same logger across the whole app. Services-that-happen-to-be-instantiated-once are fine. I do that in my own code. The `AppContainer` from the last post is literally a bag of *"there's one of each of these."*

What I'm talking about is the **Singleton Pattern** — the capital-S, design-patterns-book version. The anti-pattern. The one with the private constructor. The one with `getInstance()` or `.shared` or `.INSTANCE` on it. The one where the class itself enforces *"there is only ever one of me and everyone everywhere gets the same copy by calling a static method."* The one you can't construct in a test. The one where the internal state is global state wearing a nicer hat.

That's the thing I'm telling you to stop using. Here's the whole take in one sentence:

**Every problem you'd reach for the Singleton Pattern to solve, the `AppContainer` already solved, better.**

The rest of this post is me telling you why the singleton version is a trap — and why my own code has one tiny, very specific exception to the rule that I'll own up to at the end.

## A singleton is just a global variable in a nicer sweater

Here's the thing that kills me about all of this. I used to teach intro CS. And every intro course I ever taught — every intro book I ever assigned — had the same chapter, somewhere in the first third of the semester: **don't use global variables.** They're dangerous. They make code impossible to reason about. They couple unrelated parts of the program. They make testing hard. They hide dependencies. On day one, the student is told: *globals are bad, don't reach for them.*

And then the student graduates. They pick up a design patterns book. They flip to the "creational patterns" chapter. And right there, presented as an **advanced technique** with a capital-letter name, is:

> **The Singleton Pattern** — a class that provides globally-accessible single-instance access to itself.

That is a global variable. That is *literally, exactly, with no qualification,* a global variable. You took the thing we told the student not to do in week one, put a class around it, and now it's an *advanced pattern.* You wrote a chapter about how to do a global variable correctly and charged forty dollars for the book. Or about $32 for the Kindle version.

**I know because I wrote one of these books.** Ten years ago I wrote a book about [design patterns](https://www.amazon.com/Game-Development-Patterns-Best-Practices-ebook/dp/B01MRP7SPA/). Chapter 2 is on the Singleton Pattern. In my (weak) defense, I opened the chapter with a rant on the dangers of global variables, and framed singletons as *"use sparingly, only when you genuinely have exactly one of something."* I tried to be responsible about it. I understand the contradiction I'm talking about in these books, because I wrote one of these books. Past me wrote the chapter present me is yelling about — **everybody grows up sometime, I guess.**

We told everyone not to do this. Then you got handed a bachelor's degree, went to your first job, and were told it was actually an advanced technique. That's where the whole trap starts.

## Testability is dead

This is the biggest one. A pattern-singleton can't be mocked, can't be replaced, can't be reset between tests, can't be swapped for a fake. If your `LoginViewModel` calls `AuthManager.getInstance().login(...)`, you cannot unit test that view model. Full stop. Not *"it's hard"* — you **cannot do it**. The real `AuthManager` hits the real network, reads real secure storage, talks to real auth endpoints, because the real `AuthManager` is the only copy of itself that exists. There is nothing to pass in. There is nothing to swap. Every test you write against that view model is secretly an integration test, whether you wanted one or not.

What people do next is reach for Mockito (we'll [get to Mockito in the next post](/2026/04/14/bad-advice-roll-your-own-fakes/)) and try to paper over the problem at the framework level — class-mocking their way to a green test suite. The tests pass. And then production breaks at runtime because the real thing and the mock don't agree on some subtle detail nobody was paid to audit. But that's another post.

The point is: **a singleton inside a view model makes the view model untestable by any principled means.** You can cheat with class-level mocking, you can refuse to test it, or you can rip out the singleton. Those are your three options. Two of them are bad.

## Flexibility is dead

The Singleton Pattern means there is exactly **one implementation**, forever. Not one instance — one *implementation*. Because it's a concrete class. You can't swap it. You can't have two. You can't test it against a fake. You can't run two variants side-by-side for an experiment. You can't extend it without modifying it. Because it's a concrete class. That's what you signed up for.

Yes, I know about the Kotlin workaround. I've used it. `object UserManager : UserManagerInterface { ... }` — you make the object implement an interface, pass it around as the interface, and pretend you have polymorphism. I've done this more than once. In production.

But here's the thing: if you're doing this, **you're already 95% of the way to getting rid of your real problem. Why not go the other 5%?** If you've declared the interface and the major call sites take it as a parameter, you've already done the hard part. **Instead of making it an `object`, make it a class.** Create it in the `AppContainer`. Hand it down through constructors. That's it. The workaround and the real fix are separated by about an hour of refactoring.

And here's the part the language feature *doesn't* fix, no matter how clever you get with it: **the global name is still visible globally.** Yes, you can pass `UserManager` around politely as `UserManagerInterface` at the call sites you personally control. But `UserManager.currentUserId` is still a perfectly valid line of code anywhere else in the codebase. One file uses the interface like a good citizen. The next file reaches through the global name and parties like it's 1999. They are touching the *exact same object* — and the second file doesn't even know the first file was trying to be clean. With the `AppContainer`, that second file *can't* arbitrarily reach in and grab `currentUserId`. There is no global name to reach through, because the thing isn't global — it's a member of a container you have to ask for. **This isn't about discipline. It's about structure.** Nobody has to trust the next engineer to do the right thing, because the wrong line of code isn't a valid thing to write in the first place.

That's exactly the shape of code that the **Dependency Inversion Principle** — the D in SOLID — tells you not to write: *depend on abstractions, not concrete classes.* The Singleton Pattern is the purest possible violation of that principle. Every caller in your entire codebase depends on the concrete class by name and reaches it through a globally-resolvable method. There is no abstraction. There is no indirection. The class *is* the contract.

## Thread safety is an afterthought

Most singletons aren't written with thread safety in mind. They hold mutable state, that state gets read and written from multiple threads the moment your app is complex enough to care, and the thread-safety story gets bolted on after the fact — `@Volatile`, `synchronized`, a lock around the dangerous method, maybe a whole actor-isolation pass if you're lucky. You *can* make a singleton thread-safe, but you have to actually think about it every time, and **nothing at the call site reminds you to think about it.**

Some languages even lie to you about this. Kotlin's `object` keyword is the clearest example:

```kotlin
object UserManager {
    var currentUserId: Int? = null
}
```

Every Kotlin dev learns that `object UserManager` is *"thread-safe."* And it is — **for construction.** The JVM's class-loading lock guarantees exactly one instance ever gets built, even if a hundred threads race to touch it simultaneously. That's a real win. It's why Kotlin devs don't have to write the double-checked-locking dance that Java singletons needed in the `getInstance()` era.

But that's where the thread safety ends. The `var currentUserId` inside that object is exactly as racy as it would be in any other class. The `object` keyword gave you safe *construction*. It did not give you safe *mutation*. The Kotlin community treats `object` as *the* good singleton — safe, idiomatic, modern — but the JVM fixed the easiest problem (*"don't construct two of them"*) and left every hard one exactly where it was.

Here's the test I'd apply: *if you copy-pasted the body of your `object` into a regular `class`, would it still be thread-safe?* If yes, the `object` version is too. If no, neither is.

The real danger, you know, isn't any single singleton — it's **the habit.** Once you get used to reaching for a globally-accessible bag of state, the reach becomes muscle memory. You start doing it from coroutines, from background queues, from anywhere in the codebase you happen to be, and the compiler never warns you because there's no per-call-site thread-safety check. The audit you should have done the first time becomes a postmortem three years in.

When you finally find yourself doing that audit, the usual advice is to pick a synchronization strategy: `@Volatile`, `AtomicReference`, `Mutex`, actor isolation. Reach for a lock, pay the lock tax, hope you got it right. I'd argue for something else:

> **Instead of picking a synchronization strategy, pick an architecture where you can't screw it up.**

That's the real answer. Not *"reach for a lock"* — *look at every piece of shared mutable state and ask whether it needs to exist at all.* Nine times out of ten it's a cache you never measured, or a field you could replace with a reactive stream, or a piece of session data you could just re-read every time from the thread-safe primitive underneath. Strip it. The lock you were about to add becomes unnecessary. The race goes away because the state goes away.

## `// TODO: refactor this later`

Imagine you're building the Govee app. You know Govee — they make those color-changing LED strips, light bars, and neon panels you see on every streamer's desk. I have six in this room behind me while I'm writing this.

Neon lights are cool.

In year one of this hypothetical, the company has exactly one product: a single smart lamp. Early prototype, gets it out the door, proves the market. Your app pairs with that one lamp over Bluetooth, offers a color picker and a brightness slider, and calls it done. One phone, one lamp, one connection. **That is how anybody would build it.**

Year two, the company ships V2. It's a completely different device — more features, more modes, speakers, whatever. It's not polymorphic with V1; they share almost nothing. So on the app side, you start a new subsystem for the V2 device. New screens, new flows, new code path, entirely separate from the V1 stuff.

And here's where the trap gets laid — quietly, without anybody realizing it (except for Ackbar).

Where do the V2 methods live? Well, there's one V2 device. The code that talks to it is basically *"here's how you do stuff with the V2 device."* So you make a class called `SuperDevice` and fill it with static methods. `SuperDevice.setColor(...)`. `SuperDevice.applyScene(...)`. `SuperDevice.checkFeature(...)`. `SuperDevice.getFirmwareVersion()`. Every V2 screen calls into it. Nobody on the team even calls it a singleton — *it's just where the V2 methods live.* There's one V2 device, so there's one bag of methods for it, so those methods are `static`. It's not an architectural decision. It's not a Gang-of-Four reference. It's just where the code went.

For the next two and a half years, this is the V2 subsystem. The team builds scenes, custom animations, music sync, schedules, rotating palettes, DIY mode, all of it, and every single feature calls into `SuperDevice.something()`. The subsystem works. It ships. Customers love it. Some of them do buy two V2s — nobody really planned for that — and technically the app still works in that case, but there are definitely bugs. It probably always grabs whichever device paired first. Nobody complains loudly enough to get it prioritized, and the team moves on to the next feature.

Then the people who originally wrote it leave the team. One gets fired, one moves to another project, the manager who'd been running the team for the last four years switches orgs. The code stays. New engineers join, look at `SuperDevice`, shrug, and add their new feature as another static method on it, because that's what the surrounding code does.

Then one day leadership drops the roadmap for the next year: **four new devices shipping in six months.** A rope light you can bend around a door frame. A standing wall panel. A smaller bedside lamp with a built-in speaker. A kitchen-cabinet strip. All brand-new hardware, all in the same product family as V2, all expected to support every feature the app already has on day zero.

And this isn't that far off. Every multi-device smart-home company eventually ships a product line instead of a product. That's just what happens when a company stops being one-device and becomes a brand. The business decision is sound. The year-two engineering decision wasn't visibly wrong *when it was made.*

And then someone has to build it.

Here's the first thing you notice: the new devices aren't *different* the way V1 and V2 were different. They're the same class of hardware with per-device quirks. A rope light has different animation timing than a wall panel. A bedside lamp has a speaker the wall panel doesn't have. Each one ships with a slightly different firmware baseline, and some features require specific minimum firmware versions on specific devices. So what you actually need is a `Device` **interface** — a protocol where the device itself knows what it can do, what its firmware is, what features it supports, how to talk to it. Polymorphism. The thing you'd write if you were starting fresh today.

Here's the second thing you notice: **you can't do that.** The entire V2 codebase calls `SuperDevice.setColor(...)` directly. Every one of those call sites has to change. `SuperDevice.setColor(...)` has to become `device.setColor(...)`, where `device` is an instance of the new `Device` protocol, and that device has to be *passed in* to the code path somehow, which means every caller of every caller of every caller has to get a device parameter it didn't have before. The scene editor doesn't take a device. The music-sync loop doesn't take a device. The schedule view model doesn't take a device.

Why would they?

For two and a half years, there was only one freaking device.

Here's the third thing you notice, and this is the one that makes it a rescue op instead of a refactor: **nobody at the company has realized the code is broken yet.** Nobody on the team. Nobody in the org. And on top of that, remember the four-year manager who switched orgs? The person who replaced *them* also just left. This is your *third* manager in about eight months, and they're only two months in. They're still catching up on the backlog, and they've never heard of `SuperDevice`. The new devices don't exist in the wild. The hardware team has prototypes. The firmware team has prototypes. The app team has never plugged one into the app because nobody has asked them to. A random Jira ticket shows up: *"FYI, the new bedside lamp can't run the music-sync feature, don't know if that's intentional."* You open the ticket. You start tracing. You realize, three files in, that it's not that music-sync is broken for the bedside lamp — it's that **nothing works on the bedside lamp.** Or the rope light. Or the wall panel. Or the cabinet strip. The entire V2 subsystem is hardwired, at every call site, to *the* V2 device. There is no "which device" parameter anywhere because there was never any reason to have one. And launch is six months out and marketing already has a product page written.

Ripping it out is either a multi-month project or a rescue operation.

I wouldn't know. This is a hypothetical.

But I can tell you *hypothetically* that if it had happened to me, it might have looked like 80 hours a week for three weeks straight, two engineers, roughly 100 PRs each, war rooms, late-night rollbacks — **and all of it on top of the regular quarterly feature work that didn't pause just because the V2 subsystem was on fire.** The rescue happened on nights and weekends, because the day job kept going.

Hypothetically.

The original authors of `SuperDevice` were long gone. Everyone on the team knew the code was ugly, but nobody owned it — it was the team's code, and everybody was heads-down on higher-priority features until product shipped a roadmap that made it structurally incompatible with the codebase overnight.

*Hypothetically.*

It gets done eventually. The new devices ship. The app supports five. But the postmortem conversation is always the same:

> *"Why was this a singleton to begin with?"*

And the answer is always the same:

> *"Because two years ago, it wasn't wrong."*

## This is not a story about bad engineers

Here's the careful part, and I want to land this hard because I've seen the story told wrong. **Nobody in that story is a bad engineer.**

The person who wrote the first version of `SuperDevice` made a reasonable call. There was one V2 device. The methods to talk to it lived on a class. The methods were `static` because there was nothing to instantiate. They probably didn't even think of it as a singleton — they thought of it as *"here's the V2 subsystem."* The name never came up. Nobody wrote *"let's use the Singleton Pattern"* in a design doc. The pattern smuggled itself in through the shape of the product.

The people who maintained it through years three and four all knew the V2 subsystem was ugly. But by then it was load-bearing — you cannot rip out a god class that's called from hundreds of files while you're also shipping the next feature on a deadline. The people who could see the code was wrong couldn't afford to fix it. The people who could have avoided it in year two had no way to know the product line would eventually become four more devices.

**Everyone in this story was making tradeoffs under real constraints.** Managers pushing deadlines. PMs shipping roadmaps. Growth team running experiments. Design asking for *"just one more screen."* Nobody has infinite time to sit and think about the third-order consequences of a line of code written in year one. That's not how the job works.

**The singleton wasn't a mistake. It was a ticking time bomb with a long fuse, and the fuse was the roadmap nobody could predict yet.**

*Hypothetically.*

The fix isn't *"make smarter decisions."* You don't know enough in year one to be smart about year three. The fix is **build code that can absorb a roadmap change without a multi-month rescue operation.** That's it. That's the whole skill. Everything else in this series — DI, no singletons, no class-level mocking, wrapping third parties — is just tactics for that one strategy.

**Write code that can change.** Services behind interfaces. Dependencies through constructors. Facades over singletons. **Blast radius of one file, not a hundred.**

## The alternative (and my confession)

Instead of reaching for the Singleton Pattern, do what I did in the DI post:

1. Define a service **interface** (`SessionService`, `AuthService`, `DeviceService`, etc.).
2. Put one **implementation** of it on your `AppContainer` as a lifetime-scoped instance. It lives as long as the app, but it's not globally accessible — you have to ask the container for it.
3. If nearly every class needs it and passing it through five layers is genuinely annoying, add a thin **facade** (`Log.e(...)`, `FF.isEnabled(...)`, `Device.current()`) that configures itself from the container at app startup. The facade looks like a singleton at the call site. The thing it delegates to is fully swappable.
4. Tests pass fakes to the container. Production code passes real implementations. Everything is still swappable end-to-end because **the facade itself holds no state** — it just forwards.

I mentioned in the DI post that `Log` and `FF` are technically singletons — static wrappers everyone reaches through. Owning that again here. The reason they're safe isn't anything about the wrapper itself (no state, no logic, one line per method that forwards to an interface from the container). It's that **nothing the call site does mutates shared state.** `Log.e(...)` is fire-and-forget — I dump a message at the logger on whatever thread I happen to be on and never read anything back. `FF.isEnabled(...)` is read-only — a constants lookup that happens to get populated at runtime. Neither one has two threads fighting over anything, and that's the real test for whether a facade is safe: not *"does the wrapper hold state"* but *"does the call site do anything two threads could race on."*

The other thing the DI pattern buys you — and I want to point this out because most people don't notice it — is that **with dependency injection, you can stop using "one of" even for your services, if you ever need to.** Right now I only create one `APIService` on my container because it's convenient and nothing forces me to create more. But there's no architectural reason my `LoginViewModel` and my `DashboardViewModel` have to share the same `APIService`. They're both just making HTTP calls. They don't rely on each other's state. If I ever hit a situation where sharing caused a race or a lock contention issue, I could trivially give each view model its own `APIService` instance — change one line in the container — and the rest of the app wouldn't notice. **That optionality is what DI buys you.** You literally can't do it if you built your services as static methods hanging off a class with a private constructor.

## The Vibe Check

And look — none of this is an original take. Singletons have been called an anti-pattern for twenty years. Every architecture book in the last decade has said some version of this. I'm not here to tell you something you've never heard. I'm here to tell you the version of it that I personally watched cost a team a quarter of engineering capacity. The academic argument is true. The lived argument is worse.

A singleton is a service you were afraid to write an interface for. The moment you put it behind an interface, you've done 95% of the work to not need the singleton anymore.

**The other 5% is deleting the word `static`.**

Thanks for sticking around this long. Next round's on me.
