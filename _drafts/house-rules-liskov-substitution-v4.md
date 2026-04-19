---
layout: post
title: "House Rules: It Compiled. It Lied."
series: house-rules
tags: [architecture, solid, opinion, swift, mobile]
---

Years ago, before I got my current job, I was interviewing at another big tech company. Somewhere in the loop, the interviewer asked me to name the SOLID principles.

Not explain them. Just name them.

It was a gotcha. You could hear it in the tone — the tone you use when you don't expect someone to know the answer.

I rattled them off. Cold. In order.

He didn't seem thrilled about that.

I didn't get the job. But I did leave the room with a theory: somewhere in the industry there's a running bet on which letter people can't remember. My money's on L.

## Why L Is the Letter Nobody Remembers

S is for Single Responsibility. It tells you what it is.

O is for Open-Closed. It tells you what it is.

L is for Barbara Liskov, a computer scientist who wrote a paper in 1987.[^1] That's not a description. That's a name. You can't deduce the principle from the name the way you can with the other four. So people see "Liskov Substitution" on a whiteboard and their brain files it under "obscure academic trivia" — the kind of thing you only know if you read the right textbook. Like Dijkstra's algorithm. Or the Picard Maneuver. Interviewers know this. That's why it's the one they ask.

It's not obscure. It's not academic. Once you strip the name off, it's probably the simplest principle in SOLID.

## The Actual Principle, in Four Words

Formal definition, for the tattoo:

> If S is a subtype of T, objects of type T should be replaceable with objects of type S without breaking the program.

Forget it. Here's the working version:

**Add is fine. Change is lying.**

That's the whole thing. Every class makes promises. A subclass inherits those promises. You're allowed to ADD new promises on top — that's just extension, that's the [O we just covered](/2026/04/17/house-rules-whatever-happens-happens/). What you can't do is quietly *change* an existing promise and keep the same name on the door.

If you do, every caller that trusted the name gets lied to. No compile error. No failed test. Just a world where the same function does different things depending on what got injected, and nobody has any way to tell.

That's the bug. It's the kind of bug that ships. It's the kind of bug you find six months later, when a customer's credit card gets charged twice on Tuesdays and nobody can explain why.

## The Caller Can't Save Themselves

Here's the part that makes LSP violations mean.

Say you have a function somewhere in your app:

```swift
func fetchUsers(client: HTTPClient) async throws -> [User] {
    return try await client.get("/api/users")
}
```

It takes an `HTTPClient`. Your code has assumptions baked in about what `HTTPClient` does. *One call, one request.* If it fails, it throws. If you want retries, you wrap it in your own retry logic at a higher layer. That's the mental model. That's the contract.

Now I hand you a `RetryingHTTPClient` that silently retries three times on failure. Same base class. Same `get()` signature. The only thing different is the behavior.

Your function still compiles. Your tests still pass. But now your app makes three requests where it expected one. Your own retry logic wraps it, so that's three times three. If there's rate-limiting, you hit it with 9x the traffic. If there's telemetry counting request volumes, your graphs spike for reasons nobody can explain. If the failing request is a POST that's not idempotent, you just charged somebody three times.

None of it is in your code. None of it shows up in a review of your code. The type checker says you're fine.

**Compiled. Lied.**

You would have no way to know unless you read the source of the thing you were handed — which is exactly the situation interfaces were supposed to save you from.

## Real Code, Real Trap

This isn't hypothetical. Let me show you with actual code from the iOS app I'm shipping right now. I have a real `HTTPClient` class that's set up to be subclassable, and this is the exact shape of the trap.

Here's the relevant slice:

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

One request, one response. That's the promise of `get()`. Hit the endpoint, get the data back or throw an error. The caller's problem from there.

Now let's break it. Here's a subclass that *feels* like a reasonable addition — everyone loves retries:

```swift
// DON'T DO THIS
class RetryingHTTPClient: HTTPClient {
    override func get<T: Decodable>(
        _ path: String,
        queryParams: [String: String] = [:],
        authenticated: Bool = true,
    ) async throws -> T {
        for attempt in 1...3 {
            do {
                return try await super.get(
                    path,
                    queryParams: queryParams,
                    authenticated: authenticated,
                )
            } catch {
                if attempt == 3 { throw error }
                try await Task.sleep(for: .seconds(attempt))
            }
        }
        fatalError("unreachable")
    }
}
```

Identical signature. Identical return type. Identical error type. Compiles clean.

Lies cleanly.

The base class promised one request per call. This one makes up to three. Every caller that wraps its own retry logic on top — and mine do — now multiplies that by whatever it's already doing. The promise is broken silently, at runtime, in prod.

The stupid part is: nothing stops me from doing this. I wrote `HTTPClient` with `@unchecked Sendable` and didn't mark it `final`. I even left a `// swiftformat:disable:next preferFinalClasses` comment above it, specifically telling the linter to shut up about the class being subclassable. I did that on purpose — I wanted the option to extend. **LSP is the principle that says: *you can extend, but you have to keep the contract.*** Nothing in the language enforces that. Nothing in the test suite catches it. It's on me to know the rule.

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

That's the pattern you'll notice once you start looking. **When you want to add behavior, you usually don't actually want a subclass — you want injection.** When you reach for a subclass, it's usually because you want to *change* behavior. And changing behavior behind the same signature is exactly the thing LSP says not to do.

So the rule in practice is even tighter than the formal version:

- **Adding behavior?** Inject it. Compose it. Don't subclass.
- **Changing behavior?** Give it a new name. Don't overload the old one.

If you follow those two, you almost can't violate LSP. Which is why —

## The Awkward Admission

— this post was hard to write.

Not because the concept is hard. Because I genuinely don't have a Liskov violation in any of my code. The whole point of the principle is avoiding the bug I just showed you, and in my actual codebases I've already avoided it, mostly by accident.

In [LiquidMetal2D](https://github.com/mattcasanova/LiquidMetal2D), `GameObj` is the base class for every object in the engine. It has a `toUniform()` method — the one I [rewrote for the OCP post](/2026/04/17/house-rules-whatever-happens-happens/) — that any subclass can override to pack its own GPU data for a custom shader.

How many subclasses of `GameObj` override `toUniform()` in my actual engine?

Zero.

Every object the engine ships with uses the base implementation. The extension point exists. Nobody's extended it. No overrides means no chance to break the contract. No chance to break the contract means no LSP bugs. The canary is alive because there's no gas in the mine.

That isn't a point of pride. It's just the consequence of a rule I picked up years ago and never thought much about: **prefer composition over inheritance.** Shallow hierarchies. Interfaces where they earn their keep. When you do subclass, add, don't change. If what you actually want is different behavior, give the thing a different name and a different contract.

That's all of LSP, distributed across a bunch of small choices that don't feel like LSP when you're making them.

## The 90s Version, in One Paragraph

The textbook example for LSP is Square-extends-Rectangle. `Square` inherits from `Rectangle`, overrides `setHeight()` to also call `setWidth()` — because, you know, squares are squares — and now every caller expecting a rectangle gets a liar. Every blog post about SOLID since 2004 has told this story. Nobody writes this code. I taught this example to undergrads for seven years and every semester I felt a little dumber saying it, because I was *also* teaching them to never build deep inheritance hierarchies in the first place.[^2] I was using a 90s-shaped example to illustrate a principle that matters in non-90s-shaped codebases, and I never closed the loop. The retry client is the example I should have been using the whole time.

## The Canary

Here's the last thing, and then you can go to bed.

LSP isn't a principle you sit down and apply. There's no checklist. There's no "LSP pass" on the codebase. It's a **symptom you watch for** — the same way a canary in a coal mine is a symptom. When you see a subclass lying about its behavior, the bug is usually upstream. Deep hierarchy? That's an S violation. Implementation that secretly does something different? Probably O or I. Dependency hardcoded instead of injected? That's D.

If you follow S, O, I, and D, L mostly follows from them. When the canary drops dead, don't blame the canary — check the other four.

And if someone asks you to name the five principles in an interview someday, and you can, and the interviewer doesn't seem thrilled about that — congratulations. That's just a slightly different kind of LSP violation. They had a promise about what an answer to that question would look like. You gave them something different, with the same signature.

You didn't get the job. It's fine.

Still keeping the rules.

---

*This is the third entry in the **House Rules** series. Next up: the I. Interface Segregation. My Detekt linter caught it. I told it to shut up. Turns out the linter was right and I was the retrying HTTP client.*

*Halfway through SOLID. So far the answer to all of them is "interfaces." I'll let you know if that changes.*

*I didn't write 'em, but those are the rules.*

[^1]: Her paper was "Data Abstraction and Hierarchy." She later won the Turing Award for other work. None of that makes the name a description. "Add is fine, change is lying" is still the thing you need to remember.
[^2]: If I taught you C++ or intro programming between 2015 and 2022 in Singapore or Korea, consider this the apology.
