---
layout: post
title: "You're Gonna Carry That Weight"
tags: [architecture, error-handling, logging, mobile, ai, opinion]
---

It's late, I'm reading a diff, and I can't answer a simple question about my own app: if this call fails tonight, does anybody write it down?

Not "does the user see an error." That part works. The view model catches, the screen shows something honest, everyone moves on.

I mean *write it down*.

A record.

Evidence.

The thing you pull up three weeks later when a user says login's been weird and you need to know what actually happened at 9:47 on a Tuesday.

I traced the path. The throw was real. The catch was real. And between them? Maybe a log, maybe not. The only way to know was to walk every layer and check by hand. Then re-check tomorrow, because tomorrow there'd be new layers.

I pulled the thread expecting a small refactor. It's been three days rewiring how errors move through both apps.[^spoon]

### The receipts

I did the smart things early on this project. I'm not going to pretend otherwise. The receipts are on this blog.

The logger is a protocol, and [exactly one file in the codebase knows Sentry exists](/2026/04/15/bad-advice-wrap-your-vendors/). The HTTP layer threw typed errors: an enum on iOS, a sealed class on Android. View models caught them and collapsed them into a small set of client errors for the view, [and I showed you that code too](/2026/04/15/house-rules-the-other-four-letters/). The core library doesn't know what a screen looks like; the views don't know what a network looks like. There's even a separate event channel for the emergencies (force update, kill switch), because I've been on enough apps to know you build the brakes before the car is moving, not after it's through the guardrail.[^killswitch]

And I had a rule about logging. **Log at the throw site**, where the rich context lives: the status code, the endpoint, the thing that actually broke. Then rethrow, and let the view model catch it and set state for the screen. Views don't log; [their job is showing shit and reacting to shit](/2026/04/15/house-rules-the-other-four-letters/). Clean separation of concerns. I thought I was very smart.

Remember that sentence. It's load-bearing.

### The crack

Since then, the apps have been in full crank: [billing](/2026/07/06/nightcap-no-fireworks/), video players, offline handling, feature after feature chasing parity with a website that will not sit still. Which means AI has been writing a lot of this code, fast, with me [reading behind it](/2026/07/07/blade-runners/).

Two things started to itch.

The first was small and concrete: the structured data going to Sentry, the key/value context attached to each error, was stringly typed. `"status_code"` here, hopefully `"status_code"` there, nothing stopping a `"statusCode"` from quietly forking the data. My reporting queries would just... miss. No error, no warning, a dashboard that's confidently wrong. I don't accept magic strings from myself; I'm certainly not accepting them from a machine that generates code by the pound.

The second itch was the one that mattered. I kept looking at fresh error-handling code and thinking: I don't think it logged. My rule, log at the throw site and rethrow, lived in my head. It worked when I was the only author, because I knew it and I followed it.

I might know the rule.

I might even follow the rule.

**There is no guarantee the AI follows the rule.**

A convention is enforced exactly as often as you personally are the one typing. And these days I am, at most, one of the authors. The slowest one, reviewing the fastest ones. Any given throw site, written at machine speed on a Tuesday night, might log with full context or might not, and nothing in the build, the tests, or the type system would ever tell me which.

### Nothing to debug

Here's what that looks like from the other end. Stand in the view model's catch block. An error just arrived. Question: was it logged upstream?

You cannot tell.

Not "it's hard to tell." There is no way, standing at the catch site, to know. The error object doesn't carry a receipt. If the origin logged it, great: you log nothing, everything's fine. If the origin *didn't* log it and you log nothing, the failure never happened as far as Sentry's concerned. The user got an apology, I got silence, and the bug gets to age like it's on a beach somewhere with a new name.

So my options were: log defensively everywhere, drowning in duplicates as every layer charged the same failure twice, or trust every author of every throw site forever, including the ones that aren't people.

I've actually been burned by the polite version of this before. [The funnel logger that wrote `"unknown"`](/2026/05/01/think-mcfly-think/): a nullable parameter passed down ten layers to a logger that shrugged at the bottom. Same disease, different organ. Information that only exists at the origin, quietly dying somewhere in the middle, and a terminal that can't tell missing from fine.

And the worst part is what my old rule turns into at scale. Log-at-throw-then-rethrow means the middle of the stack is full of this. Real code, sitting in my repo as I write this:

```swift
} catch {
    logging.e("Video fetch failed", error: classify(error), extra: ["slug": slug])
    throw error
}
```

Logged *and* rethrown. Now every layer downstream inherits the ambiguity: it's already been reported, so if the view model logs too, that's two Sentry issues for one failure. And if the view model assumes someone upstream handled it, we're right back to hoping. A middle layer that logs-and-rethrows doesn't add safety. It poisons the well for everyone below it.

### Settle the tab once

So I tore the rule out and replaced it with one that doesn't need my memory, my discipline, or the AI's cooperation to be checkable:

**If you rethrow, you don't log. If you stop the error, you log.**

That's the whole rule. The error gets logged exactly once, at the *terminal*, wherever it stops propagating, and nowhere else.

Think of it as a bar tab. The origin opens the tab: it catches whatever foreign thing actually failed (a `URLError`, a decode miss, a 500), packages it into a rich, sanitized error of mine, and throws. Every layer the error passes through on the way down can add a line to the tab. More context. A breadcrumb for the itemization. But nobody charges the card mid-crawl. The tab gets settled once, at last call, wherever the night actually ends.

Usually that's the view model: it stops the error, turns it into screen state, and now it logs it too, with everything the error carried from the origin. But *terminal* means wherever the error stops, not "always the view model." A repository with a warm cache is allowed to catch a refresh failure, log it, and swallow it; the user has data, the screen doesn't need to hear about it. But swallowing doesn't waive the tab. The moment the repo stops that error, *it* is the terminal, and it settles up in full. The server still failed. The user getting lucky with a warm cache doesn't un-fail it. Important enough to swallow is important enough to log; a silent swallow is just the walkout with better manners. What the swallow changes is who has to hear about the error, not whether it gets written down.

Full disclosure: the first draft of this paragraph called that swallowed failure a breadcrumb, not an issue. Nobody saw it, nobody hurt, right? I read it back, disagreed with myself in real time, and went and changed the code. [The blog is still making the code better.](/2026/04/25/nightcap-twenty-one-posts-deep/)

One subtlety, because it looks like a violation and isn't. The response handler gets a 500 back and tries to decode the server's error body, and the decode *also* fails. Two errors are now standing in one catch block: the server fault, still on its way somewhere, and a decode fault whose road ends right here. So the handler logs the decode failure (its own event, its own tab, settled on the spot) and throws the server error onward, untouched and unlogged. A log sitting right next to a throw, and still not log-and-rethrow, because they're about different errors. The rule is per error, not per call site. Two failures. Two tabs. Each paid at its own terminal.

Log-and-rethrow, in this frame, is charging the customer's card at every bar on the crawl and then letting the tab run the card again at close. And my old failure mode, the unlogged error, is the other guy: the one who walks out on the tab entirely, and you find out three weeks later when the books don't balance.

One tab. One settlement. Everyone in the middle just carries it.

### Make the mistake un-compilable

A rule is nice. What I actually wanted was for the *wrong version to not compile*, because the compiler is the one reviewer that reads every line of every author's work, every time, and never gets tired at 11pm.

Start with the errors themselves. Here's what iOS had. One enum, one central switch, the shape I'd bet most codebases have:

```swift
enum APIError: Error {
    case unauthorized
    case forceUpdate(message: String, updateURL: String?)
    case killSwitch(message: String)
    case validationError(message: String, errors: [String: [String]])
    case networkError(Error)
    case decodingError(Error)
    case serverError(statusCode: Int, message: String?)
    case unknown

    func toClientError() -> ClientError {
        switch self { /* every case, one switch, forever */ }
    }
}
```

Fine for deciding what the screen shows. Useless for diagnostics: a case can't carry its own reporting behavior, so all the "what do we tell Sentry" logic ends up bolted on somewhere central, stringly, and forgettable.

Now each error is its own type, conforming to a small protocol that makes self-description part of the contract:

```swift
public protocol PRError: Error {
    /// What this failure is called in a report. The error knows; the call site doesn't restate it.
    var logMessage: String { get }
    var diagnosticContext: [ErrorKey: String] { get }
    func toClientError() -> ClientError
}
```

Every error answers for itself. Here's one of them, and I picked this one for a reason:

```swift
struct ValidationError: APIError {
    let message: String
    let errors: [String: [String]]
    let endpoint: String?

    var apiCase: APICase { .validation }

    /// Field names only. The messages echo the values the user submitted.
    var fields: [ErrorKey: String] {
        [.message: message, .validationKeys: errors.keys.sorted().joined(separator: ",")]
    }

    func toClientError() -> ClientError { .generic }
}
```

Look at the comment on `fields`. A validation failure knows which *fields* failed, and deliberately does not report what the user typed into them, because validation messages echo input back. More on that in a second.

The Android twin kept its sealed-class bones (Kotlin was already halfway there; sealed subclasses are real types) and picked up the same protocol. The doc comment on it states the whole design in one line: *adding a case is a compile error until it answers for itself; nothing can be added that reports nothing.*

Then the keys. The diagnostic dictionary isn't `[String: String]` anymore. It's `[ErrorKey: String]`, a closed vocabulary with the wire spellings in exactly one place. A stray key can't be spelled, only chosen. And those wire spellings are a contract shared verbatim with Android, because one Sentry query has to match one spelling, not two platforms' worth of creative casing.

Then the logger enforces classification at the signature:

```swift
func e(_ message: String, error: any PRError, extra: [String: String])
```

You cannot hand the error channel a raw `URLError` or a naked `DecodingError`. If it isn't one of mine (classified, sanitized, self-describing), it doesn't type-check. The AI doesn't have to remember the rule. The signature *is* the rule.

And the part I like most: the de-log sweep strips `LoggingService` out of every layer that only rethrows. A repo that never stops an error has no logger to misuse. The dependency isn't there. The layers that do stop errors (the view models, the repos that swallow behind a cache) keep theirs, and now the constructor is documentation: if a class holds a logger, it's a terminal for something. Delete the dependency everywhere else and the compiler personally walks you to every orphaned log call. I didn't sweep the codebase; the build did.

Levels got teeth in the same pass. The line that matters isn't severity first; it's fault versus context. A fault becomes a Sentry issue even if the user never felt it. Context never becomes an issue, no matter how much of it piles up:

| level | where it goes | what it's for |
|---|---|---|
| `.d` | console, debug builds only | dev tracing; never leaves the device |
| `.i` | breadcrumb: a rolling trail on the device, **not** an issue | context; the quiet lines on the tab |
| `.w` / `.e` / `.f` | event: a real Sentry issue, at matching severity | faults, ranked by how loud they should be |

Breadcrumbs cost nothing and raise nothing. They only ever leave the phone riding along with a real event, as its trail. A user whose session has no fault sends me nothing at all. When something real breaks, the last hundred quiet lines of that user's tab come with it. The forcing question per call site is simple: *is this a fault?* Event, at whatever severity it deserves. *Only useful as context when something else breaks?* Breadcrumb.

### Whose data is it

Threaded through all of this is the other thing that kicked off the rabbit hole: PII.

Sentry is a third party. A good one. I [chose it on purpose](/2026/04/15/bad-advice-wrap-your-vendors/). But my users didn't choose it, and they don't know it exists. What they type into a login form, what they search for, their email address: none of that is Sentry's business, because none of it is necessary to fix a bug. A user ID tells me who hit the error. That's enough.

So the errors sanitize at the source, by construction. Validation errors report field names, never values. The `setUser` call sends an ID and nothing else; the email came out this week. A search-repository log line that included the raw search text dies in the sweep, not scrubbed but *deleted*, because its constructor loses the logger entirely. The diagnostic context is safe not because every author remembers to be careful but because the types only have slots for the safe things.

Same lesson as the logging rule, one layer down: privacy-by-vigilance doesn't survive multiple authors either. Privacy-by-construction does.

### The only reviewer that reads every line

I've spent the last week building [blade runners](/2026/07/07/blade-runners/): reviewers and auditors, AI reading AI, briefings before I read a diff. I trust that machinery and it catches real things.

But there's a cheaper enforcer below all of it, and this refactor is really about relocating my rules down to it. A rule in my head is enforced when I'm typing. A rule in a doc is enforced when someone reads the doc, and the AI reads the doc *most* of the time, which is a different thing from all of the time. A rule in the type system is enforced on every build, for every author, at zero marginal cost, forever.

The hierarchy of where a rule can live turns out to be the whole game now. Head → doc → review → compiler. Every step down is a step from *probably* toward *provably*. AI throughput doesn't change what good architecture is. It changes how much of your architecture you can afford to leave as etiquette.

Conventions don't survive multiple authors. Structure does. I used to know that about human teams. Turns out it's the same lesson when half the team generates code by the pound and never went to your onboarding.

### What it costs

House style is honest about the bill, so: this isn't free.

An enum case was one line; a self-describing error type is thirty. The old `toClientError()` switch was ugly but it was *one place to read*. There's a window mid-migration where the terminal seam lands before the middle logs are stripped, and some failures get charged twice. I'll take a brief double-charge over a walkout, but it's real noise while it lasts. And "terminal" takes actual judgment at the edges: fire-and-forget calls (a progress ping on a timer, nobody awaiting it) have no view model to stop in, and if you don't deliberately give them a terminal, they become the new silent walkouts. That one's on the checklist *because* it almost wasn't.

Machinery's landed on both platforms: the error family, the typed keys, the level routing. The sweep that relocates every log to its terminal is this weekend's work.[^fable] The rule's already earning it: the response handler and half the repositories got *simpler*, because a layer that just throws is smaller than a layer that throws, logs, and wonders whether it should.

---

An error is weight. It gets picked up once, at the origin, by the only code that knows what actually happened. Everything in the middle carries it, maybe adds a line to the tab, but nobody sets it down early, and nobody pays twice. Whoever finally stops it settles the whole thing, once, with the full itemized night attached.

That's the deal I couldn't enforce by memo. So now it's in the types, where nobody gets to forget it: not me at midnight, not the machine at machine speed.

You're gonna carry that weight.

[^spoon]: Yes, this jumps the queue. The [spoon post](/2026/06/19/house-rules-two-drink-minimum/) is still owed. The rant cooled and started turning into something else, and I've learned not to bottle those early. The tab stays open.

[^killswitch]: Force update and a kill switch, wired into the error channel from day one, because every app I've ever seen need them built them *during* the emergency: something broken already shipped, and there was no way to stop it and no minimum version to hide behind. Those are expected conditions, not faults, which is why they ride a separate event channel instead of the throw channel. The app shell handles them; no view model ever logs them as errors. Build the brakes before the car moves. Then, [as I learned last week](/2026/07/06/nightcap-no-fireworks/), check that they're actually connected to the pedal.

[^fable]: The good model came back in the bottle this weekend, which is exactly the kind of help a two-platform sweep wants. [You know my policy.](/2026/07/07/blade-runners/)
