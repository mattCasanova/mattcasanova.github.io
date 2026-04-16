---
layout: post
title: "House Rules: I (Interface Segregation)"
series: house-rules
tags: [architecture, solid, mobile, ios, swift, opinion]
status: seed
---

## Pre-draft seed — not a post yet

### The hook

I knew this was wrong when I wrote it. I built every other service in my app behind a focused interface — `AuthService` handles auth, `LoggingService` handles logging, `StorageService` handles storage. Each one is tight. Each one does one thing. Then I wrote `APIService` and put twenty-two methods on one protocol.

Auth. Dashboard. Videos. Programs. Browse. Search. Favorites. Stats. Profile. Feature flags. One interface. Twenty-two methods. Every domain in the entire app, on one menu.

I knew it was an ISP violation. I turned my Detekt linter up to strict specifically so it would catch stuff like this — and it did. It flagged the interface for having too many functions. And I looked at the warning, typed `@Suppress("TooManyFunctions")`, and moved on. The linter caught it. I told the linter to shut up.

```kotlin
@Suppress("TooManyFunctions")
interface ApiService {
    // ... 23 methods across every domain in the app
}
```

That annotation is the receipt. I knew. I deferred it because the app was still taking shape and I didn't want to commit to domain boundaries before the domains were clear. That's a fair reason. It's also the reason every god-interface in every codebase gives you.

So I had Claude rip into me about it. Bourdain-style.

### The before (snapshot this code before the refactor lands)

APIService protocol — 22 methods, one interface:

```swift
public protocol APIService: Sendable {
    // Auth
    func login(email: String, password: String) async throws -> AuthResponse
    func register(name: String, email: String, password: String, passwordConfirmation: String) async throws -> AuthResponse
    func forgotPassword(email: String) async throws
    func logout() async throws

    // User
    func getUser() async throws -> User

    // Dashboard
    func getDashboard() async throws -> DashboardResponse

    // Videos
    func getVideo(slug: String) async throws -> VideoDetailResponse
    func updateProgress(slug: String, watchedSeconds: Int, completed: Bool) async throws
    func toggleVideoFavorite(slug: String) async throws

    // Programs
    func getPrograms() async throws -> ProgramsResponse
    func getProgram(slug: String) async throws -> ProgramDetailResponse
    func toggleProgramFavorite(slug: String) async throws

    // Browse & Search
    func browse(duration: String?) async throws -> BrowseResponse
    func search(query: String, tags: [String]) async throws -> SearchResponse
    func getVideosByTag(slug: String) async throws -> [Video]

    // Favorites & Stats
    func getFavorites() async throws -> FavoritesResponse
    func getStats() async throws -> StatsResponse

    // Profile
    func getProfile() async throws -> User
    func updateProfile(name: String, email: String) async throws -> User
    func updatePassword(currentPassword: String, password: String, passwordConfirmation: String) async throws

    // Flags
    func getFlags() async throws -> FeatureFlagResponse
}
```

LoginViewModel depends on this. It calls `login()`. One method. But it "depends on" twenty-two.

### The Bourdain line that sells the post

"It's like walking into a restaurant and the waiter hands you a menu with 200 items. You wanted a drink. You're at the bar. You don't need the prix fixe, you don't need the kids' menu, you don't need the catering options. But somebody decided everything goes on one menu, and now you're flipping through fifteen pages to find the whiskey list."

### The ISP definition (for the post)

**No client should be forced to depend on methods it doesn't use.**

### The after (fill in after the refactor)

Split into domain-specific protocols: `AuthAPI`, `DashboardAPI`, `VideoAPI`, `ProgramAPI`, `BrowseAPI`, `FavoritesAPI`, `ProfileAPI`, `FeatureFlagAPI`. `DefaultAPIService` implements all of them. Consumers depend on the narrow slice they need.

LoginViewModel's `AuthService` now takes `AuthAPI` — 4 methods. The fake in tests stubs 4 methods, not 22.

### The real-world friction (the part that makes this not a textbook post)

Splitting the interface isn't as simple as "big interface bad, small interfaces good, done." There's a real consideration:

The `HttpClient` base class handles unauthorized responses globally — if any API call gets a 401, the base class emits `AppEvent.Unauthorized` through a `SharedFlow`/`AsyncStream`, and the `AppContainer` listens for that to trigger logout. That's a cross-cutting concern that lives on the *class*, not the interface.

The naive refactor would be: split the interface into domain protocols AND split the implementation into separate classes — `AuthApiClient`, `DashboardApiClient`, etc. But then each one has its own `HttpClient` instance, its own event stream, and the `AppContainer` has to listen to N streams for unauthorized events. That's worse.

The right answer: **split the interface, keep the implementation.** One `DefaultAPIService` class. One `HttpClient`. One event stream. The class conforms to all the domain protocols. `AppContainer` creates one instance and passes it to each consumer as the narrow type — the consumer only sees its slice, but it's the same object underneath. Unauthorized handling doesn't change because there's still one `HttpClient` emitting one stream.

This is the ISP insight nobody talks about: **ISP is about the contract, not the class.** The textbook says "split the interface." It doesn't say "split the implementation." Those are different operations with very different consequences, and the blog post should make that distinction clearly because every real-world ISP refactor hits this same question.

### The self-aware angle

I knew this was wrong. I wrote about ISP in the S post ("I know what the other four letters are") and my own codebase was violating the I the whole time. I suppressed the linter warning that caught it. The fun of the post is the before/after — I caught it because I was literally writing a blog post about SOLID and had to look at my own code with fresh eyes. And the refactor wasn't clean — it required thinking through the HttpClient/unauthorized constraint before the "just split it" answer became obvious.

### Connection to the arc

- S post: "one reason to change" — the foundation
- O post: (TBD)
- L post: (TBD)
- **I post: "no client should depend on methods it doesn't use"** — and here's my own twenty-two-method protocol as evidence
- D post: already covered in the Bad Advice arc (DI, singletons, fakes, vendors)

### Potential title ideas

- House Rules: The I Nobody Talks About
- House Rules: Twenty-Two Methods, One Interface, Zero Excuses
- House Rules: The God Menu
