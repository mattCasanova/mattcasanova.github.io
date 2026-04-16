---
layout: post
title: "A Confident Dose of Bad Advice: Just Roll Your Own DI"
date: 2026-04-14
series: bad-advice
tags: [mobile, architecture, opinion, ios, android]
---

Quick context before I give you a confident dose of bad advice. I've been writing iOS and Android code since about 2010 — starting with a game I shipped called [Sync-Ball](https://apps.apple.com/ca/app/sync-ball/id356277312), then Android games around 2014 back when you had to use Eclipse because Android Studio wasn't out yet. I've been building *non-game* mobile apps since around 2017. So I've gone through a few iterations of the "how do I organize services and view models without losing my mind" question.

I'm also the guy who's currently [writing his own terminal emulator](/2026/04/13/vim-can-wait/), who [rebuilt his own 2D game engine](/2026/04/12/the-right-algorithm-still-slower/) instead of using Unity, and whose [first professional life was teaching students to write their own game engines](/2026/04/13/standing-on-the-shoulders-of-xterm/) instead of starting with existing ones. If "roll your own" sounds like a recurring pattern in my life, that's because it is. Which means the take below should not be shocking coming from me. It doesn't mean I'm right. It means you should read the whole thing before deciding I'm wrong.

## The Take

**Unless you're on a 400-engineer team sharing one monorepo, you probably don't need a DI framework for your mobile app. Just roll your own service container. It's twenty lines of code, maybe thirty if you're feeling fancy, and you'll learn more about dependency injection from writing it than you ever will from reading Dagger docs.**

Fair warning before you keep reading: this is a topic that turns me into a ranter. Once you start me on service boundaries, testability, and why the framework isn't the architecture, you get the full 2am bar monologue. Which, I suppose, is exactly why it's in this series.

## Why I Think So

### The V-Shred story

In 2019 I was working at V-Shred on their brand-new Android app. Fresh greenfield project, no legacy decisions to inherit, no existing framework choices, nothing. My dream. I got to pick the whole stack.

Quick aside, because it fits the theme of this post: I had never shipped Kotlin in production before. Google had recently made it their preferred Android language, but most of my previous Android experience was Java. I walked into the VP of Engineering's office and told him "I'm writing this in Kotlin, not Java, because Kotlin is better and if we start in Java we'll end up migrating later anyway." He shrugged, said "fine, you own it," and that was the whole conversation. Same week, same project, I was also deciding how to handle dependency injection.

I looked at Dagger. This was pre-Hilt — Hilt didn't exist yet — so the options were "roll your own" or "take on the full weight of Dagger 2 with its component graphs and subcomponents and @Provides annotations *while also learning Kotlin*." I read the Dagger docs for about an hour, closed the tab, and wrote a big factory class that manually constructed every service and every view model and passed the dependencies in through their constructors. It was crude. It worked. We shipped it. I was the only Android engineer on the project at the time, and I never regretted the choice once.

It wasn't that Dagger was *bad.* It was that the cost-benefit of "spend three days learning Dagger and debugging its generated code" versus "spend one day writing a factory" was obviously the factory. If V-Shred had grown to fifty Android engineers, we might have had to migrate. It didn't, so we didn't.

Six years later, I'm on another mobile project. Cleaner code, better patterns, same exact philosophy. Here's what it looks like.

### What "roll your own" actually looks like in practice

This is the Android `AppContainer` from the app I'm currently building — trimmed slightly for space, but structurally it's the whole thing:

```kotlin
class AppContainer(context: Context) {
    val logging: LoggingService = DefaultLoggingService(context)

    // Services (singletons)
    val secureStorage: SecureStorage = DefaultSecureStorage(context, logging)
    val storage: StorageService = SharedPrefsStorageService(context, logging)
    val staleness: StalenessTracker = StalenessTracker()

    val apiService: ApiService =
        DefaultApiService(
            tokenProvider = { secureStorage.get("auth_token") },
            logging = logging,
        )

    val authService: AuthService =
        DefaultAuthService(
            apiService = apiService,
            secureStorage = secureStorage,
            storage = storage,
        )

    val featureService: FeatureService =
        DefaultFeatureService(
            apiService = apiService,
            storage = storage,
            logging = logging,
        )

    // Repositories (singletons)
    val userRepository: UserRepository =
        DefaultUserRepository(
            apiService = apiService,
            storage = storage,
            staleness = staleness,
        )

    init {
        Log.configure(logging)
        FF.configure(featureService)
        setupSessionHooks()
    }

    // ViewModel Factories
    fun makeLoginViewModel(): LoginViewModel =
        LoginViewModel(authService = authService)

    fun makeRegisterViewModel(): RegisterViewModel =
        RegisterViewModel(authService = authService)

    private fun setupSessionHooks() {
        authService.onLogin = { user ->
            logging.setUser(user.id.toString(), user.email)
        }
        authService.onLogout = {
            logging.clearUser()
        }
    }
}
```

That's it. That's the whole DI framework. You construct the app-level services in order of their dependencies — logging first, then secure storage (which needs logging), then the API service (which needs secure storage for the token), then session (which needs API and storage), and so on. You add factory methods for any view model that needs services. You wire up cross-service listeners in `init`. The whole thing reads top-to-bottom. There's no magic.

And here's the view model side:

```kotlin
class LoginViewModel(
    private val authService: AuthService,
) : ViewModel() {
    val email = MutableStateFlow("")
    val password = MutableStateFlow("")
    // ... state flows for loading, error, validation

    fun login() {
        if (email.value.trim().isEmpty() || password.value.isEmpty()) return
        viewModelScope.launch {
            _isLoading.value = true
            try {
                authService.login(email.value, password.value)
                _didLoginSuccessfully.value = true
            } catch (e: ApiError.ValidationError) {
                _fieldErrors.value = e.errors
            } catch (_: Exception) {
                _error.value = ClientError.GENERIC
            }
            _isLoading.value = false
        }
    }
}
```

Notice what's missing: `@HiltViewModel`, `@Inject`, `@Singleton`, `@Provides`, `@Binds`, `@ActivityScoped`, `@AssistedInject`, any module classes, any component interfaces, any `KAPT` code generation, any generated glue files you can't read. The view model extends the real `androidx.lifecycle.ViewModel` from Android's lifecycle library. It takes a `AuthService` through its constructor. The `AppContainer` creates it through `makeLoginViewModel()`. That's the whole wiring story.

- **Testable:** pass a fake `AuthService` (that implements the same interface) in your unit test. Done.
- **Scoped:** the lifetime of any dependency is just "as long as the thing that holds it." App-wide? Put it on `AppContainer`. Per-view-model? Construct it in the factory method. Session-scoped? Use lifecycle hooks (I'll get to this in a minute).
- **Debuggable:** put a breakpoint anywhere in the construction chain and walk through the whole dependency graph.
- **Portable:** if you ever *do* need to migrate to Hilt, every service is already behind an interface and already takes dependencies through its constructor. The migration is mechanical.

And before someone in the comments says it: **this is still compile-time graph validation.** The main sales pitch for Dagger is "we catch missing or misconfigured bindings at compile time instead of at runtime." Hand-rolled containers get that for free from the language. Look at the code again — the `AppContainer`'s `init` block constructs `DefaultApiService(tokenProvider = ..., logging = logging)`. If I forget the `logging` argument, Kotlin's compiler refuses to build. If I pass the wrong type, the compiler refuses to build. If I try to use `authService` before it's constructed, the compiler refuses to build. I'm not wiring this up through reflection or annotation processing — I'm writing actual code that actually runs, and the type system is the validator. Dagger generates code that provides this guarantee. Hand-rolled containers just *are* the code that provides it. Same outcome, no build-step magic, better error messages when something's wrong ("you forgot a parameter" vs. "Dagger cannot find a binding for `LoggingService` in the `AppContainer` component graph").

And while I'm on rebuttals: **you also still get per-view-model scoping, for free.** The other big Dagger/Hilt sales pitch is the `@Scoped` family of annotations — `@Singleton`, `@ActivityScoped`, `@ViewModelScoped`, and so on — which control how long each dependency lives. The implicit claim is that hand-rolled containers force everything to be a singleton because you construct everything once in the `AppContainer` constructor. That's wrong. It's wrong because you control the factory methods.

Right now, in the `AppContainer` above, every service happens to be a singleton because every `makeXViewModel` passes the container's singleton instance in. But that's my choice, not a limitation of the approach. If I wanted a per-view-model `AuthService` — say, because I want each login attempt to get a fresh session state that dies when the view model dies — I'd just change the factory to construct a new one:

```kotlin
fun makeLoginViewModel(): LoginViewModel =
    LoginViewModel(
        authService = DefaultAuthService(
            apiService = apiService,
            secureStorage = secureStorage,
            storage = storage,
        )
    )
```

Same idea for activity scope, fragment scope, request scope, whatever. The lifetime of any dependency is "as long as the thing that holds a reference to it." The factory method is where you decide who holds the reference. In my example, singletons on `AppContainer` live as long as the app. A dependency constructed inside `makeLoginViewModel` lives as long as the `LoginViewModel` does. A dependency constructed inside a per-fragment container lives as long as the fragment. It's all just object lifetimes. The framework didn't invent this; Java and Kotlin and Swift have always had it.

And one more rebuttal, because this is the one that people *really* think they need Dagger for: **session scope.** The "user logs in, now I need services that know about the session, then they log out and the services should reset" problem. DI frameworks have elaborate machinery for this — Dagger's user-scoped components, Hilt's `@UserScoped`, complete component graphs that exist during a login and get thrown away on logout. It turns out you don't need any of it.

Look at the `AppContainer` code again and notice the `setupSessionHooks()` call in `init`. That's my entire session-scope handling, and the method is about a dozen lines:

```kotlin
private fun setupSessionHooks() {
    authService.onLogin = { user ->
        logging.setUser(user.id.toString(), user.email)
    }
    authService.onLogout = {
        logging.clearUser()
    }
}
```

That's it. I don't have a separate `SessionContainer` that gets constructed at login and nulled out at logout. I don't need one. The `AuthService` exposes `onLogin` and `onLogout` callbacks, and at app startup every service that cares about user lifecycle subscribes to those events exactly once. When a user logs in, the hooks fire and the subscribed services react. When they log out, the hooks fire again and the services reset. Services that don't care about user lifecycle never have to know any of this machinery exists.

The insight underneath this is: **"session scope" is just event subscription with extra steps.** A session-scoped service isn't really one that "lives during the session" — it's one that *cares about* session events. Frameworks handle that by creating and destroying service instances on login/logout. Hand-rolled handles it by having a singleton that reacts to events. Same behavior, different implementation, dramatically different amount of glue code.

If my app grows to the point where I need to actually reset service state on logout — cache eviction, scratchpad clearing, auth token wiping, whatever — I add a line to the `onLogout` hook. The AuthService doesn't need to know about the services that care about it. The services that care don't need to know about each other. And I still don't need Dagger.

### The real insight: interfaces, not frameworks

Here's the thing that's more important than anything I just said about hand-rolling your container:

**Every service in that `AppContainer` is behind an interface.** `LoggingService`, `AuthService`, `ApiService`, `StorageService`, `SecureStorage`, `UserRepository`, `FeatureService`. The `AppContainer` constructs the `Default*` concrete classes, but everything *else* in the codebase — every view model, every repository, every other service — only ever sees the interface.

This is the thing that makes the code testable. Not Dagger. Not Hilt. *Interfaces.*

And I'll tell you a secret about real-world Dagger and Hilt codebases: plenty of them do this wrong. They inject concrete types instead of protocols. The `@Inject` annotation on a class is just "please construct me," it doesn't force you to program against an abstraction. You can have a Hilt-based codebase where every view model depends on `DefaultApiService` directly, and then you go to write a unit test and realize you can't fake the network because the class you needed to replace isn't behind an interface.

**The framework doesn't save you from bad architecture. The architecture is the work.** The framework is just the way the pieces get glued together. If you roll your own container but you program every service behind an interface, you have better architecture than a lot of professionally-shipped Hilt apps. If you use Hilt but you inject concrete classes everywhere, you have worse architecture than the factory class I wrote at V-Shred in 2019.

The facade bits (`Log.configure(logging)` and `FF.configure(featureService)`) are a related point worth being explicit about. `Log` is a global access point for logging. `FF` is a global access point for feature flags. But neither of them is a static class or a singleton-with-behavior — they're both thin facades over the real, swappable services (`LoggingService` and `FeatureService`), and `AppContainer.init` wires each facade to its real implementation at startup. In a test, you pass a fake `LoggingService` to the facade's `configure()` method and every call to `Log.info(...)` goes through the fake. Everything stays testable.

Why facades instead of passing `logging` and `featureService` through like the other services? **Convenience.** Practically every class in the app needs to log something, and a growing number need to check feature flags. Threading those through every constructor adds boilerplate for a dependency everyone has. It's just an architectural choice — you could pass them through like regular services if you wanted — but I've rolled them into short-named global facades because it's cleaner in practice.

**A big warning about static classes**, since this is adjacent to the topic: *don't* do this with plain static classes. If I'd written a `Log` class with static `info(msg)`, `warn(msg)`, `error(msg)` methods that wrote directly to logcat or OSLog, I'd get the convenience but lose all the testability — static calls can't be mocked, so any class that calls `Log.info()` directly can't be tested without a log being written somewhere real. And if you ever have to refactor the logging backend, you're touching every file in the codebase that called `Log.info()`. I've had to do this refactor. It's a multi-day rescue operation. You'll touch a hundred files.

The facade pattern is the middle ground: **the *convenience* of a global static call, with the *testability* of a service injected through the container.** The call site looks static (`Log.info(msg)`) but internally it's delegating to whatever `LoggingService` implementation was handed to `Log.configure()` at startup. In production that's the real service. In tests it's a fake. Call sites don't know or care.

If you *really* want to, you can skip the facade entirely and pass `logging` and `featureService` through every constructor. It would work, it would be slightly more explicit about dependencies, and it would add a little boilerplate to every class. Your call. I've done it both ways and landed on facades for the things that nearly every class needs, constructor injection for everything else.

### You can go one layer deeper: view models behind interfaces too

Here's something I've done on past projects that I haven't done yet on this one — and I want to be honest about why.

In previous apps I've put *view models* behind interfaces too, so the views only ever interact with a `LoginViewModelProtocol` instead of `LoginViewModel`. That way every layer of the app, all the way out to the view, is testable in isolation — the view doesn't depend on the concrete view model, it depends on an interface that any mock can satisfy.

I haven't done that for this project. Not because I don't believe in the pattern, but because **my view models are currently thin.** They contain almost no business logic of their own — they defer to the session service, the API service, the repositories, and the feature service. There's not much worth testing in isolation at the view model layer. A unit test for `LoginViewModel` right now would mostly just assert "it called `authService.login(email, password)`," which is a tautology.

But here's the discipline that matters: **the reason I can get away with thin view models is that I push the real logic down into services and repositories,** which *are* behind interfaces, and *are* properly testable. If a view model starts accumulating complex logic — multi-step workflows, validation rules, branching state machines — that's a signal that the logic should be pulled out into a service (or a repository, or a use case, depending on the domain) where it has a proper home and a proper interface.

The failure mode I'm avoiding is the one most mobile codebases hit: you start with a thin view model, you add "just one more thing," and then six months later the view model has 400 lines of business logic tangled up with UI state, and now you want to test it but you can't because it's not behind an interface, and refactoring it behind an interface at this point is a multi-day job. I'd rather put the logic in a service from the start than have to rescue it out of a view model later.

If my view models grow complex despite this discipline, I'll add interfaces on top of them. But the move isn't "reflexively put every view model behind an interface on day one because you might need it." The move is "keep view models thin, and if a specific one breaks that rule, give it an interface then."

This is the kind of architectural decision that DI frameworks can't make for you. Hilt doesn't care whether your view models are thin or thick. It just wires them up. The thinking about where logic lives — that's on you, regardless of which DI pattern you use.

### Cross-platform symmetry is a huge hidden win

Here's the other thing that almost never gets talked about in the "framework vs. roll-your-own" debate, and it's the thing I love most: **when you roll your own, you get to use the exact same patterns on both platforms.**

Here's the same `AppContainer` in Swift for iOS, alongside the Kotlin one:

```swift
@MainActor
public final class AppContainer {
    // Services (singletons)
    public let keychain: SecureStorage
    public let storage: StorageService
    public var apiService: APIService
    public let authService: AuthService
    public let featureService: FeatureService
    public let logging: LoggingService
    public let staleness: StalenessTracker

    // Repositories
    public let userRepository: UserRepository

    public init() {
        logging = DefaultLoggingService()
        keychain = DefaultSecureStorage(logging: logging)
        storage = UserDefaultsStorageService(logging: logging)
        staleness = StalenessTracker()

        let tempKeychain = keychain
        apiService = DefaultAPIService(
            tokenProvider: { tempKeychain.get(key: "auth_token") },
            logging: logging
        )

        authService = DefaultAuthService(
            apiService: apiService,
            keychain: keychain,
            storage: storage,
        )
        // ... featureService, userRepository same shape ...

        Log.configure(logging)
        FF.configure(featureService)
        setupSessionHooks()
    }

    public func makeLoginViewModel() -> LoginViewModel {
        LoginViewModel(authService: authService)
    }
    // ... other factories ...
}
```

Same structure. Same naming (modulo `let` vs `val`). Same cross-service wiring in `init`. Same factory method pattern for view models. A person who knows one can jump straight into the other and be productive in about ten minutes.

I've worked at multiple small dev shops where I was the one guy doing both platforms. That's not rare anymore — it's increasingly normal for small teams. Plenty of developers who've been "pure Android" for five years suddenly get dropped onto an iOS codebase because their team got restructured. If the two apps use the same architectural pattern, the cross-platform context switch is nearly free. If one app uses Hilt and the other uses Factory, the developer has to learn *both framework conventions* before they can read the code. That's cost.

Rolling your own means the *pattern* is the standard, not the framework. And patterns port across languages way better than frameworks do.

### The DigiPen philosophy: concepts, not tools

I have to zoom out for one paragraph, because this ties to something I believed long before I had strong opinions about mobile DI.

I taught at [DigiPen](https://www.digipen.edu/) for seven years. One of the pedagogical choices DigiPen made that I always loved was that students learned to build their own game engines and their own renderers instead of starting on Unity or Unreal. People outside the school would sometimes ask why — "isn't that teaching an obsolete skill? Everyone uses Unity in industry." The answer is: **Unity is a tool, and tools change.** In ten years Unity might be dead and some new engine might dominate. If you taught students Unity, you'd have to re-teach them the new tool every time. But if you taught them the *concepts* — game loops, vertex buffers, instanced rendering, sprite batching, collision detection, component systems — then they could pick up any engine in a week because they already understood what was under the hood.

The exact same logic applies here. Dagger is a tool. Hilt is a tool. Factory is a tool. Koin is a tool. All of them implement the same underlying concept: **compose your services so that every class depends on abstractions it doesn't construct itself.** If you learn that concept by building it once, you understand every DI framework afterward — because every DI framework is just a fancier version of the factory class you already wrote. If you *start* with Hilt, you learn Hilt. You don't necessarily learn the concept. And when Hilt gets replaced in 2029 by the next thing, you get to start over.

Rolling your own once is the fastest way to understand what every framework is doing under the hood. The 20 lines of code you write teach you more than a week of reading Dagger docs.

## Where I'm Probably Wrong

Two honest exceptions I'll acknowledge:

1. **If your codebase has many engineers and nobody holds the whole graph in their head.** At big-company scale, the annotation-based contracts in Dagger and Hilt become a shared language. You write `@Singleton` and every engineer reading the code immediately knows what that means. Hand-rolled containers rely on everyone understanding the construction order and the lifecycle conventions you set — and at scale, that knowledge leaks. If you're at a 400-engineer org on a shared monorepo, use what your platform team standardized on. I'm not talking to you.

2. **If your app genuinely has complex scope requirements.** Some apps actually need user-session scopes, activity scopes, request-scoped dependencies, and feature-flag-dependent variants all at once. At that point the framework is doing real work. Be honest about whether that's you or whether you just *imagine* it might be someday. In six years of shipping mobile apps, I can count on one hand the times the scope complexity was actually non-trivial, and every one of them was late-stage in a mature product. Your v0 doesn't need it.

I am *not* saying DI frameworks are bad. I'm saying they're often overkill for the app you're actually shipping *this quarter*, and the pattern underneath them is so simple that you should learn it by building it once.

## The Vibe Check

This take is for solo devs, small teams, side projects, and greenfield commercial apps at companies under roughly thirty mobile engineers. If you're building an enterprise app on a shared platform, your platform team already chose, and this post isn't for you.

For everyone else: write less glue, ship more features. The `AppContainer` is not the interesting part of your app. Write it in an afternoon, put it behind interfaces so everything stays testable, use the same pattern on iOS and Android, and get back to the actual product.

Take it or leave it. I'll be at the end of the bar.
