---
layout: post
title: "House Rules: Don't Be Astonishing"
series: house-rules
tags: [architecture, solid, opinion, gamedev, swift]
---

Years ago, before I got my current job, I interviewed at another big company. At some point the interviewer asked me to name the SOLID principles. Not explain them — name them. It was clearly a gotcha question. You could hear it in his voice. The tone you use when you don't expect someone to know all five off the top of their head.

I rattled them off. Cold. In order.

He didn't seem thrilled about that. At least from what I could tell. Anyway, I didn't get that job. But at least I knew those SOLID principles, right?

Most people can rattle off one or two of the letters. Maybe three on a good day. But there's always one that trips people up. Liskov Substitution sounds scary and it isn't. It's just hard to remember — we don't exactly drop these names in daily conversation, and "Liskov" isn't the easiest name to pull out of thin air. Unless maybe you're a nerdy ex-CS teacher who blogs at 11pm.

"Single Responsibility" tells you what it is. "Open-Closed" tells you what it is. "Liskov Substitution" tells you that someone named Liskov had an opinion about something. That's the whole reason interviewers love it. It sounds academic. It sounds like something you'd only know if you read the right textbook — like Dijkstra's algorithm or the Picard Maneuver.

It's not. It's the simplest principle in SOLID. It just has the worst name. No offense to Uncle Bob... or Barbara Liskov.

## The Honest Part, Up Front

This one was hard to write.

Not because the concept is hard — it's probably the easiest of the five once you strip away the name. It was hard to write because I don't have any LSP violations in my code. The whole point of the principle is avoiding the bug I'm about to show you, and I've already avoided it. It's hard to find an example of a thing that isn't there.

I'm not saying I don't have bugs — of course I have bugs. It's just that none of them are LSP bugs. Great for my app. Terrible for this blog post.

So I had to think up examples. And that's when something clicked: the reason I don't have LSP bugs isn't because I'm some kind of LSP genius. I just don't have deep hierarchies. Fewer levels, fewer chances for a child to break the parent's contract. No deep trees, nothing to break.

That, and following the other four principles — the ones I've been writing about for [the](/2026/04/15/house-rules-the-other-four-letters/) [last](/2026/04/17/house-rules-whatever-happens-happens/) two posts. That's the actual punchline of this post. I'm going to work up to it.

## Don't Be Astonishing

The formal definition everyone skips past:

> **If S is a subtype of T, then objects of type T should be replaceable with objects of type S without breaking the program.**

If you squint, this is just OCP. The [last post](/2026/04/17/house-rules-whatever-happens-happens/) was about extending behavior instead of modifying it. LSP is the other half: when you extend, make sure the child doesn't break what the parent is doing.

The problem with the formal definition — "without breaking the program" — is that these violations often don't break anything. The program runs fine. No crash, no error, no failed test. The *behavior* changes — in astonishing ways. That's the real issue. Not broken programs. Broken expectations.

Uncle Bob calls this the Principle of Least Astonishment. **Don't be astonishing.** That's his version.

I have my own: **don't do unexpected shit.**

Does this thing behave the way everyone *using* it expects it to? Not the system — the people. The developer who calls your method next month. The teammate who trusts the interface. If they set a property and it silently does nothing, the program didn't break. Their mental model did. And some poor bastard is going to spend a day figuring out why.

## The Obviously (Hopefully) Bad Example

You've seen this bug. Everyone has. You just never called it an LSP violation.

Here's a deliberately stupid example to show why it matters. In my iOS app, I have a `StorageService` protocol:

```swift
public protocol StorageService: Sendable {
    func get<T: Decodable & Sendable>(key: StorageKey) -> T?
    func save(key: StorageKey, value: some Encodable & Sendable)
    func remove(key: StorageKey)
    func clearAll()
}
```

And a `UserDefaultsStorageService` that implements it:

```swift
public class UserDefaultsStorageService: StorageService {
    private let defaults: UserDefaults

    public func save(key: StorageKey, value: some Encodable & Sendable) {
        let data = try encoder.encode(value)
        defaults.set(data, forKey: prefixed(key))
    }

    public func get<T: Decodable & Sendable>(key: StorageKey) -> T? {
        guard let data = defaults.data(forKey: prefixed(key)) else {
            return nil
        }
        return try decoder.decode(T.self, from: data)
    }
}
```

Save something, kill the app, open it again — it's still there. That's the contract. Data persists.

Now imagine someone subclasses it to make a `CachedStorageService`. Not implementing the protocol — subclassing `UserDefaultsStorageService` directly:

```swift
class CachedStorageService: UserDefaultsStorageService {
    private var cache: [String: Data] = [:]

    override func save(key: StorageKey, value: some Encodable & Sendable) {
        cache[key.rawValue] = try? encoder.encode(value)
    }

    override func get<T: Decodable & Sendable>(key: StorageKey) -> T? {
        guard let data = cache[key.rawValue] else { return nil }
        return try? decoder.decode(T.self, from: data)
    }
}
```

Same methods. Same signatures. But the overrides only touch an in-memory dictionary. No UserDefaults. During the session, everything works. The data's there. Tests pass. But close the app and reopen it? Gone. The child class broke the parent's promise: *your data persists.*

Nobody would do this, right? Of course they should call `super`. Hopefully you look at this code and immediately see why it's wrong. Any function that explicitly takes a `UserDefaultsStorageService` is expecting data to persist to UserDefaults. The subclass just... doesn't do that. It's a `UserDefaultsStorageService` that doesn't touch UserDefaults. The substitution fails.

That's LSP. You just never called it that.

## The 90s Version

I'm going to get through this fast because you've heard it.

Deep inheritance trees. `Square` extends `Rectangle`, you call `setHeight()`, the square secretly changes the width too. The caller expected a rectangle. They got a liar. This example has been in every SOLID blog post written since 2004.

I never worked at Zynga, but a buddy of mine did. This was years ago, so I'm going off memory, but the story goes: they built FarmVille, then CityVille, then one of the animal-themed -Ville games. The codebase kept growing, and at some point the inheritance trees got deep enough that something like a zebra ended up inheriting from a movable building. A zebra. Inheriting from a building. Because a building inherited from a farm, and a car or a blimp or whatever needed to move so it became a movable building, and then the animal game came along and needed all that same click-to-collect behavior, so the animal inherited from... a movable building. That's astonishing.

Nobody writes code like that anymore. At least I hope not. So people hear "Liskov Substitution" and think *"90s problem, doesn't apply to me."*

It does. It just looks different now. And I'll be honest — I'm partly responsible for why people think it doesn't.

## The Wrong Version

I taught SOLID principles to college undergrads for seven years. Two semesters a year, every year. By the end, I could explain LSP in my sleep — and I did, the same way, every time. Squares and rectangles. The hierarchy that breaks.

To be clear — I wasn't teaching deep hierarchies. I never have. Even back then, I taught shallow hierarchies, composition, the stuff that actually works. But when it came time to explain LSP, I reached for the textbook examples, because those were the examples that existed. Squares and rectangles. Subtypes that lie about their dimensions.

Not once did a single one of those students hit that bug at their first job.

Because I wasn't teaching them to build deep hierarchies in the first place. The examples I used to explain LSP assumed an architecture I was actively telling them to avoid. No wonder it felt academic — I was teaching a principle using code nobody would write.

The hierarchies got shallower. The violations didn't go away — they just got harder to spot. Let me show you what they look like now. And then: how do you fix subtypes that lie?

## The Modern Version

You have an HTTP client. `BasicHttpClient`. It has a `sendRequest()` method. One call, one request. You build your app around that assumption. You write retry logic at the application layer — if it fails, wait, try again, maybe three times max. It works. It's clean.

Then somebody needs retry logic closer to the metal. So they subclass it: `RetryingHttpClient extends BasicHttpClient`. Same `sendRequest()` signature, but the override adds three retries with exponential backoff. Problem solved. Let's get a beer.

Now your application-level retry calls `sendRequest()` three times. Each of those calls internally retries three times. One failed request turns into nine. Maybe twenty-seven if there's another layer. The server gets hammered. Rate limits kick in. Somebody's pager goes off.

The subtype can't be substituted for its parent. That's the violation. `RetryingHttpClient` claims to be a `BasicHttpClient`. It isn't. You can't swap one for the other without the calling code breaking in ways nobody expects.

These are LSP violations wearing a trench coat and calling themselves "features."

## The Edge Case Problem

Here's the part nobody talks about: these violations don't happen because someone was dumb. They happen because the world has legitimate edge cases.

I'm building a [terminal emulator](https://github.com/mattcasanova/flux). Right now it runs on macOS and Linux. Eventually, Windows. Yesterday I was working on path expansion — when you type `~/` in the autocomplete, it expands to your home directory. When you type `$HOME`, same thing. That part works. But my autocomplete code splits paths on `/`, because that's how paths work on Mac and Linux. On Windows, paths use `\`.

Windows isn't broken. Nobody's dumb. It's just a legitimate edge case — the platform really does work differently. And my contract needs to handle that. I haven't fully solved it yet. I haven't decided exactly how. But I know the problem is there, and I know the wrong fix when I see it: `if windows { swap_slashes() }` somewhere in the path expansion, quietly normalizing everything to `/` before the rest of the code sees it. That's hiding the edge case, not handling it. The autocomplete works, the paths resolve, and then someone pastes a native Windows path and something downstream blows up because the abstraction lied about what it was given.

Same tradeoff from [the Fakes post](/2026/04/14/bad-advice-roll-your-own-fakes/): *the author pays a minute, the maintainer pays a month.*

## The Ghost Enemy

LSP done right just looks like normal code. There's nothing to show. So the example is contrived. And to be fair, nobody writes a `GhostEnemy` subclass anymore — modern game engines use component systems, not deep hierarchies. It might even look stupid. But contrived doesn't mean impossible. On your solo project or your eight-person dev shop, sure — you'd catch this. But at a company with tens of thousands of engineers, a codebase that's five or ten years old, with developers coming and going and making little tweaks? Someone wrote a base class years ago. It gets used in a dozen places. Nobody remembers the original contract. Somebody subclasses it and makes a reasonable-looking decision that quietly breaks something three layers away. That happens. I've seen it.

In [LiquidMetal2D](https://github.com/mattcasanova/LiquidMetal2D), every `GameObj` has a `tintColor` property and a `toUniform()` method that packs data for the GPU. The base class honors the contract — set `tintColor`, it shows up on screen:

```swift
open func toUniform() -> UniformData {
    let uniform = AlphaBlendUniform()
    uniform.transform.setToTransform2D(
        scale: scale, angle: rotation,
        translate: Vec3(position, zOrder))
    uniform.color = tintColor  // whatever you set, it gets packed
    return uniform
}
```

Now imagine someone writes a `GhostEnemy` subclass. Game rule: ghosts are always translucent white. So the developer hardcodes it:

```swift
class GhostEnemy: GameObj {
    override func toUniform() -> UniformData {
        let uniform = AlphaBlendUniform()
        uniform.transform.setToTransform2D(
            scale: scale, angle: rotation,
            translate: Vec3(position, zOrder))
        uniform.color = Vec4(1, 1, 1, 0.5)  // always translucent white
        return uniform
    }
}
```

Looks clean. Ghosts are white. Ship it.

Six weeks later, another developer writes a power-up that makes ghosts vulnerable. Flash them red:

```swift
ghost.tintColor = Vec4(1, 0, 0, 1)
```

Nothing happens. The ghost stays translucent white. No crash. No error. The property was set. It just doesn't work on this one subclass.

Not because someone forgot. Because someone *chose* to enforce a game rule inside `toUniform()`. That's what makes it LSP and not just a bug — it was a deliberate decision that broke the contract the base class made.

The fix: the uniform should just pack whatever it's given. Ghosts being translucent white is a game rule. Enforce it where game rules live — in the scene's `update()`, in the ghost's initialization, in whatever spawns the ghost. Not in `toUniform()`. The uniform's job is packing data. The game's job is deciding what the data should be.

**Don't enforce business rules by breaking the contract at the wrong layer.**

## So How Do You Actually Avoid This?

Same answer as the last two posts: interfaces and shallow hierarchies. Follow the other four principles and this one mostly takes care of itself.

And that's why this post was hard to write. I follow the other four principles. I keep my hierarchies shallow. I had to make up a ghost to kill.

---

*This is the third entry in the **House Rules** series. Next up: the I. Interface Segregation — the one where my linter caught it, I told it to shut up, and it turned out the linter was right.*

*So what have we learned so far? S: interfaces. O: interfaces. L: interfaces (and shallow hierarchies). Sensing a pattern?*

*I didn't write 'em, but those are the rules.*
