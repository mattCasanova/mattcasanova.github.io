---
layout: post
title: "House Rules: Don't Be Astonishing"
series: house-rules
tags: [architecture, solid, opinion, gamedev, swift, rust]
---

Years ago, before I got my current job, I interviewed at another big company. At some point the interviewer asked me to name the SOLID principles. Not explain them — name them. It was clearly a gotcha question. You could hear it in his voice. The tone you use when you don't expect someone to know all five off the top of their head.

I rattled them off. Cold. In order.

He didn't seem thrilled about that.

Here's the thing — Liskov Substitution sounds scary and it isn't. The other four letters tell you what they are. "Single Responsibility." "Open-Closed." "Interface Segregation." You can guess the gist from the name alone. "Liskov Substitution" tells you that someone named Liskov had an opinion about substituting. It sounds like something you'd find on a conference slide next to Dijkstra's algorithm and the Picard Maneuver — named after a person, sounds important, probably on a quiz somewhere.

It's not. It's the simplest principle in SOLID. It just has the worst name.

Here's the formal definition everyone skips past:

> **If S is a subtype of T, then objects of type T should be replaceable with objects of type S without breaking the program.**

Cool. Super helpful. Let me just tattoo that on my forearm so I can reference it during code review.

Here's what it actually means: **don't do unexpected shit.**

Does this thing behave the way everyone using it expects it to? Not the compiler — the people. The developer who calls your method next month. The teammate who trusts the interface. If yes, you're good. If no, you have a bug that no type system on earth can catch. That's the whole principle.

If you squint, it's the flip side of the [last post](/2026/04/17/house-rules-whatever-happens-happens/). OCP says "extend, don't modify." LSP says "when you extend, don't break what the parent promised." Two sides of the same coin.

## Hard to Show a Bug That Isn't There

This one was hard to write. Not because it's hard to understand — it's the easiest of the five once you strip away the name. It was hard to write because I don't have any LSP violations in my code. The whole point of the principle is avoiding the bug I'm about to show you, and I've already avoided it.

I'm not saying I don't have bugs — I absolutely have bugs. It's just that none of them are LSP bugs. Great for my app. Terrible for this blog post.

So I had to think up examples. And that's when something clicked: the reason I don't have these bugs isn't because I'm some kind of LSP savant. It's because I keep my hierarchies shallow and follow the other four principles — the ones I've been writing about for the [last](/2026/04/15/house-rules-the-other-four-letters/) [two](/2026/04/17/house-rules-whatever-happens-happens/) posts. That's actually the punchline of this post. I'm going to work up to it.

But first — the violations. You've seen them. Everyone has. You just never called them "LSP violations." You called them bugs.

## The Storage Service That Forgot to Store

In my iOS app, I have a `StorageService` protocol:

```swift
public protocol StorageService: Sendable {
    func get<T: Decodable & Sendable>(key: StorageKey) -> T?
    func save(key: StorageKey, value: some Encodable & Sendable)
    func remove(key: StorageKey)
    func clearAll()
}
```

And a `UserDefaultsStorageService` that implements it. Save something, kill the app, open it again — it's still there. That's the contract. Data persists.

Now imagine someone subclasses `UserDefaultsStorageService` to make a `CachedStorageService`. Not implementing the protocol separately — subclassing the concrete class:

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

Same methods. Same signatures. Same types. But the overrides only touch an in-memory dictionary. No UserDefaults. During the session, everything works — data's there, tests pass. Close the app and reopen it? Gone.

The child class broke the parent's promise: *your data persists.* It's a `UserDefaultsStorageService` that doesn't touch UserDefaults. Any function that takes a `UserDefaultsStorageService` is expecting persistence. This thing doesn't persist. The substitution fails.

Nobody would actually do this. Right? Hopefully you look at this code and immediately see what's wrong. That's the point — it's the same shape as every LSP violation, just obvious enough to spot.

The real ones aren't this obvious.

## Close Enough Is Where the Bugs Hide

Here's one that looks reasonable.

You have an HTTP client. `BasicHttpClient`. It has a `sendRequest()` method. One call, one request, one response. You build your app around that assumption. You write retry logic at the application layer — if it fails, wait, try again, three times max. It works. Clean architecture.

Then somebody needs retry logic closer to the metal. So they subclass: `RetryingHttpClient extends BasicHttpClient`. Same `sendRequest()` signature, but the override retries three times with exponential backoff internally. Problem solved. Ship it. Let's get a beer.

Except now your application-level retry calls `sendRequest()` three times. Each call internally retries three times. One failed request becomes nine. Maybe twenty-seven if there's another layer nobody remembers. The server gets hammered. Rate limits kick in. Somebody's pager goes off.

The subtype can't be substituted for its parent. `RetryingHttpClient` claims to be a `BasicHttpClient`. It isn't. You can't swap one for the other without the calling code breaking in ways nobody expected.

Here's the thing — there's nothing wrong with having both of these. A single-try client and a retrying client are both perfectly legitimate. The problem isn't that they exist. The problem is that one inherits from the other. Retry is claiming to BE a single-try client, and it's not. It's a different behavior wearing a familiar name.

**This is premature minimization.** You saw two things that looked similar — same method, same parameters, same return type — and you thought *"close enough, I'll just extend the existing one."* Less code. Feels clean. But the fake similarity is what blows up later. The retry client isn't "single-try plus a loop." It's a fundamentally different behavior that happens to share a method signature.

That's the core of LSP. Not the formal definition with the S and the T — this. **Claiming two things are the same to avoid the cost of admitting they're different.** The author saves a few lines today. The maintainer debugs a retry storm in six months. Same tradeoff from [the Fakes post](/2026/04/14/bad-advice-roll-your-own-fakes/): *the author pays a minute, the maintainer pays a month.*

The fix is boring: don't subclass. Have both implement an `HttpClient` interface. The caller asks for an `HttpClient` and gets one — it doesn't care which. But neither one claims to be the other. The retry client doesn't inherit from the single-try client. There's no lying. The contract is honest.

## The Ghost Enemy

This one's contrived. Fair warning. Modern game engines use component systems, not deep hierarchies. Nobody actually writes a `GhostEnemy` subclass anymore. But at a company with thousands of engineers and a codebase that's ten years old, with developers coming and going and making tweaks — someone writes a subclass that quietly breaks something three layers away. It happens. I've seen it.

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

Six weeks later, another developer writes a power-up that makes ghosts vulnerable — flash them red:

```swift
ghost.tintColor = Vec4(1, 0, 0, 1)
```

Nothing happens. The ghost stays translucent white. No crash. No error. The property was set. It just doesn't work on this one subclass. Not because someone forgot — because someone *chose* to enforce a game rule inside `toUniform()`. The base class made a promise: set `tintColor`, it shows up. The subclass broke that promise.

The fix: the uniform should just pack whatever it's given. Ghosts being translucent white is a game rule — enforce it where game rules live. In the scene's `update()`, in the ghost's initialization, in whatever spawns the ghost. Not in `toUniform()`. The uniform's job is packing data. The game's job is deciding what the data should be.

**Don't enforce business rules by breaking the contract at the wrong layer.**

## The Abstraction That Needs a Cheat Sheet

Here's one that's actually real. Not contrived, not hypothetical — it's a bug I know is waiting for me.

I'm building a [terminal emulator](https://github.com/mattcasanova/flux). Right now it runs on macOS and Linux. Eventually, Windows. My autocomplete code splits paths on `/`, because that's how paths work on Mac and Linux:

```rust
let (list_dir, prefix) = if let Some(last_slash) = partial.rfind('/') {
    let dir_part = &partial[..=last_slash];
    let file_part = &partial[last_slash + 1..];
    // resolve and list the directory...
};
```

On Windows, paths use `\`. That's not a bug in Windows — it's a legitimate platform difference. The world has real edge cases.

Here's the wrong fix. And I know the temptation, because I've seen this fix in other codebases:

```rust
fn normalize_path(path: &str) -> String {
    if cfg!(target_os = "windows") {
        path.replace('\\', "/")
    } else {
        path.to_string()
    }
}
```

Looks fine. Swap the slashes. Move on. But think about what you've done — you've hidden the edge case behind a lie. The rest of the code believes "all paths use `/`." Windows paths don't. You quietly normalize before anyone notices and hope nothing downstream cares.

Something downstream always cares. Someone pastes a native Windows path. Something displays it back to the user. Something logs it. The path separator got swapped, and now the output looks wrong on a platform where `\` is the standard. The abstraction said "I handle paths." It didn't say "I quietly change your paths to look like a different operating system."

The right fix is what I already have for the rest of the platform. In flux, I have a `Platform` trait that handles OS differences properly:

```rust
pub trait Platform {
    fn config_dir() -> PathBuf;
    fn data_dir() -> PathBuf;
    fn state_dir() -> PathBuf;
    fn cache_dir() -> PathBuf;
}
```

macOS returns `~/.config/flux`. Linux respects `$XDG_CONFIG_HOME`. Windows returns `%APPDATA%\flux`. Three different answers. The caller asks for `config_dir()` and doesn't check what platform it's on. Each implementation does its own thing. No `if windows` anywhere in the calling code.

The path separator should work the same way. A platform-specific normalization pass — part of the trait, not a hidden `if` buried in the autocomplete. Windows normalizes its paths. macOS and Linux do nothing. The calling code doesn't ask what OS it's running on. It doesn't need to.

I haven't fully solved this in flux yet — I'm being honest about that. But I know the shape of the solution because I can see what the wrong one looks like. And the signal is always the same: **when you have to write `if type == X` before calling your interface, the interface has failed.** That `if` statement is an admission that the abstraction doesn't actually abstract. You need more information than the type provides. The interface was supposed to handle this for you. It didn't.

Uncle Bob talks about this — the switch statement on type is the code smell. Every `if instanceof`, every `if type ==`, every `match` on an enum right before calling a method that should have just done the right thing — same admission. *"I need to know what you actually are because the interface didn't tell me enough."* The fix is always the same: push the behavior into the type. Don't check what it is. Let it do its own thing.

That's the difference between hiding an edge case and handling it. Hiding is a `normalize_path()` that quietly lies. Handling is a trait method that each platform implements honestly.

## Three Shells, One Contract

While I was looking for LSP violations in my code, I found something better. An example of it working. Silently. Invisibly. The way it's supposed to.

My terminal supports Bash, Zsh, and Fish. Each shell stores its command history in a completely different format.

Bash — plain text, timestamps on separate lines prefixed by `#`:
```
#1712700000
ls -la
#1712700060
git status
```

Zsh — extended format, timestamp and command on the same line, split by `;`:
```
: 1712700000:0;ls -la
: 1712700060:0;git status
```

Fish — YAML-ish, `- cmd:` prefixes:
```
- cmd: ls -la
  when: 1712700000
- cmd: git status
  when: 1712700060
```

Three shells, three completely different formats. Here's how I handle it:

```rust
pub trait Shell {
    fn parse_history_entry(&self, raw_line: &str) -> Option<String>;

    fn load_history(&self) -> Vec<String> {
        std::fs::read_to_string(self.history_file())
            .unwrap_or_default()
            .lines()
            .filter_map(|line| self.parse_history_entry(line))
            .collect()
    }
    // ...
}
```

Each shell implements `parse_history_entry()` its own way:

```rust
// Bash — timestamp lines start with #, skip them
fn parse_history_entry(&self, raw_line: &str) -> Option<String> {
    if raw_line.starts_with('#') { None }
    else { Some(raw_line.to_string()) }
}

// Zsh — strip the ": timestamp:0;" prefix
fn parse_history_entry(&self, raw_line: &str) -> Option<String> {
    if raw_line.starts_with(": ") {
        raw_line.splitn(2, ';').nth(1).map(|s| s.to_string())
    } else {
        Some(raw_line.to_string())
    }
}

// Fish — strip the "- cmd: " prefix, skip metadata lines
fn parse_history_entry(&self, raw_line: &str) -> Option<String> {
    raw_line.strip_prefix("- cmd: ").map(|s| s.to_string())
}
```

Look at `load_history()`. It reads the file, iterates lines, and calls `parse_history_entry()` on each one. It doesn't check which shell it has. Doesn't need to. Bash skips its timestamp lines. Zsh strips its prefix. Fish strips its YAML prefix. Each implementation honors the contract: *give me a raw line, I'll give you the command — or `None` if it's metadata.*

No `if shell == "bash"`. No switch statement. No cheat sheet. The caller asks for history entries and gets history entries. It doesn't know or care how the parsing works.

This is what LSP looks like when it's working. And it looks like nothing. There's no trick. No surprise. No clever abstraction. The code just does what you'd expect. Three shells. One contract. Nobody lied.

That's the whole point. When substitution works, there's nothing to see.

## The Canary

Here's the thing that clicked for me while writing this post.

LSP isn't a principle you sit down and apply. It's a **symptom you watch for.** When you see an LSP violation — when a subclass surprises you, when an implementation lies about what it does, when path three does something paths one and two would never do — it usually means one of the other four principles is broken upstream.

Deep inheritance tree where a zebra extends a building? That's an SRP violation — too many reasons for that class to exist. An implementation that secretly retries when the caller expected a single try? That's an OCP problem — you inherited when you should have composed. A dependency that lies about its behavior? That's a DIP issue — you're depending on a concretion instead of an honest abstraction.

Follow S, O, I, and D. LSP takes care of itself. It's the canary in the coal mine. When it drops dead, check the other four.

And that's why this post was hard to write. I follow the other four principles. I keep my hierarchies shallow. The canary is still alive and well. I had to make up a ghost to kill it.

---

*This is the third entry in the **House Rules** series. Next up: the I. Interface Segregation — the one where my linter caught it, I told it to shut up, and it turned out the linter was right.*

*So what have we learned so far? S: interfaces. O: interfaces. L: interfaces and shallow hierarchies. Sensing a pattern?*

*I didn't write 'em, but those are the rules.*
