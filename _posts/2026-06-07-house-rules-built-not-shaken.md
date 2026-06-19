---
layout: post
title: "House Rules: Built, Not Shaken"
date: 2026-06-07
series: house-rules
tags: [architecture, design-patterns, builder, swift, opinion]
---

*SOLID series done: [S](/2026/04/15/house-rules-the-other-four-letters/) → [O](/2026/04/17/house-rules-whatever-happens-happens/) → [L](/2026/04/22/house-rules-it-compiled-it-lied/) → [I](/2026/05/05/house-rules-i-suppressed-the-linter/) → [D](/2026/05/06/house-rules-d/). Patterns next.*

It's late. I'm staring at a function signature with eight parameters. Five of them are nullable. There are three overloads of it depending on which surface is doing the calling. The function lives on one class out of six logger classes in this codebase, every one of them a public singleton, every one of them roughly doing the same job, none of them sharing a common interface.

Nobody on the team can tell me, with confidence, which one a brand-new feature should use.

Two choices here. You write a doc — *"How to Log a Click,"* a wiki page with a decision tree and a `// TODO` for the part you're still unsure about. A few people read it once. Most don't. The ones who do misremember it three weeks later, when somebody adds a seventh logger and doesn't update the page.

Or you can rip the whole thing out and put a Builder in front of it.

Not an interface.

A Builder.

I'm doing option two, and yes, that one hurt to type. I'll get to why.

## Built, not shaken

There are three ways to make a cocktail. You can shake it, you can stir it, or you can *build* it. A built drink gets made right in the glass — base spirit first, then the modifiers, then whatever goes on top, poured in one at a time, in order, until it's a drink. An Old Fashioned is a built drink. Sugar, bitters, a splash of water, the whiskey, the ice, the twist. In that order. In the glass.

That's the Builder pattern. "Built" isn't a metaphor I went reaching for. It's the same word the bartender uses.

You set up your inputs in order. The base goes in first — the part there's no drink without. The optional stuff goes in if the drink calls for it. Then it's done, and you serve it. Or, if you're me on a Tuesday: `.logClick()`.

Disclaimer, since I'm about to spend a whole post talking like I work behind a bar: I don't. I've never pulled a shift in my life. What I *do* have is an unreasonable number of amateur hours logged on the customer side of the rail — and a home bar I've over-invested in to the point that, if every Old Fashioned I've built at my own kitchen counter counted toward a degree, I'd be writing this with a PhD. I can't mix for a room on a Friday night. But I know how to *order* one, and I know a good order from a bad one. That's the side this whole metaphor lives on.

This kicks off a design-patterns sub-arc in House Rules, now that SOLID's in the can (five letters, one answer, all interfaces). I'm not writing a textbook — I already wrote half of one a decade ago and have spent [two](/2026/04/14/bad-advice-roll-your-own-di/) [posts](/2026/04/14/bad-advice-delete-your-singletons/) apologizing for it. I'll just write the patterns I'm actually using. Builder's first because I'm neck-deep in one, and it's the cleanest motivation for the pattern I've ever had.

## The simple rule I taught for years

**When you have more than three nullable parameters, a `Map<String, Any>`, or several god-class singletons that do roughly the same job — reach for a Builder.**

Or, the way I told my students for years and still say to anyone on this team who'll listen:

> **Right now, this is hard to use, easy to break. We need it to be easy to use, hard to break.**

That's the whole game. The Builder is one of the cleanest tools for getting from the first sentence to the second.

## The look-alike

If you've read the [engine posts](/2026/04/19/five-year-particle-system/), you've seen a `build()` on every scene in LiquidMetal2D, and you might be winding up to ask if *that's* the Builder pattern. It isn't. That `build()` takes no parameters, accumulates nothing, and hands back a default instance — it's a **Factory Method**, Builder's one-line cousin. Builder accumulates state across a chain and assembles something composite at the end. Same word in the API, different pattern under it. (The engine's version leans on a nice Swift trick — an overridable `class func` that returns `Self()`, so every subclass gets the right-typed factory for free. Worth its own post someday. Not today.)

## Now, the work

I have a rule about [the day job](/2026/05/09/nightcap-still-doing-it/): I'll talk about the work going well, but I don't tell war stories about the place that pays me. So — **this is not one.** It's a refactor, and a happy one: the right answer was sitting in the design-patterns book waiting to be picked up. What follows is a *hypothetical* shape — not the real classes, names, or schema. Anybody who's worked on a five-year-old multi-platform app will recognize it.

The job is funnel analytics — attributing a purchase or a signup back to the entry point that started the journey, so we can see which paths convert and which leak. Clicks, impressions, entry points. Logged on user actions, not in any loop.

### Six loggers walk into a bar

Imagine a codebase. Big one. Old enough that it's outlived most of the engineers who started it. Multi-platform — web, mobile, some kind of cross-platform UI runtime, an experimentation system layered on top.

Over the years it's accumulated logging classes. Each one was reasonable when it was written; each solved a real problem at the time. Today (names made up):

- `WebFunnelLogger` — the original web logger. Somebody later drafted it into being the "unified router" everything else writes through. Still named `Web`.
- `MobileUIActionLogger` — doesn't *log*; returns an *action* you attach to a button, fired on tap. Command pattern, welded to the UI layer.
- `ExperimentEntryLogger` — logs entry into an A/B-tested surface. Doesn't do clicks. Still public.
- `LegacyClickTracker` — deprecated three years ago. Nobody pulled the call sites, so new code copies the old code and keeps using it.
- `NewSurfaceLogger` — the new system. One parameter: a `[String: Any]` dictionary. Typo-safe? Catastrophically not.
- `RealtimeMetricLogger` — dashboard-in-30-seconds stuff. Bypasses the router. Own table, own schema.

All six are public singletons, reachable from anywhere, sharing no common interface — because the shapes really are different underneath. Three years ago somebody tried to unify them by making `WebFunnelLogger` route for the others. It's the closest thing to a unified surface this system has, and even it isn't really unified: most call sites predate it and bypass it, and nobody's done the sweep.

The unified router's `logClick` method, today, looks roughly like this:

```swift
public func logClick(
    session: Session,
    entryPoint: EntryPoint,
    productContext: ProductContext,
    eventTime: Date? = nil,
    contentOverride: String? = nil,
    surface: Surface? = nil,
    eventTarget: EventTarget? = nil,
    flowID: String? = nil
) { ... }
```

Eight parameters — and I'm being kind; the real one has more, plus a few `logClick` overloads sitting next to it for callers who wanted a slightly different shape and didn't want to touch the existing sites. Five-plus nullable. Every quarter, somebody adds another. The signature is a graveyard of *"I needed this for one feature."*

This is shouting a dozen ingredients at the bartender and praying he pours them in the right order. Nobody orders a drink like that. You don't lean over the rail and yell *"two ounces rye, quarter ounce simple, two dashes Angostura, no orange, yes cherry, big rock, express the peel, also nil, also nil, also nil."* You order an Old Fashioned and you say how you want it. The bartender knows the base. You add the modifications that matter. That's the whole difference.

The new `NewSurfaceLogger` takes the opposite approach:

```swift
NewSurfaceLogger.shared.log([
    "session": session.id,
    "entryPoint": "feed_compose",
    "product_context": productContext.dict,
    "event_time": Date()
])
```

One parameter: a dictionary. Beautiful in theory. In practice, typo `"entryPoint"` as `"entry_point"` once and your dashboard shows nothing for a week, because the table partitions on the spelling and nobody validates strings at runtime.

So a new feature has to know: which logger, what params, which spelling, override-or-`nil`, and which table it lands in three hops later. **Hard to use. Easy to break.** This is the system that quietly produces missing data — and you find out three weeks later when marketing asks why the funnel went flat. The data was never missing. It went into the wrong logger under the wrong key into a table the dashboard wasn't reading.

This isn't a story about a bad team. It's a story about [a roadmap that ran over a codebase nobody had time to refactor](/2026/04/14/bad-advice-delete-your-singletons/). Sound familiar.

### Why not interfaces?

If you've read any of the posts before this one, you know my reflex. Singleton problem? [Interface.](/2026/04/14/bad-advice-delete-your-singletons/) Vendor problem? [Interface.](/2026/04/15/bad-advice-wrap-your-vendors/) SOLID? Five letters, [one answer](/2026/05/06/house-rules-d/), all interfaces. *Put an interface on it* is, statistically, my answer to everything.

So the first thing I did with this logger mess — before I'd even read half of them — was sit down and try to draft an interface for it.

I couldn't. And it stung a little to learn there's a whole class of problem interfaces won't solve. Two reasons.

**One: there's no DI seam.** No `AppContainer`, no threading. The loggers are reached through static `.shared` calls from a hundred code paths. Retrofitting an interface and a container under all of that is a year-long project that gets cancelled the first time a feature roadmap lands on it. Political reality.

**Two: there's no common shape to abstract.** This is the deeper one. The six don't agree on a shape — one returns *actions*, one writes synchronously, one takes a dictionary, one queues. They aren't six implementations of an `EventLogger`; they're six different patterns that happen to emit log lines. A `protocol EventLogger` over them would be a lie — half the methods would be no-ops on half the types.

Interface unification works when the implementations agree on a shape. These don't. So the seam can't be at the *implementation* layer. It has to be at the *call* layer.

That's exactly what Builder gives you.

## Building the drink

Here's what the API I'm building looks like, sketched in Swift:

```swift
Logger
    .buildFor(session: session, entryPoint: .feedCompose, product: productContext)
    .withEventTime(Date())
    .withContentOverride("Custom CTA")
    .logClick()
```

Read it left to right.

**`Logger.buildFor(...)`** takes what's required — the base spirit: session, entry point, product context. The factory takes them, and the builder it hands back has no public constructor of its own. No base, no drink, and the compiler enforces it.

**`with...`** takes what's optional — the modifiers. Override the event time? `.withEventTime(...)`. Don't care? Leave it off. The nullable parameters disappear from the call site because they aren't parameters anymore — they're optional pours on the chain. Five nullables became zero.

**`logClick()`** is the terminal verb. Make it. It assembles the accumulated state into a `LogPayload` and hands it off to be written. There's also `logImpression()`, and whatever else the system needs — the terminals are the only places the drink actually gets poured.

The builder itself — the workhorse — looks like this:

```swift
public final class LogBuilder {
    private let session: Session
    private let entryPoint: EntryPoint
    private let product: ProductContext
    private var eventTime: Date?
    private var contentOverride: String?
    private var surface: Surface?
    private var eventTarget: EventTarget?
    private var flowID: String?

    // No public constructor. You only get one of these from the dispatcher.
    fileprivate init(session: Session, entryPoint: EntryPoint, product: ProductContext) {
        self.session = session
        self.entryPoint = entryPoint
        self.product = product
    }

    public func withEventTime(_ t: Date) -> LogBuilder {
        self.eventTime = t
        return self
    }

    public func withContentOverride(_ s: String) -> LogBuilder {
        self.contentOverride = s
        return self
    }

    // ... withSurface, withEventTarget, withFlowID, etc.

    public func logClick()      { WebFunnelLogger.shared.logEvent(payload(.click)) }
    public func logImpression() { WebFunnelLogger.shared.logEvent(payload(.impression)) }

    private func payload(_ kind: EventKind) -> LogPayload {
        LogPayload(
            kind: kind,
            session: session,
            entryPoint: entryPoint,
            product: product,
            eventTime: eventTime ?? Date(),
            contentOverride: contentOverride,
            surface: surface,
            eventTarget: eventTarget,
            flowID: flowID
        )
    }
}
```

A few things to notice.

It's a `class`, and each `with...` mutates and returns `self` — the standard Builder shape *Effective Java* has been preaching since the early 2000s, the same one you'd write in Kotlin or TypeScript. (You *could* make it a `struct` with copy-on-mutation for value semantics, but you'd pay `var copy = self; …; return copy` on every method for a thread-safety win most teams never cash in. Class is simpler.)

The constructor is `fileprivate` — there is no public way to make a `LogBuilder`. The only door in is the dispatcher, which we'll get to in a second.

The terminal verb is the one place that knows where a log goes. `logClick()` builds the payload and hands it to `WebFunnelLogger` — the old eight-parameter "unified router" from earlier, except I collapsed its whole signature into a single `logEvent(payload)` and made it `internal`. Every nullable that used to live on `logClick` now lives inside `LogPayload`, assembled once instead of at every call site. The caller doesn't know any of that happened, or that `WebFunnelLogger` still exists. They built a log and said `.logClick()`.

That's half the post. **Routing collapses into the terminal verb.** The next time we change which underlying logger handles which surface — and we will, because migrations don't end, they just pause for a quarter — that's a one-file change. Not four hundred call sites.

The other half is what you call to get a builder in the first place.

### Pick the drink when you order

When I first sketched this, I let you flag the variant at the *end* of the chain — build the whole thing up, then call something like `.asUILog()` to change what came out. I wrote it that way. Then I looked at it and realized it was backwards. You don't order an Old Fashioned, watch the bartender build it, and *then* tell him you actually wanted it to-go. You say what you're drinking when you order.

So the choice moves to the front. The public entry point isn't one factory — it's a few, and picking one is the first thing you do:

```swift
public enum Logger {
    public static func buildFor(
        session: Session, entryPoint: EntryPoint, product: ProductContext
    ) -> LogBuilder {
        LogBuilder(session: session, entryPoint: entryPoint, product: product)
    }

    public static func buildForUI(
        session: Session, entryPoint: EntryPoint, product: ProductContext
    ) -> UILogBuilder {
        UILogBuilder(session: session, entryPoint: entryPoint, product: product)
    }

    public static func buildForExperiment(
        session: Session, entryPoint: EntryPoint, product: ProductContext
    ) -> ExperimentLogBuilder {
        ExperimentLogBuilder(session: session, entryPoint: entryPoint, product: product)
    }
}
```

`Logger` is deliberately stupid. It holds no state. It makes one decision — which builder does this caller want — and gets out of the way, handing back a *different builder type* per variant. (`buildFor` rather than `for` because `for` is reserved in Swift; the `buildFor` / `buildForUI` / `buildForExperiment` family reads fine.)

And yes — I see it. `Logger` is a global I reach by static name, the exact thing I [told you to delete](/2026/04/14/bad-advice-delete-your-singletons/) a few posts ago. I can't dependency-inject it yet, and I'm not going to pretend I love that. But this is a five-year-old codebase with the logging entry point wired into hundreds of call sites, and you don't fix that in one PR — you fix it in passes. **Pass one is the clean interface**: every caller through one front door with the right shape. That's this post. Pass two makes the door swappable — hide the implementation behind something I can replace in a test, the [vendor-wrapper move](/2026/04/15/bad-advice-wrap-your-vendors/). The static name stays; what's behind it stops being a hardcoded global. I'm flagging it on purpose, because the "temporary" singleton nobody flags is how you get a global somebody swore they'd kill in 2021. The caseless `enum` keeps me honest meanwhile: no instance, no state, nothing to race.

Now the call sites. Each builder offers only the verbs that make sense for it:

```swift
// the common case → logs directly
Logger.buildFor(session: s, entryPoint: .feedCompose, product: p)
    .withContentOverride("Custom CTA")
    .logClick()

// the UI case → returns an action the button fires later
Logger.buildForUI(session: s, entryPoint: .feedCompose, product: p)
    .withContentOverride("Custom CTA")
    .getClickAction()

// the experiment case → records an exposure into the experiment pipeline
Logger.buildForExperiment(session: s, entryPoint: .feedCompose, product: p)
    .withVariant(.treatment)
    .logExposure()
```

`buildForUI` returns a `UILogBuilder`, and a `UILogBuilder` does not have a `logClick()`. It can't — the method isn't on the type. So you cannot pick the UI variant and then accidentally log directly; the compiler doesn't have the vocabulary for that mistake. It runs the other way too: a plain `LogBuilder` has no `getClickAction()`, and the experiment builder is the only thing carrying `withVariant(...)` and `logExposure()`. You chose your terminal when you chose your factory, and the type system holds you to it.

That's the win of moving the choice to the front: **the wrong terminal isn't a runtime check or a code-review catch. It doesn't compile.**

The variant builders are the same story as `LogBuilder` with different verbs. `UILogBuilder` doesn't log — its terminals (`getClickAction()`, `getImpressionAction()`) hand back an action the UI runtime fires on tap. `ExperimentLogBuilder` carries `.withVariant(...)` and a `logExposure()` terminal that routes into the experiment pipeline instead of the funnel tables. (In real life the three share their guts — required fields, common modifiers, payload assembly — through a base they all extend, so none of it is actually copy-pasted.)

## On the house

Two payoffs I haven't already hammered:

**The `Map<String, Any>` is contained.** Some of the underlying systems still want a dictionary, and the Builder still hands them one — but the *typo surface* is now an `EntryPoint` enum, a `Surface` enum, a typed `Date`. The compiler fails before the dashboard does.

**Migration is a sweep, not a rewrite.** I still have to touch every call site using one of the six loggers — real work, but mechanical: find the singleton call, replace with the chain, no judgment calls. When the sweep's done, the old loggers go `internal` and there's no path back.

Everything else — required-vs-optional enforced at the type level, the wrong terminal refusing to compile, one autocomplete-discoverable door instead of six — falls out of the shape for free.

## Where it doesn't fit — and what it costs

Every rule has exceptions, and every abstraction has a tax.

**Two parameters, no optionals.** `User(id: 1, name: "x")` doesn't need a Builder; a constructor reads fine. Builder earns its weight only when there are enough optional parameters to make a signature ugly.

**A hot loop.** Every `Logger.buildFor(...)` is a heap allocation. Building 100,000 of these a frame in a [particle system](/2026/04/19/five-year-particle-system/) will light up your profiler. Builder is for *occasional* assembly of *complex* things, not inner loops.

**You actually have a clean common interface.** If the loggers genuinely shared `EventLogger.log(event:)`, I wouldn't need a Builder — I'd need [DI and one interface](/2026/04/14/bad-advice-roll-your-own-di/). Builder shines precisely when you *don't* have that.

**You're tempted to put logic in it.** Builders are dumb; they accumulate fields. The moment `.withX()` validates or fetches or computes, you've built a service in a Builder costume. Move the logic to the terminal.

And the tax, even when it fits: it's a new vocabulary reviewers have to learn, you trade flat-arg autocomplete (every parameter at once) for "which `.withX()` exists," and the Builder has to track the schema — a new field means a new `.withX()`. All real. All cheaper than silently missing one of those nullables.

## Closing time

I've been writing the same post for two months. Singletons, fakes, vendors, all five letters of SOLID — every one had the same answer. *Put an interface on it.* I made it a running gag and [closed the SOLID arc](/2026/05/06/house-rules-d/) on it. *Five letters. One answer.* Mic drop, walk off stage.

This is the first post where the answer isn't interfaces — and that's the lesson under the lesson. **The principle isn't the tool.** The principle is **shape the seam between what a caller knows and what a system does.** Interfaces are one tool for that seam, and most days the right one. But when the implementations underneath don't agree on a shape, the seam can't sit at the implementation layer — it moves up to the call site, and there the right tool isn't an interface. It's a Builder.

SOLID was about the *shape* of the seam. Builder is about the *ergonomics* of one kind of seam — where the call site assembles a complex thing before handing it off.

A built drink isn't fancy. It's just the honest way to make something with more than two ingredients without forgetting one — base first, modifiers next, served when it's done. The Builder isn't fancy either. It's the honest way to call a system with more than two parameters and six front doors.

You don't make the customer recite the recipe. You don't make the new engineer memorize the back bar. You hand them one order. Base, modifiers, make it.

Right now, this is hard to use, easy to break.

Soon, it's going to be easy to use, hard to break.

That's the whole bit. Last call.

---

*I didn't write 'em, but those are the rules.*
