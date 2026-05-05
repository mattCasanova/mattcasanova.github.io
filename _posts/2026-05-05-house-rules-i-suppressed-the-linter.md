---
layout: post
title: "House Rules: I Suppressed the Linter"
series: house-rules
tags: [architecture, solid, mobile, android, kotlin, opinion]
---

*SOLID series: [S](/2026/04/15/house-rules-the-other-four-letters/) → [O](/2026/04/17/house-rules-whatever-happens-happens/) → [L](/2026/04/22/house-rules-it-compiled-it-lied/) → **I** → D*

I knew this code was wrong when I wrote it.

I'd built every other service in the app behind a focused interface. `LoggingService` handled logging. `StorageService` handled storage. `SecureStorage` handled the keychain. Each one tight. Each one doing one thing. Then I wrote `ApiService` and put twenty-two methods on it.

Auth. Dashboard. Videos. Programs. Browse. Search. Favorites. Stats. Profile. Feature flags. One interface. Twenty-two methods. Anyone who needs one method gets all twenty-two.

I'd turned my Detekt linter up to strict specifically so it would catch stuff like this. It did. It flagged the interface for having too many functions. I looked at the warning, typed `@Suppress("TooManyFunctions")`, and moved on.

The linter caught it. I told the linter to shut up.

```kotlin
@Suppress("TooManyFunctions")
interface ApiService {
    // ... 22 methods, every domain in the app
}
```

That annotation is the receipt. I knew. I deferred it because the app was still taking shape and I didn't want to commit to domain boundaries before the domains were clear. That's a fair reason. It's also the reason every god-interface in every codebase has ever been written.

So when I sat down to write the I post, I had to look at my own code with fresh eyes. Turns out fresh eyes are exactly what blog posts are good for.

## The Crime, in Full

Here's the actual interface — receipt and all:

```kotlin
@Suppress("TooManyFunctions")
interface ApiService {
    val events: SharedFlow<AppEvent>

    // Auth
    suspend fun login(
        email: String,
        password: String,
    ): AuthResponse

    suspend fun register(
        name: String,
        email: String,
        password: String,
        passwordConfirmation: String,
    ): AuthResponse

    suspend fun forgotPassword(email: String)

    suspend fun logout()

    // User
    suspend fun getUser(): User

    // Dashboard
    suspend fun getDashboard(): DashboardResponse

    // Videos
    suspend fun getVideo(slug: String): VideoDetailResponse

    suspend fun updateProgress(
        slug: String,
        watchedSeconds: Int,
        completed: Boolean,
    )

    suspend fun toggleVideoFavorite(slug: String)

    // Programs
    suspend fun getPrograms(): ProgramsResponse

    suspend fun getProgram(slug: String): ProgramDetailResponse

    suspend fun toggleProgramFavorite(slug: String)

    // Browse & Search
    suspend fun browse(duration: String?): BrowseResponse

    suspend fun search(
        query: String,
        tags: List<String>,
    ): SearchResponse

    suspend fun getVideosByTag(slug: String): List<Video>

    // Favorites & Stats
    suspend fun getFavorites(): FavoritesResponse

    suspend fun getStats(): StatsResponse

    // Feature Flags
    suspend fun getFlags(): FeatureFlagResponse

    // Profile
    suspend fun getProfile(): User

    suspend fun updateProfile(
        name: String,
        email: String,
    ): User

    suspend fun updatePassword(
        currentPassword: String,
        password: String,
        passwordConfirmation: String,
    )
}
```

Twenty-two methods. One interface. Every screen depends on it. `LoginViewModel` calls `login()` — one method. But it "depends on" twenty-two.

That's the violation.

## What the Letter Actually Says

The textbook line, for the tattoo:

> **No client should be forced to depend on methods it doesn't use.**

That's it. That's the I.

Two words to underline: *forced* and *client*. The principle isn't about how big interfaces should be. It's about who gets stuck depending on what. The interface is just where the rule lands.

If you internalized [the S post](/2026/04/15/house-rules-the-other-four-letters/), I lands for free. Here's the through-line that carries you across both:

**ISP is SRP for interfaces.**

S argues a class should have one reason to change. I argues an interface should have one reason to grow. Same idea at a different scope. S asks *who can force this code to change.* I asks *who has to look at this contract to do their job.* The unit is different. The question is the same: minimize the blast radius.

The `LoginViewModel` from the S post had one reason to change. The `ApiService` above has one reason to grow per domain that touches the network — auth, dashboard, videos, programs, profile, all of it. The interface is under permanent pressure to expand, and it will keep expanding as long as it exists.

## I Just Wanted a Beer

Here's how this lands when you actually try to use it.

You walk into a bar. Sit down. Order a drink. The bartender hands you a menu with 200 items.

You wanted a beer.

You're at the bar.

You don't need the caviar. You don't need the 1am bar sushi. You don't need the kids' menu. You don't need the catering options. You don't need the wedding tasting flight. But somebody decided everything goes on one menu, and now you're flipping through fifteen pages to find a hazy IPA.

That's the `LoginViewModel` getting handed `ApiService`.

It wanted `login()`. It got the caviar.

## Why the I Is the Subtle One

Out of the five letters, this is the one where the cost is the most delayed — and the one your tools are worst at flagging.

Day one, an ISP violation is invisible. The view model that "depends on" twenty-two methods compiles fine. It runs fine. The login screen calls `login()` and the login screen works. There's no compile error, no failing test, no smell.

The other letters don't get to hide like that. Skip DI and your unit test instantly drags in real networking. You notice. Tangle two responsibilities in one class and you can't isolate either of them under test. You notice. Lie about a subclass contract and the parent class's test suite goes red against the subclass. You notice.

Violate I and the test suite eventually goes red too — but in a language that doesn't look like a violation. You add a method to your fat interface. Twenty mocks across twenty test files break. You shrug — *of course I have to update the mocks, I just added a method* — and update each one. The signal *is* there: you shouldn't have had to update twenty mocks. You should have updated *one* — the one for the consumer that actually calls the new method. But that distinction is invisible while you're updating. It just feels like normal test maintenance.

That's why it's subtle. The breakage looks like work, not like an architecture problem.

In rough order of how often they actually bite, in real codebases:

**1. Mock churn.** Every test that needs to fake `ApiService` has to stub all twenty-two methods, even if the test only cares about `login()`. Add a twenty-third method tomorrow and every test in the suite goes red until each one is updated. This is the version that bites in real codebases over and over. Even if you stop caring about ISP for principle, the test suite makes you care for survival.

**2. The footgun.** `grantAdminRights()` is supposed to be reachable only from the admin dashboard, where it's gated by the right checks. With the fat interface, a view model on the login screen can call it just as easily — the method is right there. Programmers reach for what's reachable. The interface is bad documentation about what each consumer is *supposed* to do — it gives every consumer the keys to every domain.

**3. Refactor cost as the codebase grows.** Once a 22-method interface is load-bearing, splitting it later means touching every consumer and every test. That's the version that hurts the day you decide to stop ignoring it. The longer you wait, the bigger the rescue.

**4. Recompile time.** In C++ or large compiled libraries this can sting. In modern mobile, builds are fast enough that this is barely a tax. Worth a footnote, not an argument.

None of these hurt on day one. They compound over months and years.

That's why it's subtle. The bill comes due quietly.

## The Hard Part

This is the hard part of every SOLID principle, and it's harder for ISP than most.

You can always argue something is "one responsibility" by going up a level of abstraction. The `DefaultApiService` *class* has twenty-two methods. Is that an SRP violation? No — its responsibility is *being the API wrapper.* That's gaming the rule a little, but it's defensible.

And not just For bullshit reasons: nobody actually holds a reference to `DefaultApiService`. The class lives in the library, gets built once in the `AppContainer`, and gets handed to each consumer as the narrow interface they need.

The class can have twenty-two methods because *no consumer ever sees twenty-two methods.* The fix isn't necessarily to split the class — sometimes splitting the class is exactly right. But for an ISP violation, the move is usually to split the *interface* and let the class implement the slices.

You can also reduce ISP to absurdity in the other direction. Every interface should have one method. `LoginApi`, `ForgotPasswordApi`, `LogoutApi`. Pure ISP. Now there are ten files for what could be one tight `AuthApi` with three methods.

It's the same shape as the view-controller question. Every view controller "violates SRP" if you look narrowly enough — it handles many UI elements. But its responsibility *is* to be a view controller. The judgment call is: at what level of abstraction do you draw the line?

And to be fair, that line can be blurry. It's the same with most architecture questions.

For ISP, I think the line lives at the *consumer.*

Each consumer should see only the methods it actually needs. The login screen shouldn't see `grantAdminRights()`. The dashboard shouldn't see `forgotPassword()`. Whatever grouping makes that true is the right grouping.

That doesn't tell you exactly where the seams go. It tells you the question to ask.

## After the Crime

Same Android app, same `DefaultApiService` class. The interface side becomes a stack of small interfaces, one per domain:

```kotlin
interface AuthApi {
    suspend fun login(
        email: String,
        password: String,
    ): AuthResponse

    suspend fun register(
        name: String,
        email: String,
        password: String,
        passwordConfirmation: String,
    ): AuthResponse

    suspend fun forgotPassword(email: String)

    suspend fun logout()

    suspend fun getUser(): User
}

interface DashboardApi {
    suspend fun getDashboard(): DashboardResponse
}

interface VideoApi {
    suspend fun getVideo(slug: String): VideoDetailResponse

    suspend fun updateProgress(
        slug: String,
        watchedSeconds: Int,
        completed: Boolean,
    )

    suspend fun toggleVideoFavorite(slug: String)
}

// ProgramApi, BrowseApi, FavoritesApi, ProfileApi, FeatureFlagApi...
```

`DefaultApiService` implements all of them. Same class. Same instance. The container hands the same singleton to every consumer — but each consumer only sees its slice:

```kotlin
val loginVM = LoginViewModel(authApi = defaultApiService)              // sees 5 methods
val dashboardVM = DashboardViewModel(dashboardApi = defaultApiService) // sees 1 method
```

The `LoginViewModel` no longer "depends on" twenty-two methods. The fake in tests stubs five, not twenty-two. Add a `getNotifications()` method to a future `NotificationsApi` and not a single login test goes red.

## The Part Nobody Talks About

The textbook says "split the interface." It doesn't say "split the implementation." Those are different operations with very different consequences, and every real-world ISP refactor hits this fork.

Here's the trap. The naive refactor is: split the interface into domain interfaces *and* split the implementation into one class per interface. `AuthApiClient`, `DashboardApiClient`, `VideoApiClient` — eight separate networking classes, each with its own `HttpClient`.

Sounds clean.

It isn't.

The base `HttpClient` in this app handles unauthorized responses globally. If any API call returns a 401, the client emits an `AppEvent.Unauthorized` through a `SharedFlow`, and the `AppContainer` listens for that event and triggers logout. That's a cross-cutting concern that lives on the *class*, not on the interface. It only works because there's exactly one `HttpClient`, exactly one flow, exactly one listener.

Split the implementation and now you have eight `HttpClient`s, each emitting its own unauthorized flow. The container has to merge eight flows. Or pick one. Or — most likely — silently break the auto-logout flow on whichever clients you forgot to wire up.

The split-the-interface version doesn't have that problem. One `DefaultApiService`. One `HttpClient`. One unauthorized flow. The class implements all the small interfaces, the container hands the same instance to every consumer as the narrow type, and the cross-cutting concern stays where it belongs.

**ISP is about the contract, not the class.**

Look at the `ApiService` from earlier — the one with twenty-two methods. The `val events: SharedFlow<AppEvent>` line at the top, before the auth methods. Every screen that depended on `ApiService` saw an event flow it had no business reading. Most of them never touched it. They all had access.

After the refactor, none of the eight domain interfaces expose `events`. The flow lives on `HttpClient`, the container proxies it to the one listener that actually cares, and the rest of the codebase doesn't even know it exists. The class kept the cross-cutting concern. The interfaces stopped advertising it.

Splitting the interface gives you the consumer-side benefits. Splitting the implementation costs you the cross-cutting wins you didn't realize you had. They are not the same operation. Nobody told me this. I had to walk into it and back out before the design clicked.

## Where I'm Still Wrong

Real talk. I might not refactor this all the way before the post lands.

The win from splitting `ApiService` into eight domain interfaces is real and obvious. The next layer of judgment — *should `AuthApi` itself shrink to `LoginApi` plus `LogoutApi` plus `RegistrationApi`* — is genuinely less obvious. Login uses `login()`. Forgot Password uses `forgotPassword()`. Are those the same client, or two? At what point is splitting more valuable than the boilerplate it costs?

The post is about the *decision*, not the perfect refactor. The login screen probably still has access to too much by the time this lands. I might shrink the consumer slices another notch while writing. The honest move is to talk about *finding the balance* rather than pretending I shipped a textbook split.

This is the same trap [I caught myself in for the L post](/2026/04/22/house-rules-it-compiled-it-lied/). Writing about a principle forces me to look at my own code with fresh eyes — and the code keeps having more to say than the post originally planned for.

## Coupling Compounds

Here's the part that makes ISP feel like air. You don't notice when it's there. You don't really notice when it's being violated either — until the day something has to change and you have to decouple things that have been coupled for a year.

That's the day you wish you'd spent twenty extra minutes on day one.

Coupling compounds. Once two things are tightly coupled, they tend to get *more* coupled, not less. New consumers get added against the fat interface because the fat interface is what's there. New methods get tacked on because that's where the methods go. Six months later your blast radius is a hundred files instead of one.

Decouple them on day one and decoupled is the *natural state.* Maintenance gets cheaper forever after.

That's not a crusade. That's the math.

## Interfaces Again. Of Course.

If every service is already behind an interface — the [Bad Advice arc](/series/bad-advice/) thesis — that's 90% of the win. ISP is the refinement: shrink each interface so each consumer only sees what it actually uses.

That makes this post a continuation of [Bad Advice](/series/bad-advice/) and the rest of [House Rules](/series/house-rules/), not a new argument.

*More* interfaces. Tighter ones.

What's better than one interface? Two. What's better than two? Eight, if eight is what your consumers actually need.

What is interface segregation about?

More interfaces.

---

*This is the fourth entry in the **House Rules** series. One letter to go. D — Dependency Inversion — was already the whole [Bad Advice arc](/series/bad-advice/) wearing different clothes, so the closing post will mostly be tying the bow.*

*Five posts in, the answer to every letter has been: interfaces. Interfaces. Interfaces. Interfaces. I don't think that's going to change.*

*I didn't write 'em, but those are the rules.*
