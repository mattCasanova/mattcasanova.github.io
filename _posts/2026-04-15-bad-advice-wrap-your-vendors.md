---
layout: post
title: "A Confident Dose of Bad Advice: Wrap Your Vendors"
series: bad-advice
tags: [architecture, opinion, testing, mobile, third-party, logging]
---

This is the fourth and final entry in a four-post arc. The first three — [Roll Your Own DI](/2026/04/14/bad-advice-roll-your-own-di/), [Delete Your Singletons](/2026/04/14/bad-advice-delete-your-singletons/), and [Roll Your Own Fakes](/2026/04/14/bad-advice-roll-your-own-fakes/) — were all about code you write. This post is about code you *don't* write and will never own: the SDKs that show up in your `build.gradle` and `Package.swift` and quietly take over your codebase.

**Every third-party SDK you import should be wrapped behind an interface you control. Your app talks to your interface. The interface talks to the vendor. The vendor never leaks anywhere else in your codebase. Period. No exceptions.**

If you built the `AppContainer` from the DI post, you already know where this wrapper lives: it's just another service, behind an interface, injected like everything else. The wrapper is your code. The vendor's SDK is an implementation detail of the wrapper. Your app treats them as unrelated.

## What I actually mean by "third party"

*Third party* in this post means **anything you didn't write and don't control.** Vendor SDKs. Platform system libraries. Framework-provided storage APIs. Anything that could change under you, at any time, for any reason — and *any reason* includes *business decisions made by people you've never met.*

So when I say "wrap your third parties," I mean all of these:

- **Analytics SDKs** — Firebase Analytics, Facebook Analytics, Mixpanel, Amplitude, Segment
- **Push and engagement** — Airship, OneSignal, Braze
- **Attribution** — AppsFlyer, Adjust, Branch
- **Auth providers** — Firebase Auth, Auth0, Okta
- **Any networking library** — Retrofit, Alamofire, Ktor
- **Graphics and GPU APIs** — OpenGL, Vulkan, Metal, DirectX, wgpu
- **Physics engines** — Box2D, Bullet, PhysX

**Whatever third-party library you're currently using is an implementation detail. The rest of your app doesn't need to know about it, so don't tell them.**

**One caveat before you go too far with this:** don't try to wrap the *platform itself.* You're not going to wrap `UIViewController`. You're not going to wrap Android's `Activity` or its lifecycle-aware `ViewModel`. Those are the scaffolding the platform gives you — the shape of building on top of it, not libraries you'd realistically swap. There's no `UserDefaults`-versus-Core-Data choice for `UIViewController`. It's just how iOS works.

## Marketing Wants a New Tool

Years ago I worked on a mobile betting app that had, at various times, **five different third-party loggers and analytics SDKs:**

- **Urban Airship** — push notifications and user engagement.
- **Firebase Crashlytics** — crash and error reporting.
- **Firebase Analytics** — event analytics.
- **Facebook Analytics** — event analytics.
- **AppsFlyer** — mobile attribution.

None of them were behind interfaces. Each SDK was imported directly into the files that needed it, and each SDK exposed itself as a singleton — `UAirship.shared()`, `FIRCrashlytics.crashlytics()`, `AppEventsLogger.newLogger(...)`, `AppsFlyerLib.shared()` — that you just called wherever you needed it. This is the default shape of almost every mobile SDK. They ship that way because it makes the getting-started tutorial work in thirty seconds. The cost is hidden until the day you need to change something.

When I joined the project, the first two — Urban Airship and Firebase — were already scattered through maybe forty or fifty files each. I hadn't written any of that code. Then, over the span of about eight months, three things happened.

**First, marketing decided to replace Firebase Analytics with Facebook Analytics.** Not because Firebase was broken, but because the Facebook team was offering something the marketing team wanted: better campaign attribution, a dashboard the marketers already knew how to use, whatever. A business decision. Nobody asked the engineering team whether this was a good idea, because it wasn't an engineering decision. It was a marketing decision that happened to require a week of engineering work.

I had to rewrite every call site. Every file that imported `FIRAnalytics` had to be changed to import `FBSDKCoreKit`, rename the method calls, remap the event names, convert the parameters from Firebase's shape to Facebook's shape. I spent several days on grep-and-replace work while trying not to break any of the three hundred actual features of the app.

**Partway through, I wised up.** Instead of finishing the rewrite as a direct call-site-by-call-site replacement, I extracted a `LoggingService` interface, built a thin wrapper around Facebook Analytics behind it, and migrated the remaining call sites to the interface instead of the vendor. Not all of them — just the new ones and the ones I was already touching. Urban Airship and Firebase Crashlytics stayed scattered, because I didn't have the budget to refactor everything at once.

**Then marketing added AppsFlyer.** A few months later, the same kind of conversation: *"we need to track installs for our ad campaigns, here's the new SDK."* This time, because I had already started the wrapper pattern, I added the AppsFlyer implementation to the `LoggingService` fanout in *one place.* The new feature code never touched the AppsFlyer SDK directly. Adding a third analytics tool was genuinely easier than adding the second.

The pattern was obvious by the time I left: **every time a third-party SDK touched more than a dozen files, changing anything about it meant hunting down every call site by hand.**

**A third party's SDK is always one business decision away from being ripped out. And marketing gets what marketing wants.**

## Marketing Wants a New Tool, Part II

When I went to V-Shred in 2019 and got a greenfield Android project, this was lesson number one I brought with me. Before I wrote a single feature, I built a `LoggingService` interface:

```kotlin
interface LoggingService {
    fun event(type: AppEvent, extra: Map<String, Any> = emptyMap())
    fun error(message: String, throwable: Throwable? = null)
}
```

And a corresponding enum of every event the app actually cared about:

```kotlin
enum class AppEvent {
    UserSignedIn,
    UserSignedOut,
    WorkoutStarted,
    WorkoutCompleted,
    VideoWatched,
    SubscriptionPurchased,
    // ... more as the app grew
}
```

**I own these names.** They live in my codebase, not the vendor's. Every vendor has its own naming conventions, and the translation from *my* vocabulary to *theirs* happens exactly once — inside the reporter implementation, and nowhere else. If a vendor renames an event tomorrow, I change one line in one file.

Every view model called `logging.event(AppEvent.WorkoutStarted)` through the interface. Nobody imported a vendor SDK anywhere else.

When Firebase came in later for crash reporting, I added a `FirebaseCrashReporter` implementation — the only file in the codebase that imported `FirebaseCrashlytics`. Zero feature-code changes.

When Facebook Analytics came in later — same marketing dance, new vendor — I refactored the default logging service to hold an array of sub-reporters and fan out each event:

```kotlin
class DefaultLoggingService(
    private val reporters: List<EventReporter>,
    private val crashReporter: CrashReporter,
) : LoggingService {

    override fun event(type: AppEvent, extra: Map<String, Any>) {
        reporters.forEach { it.report(type, extra) }
    }

    override fun error(message: String, throwable: Throwable?) {
        crashReporter.record(message, throwable)
    }
}
```

Each sub-reporter owned the translation from my app's event enum to the vendor's format:

```kotlin
class FacebookEventReporter(private val context: Context) : EventReporter {
    override fun report(event: AppEvent, extra: Map<String, Any>) {
        val vendorName = when (event) {
            AppEvent.UserSignedIn        -> "fb_mobile_complete_registration"
            AppEvent.WorkoutStarted      -> "custom_workout_started"
            AppEvent.SubscriptionPurchased -> "fb_mobile_subscribe"
            // ...
        }
        AppEventsLogger.newLogger(context).logEvent(vendorName, extra.toBundle())
    }
}
```

The Firebase reporter was nearly identical with Firebase's naming conventions. `CrashReporter` stayed a separate interface because Facebook Analytics doesn't do crashes.

**Total effort to add Facebook Analytics alongside Firebase: one new class, one line added to `AppContainer`, zero call site changes in the rest of the app.** Marketing got their new tool in a day. I went back to whatever I was supposed to be working on.

## Where We're Going, We Don't Need Marketing (to Ask)

On the mobile side project I'm working on now, the same principle applies to two separate things.

**The logger** is Sentry. Will it be Sentry forever? Probably not. Vendors change, pricing changes, tooling changes, and the whole point of this post is that *you should not bet your codebase on any single vendor surviving forever.* Every view model in my current app calls `Log.e(...)` through a facade that delegates to a `LoggingService` interface, implemented by a `SentryLoggingService` class that is the *only* file in the entire project that imports the Sentry SDK. The day I want to replace Sentry, I'll write a new implementation class and change one line in the `AppContainer`. Zero `ViewModel`s touched. Zero tests rewritten.

**Storage** is the other one, and it's the example I want you to internalize because it proves the pattern applies even when the "vendor" is Apple or Google themselves.

```kotlin
interface StorageService {
    fun <T> get(key: StorageKey, serializer: KSerializer<T>): T?
    fun <T> save(key: StorageKey, value: T, serializer: KSerializer<T>)
    fun remove(key: StorageKey)
    fun clearAll()
}
```

(The `KSerializer<T>` parameter is just generics plumbing — you can ignore it if you're not a Kotlin person. It's there so I can pass in any serializable type I want instead of having to write separate `getString`, `getInt`, `getBool`, `getObject` methods for every possible type. Kotlin needs it at runtime because of type erasure. Swift handles the same thing with `some Encodable & Sendable`. Same idea, different syntax.)

And `StorageKey` is a type-safe enum of every key the app actually uses:

```kotlin
enum class StorageKey(val key: String) {
    CURRENT_USER("current_user"),
    DASHBOARD("dashboard"),
    VIDEOS("videos"),
    PROGRAMS("programs"),
    FAVORITES_VIDEOS("favorites_videos"),
    FAVORITES_PROGRAMS("favorites_programs"),
    TAGS("tags"),
    STATS("stats"),
    FEATURE_FLAGS("feature_flags"),
}
```

Nobody in the app can pass a typo'd string as a key. The compiler forces you to pick from the enum cases, and the backing string that actually gets written to `SharedPreferences` or `UserDefaults` is an implementation detail that never escapes the storage module. Same principle as the `AppEvent` enum from the V-Shred logging example, applied to storage instead of analytics.

The implementation today writes to **`SharedPreferences`** on Android and **`UserDefaults`** on iOS. Both are the obvious, built-in, *"here's where you put key-value stuff"* APIs for their respective platforms. They're fine for what I need right now.

But I know they won't be fine forever. Schemas will get more complex. I'll want queries I can't express in a key-value bag. I'll hit the size limits. I'll want encryption for some fields and not others. Room (Android), Core Data (iOS), SwiftData (iOS, newer), SQLite directly, or some hybrid — any of these might be the right call in a year, or two, or three. The point is: **the day I decide to swap, I want it to be a one-file change.** The `StorageService` interface stays the same. The implementation gets a new class behind it. Zero `ViewModel`s or repositories know or care.

This is why I keep saying *"third parties"* loosely. `UserDefaults` isn't a third party in the strict sense — it's in the standard library, it was there the day I installed Xcode. But it *is* something I didn't write and don't control, and that's the only distinction that matters for this rule. **Wrap it anyway.**

## Yes, You Are Gonna Need It

Here comes the objection. Purists will tell you this is over-engineering. YAGNI — *you ain't gonna need it.* Premature abstraction. Adding complexity without immediate benefit. *"Don't add an abstraction until you need one."*

And honestly? **That's a decent rule of thumb — for low-level code.** Let me give you two examples from my own projects where YAGNI is exactly right and I follow it religiously.

**In [LiquidMetal2D](https://github.com/mattcasanova/LiquidMetal2D), my 2D game engine, I'm not building a hierarchical `Texture` interface** with a `RemoteTexture` and a `LocalTexture` and an `AtlasTexture` each implementing some grand `ITexture` contract. The texture manager loads an image, stuffs it in a hashmap keyed by the URL, and returns an `Int` — call it the texture ID, or the texture alias — to whoever asked. From the rest of the engine's perspective, a texture is just an int. If you need anything else about it, you hand the manager the int and ask. There's no wrapped object, no polymorphism, no clever factory pattern. It would be *worse* if there were — I'd be writing five files where one int will do.

**Same thing on the side project for video loading.** I'm loading videos from multiple sources — YouTube, Vimeo, maybe Rumble later — and I'm *not* building a `PlayableVideo` interface with a `YouTubeVideo` and a `VimeoVideo` subclass each knowing how to render themselves. I have a single `Video` type with a `source` field, and the one view that displays it does a switch on the source: *"YouTube → use the YouTube player; Vimeo → use the Vimeo player; Rumble → use whatever Rumble does."* A switch statement. Done. Making each video type its own class with polymorphic `play()` methods would be more code, more indirection, and zero benefit, because I have exactly one consumer: the single display view.

Those are cases where YAGNI is correct. I'm not abstracting because I don't need to abstract. The concrete code is cleaner, shorter, and easier to reason about. The day I need to do something different, I'll change the int or the switch, and it'll be a small change in one place.

**But wrapping third-party vendors is different, because it isn't low-level code — it's high-level architecture.** The thing you're abstracting isn't a data structure or a rendering routine. It's a *dependency on somebody else's business decisions.*

This is exactly what the **Dependency Inversion Principle** — the D in SOLID, the same principle from [the singletons post](/2026/04/14/bad-advice-delete-your-singletons/) — tells you to avoid: *high-level modules should not depend on low-level modules. Both should depend on abstractions.* **Your `ViewModel`, whose entire job is coordinating a screen of user interaction, should not depend directly on a very specific logging SDK made by a very specific third party you don't own.** They should meet at an abstraction — an interface you wrote, living in one file on your side of the boundary. That isn't premature abstraction. It's exactly the shape the architecture wants.

And it isn't premature for two more reasons that can't wait.

**First, you need it from day one to test your code.** The moment you write a view model that calls `FirebaseAnalytics.logEvent(...)` directly, you have a view model you can't unit-test. Every test of that view model will trigger a real Firebase call, which will either fail (CI has no credentials) or pollute your real analytics dashboards with fake data from test runs. Writing the interface isn't abstracting something you don't need yet. It's creating the seam where a fake implementation slots in for your tests. You need that seam the first day you write a test, which is the first day of the project. (For more on fakes, see [the previous post](/2026/04/14/bad-advice-roll-your-own-fakes/).)

**Second, consider the cost.** Twenty lines of Kotlin today versus fifty files of refactor later. That's actually the exact cost-benefit math Fowler uses in [his YAGNI article](https://martinfowler.com/bliki/Yagni.html) — the question isn't *"do I need this right now?"* It's *"what's the cost to build it now versus the cost to add it later if I'd skipped it?"* For wrapping a vendor, the cost-to-build-now is maybe twenty lines. Five minutes by hand, thirty seconds with Claude — *"give me an interface that wraps Sentry with these four methods, plus the default implementation and a fake for tests,"* and you have all three files before you've finished your coffee. The cost-to-refactor-later is fifty, a hundred, sometimes hundreds of files. **The YAGNI framework itself, run honestly, says wrap the vendor.** The only way the math comes out the other direction is if you're certain you'll never replace this specific vendor, and over eighteen months of shipping, I've never seen that bet pay off. **The interface is cheap. The rewrite is not.**

Because the vendor is going to change. Always. SDKs break. Prices triple. Products get deprecated. Companies get acquired. App store rules shift. *Something, eventually, will make you want to replace this specific vendor.* When that day comes — and it comes for every app that ships for more than eighteen months — **you want to change one file, not a hundred.**

That's not premature optimization. **It's optimization based on years of experience doing the wrong thing.**

## The deeper connection

This is the fourth post in a four-post arc that all keeps pointing at the same underlying idea from different angles:

- **[Roll Your Own DI](/2026/04/14/bad-advice-roll-your-own-di/)** — minimize the blast radius of how services get wired together.
- **[Delete Your Singletons](/2026/04/14/bad-advice-delete-your-singletons/)** — minimize the blast radius when a service's shape has to change.
- **[Roll Your Own Fakes](/2026/04/14/bad-advice-roll-your-own-fakes/)** — minimize the blast radius of bad architectural decisions by not letting tests paper over them.
- **Wrap Your Vendors** (this post) — minimize the blast radius when *someone else's* code changes.

All four are the same principle: **make change cheap.** That's the whole game. If you can change X in one file instead of a hundred, your code is good. If you can't, it isn't. Every architectural decision in your codebase should be judged against that question — *when this needs to change, how many files will I touch?* — and the answer should almost always be a small number.

The reason wrapping vendors is the fourth post and not the first is that it's the most *visible* version of the problem. Bad DI is invisible until your first test. Singletons are invisible until year three. Mockito-infested tests are invisible until your first refactor. But **third-party SDKs announce their problems in the release notes.** Every time the vendor ships an update, every time the pricing page changes, every time the app store rejects something, you get a visible reminder that the vendor is not on your team. The third-party wrapping principle is the easy sell, because the pain is obvious.

It's also the version of the argument that's hardest to hand-wave away. *"Yeah, but I don't need DI"* is a line I've heard. *"Yeah, but our singletons are fine"* is a line I've heard. *"Yeah, but Mockito is just how we test"* is a line I've heard. Nobody has ever said *"yeah, but rewriting eighty files every time our analytics vendor changes is fine."* Nobody.

## The Vibe Check

You don't own the third party. You own the wrapper. **Act like it.**

Every SDK you import is a tenant, not a homeowner. It's renting space in your `build.gradle`, in your binary, in your mental model. It pays rent by solving some problem you couldn't solve yourself. And like any tenant, it will eventually move out — because its pricing changed, because its parent company got acquired, because the platform rules changed, because your business decided the other vendor is better this year.

Never let a tenant own the building.

Write the interface. Wrap the SDK. Ship the one file. When the tenant moves out, all you touch is the lease.

That's the arc. Tab's on me. See you at the next round.