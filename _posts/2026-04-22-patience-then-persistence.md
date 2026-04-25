---
layout: post
title: "Patience, Then Persistence"
tags: [liquidmetal, gamedev, architecture, swift, ios]
---

I started LiquidMetal2D in 2020. It couldn't save a file until last week. I wrote the persistence layer in Vegas — a city that, as a rule, takes a dim view of saving anything.

Six years feels like a lot. From the inside, it never did.

That's not neglect. That's just how projects work — and when it's a solo engine, every feature is a negotiation with yourself. *Do I want to deal with this now?*

Is persistence more important than a [particle system](/2026/04/19/five-year-particle-system/)? No way, that stuff's way cooler. More important than [broadphase collision detection](/2026/04/12/the-right-algorithm-still-slower/)? No way, that stuff's way cooler. More important than [multiple shaders in one render pass](/2026/04/19/render-pass-strikes-back/)? No way, that shit is cooler.

So I got a bunch of cool stuff out of the way. I still wasn't planning on doing persistence.

Then I started building a particle editor that needs to save config presets to disk, and I realized those presets weren't going to do anyone any good if they lived in some hidden sandbox. They need to land where the user can see them — Dropbox, iCloud Drive, Files.app. Otherwise it's not an editor, it's a toy.

The negotiation ended. Persistence got built.

Two kinds of persistence. Different enough that trying to unify them was a dead end.

## Two Problems, Not One

The first thing I realized before writing any code was that *persistence* is a word for two separate problems. They barely overlap.

**App-controlled.** The app picks the filename. The file is invisible to the user. Save games, settings, progress, caches. You don't even know these files exist; they live inside the app's sandbox, and the game decides when to read and write them.

**User-controlled.** The user picks the destination. The file shows up in Files.app, in iCloud Drive, in Dropbox, wherever. Exports, imports, user-owned content. The app can't just write one — it has to *ask* the user, via the system picker.

Most games only need the first one. Save games, settings, caches — invisible files the user never asks about.

Tools need both. A particle editor, a sprite animator — anything that produces files the user owns — needs the second too.

I'm building both kinds. So I needed both tiers.

I spent a few minutes trying to imagine a unified `save(...)` / `load(...)` API that handled both. It leaked both ways. An app-controlled save has no file browser. A user-controlled save has no predictable filename. Making them share one API meant lying to somebody.

So: two separate things. They share `Data` as the currency. That's the only thing they have in common.

## BlobStore, Back Room

App-controlled persistence. Four methods. String keys.

Wrap them in an enum if you want type safety.

```swift
public protocol BlobStore {
    func put(_ data: Data, key: String) throws
    func get(key: String) throws -> Data
    func list() throws -> [String]
    func delete(key: String) throws
}
```

That's the whole protocol.

Three things shipped with it:

- `FileBlobStore` — writes to `Documents/<subdirectory>/<key>`. The subdirectory lets an app partition namespaces: `"saves"`, `"settings"`, `"presets"`. About 40 lines of `FileManager` glue.
- `InMemoryBlobStore` — `Dictionary<String, Data>`. Exists so tests don't touch disk.
- `CodableBlobStore<T>` — wraps any `BlobStore` and eats the `JSONEncoder` / `JSONDecoder` boilerplate. Typed `put(T, key:)` / `get(key:) -> T`.

Had one *do we even need a protocol here* moment.

`FileBlobStore` could stand alone — it works.

Two things kept the protocol in.

First, testability. If `CodableBlobStore<Player>` was hardcoded to file I/O, you couldn't unit-test save/load without touching disk. Swap `InMemoryBlobStore` in, same code path runs in-memory, tests stay fast.

Second, a future `HTTPBlobStore` drops in the same way. If a game wants cloud saves, it attaches a different backend without changing any of the `CodableBlobStore<T>` usage.

Two reasons to keep the protocol. Both paid for it.

## Sync, On Purpose

Same logic for sync vs async.

`BlobStore` is sync. Could've gone async — `async throws` on every method, force structured concurrency at the protocol level. Didn't.

Most saves are menu taps and scene transitions, not the game-loop hot path. A 5KB JSON write is sub-millisecond; sync is fine. If a save gets large enough to stutter, the caller wraps:

```swift
Task.detached {
    try? store.put(blob, key: "checkpoint")
}
```

One extra set of curly braces. Flexibility at the call site, not in the protocol surface.

Async-by-default would have meant every test wraps in `await`, every UIKit handler bridges via `Task` either way, and the protocol doubles the day `HTTPBlobStore` shows up.

I make you wait. You can choose not to.

## Three Layers, All Optional

Three layers, each optional depending on what a game needs:

```swift
// Layer 1 — raw bytes, for anything you've already serialized.
try store.put(data, key: "slot1")

// Layer 2 — any Codable type, no JSON ceremony at the call site.
let players = CodableBlobStore<Player>(
    store: FileBlobStore(subdirectory: "players"))
try players.put(player, key: "slot1")

// Layer 3 — app-specific store that groups the typed stores into one thing.
final class GameStore {
    let players:  CodableBlobStore<Player>
    let settings: CodableBlobStore<Settings>

    func savePlayer(_ p: Player, slot: Int) throws { ... }
    func loadSettings() throws -> Settings            { ... }
}
```

A small game can get away with one `GameStore` mega object and reach into it from scenes. A bigger app with many typed concerns can build per-type stores (`SaveGameStore`, `SettingsStore`, `LevelStore`) and hand them out individually. The engine provides the primitives. How you compose them is your call.

I almost added a convenience init to `CodableBlobStore<T>` that would auto-create a `FileBlobStore` inside (`CodableBlobStore(subdirectory: "saves")`).

Talked myself out of it.

That convenience is a stealth attack that would kill testing.

Users would reach for it, skip the store injection, and make their code harder to mock later.

Small programming hit beats a sugar-coated bypass.

## DocumentIO, Front of House

User-controlled persistence. Wraps `UIDocumentPickerViewController` in `async/await` so scene code doesn't deal with picker delegates.

First draft had static methods:

```swift
DocumentIO.save(data:, suggestedFilename:, presentingFrom vc: UIViewController)
```

Every call site passed in the view controller. Fine. Until I realized a game engine has exactly *one* long-lived view controller — `LiquidViewController`, created once in `viewDidLoad`, alive for the lifetime of the app. Every scene's every call to DocumentIO would be handing in the same VC.

Refactored to a class that holds the VC once:

```swift
@MainActor
public final class DocumentIO {
    public init(presentingVC: UIViewController)

    public func save(data: Data, suggestedFilename: String) async throws
    public func load(contentTypes: [UTType]) async throws -> Data
}
```

The app builds it once in `viewDidLoad`. Scene code stops knowing about UIKit:

```swift
try await game.documents.save(
    data: json,
    suggestedFilename: "preset.particlepreset")
```

No VC. No picker. Just *"save this data, let the user pick where."*

### The Delegate Lifetime Thing

One non-obvious bit worth writing down, because it took me longer than it should have.

`UIDocumentPickerViewController.delegate` is a **weak** reference. That's fine for the old closure-then-forget pattern, but `async/await` needs the delegate to stay alive until the user actually finishes picking a file. Two patterns for handling this:

1. `objc_setAssociatedObject` — attach the delegate's lifetime to the picker via the Objective-C runtime. Idiomatic in UIKit-to-async bridges. Looks like black magic to readers who've only written Swift.
2. A self-retaining coordinator — the coordinator holds a strong reference to itself in `init`, clears it in each completion callback.

I picked (2). Readability won. *"Manual retain cycle"* is its own code smell, but every Swift reader can follow what's happening, and the one-line comment above `selfRef = self` explains the rest. The ObjC runtime version would've been two lines shorter and a month harder to debug.

## Almost Built a Singleton

Here's the question I asked Claude before I wrote a single line of persistence code. Not *how do I implement BlobStore.* Not *how do I wrap UIDocumentPickerViewController.* The question was: **how is any scene in any game going to actually reach this stuff?**

Because the default-bad answer is obvious. You build a class, it does a job, you add it as a singleton so anyone can reach it from anywhere, and a few months later you're writing a post on a blog called [*Delete Your Singletons*](/2026/04/14/bad-advice-delete-your-singletons/) and cursing yourself from the past.

The easier path is the trap. If you don't ask the access question *before* you write the code, you find yourself halfway through an implementation with a concrete class and the path of least resistance is `DocumentIO.shared` somewhere that everyone in every scene in every game can reach. It works for a while. It tests like shit. It migrates worse.

So: before a line of `FileBlobStore` got written, we talked about how consumers would get at it.

The current scene init looks like this:

```swift
func initialize(
    sceneMgr: SceneManager,
    renderer: Renderer,
    input: InputReader)
```

Three arguments. Fine for today. Breaks the moment I add another service. Every scene in every game ever written against this engine would have to update the signature. And then the next one. And the one after.

A few options I rejected before settling on one.

**Keep expanding the initializer signature.** No.

**Service locator on `SceneManager`** (`sceneMgr.resolve(DocumentIO.self)`). Hides dependencies, needs a whole registry to mock — still secretly a singleton, just wearing a different jacket.

**Opaque `AnyObject?` context bag.** Every scene does `guard let ctx = bag as? MyContext` at the top. Fragile. Lies about what scenes depend on.

What I went with: a typed protocol the app extends.

```swift
// Engine provides the floor:
public protocol SceneServices: AnyObject {
    var renderer: Renderer      { get }
    var input:    InputReader   { get }
    var sceneMgr: SceneManager  { get }
}

// App extends with whatever it needs:
protocol GameServices: SceneServices {
    var players:   CodableBlobStore<Player>   { get }
    var settings:  CodableBlobStore<Settings> { get }
    var documents: DocumentIO                 { get }
}
```

The app builds one concrete `AppGameServices` object at startup, holding real instances of everything the game uses. The engine passes it to every scene. A scene casts once to `GameServices` up front and reads services off a single typed reference. New engine service? Extend `SceneServices`. New app service? Extend `GameServices`. The scene protocol itself never churns.

Everything swappable. Everything injectable. Nothing global. If the app wires the wrong services class, the cast fails fatally on boot — which is the right signal, because that's a programming error, not a runtime condition.

The takeaway is small. **Before you build a thing, decide how people are going to reach it.** That's a thirty-minute conversation that saves you a multi-month rescue operation years later. Ask the question. Don't build the singleton.

Filed as [issue #117](https://github.com/mattCasanova/LiquidMetal2D/issues/117). The persistence PR lands first; the services refactor follows as its own PR, because the persistence work is what forced the services question to exist in the first place.

## What's Not Here Yet

A few things not in this PR:

- **Versioning.** Any save-file format will need it the moment the app ships a second version and somebody has an old save. Haven't solved it yet. Probably a per-file schema version and a migration hook, but I want to see real game code try `CodableBlobStore<T>` before I over-design the migration story.
- **Encryption.** Not included. If a game needs it, wrap a `BlobStore` with an encrypting decorator — same pattern as `CodableBlobStore<T>` wraps a `BlobStore` for typed storage.
- **Cloud sync.** iCloud ubiquity container is its own project. `HTTPBlobStore` is sketched — I have a reference implementation in another codebase — but isn't in this PR. When it lands, consumer code won't change.
- **Demos.** Neither API is wired into a demo yet. Easy follow-ups: a *"save your last position, resume from it"* demo for `BlobStore`, and an *"export / import a particle preset"* demo for `DocumentIO`. Both coming.

---

Six years of saving nothing. Now the engine saves things. The particle editor finally has a place to put its files.

More soon.
