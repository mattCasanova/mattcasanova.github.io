---
layout: post
title: "House Rules: It Compiled. It Lied."
series: house-rules
tags: [architecture, solid, opinion, swift, mobile]
---

*SOLID series: [S](/2026/04/15/house-rules-the-other-four-letters/) → [O](/2026/04/17/house-rules-whatever-happens-happens/) → **L** → I → D*

Last time I [rattled off the SOLID principles](/2026/04/15/house-rules-the-other-four-letters/), it was at a work dinner, several drinks in.

Years before that one and before my current job, I was interviewing at a different big company. Sometime during the loop, the interviewer asked me to name the SOLID principles. Not explain them. Just name them. It kind of felt like a gotcha. You could hear it in his tone — the tone you use when you don't expect someone to know the answer.

I rattled them off.

Cold.

In order.

To be honest, he didn't seem thrilled about that...or maybe I just misread the room. Either way, I still didn't get the job. But I did leave the room with a theory: somewhere in the industry there's a running bet on which letter people can't remember.

My money's on L.

## What the F Is the L?

S is for Single Responsibility. Tells you what it is.

O is for Open-Closed. Tells you what it is.

L is for...

Barbara Liskov?

...a computer scientist who wrote a paper in 1987.[^1]

That's not a description. That's a name. You can't figure the principle from the name the way you can with the others. So people see "Liskov Substitution" on a whiteboard and their brain files it under "obscure academic trivia" — the kind of thing you only know if you read the right textbook. Like Dijkstra's algorithm. Or the Picard Maneuver. Interviewers know this. That's why it gets asked.

Most engineers can name three or four. The fifth needs a Google.

It's not actually obscure. It's not really very academic.

Once you strip the name off, it's probably the simplest principle in SOLID. And one you are probably doing already and just didn't know it.

## Beer Logic

Formal definition, for the tattoo:

> If S is a subtype of T, objects of type T should be replaceable with objects of type S without breaking the program.

Forget it. Here's the working version:

**Add is fine. Change is lying.**

Here the easy example:

Patty's Pub is down the street. You love that place. You get your beer there a couple of (*eight*) times a week. One day Patty sells. New owner. Still **Patty's Pub** on the door. They add wine to the menu. You don't drink wine. You don't care. You order your beer.

They pour you a White Claw — *Mango flavor*.

Adding wine was fine — bigger menu, didn't affect you. *Changing* what *beer* means at this bar? That's the lie. Same name. Same promise. Different pour.

Adding is the [O we just covered](/2026/04/17/house-rules-whatever-happens-happens/). L is the rule that says you can't quietly swap the beer. That's the whole principle, but with beer.

In code, every caller that trusted the name gets lied to. No compile error. No failed test. Just a world where the same function does different things depending on what got injected. Nobody has any way to tell, but it is wrong and still tastes like mango.

That's the bug. It's the kind of bug that ships. It's the kind of bug you find six months later, when half your users swear pull-to-refresh stopped working and nobody can explain why.

## The 90s Version, in One Paragraph

The textbook example for LSP is Square-extends-Rectangle. `Square` inherits from `Rectangle`, overrides `setHeight()` to also call `setWidth()` — because, you know, squares are squares — and now every caller expecting a rectangle gets a liar. Every blog post about SOLID since 2004 has told this story. Nobody writes this code. I taught this example to undergrads for seven years and every semester I felt a little dumber saying it, because I was *also* teaching them to never build deep inheritance hierarchies in the first place.[^2] I was using a 90s-shaped example to illustrate a principle that matters in non-90s-shaped codebases, and I never closed the loop. The cached client is the example I should have been using the whole time.

## Same Signature, Different Pour

Here's the part that makes LSP violations damn hard.

Say you have a function somewhere in your app:

```swift
func fetchUsers(client: HTTPClient) async throws -> [User] {
    return try await client.get("/api/users")
}
```

It takes an `HTTPClient`. Your code has assumptions baked in about what `HTTPClient` does. *One call, one network request — fresh bytes from the server.* If you want stale data to be okay, you handle that at a higher layer. That's the mental model. That's the contract.

Now I hand you a `CachedHTTPClient` that silently returns whatever response it saw the last time you called the same path. Same base class. Same `get()` signature. The only thing different is that no network request happens on cache hits.

Your function still compiles. Your tests still pass. But now you're getting yesterday's data dressed up as today's. The user pulls to refresh; you "fetch"; nothing actually leaves the device. The server-side change to their account, the new content that just landed, the price that updated this morning — none of it shows up. The user keeps seeing the same view they saw an hour ago, with no way to know.

None of it is in your code. None of it shows up in a review of your code. The type checker says you're fine.

**Compiled. Lied.**

You would have no way to know unless you read the source of the thing you were handed — which is exactly the situation interfaces were supposed to save you from.

## Real Code, Real Trap

This isn't hypothetical — at least not the base class. Here's a real `HTTPClient` from the iOS app I'm shipping right now, set up to be subclassable (is that a word?). You've already seen the abstract version of the trap; here's how it would land in actual production code.

```swift
// HTTPClient.swift — the base class
// swiftformat:disable:next preferFinalClasses
public class HTTPClient: @unchecked Sendable {
    private let baseURL: URL
    private let session: URLSession
    private let tokenProvider: @Sendable () -> String?
    private let logging: LoggingService

    init(
        tokenProvider: @escaping @Sendable () -> String?,
        logging: LoggingService,
        baseURL: URL = AppEnvironment.baseURL,
        session: URLSession = .shared,
    ) { ... }

    func get<T: Decodable>(
        _ path: String,
        queryParams: [String: String] = [:],
        authenticated: Bool = true,
    ) async throws -> T {
        // ... build URLRequest, execute once, decode, return ...
    }
}
```

One promise: `get()` hits the network. That's it. No caching, no retries, no quietly returning yesterday's data.

The reason `HTTPClient` doesn't cache is because *caching is a different layer's job.* Specifically, it's the repository layer's job. Here's an example repo from the same codebase, simplified:

```swift
@MainActor
public final class UserRepository {
    private let apiService: APIService
    private let storage: StorageService
    private let staleness: StalenessTracker

    public private(set) var user: User?

    public func fetchUser(forceRefresh: Bool = false) async {
        loadCachedUser()  // show whatever's in local storage immediately

        guard forceRefresh || staleness.isStale(key: .currentUser) else {
            return  // not stale yet — don't hit the network at all
        }

        do {
            let fresh = try await apiService.getUser()
            storage.save(key: .currentUser, value: fresh)
            staleness.markFetched(key: .currentUser)
            user = fresh
        } catch { ... }
    }
}
```

The repository owns the *data lifecycle*: what's cached, when it's stale, when to re-fetch. Pull-to-refresh on the UI sets `forceRefresh: true`, which skips the staleness check and forces a real network call. The user just navigating around hits the staleness tracker first and skips the network if data is still fresh enough.

More clear: If the user navigates to the same tab, we don't make another API call. If they pull to refresh, we do.

`HTTPClient` doesn't know any of that. It doesn't need to. Its job is *bytes over the wire.*

Now imagine somebody — a well-meaning, performance-conscious asshole — sees the app firing requests as the user navigates and decides to "extend" `HTTPClient`:

```swift
// DON'T DO THIS
class CachedHTTPClient: HTTPClient {
    private var cache: [String: Any] = [:]

    override func get<T: Decodable>(
        _ path: String,
        queryParams: [String: String] = [:],
        authenticated: Bool = true,
    ) async throws -> T {
        if let cached = cache[path] as? T { return cached }
        let result: T = try await super.get(
            path,
            queryParams: queryParams,
            authenticated: authenticated,
        )
        cache[path] = result
        return result
    }
}
```

Identical signature. Identical return type. Identical error type.

Compiles cleanly. Lies cleanly.

Follow what happens. The user pulls to refresh. `UserRepository.fetchUser(forceRefresh: true)` skips the staleness check and calls `apiService.getUser()`, which calls `HTTPClient.get(...)`. The `CachedHTTPClient` returns the previous response from its in-memory dictionary — *the network is never hit.* The repository writes the same data into storage. It marks it as freshly fetched. The UI's spinner finishes. Everything looks fine.

It isn't.

The pull-to-refresh contract — *"I, the user, am asking you to check with the server right now"* — was silently broken.[^3] The base class promised network bytes; the subclass returned in-memory leftovers. The repository, which expected a real fetch every time the staleness tracker said *go*, was lied to. The staleness tracker now thinks the data is fresh as of right now, when it actually hasn't been touched in hours.

The stupid part is: nothing stops me from doing this. I wrote `HTTPClient` with `@unchecked Sendable` and didn't mark it `final`. I even left a `// swiftformat:disable:next preferFinalClasses` comment above it, specifically telling the linter to shut up about the class being subclassable. I did that on purpose — I wanted the option to extend. **LSP is the principle that says: *you can extend, but you have to keep the contract.*** Nothing in the language enforces that. Nothing in the test suite catches it. It's on me to know the rule.

And here's the architectural twist that makes this kind of bug particularly nasty: **caching belongs at the layer that owns the data lifecycle, not at the transport layer.** A cache inside `HTTPClient` doesn't just break LSP — it breaks the entire architecture's idea of *who's allowed to decide when data is stale.* The repository is the only thing that knows about user intent (pull-to-refresh) and about staleness windows. Move that decision down into the network plumbing and the upper layers become liars by accident, every time.

## "But Logging Is Adding, Right?"

Somebody reading this will think: *"OK, but what about a `LoggingHTTPClient` that logs every request? That's adding behavior. Is that allowed?"*

It is. But you wouldn't write it as a subclass anyway.

Look at how `HTTPClient` actually takes its logger:

```swift
init(
    tokenProvider: @escaping @Sendable () -> String?,
    logging: LoggingService,
    ...
) { ... }
```

`LoggingService` is injected. It's a dependency. The client already logs every request through whatever logger you hand it — because that's what D (Dependency Inversion) is for. Want more logging? Configure the logger. Want different logging in tests? Inject a fake. You don't subclass the client, you compose it.

That's the pattern you'll notice once you start looking. **When you want to add behavior, you probably don't really want a subclass — you want injection.** When you reach for a subclass, it's probably because you want to *change* behavior. And changing behavior behind the same signature is exactly the thing LSP says not to do.

So the rule in practice is even tighter than the formal version:

- **Adding behavior?** Inject it. Compose it. Don't subclass.
- **Changing behavior?** Give it a new name. Don't overload the old one.

If you follow those two, you almost can't violate LSP. Which is why —

## The Awkward Admission

— this post was hard to write.

Not because the concept is hard. Because I genuinely don't have a Liskov violation in any of my code, and while writing this post I went and made it impossible to add one.

Not that subclassing is always wrong. Strong libraries, framework extension points, designed-to-extend APIs — those earn it. But most application code? There are a lot of easier ways to skip this trap entirely.

In [LiquidMetal2D](https://github.com/mattcasanova/LiquidMetal2D), `GameObj` *used to be* a base class for every object in the engine. It had a `toUniform()` method — the one I [rewrote for the OCP post](/2026/04/17/house-rules-whatever-happens-happens/) — that any subclass could override to pack its own GPU data for a custom shader.

How many subclasses overrode it?

Zero.

Then [I added multi-shader support](/2026/04/19/render-pass-strikes-back/), and the question of *where does custom GPU data live* got an answer that wasn't *"on a subclass."* It went on components. `AlphaBlendComponent`. `WireframeComponent`. `RippleComponent`. `ParticleEmitterComponent`. Same `GameObj`, multiple components, multiple shaders, no inheritance. So I made `GameObj` `final`.

*Composition or bust.*

The "extension point nobody used" isn't even an extension point anymore. The canary isn't just alive. I sealed the mine and built a different mine.

That isn't a point of pride. Maybe it should be. It's just the consequence of a rule I picked up years ago and never thought much about: **prefer composition over inheritance.** Shallow hierarchies. Interfaces where they earn their keep. When you do subclass, add, don't change. If what you actually want is different behavior, give the thing a different damn name and contract.

That's all of LSP. Spread across a bunch of small choices that don't feel like LSP when you're making them. Like I said, we've probably been doing this all along. We just didn't know the name.

## The Canary

Here's the last thing, and then you can go to bed.

LSP isn't a principle you sit down and apply. There's no checklist. There's no "LSP pass" on the codebase. It's a **symptom you watch for** — the same way a canary in a coal mine is a symptom. If you catch a subclass lying about its behavior, you don't fix the subclass. You fix whatever made inheritance the chosen tool. Composition didn't fit? Interface too fat? Dependency not injectable? Each of those funnels somebody toward inheritance, and inheritance is where the lie lives.

If you follow S, O, I, and D, L mostly follows from them. When the canary drops dead, don't blame the canary — check the air.

And if someone asks you to name the five principles in an interview someday, and you can, and the interviewer doesn't seem thrilled about that — congratulations. You just caught an LSP violation in the wild. The contract was: they ask a question, you get it right, they're happy. You got it right. They weren't happy. Same signature, different pour.

You didn't get the job. It's fine.

Knowing the rules is more important than any one job.

---

*This is the third entry in the **House Rules** series. Next up: the I. Interface Segregation. My Detekt linter caught it. I told it to shut up. Turns out the linter was right and I was the cached HTTP client.*

*Halfway through SOLID. So far the answer to all of them is "interfaces." I'll let you know if that changes.*

*I didn't write 'em, but those are the rules.*

[^1]: Her paper was "Data Abstraction and Hierarchy." She later won the Turing Award for other work. None of that makes the name a description. "Add is fine, change is lying" is still the thing you need to remember.

    But seriously — shout out to Barbara Liskov. Uncle Bob. Martin Fowler. The Gang of Four — Gamma, Helm, Johnson, Vlissides. Grady Booch. Every other genius who came before us. You all crawled so we could triple kickflip our way into jobs built on your work, that we probably don't deserve. All while some asshole writes blogs about it online and most of us don't even remember your names. Damn shame.
[^2]: If I taught you C++ or intro programming between 2011 and 2017 in Singapore or Korea, consider this the apology.

[^3]: I worked at a place — names withheld — that did this wrong twice in opposite directions. No caching layer, no staleness tracker, so every tab switch refreshed the whole thing whether you needed it or not. Meanwhile the *actual* pull-to-refresh? Just a spinner that ran for three seconds and dismissed. No fetch. No invalidation. Nothing. The pull worked. The refresh didn't. Damn shame.
