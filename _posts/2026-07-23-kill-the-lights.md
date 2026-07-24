---
layout: post
title: "Kill the Lights"
tags: [architecture, testing, ios, android, swift, kotlin, opinion]
summary: "In April I told you not to wrap the platform. This week I wrapped the platform. The rule was fine; the test I gave you for applying it was wrong, and the thing that proved it was a simulator that has never once lost signal."
---

In April I wrote four posts telling you to put everything behind an interface, and then I put a caveat on the last one.

> **One caveat before you go too far with this:** don't try to wrap the *platform itself.* You're not going to wrap `UIViewController`. You're not going to wrap Android's `Activity` or its lifecycle-aware `ViewModel`. Those are the scaffolding the platform gives you, the shape of building on top of it, not libraries you'd realistically swap.

I still believe most of that. This week I wrapped a piece of the platform my own caveat said to leave alone, and I'd do it again.

### The Rule was Good, until I couldn't test it.

Look at what that caveat actually uses to decide: *libraries you'd realistically swap.* Would you ever replace this with something else? No? Then don't build the seam.

Think about this for connectivity. On iOS you ask `NWPathMonitor`. On Android you register a default network callback.

There is exactly one way to ask the OS whether you're online, I'm going to use it, and I am never writing a second one.

Chances of swapping it for something else? Zero.

By that test, a `ConnectivityMonitor` protocol is a waste of time.

Here's the part that stings. In [that same post](/2026/04/15/bad-advice-wrap-your-vendors/), a few sections down from the caveat, I wrote this about why you wrap anything:

> **First, you need it from day one to test your code.** [...] Writing the interface isn't abstracting something you don't need yet. It's creating the seam where a fake implementation slots in for your tests.

Two arguments, one post. The swap argument and the test argument.

The swap argument is the one everybody remembers, mine included, because it's the one that sounds like architecture. It has a business case in it. It comes with a story about the vendor who changed their pricing.

The test argument is the one that actually decides.

I gave you both and then went and applied only the first.

### You can't wait for a blackout

Unit tests never made me care about any of this. Test a view model, hand it a mock API service, and connectivity never walks into the room. That seam was one floor up and I already had it. Fine.

Then [end-to-end tests started to matter](/2026/07/22/the-rate-went-up/), and end-to-end is a different animal. It runs the real app, on a real emulator, against the real OS.

And the emulator has never once lost signal. It sits on a machine with better internet than my apartment...or exactly as good as my apartment since that is where I am testing.

So: does the offline banner come up? Does the app hold, and then refetch when the signal comes back? None of that behavior is driven by a request failing. It's driven by one boolean that the connectivity monitor publishes. Which means testing any of it requires that boolean to say what I want, when I want it. Offline at cold start. Online three seconds later. On command.

The real monitor will not lie for me. It reports the honest truth about a machine that is always online, which is exactly correct for real users but exactly wrong for the test I need to run.

You can't wait for the power to actually go out to find out whether the bar handles it. You put in a breaker you can throw yourself.

### Same pattern, opposite reason

So the protocol went in, and the fake behind it is the entire point:

```swift
@MainActor
final class FakeConnectivityMonitor: ConnectivityMonitor {
    private(set) var isOnline: Bool
    let updates: AsyncStream<Bool>

    func start() {} // no OS to observe; state is set explicitly

    func setOnline(_ online: Bool) {
        guard online != isOnline else { return }
        isOnline = online
        continuation.yield(online)   // drives the banner and the reconnect
    }
}
```

That's the breaker. `setOnline(false)` and the whole app believes the world went dark.

And notice the motive has flipped completely. Storage gets a protocol because the implementation might change: `UserDefaults` today, Core Data tomorrow, maybe a local store and a cloud store behind one service the day after. Connectivity gets a protocol because the implementation *never* will. I will not ship a second `ConnectivityMonitor`. I built the seam so a test could ship one.

Same pattern. Opposite reason. Worth knowing which one you're acting on, because only one of them tells you when to stop.

Which finally explains what `URLSession` was doing in my head all along. I told you not to bother wrapping it. I was right, and the reason I gave was wrong.

It isn't that it's OS-level. It's that my API service was already covering it. Every call in the app goes through an `APIService` that I wrote and put behind a protocol months ago. Test a view model and I hand it a fake API service, and no network happens at all. Test the real API service and the entire point is that it really talks to the internet.

There is no test I want where a real API service sits on top of a fake `URLSession`. I suppose I could build one. I have never once wanted it.

So `URLSession` didn't need a seam because I already had a seam above it, at the layer where I'd actually want to lie.

Connectivity had nothing above it. The boolean comes off `NWPathMonitor` and goes more or less straight to the thing that decides whether the banner is up. No layer of mine in between. Nowhere higher to stand.

That's the difference, and it has nothing to do with which side of the OS line either one sits on.

### One panel for the whole floor

Rather than special-case connectivity, I put the whole OS-bound set behind one provider. Same three properties everything else in the app has to go through the front door for:

```swift
@MainActor
protocol OSServices {
    var connectivity: ConnectivityMonitor { get }
    var secureStorage: SecureStorage { get }
    var storage: StorageService { get }
}
```

Secure storage is the middle case, incidentally. Exactly one implementation, forever, same as connectivity, and it needed the seam anyway because no test is touching the real keychain. Location goes in there the day I need location.

The DI container takes an `OSServices` with the real one as the default, so production passes nothing and gets the truth, a unit test passes a mock and never touches the OS, and a UI test running the actual app gets real storage with a fake network. Which is the combination that started all of this.

### There is no test build

Here's the problem, and it's the one that actually ate the evening.

I want the fake OS in UI tests. *Only* in UI tests. When I launch the emulator by hand on a Tuesday night I want the real radio, the real keychain, the truth.

A debug build I'm poking at is not a test run and it should not behave like one.

But both platforms hand me exactly two environments. Debug and release. There is no test environment.

And `#if DEBUG` only ever answers one question: how was this compiled. Both of my situations compile the same way. The UI test runner launches a debug build. I launch a debug build. Same answer both times.

That was never the question I needed answered. I don't need to know how it was built. I need to know who's driving.

I could go build that third environment myself. A separate configuration on iOS, another build variant on Android, and now the toolchain has a third column forever: one more scheme to maintain, one more leg on the CI matrix, one more thing to be quietly wrong for six months.

I thought there was a better way.

So: `isUITest`, set by the harness at launch and off everywhere else. Debug builds go on behaving like debug builds until a test says otherwise.

Then I sat and looked at what I'd just built. A runtime switch that swaps my app's entire platform layer out for fakes.

In a debug build that's the feature. In a shipped build it's a hook nobody should ever be able to reach, sitting in the binary anyway.

*It's behind a flag* is a promise. I've watched what happens to promises in this codebase when [I'm not the only author](/2026/07/11/carry-that-weight/).

The breaker was in. It needed a lock.

### Make the panel not exist

iOS has a real compile-time conditional, so the flag gets declared inside one:

```swift
enum AppEnvironment {
    #if DEBUG
    static var isUITest: Bool {
        ProcessInfo.processInfo.environment["PRPILATES_UITEST"] == "1"
    }
    #endif
}
```

In a release build `isUITest` doesn't exist as a symbol. Not disabled. Absent. Reference it from anywhere outside a `#if DEBUG` and you don't get a warning, you get a failed release compile.

Kotlin has no preprocessor. `BuildConfig.DEBUG` is a runtime constant, and the symbol still exists in the release build, which is precisely the thing I said wasn't good enough. But Android has build-type source sets, the same mechanism that hands you a different drawable per density or a different string per locale. Code in `src/debug/` compiles into debug builds only.

So the seam lives there:

```kotlin
// shared/src/debug/.../TestSeam.kt  (DEBUG ONLY)
object TestSeam {
    var isUITest: Boolean = false
    var connectivityOverride: ConnectivityMonitor? = null
}

internal fun testConnectivityOverride(): ConnectivityMonitor? =
    if (TestSeam.isUITest) TestSeam.connectivityOverride else null
```

With one catch: production code in `src/main` calls that hook, so the *function* has to exist in both variants or main won't compile for release. `src/release/` gets a stub hardcoded to nothing.

What's not in the release file is `TestSeam` itself. Main only ever calls the hook. Touch the object from the wrong side and the release variant has no such symbol.

Which makes verifying the whole thing take no cleverness at all:

```
swift build -c release
./gradlew :app:compileReleaseKotlin
```

If either one reached the seam from somewhere it shouldn't, that build fails. It doesn't. So the test hook is provably absent from what ships, and not because I remembered to check.

Same move as the error refactor two weeks ago, one floor down. A rule in my head holds while I'm the one typing. A rule in the toolchain holds for everybody, including the authors who generate code by the pound and never came to onboarding.

### Last call

I've shipped apps that handled a dropped signal badly. Not dramatically. Just badly: the spinner that spins forever, the screen that comes back empty and stays empty, the retry firing into a void. Nobody catches it in the office, because the office has wifi. You find out from somebody on a train, in a basement gym, in an elevator, and by then it's been broken for months and it was never once broken on your desk.

That's the class of bug that only exists in a state you can't reach on purpose. Which is the actual lesson here, and it's bigger than one monitor.

Wrap more than you think you should. But change the question. *Will I ever swap this* is the one that sounds smart, and it's the wrong one to lead with. Try the other one:

**If I need this thing to lie to my app, where do I stand?**

Then check whether you already own something at that spot. For the network I did, and that's why skipping `URLSession` was the right call for a reason I couldn't have told you in April. For connectivity I owned nothing, all the way down to the radio.

When the answer is nowhere, you need a seam, and it doesn't matter one bit that you'll only ever ship a single implementation. Build it. Then take the switch out of the wall before the customers show up.

Kill the lights.
