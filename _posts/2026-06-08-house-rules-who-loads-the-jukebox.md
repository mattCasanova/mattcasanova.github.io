---
layout: post
title: "House Rules: Who Loads the Jukebox?"
series: house-rules
tags: [architecture, design-patterns, factory, swift, opinion]
---

*Design-patterns arc: [Builder](/2026/06/07/house-rules-built-not-shaken/) → Factory. SOLID's [in the can](/2026/05/06/house-rules-d/).*

I'll confess the thing I've been saying for three months straight: my answer to almost every architecture question is *put an interface on it.* [S was interfaces. O was interfaces. All five letters, one answer.](/2026/05/06/house-rules-d/) I love interfaces. I would marry an interface. If a problem holds still long enough, I'll hide it behind a protocol and walk away whistling.

So let's do the post where I admit the one thing an interface flat-out cannot do for you.

You can't `new` an interface.

Polymorphism handles everything you do to an object *after* it exists — call its methods, pass it around, swap its guts — and never once makes you name a concrete type. Beautiful. Right up until somebody has to actually *build the thing.* The real `MenuScene`. The real `LightStrip`. An interface won't construct itself, and the line where you type the real class name is the one place all that lovely decoupling springs a leak. **Interfaces don't solve construction.** No matter how hard I try.

Damn it.

And before you reach for it — yes, I love dependency injection too. I spent [a whole post](/2026/04/14/bad-advice-roll-your-own-di/) rolling my own container and telling you to delete your singletons. But DI doesn't make construction vanish either; it just marches it to one spot — the composition root — and does it all up front, by hand. My `AppContainer` still `new`s every concrete in its graph with its own two hands. Interfaces *hide* the concrete. DI *relocates* the concrete. Neither one *builds* it for you. Something, somewhere, always has to say the real class name out loud — and that's not a thing you abstract away. It's a thing you *corner.*

That leak has a name, a pattern that plugs it, and — unlike most of what I write here, where I'm showing you a scar — **this is one I just get right.** Have for over a decade. I got it wrong for about two years starting out; I've gotten it right for eighteen since. No hedge coming, no "or maybe I'm full of it." The pattern is rock solid, and the extreme version of it living in my game engine is, and I mean this with full technical precision, *fucking badass.* That's a quarter in the swear jar, and I'd pay it again before the post is over.

It's the Factory. Let me show you why I'm so smug about it.

## My first real architecture lesson

I started programming in 2006 at DigiPen, and here's the part nobody warns you about: the *coding* came easy. Loops, pointers, syntax — fine. **Architecture** was what wrecked me. *Where does this go? Who's allowed to know about that? Why does changing one thing detonate five others?* I was a halfway-decent programmer and a genuinely bad architect, and I knew it.

The Factory is where that started to turn. It was the first time a piece of architecture went from *"thing I copy off someone smarter"* to *"thing I just have."* So no, I'm not going to dramatically discover it in front of you. I've known this one was right for longer than some of you have been writing code.

Here's where it came from. My first game, 2008, had an "engine" that was, in its entirety, a `switch` on the current scene — menu, level one, level two, every screen's logic crammed in one file.

I already dragged that switch out once, in [the OCP post](/2026/04/17/house-rules-whatever-happens-happens/), as the poster child for *open for modification* — every new scene meant surgery on the engine. All true. But it commits a *second* crime, and that one's this post's: **the engine knew every scene by name.** It was welded to the one specific game sitting on top of it.

Then [Sync Ball](https://apps.apple.com/us/app/sync-ball/id356277312), 2009–2010 — a real, shipped, contracted game, two of us and a project manager — taught me the lesson for keeps. We scoped it tight: a menu, a gameplay screen, levels from data. Then scope crept the way scope always does — settings, transitions, a final screen — and the ugly truth came out: I had *no scene management at all.* Just hacks bolted onto one hardcoded loop, screen by screen, as the deadline closed in. That's when it landed. Every game is a loop that switches scenes; you never know how many there'll be; so the switching has to become *its own thing that doesn't know your scenes.* That thing became the skeleton of every game I've built since. I didn't have the word "factory" yet. I just knew the switch had to die.

By 2012 I was teaching this to sophomores, twice a year, for years. So when I tell you I'm confident here, it's not bravado — I've explained this exact pattern out loud to a few hundred people and watched it click for them. It is load-bearing in my hands.

## Who loads the jukebox?

There's a jukebox in the corner of the bar. You walk up, drop a coin, punch **B7**, and a song plays.

You don't know which record it is. You don't know how it's wired, where B7 sits on the spindle, what mechanism drops it onto the platter. You punched a code and got music. Tomorrow the owner swaps the records — B7 is a different song now — and you punch the same button the same way. New catalog, same buttons.

And here's the part this whole post is named after: **this jukebox doesn't ship with the songs.** It ships empty. Somebody — the bar owner, not the machine — *loads it.* The machine knows how to map a code to a slot and drop a record. It knows nothing about what music is in town this month. (Hold that "empty" thought; not every factory is loaded this way, and the difference is the third point below.)

That's the Factory. Two things are always true — and a third tells you which kind you're building.

Always true:

1. **Everything that comes out shares a shape.** Punch any code, you get a *record* — a thing that plays. If B7 gave you a song and B8 gave you a sandwich, there'd be no common "play it" to build the machine around.
2. **The choice is made at runtime, and the key comes from outside.** The jukebox doesn't know which button gets pressed. The scene manager doesn't know which `SceneType` the level you're leaving will name next. A deep link, a query param, a dropdown, the previous scene — the key shows up from somewhere the factory doesn't control, at a moment it can't predict. That's the whole reason this can't be a plain constructor call: *at the call site, you don't know the answer yet.*

Miss either of those and you don't have a factory — you've got something else wearing the coat.

The third one is optional, and it just sets the flavor:

3. **Maybe the catalog comes from outside too.** Sometimes the set of products is fixed and known — a `switch` over six enum cases, all of them right there in the file. Still a factory. But sometimes the factory doesn't even know the *list*; it's loaded at startup by code the factory has never met. My game engine lives at that extreme, by necessity — it cannot know what scenes your game has. That open end is the hard, rare, *fun* case, and it's the one that earns the badass label later. A closed enum factory needs none of that machinery.

## Eight in my office, and counting

Before any engine talk, the version you'll actually write this month — and we've got a running example for it. Back in [the singletons post](/2026/04/14/bad-advice-delete-your-singletons/) we built the Govee app — you know Govee, the color-changing light strips and bars and neon panels, eight of which are glowing behind me as I type this — and we did the hard part there: killed the `SuperDevice` god-class and gave every product a shared `Device` interface.

But an interface doesn't build itself (you're going to hear me say that a lot). When the account API hands back the user's gear, *something* has to turn each blob of JSON into the right concrete `Device`. That something is a factory, and it's exactly one spot:

```swift
enum DeviceFactory {
    static func make(from json: DeviceJSON) -> Device {
        switch json.type {
        case "light_strip": return LightStrip(json)
        case "light_bar":   return LightBar(json)
        case "neon_panel":  return NeonPanel(json)
        default:            return UnknownDevice(json)
        }
    }
}
```

That's the **closed** kind. The catalog is right there — a handful of cases, known at compile time, no registry, no tricks. But run it past the two gates. Every branch returns a `Device` — shared shape. And *which* one is decided at runtime by a key from **outside** — Govee's API picked `type`, not you. A deep link does the identical job: `govee://device?type=neon_panel` builds the right one and your router only ever holds `Device`.

This is the factory you reach for without thinking, and it's worth saying out loud what it bought you. Every `if json.type == "light_strip"` that *would* have metastasized across your rendering, your controls, your scene-sync code — gone, collapsed into this one switch. Downstream, nothing ever types `LightStrip` again. Govee ships a new panel next quarter and you touch one place, not forty call sites — the same [open-closed win](/2026/04/17/house-rules-whatever-happens-happens/) from the OCP post, this time bought at the construction seam. You've written ten of these. You probably never called it "the Factory pattern." It is.

And it's the tame version. Now the fun one.

## I have built this in six languages

I have built this exact machine dozens of times. C, C++, Objective-C, Java, Kotlin, Swift. In shipped game engines and in the bare console engines my students wrote to put tic-tac-toe inside a real architecture — a class I taught in Korea, where the whole point was that even a one-screen game earns an honest way to manage screens. The pattern never changed across any of it. What changed is how much scaffolding each language made me carry — and watching that scaffolding fall away over a decade is the part I actually want to show you. So let me spell the C++ out completely. If you can read this, you can build it in any language.

**2016, C++, the engine from my book** — *[Game Development Patterns and Best Practices](https://www.amazon.com/Game-Development-Patterns-Best-Practices-ebook/dp/B01MRP7SPA)*. Start with the factory. It's a registry: a map from a stage-type enum to a *builder*, and a `Build` that looks the builder up and asks it to make a stage.

```cpp
class M5StageFactory {
public:
    void     AddBuilder(M5StageTypes type, M5StageBuilder* builder);
    M5Stage* Build(M5StageTypes type);   // find the builder for `type`, return builder->Build()
private:
    std::unordered_map<M5StageTypes, M5StageBuilder*> m_builderMap;
};
```

So what's a *builder*? It's an interface with exactly one job — make one stage. Here's the abstract base, and then the trick:

```cpp
// The abstract builder. Every builder knows how to Build() exactly one stage.
class M5StageBuilder {
public:
    virtual ~M5StageBuilder() {}
    virtual M5Stage* Build(void) = 0;   // pure virtual — the subclass does the actual new
};

// The naive move is a hand-written builder class per stage:
//   class MenuStageBuilder : public M5StageBuilder {
//       M5Stage* Build() { return new MenuStage(); }
//   };
// ...one of those for every stage, forever. I didn't do that. I wrote ONE
// templated builder that works for any stage type:
template <typename T>
class M5StageTBuilder : public M5StageBuilder {
public:
    M5Stage* Build(void) { return new T(); }   // <-- the new. the whole pattern, one line.
};
```

That `return new T();` is the entire point, in one line. It is the *only* place in the engine that says `new` for a stage. Everything downstream holds an `M5Stage*` and never knows the concrete type.

And yes — I've written the other version. A real `MenuStageBuilder`, a real `GamePlayStageBuilder`, a hand-written class apiece, in earlier engines, before I trusted a template to do it for me. A folder full of three-line builder classes that each exist to say one `new` is *exactly* the boilerplate this template erases. By the time I built Mach5 I knew better, so I reached for the template on purpose. That's the first piece of scaffolding coming down — and we've got a decade of it left to go.

And here's the jukebox actually getting loaded — the real registration from the engine (which, I'm a little proud to say, was *auto-generated* from the `*Stage.h` files in the project):

```cpp
// RegisterStages.cpp
void RegisterStages(void) {
    M5StageManager::AddStage(ST_GamePlayStage, new M5StageTBuilder<GamePlayStage>());
    M5StageManager::AddStage(ST_MenuStage,     new M5StageTBuilder<MenuStage>());
    M5StageManager::AddStage(ST_SplashStage,   new M5StageTBuilder<SplashStage>());
}
```

One line per stage. The engine now knows how to build a `GamePlayStage` without `GamePlayStage` appearing anywhere in its own code — the game handed it a builder, keyed by an enum. Punch `ST_MenuStage`, get an `M5Stage*` that's secretly a `MenuStage`. The factory has never heard of `MenuStage`. (The book generalizes this one more step — `M5Factory<EnumType, BuilderType, ReturnType>` with an `M5TBuilder<Return, T>` — the same machine parameterized to stamp out a factory for *any* type keyed by *any* enum, not just stages. Same `new T()` at the bottom, more angle brackets on top.)

That's the whole pattern, fully spelled out. Three small pieces — a factory, an abstract builder, one templated builder — and a line per type to load it. Now watch the pieces disappear.

**2020, Swift.** I ported it almost shape-for-shape. A `SceneBuilder` protocol, one generic `TSceneBuilder<T>` standing in for `M5StageTBuilder<T>`, register an instance per type:

```swift
public class TSceneBuilder<T: Scene>: SceneBuilder {
    public func build() -> Scene { return T.self.build() }
}
// load the jukebox:
factory.addScene(type: .menu, builder: TSceneBuilder<MenuScene>())
```

Four years and a new language later, the structure is *identical*: abstract builder, one templated builder, an instance per type. C++ in 2016, Swift in 2020, same scaffolding standing in the same place.

**2026, Swift, same engine.** Here's where I finally got to tear the scaffolding out — the part I'll happily bore you about at a party. Swift lets a base class construct its own *real subclass*:

```swift
open class DefaultScene: Scene {
    public required init() { ... }                      // required, so Self() is legal everywhere
    open class func build() -> Scene { return Self() }  // Self() = the actual subclass, not the base
}
```

`Self()` means the one base implementation is correct for every scene that will ever exist, so the whole builder layer deletes and the registry just stores the classes. Here's the *entire* scene setup to launch a game today:

```swift
let factory = SceneFactory()
factory.addScenes([
    MenuScene.self,
    GameplayScene.self,
    GameOverScene.self,
])
```

Now set that beside the C++ register from up top, line for line:

```text
2016, C++:    AddStage(ST_GamePlayStage, new M5StageTBuilder<GamePlayStage>());
2026, Swift:  GameplayScene.self
```

Same job — *tell the factory this type exists* — minus the builder object that used to wrap every single line. You hand the factory the class and walk away. (That's the real engine, not a blog toy; the [OCP post](/2026/04/17/house-rules-whatever-happens-happens/) shows the full eleven-scene version booting an actual game.)

Count what fell away. C++ 2016 and Swift 2020: a factory, an abstract builder, a templated builder, and a registered builder object per type. Swift 2026: a factory, and you hand it the class. The abstract builder — gone. The templated builder — gone. The per-type builder instance — gone. Same machine, half the parts.

Remember that quarter I dropped in the swear jar up top? Here's what it bought. My engine can run **any scene you invent, that I have never seen, that didn't exist when I shipped the engine** — and the cost to support a brand-new one is: write the class, add an enum case, register it. Zero builder classes. Zero overrides. Zero engine changes. The open catalog — the jukebox that ships empty and lets *you* load the records — costs almost nothing now, because Swift hands me self-typed construction for free. Ten years of shaving the same pattern down, and it finally hit the floor.

That's fucking badass, and I'm not going to pretend otherwise.

## Isn't this just dependency injection?

It's the first thing people ask, and the line is clean once you see it.

I have two tools for "confine the `new` to one spot." One is the [`AppContainer`](/2026/04/14/bad-advice-roll-your-own-di/) — manual dependency injection, the thing I spent the whole intro telling you I love. The other is this factory. Both keep concrete-type knowledge in one place. But they're not the same move:

- **`AppContainer` wires a *known* graph, *once*.** I know at compile time I need a logger, a network client, a session store — *different* types, *different* jobs. I build them in order at startup and hand out the interfaces. There's no choosing: you ask for the logger, you get the logger.
- **The factory makes *one of many interchangeable* implementations of *one* interface, and which one is decided at *runtime*, by a key.** Same shape, many concretes, picked when the key lands. The container hands you *the* thing. The factory picks *which* thing.

That's the whole difference: **a container *wires*, a factory *chooses*.** (My engine piles a second reason on top — it can't even *name* its scenes — but that's the open extreme, not the rule. A closed factory choosing among four types it knows perfectly well is still doing the thing a container never does: deciding at runtime.) Use DI for the fixed graph you know at startup. Use a factory when the choice shows up later, from outside, by a key.

## Where the jukebox is wrong

I love this pattern. I'm still not going to let you over-apply it, because a jukebox in the wrong corner is just furniture.

**Two or three types, known, never chosen at runtime.** Then call the constructor. No variable code, no jukebox — you want a record player and one album.

**No shared interface.** Then it isn't this pattern, full stop. A `switch` that returns unrelated concrete types is a `switch`, and that's fine; just don't dress it in a coat it isn't wearing. (This, by the way, is why my [logger mess from the Builder post](/2026/06/07/house-rules-built-not-shaken/) needed a Builder and *not* a factory — six classes that shared no shape. Different problem, different tool.)

**Heterogeneous things that each do their own job.** View models. Every screen's is a different shape; a factory there buys you indirection and nothing else. It's also why you've never reached for one on your screen flow and never missed it.

And now zoom all the way out, because this is the one that actually matters — and it's bigger than factories.

**You're already inside a walled-off module.** Factories are a great tool, but that doesn't mean you don't ever call concrete objects. Those are real implementation details, and those are abstracted behind a function or a class or a module.

I spent this whole post telling you to corner construction. But you corner it at the seams *between* modules, not at every line *within* one.

My graphics engine meets the rest of the system through a `Renderer` interface — mockable at that boundary, swappable in a test, the whole game blind to what's behind it. And *inside* that engine I construct concrete Metal types everywhere, on purpose, and I'd be a lunatic not to. A Metal renderer that hid all its Metal types behind interfaces is a Metal renderer drowning in spaghetti for nothing.

Same story with an Android network client: the login screen mocks the *client* at the boundary and never sees one Android networking class — but the client itself calls those concrete classes directly, because that is *literally its job.* **The module is the abstraction. The details inside it are not.** You don't abstract everything away into oblivion; that's not architecture, it's just a more expensive way to write the same program. Find the seam, and abstract *that.*

And the tax, even when it fits — this is the *other* half of why you don't abstract everything: every layer of indirection is a layer between you and a bug. Call a constructor and you know the exact type you're holding and the exact line it was born on — you can click it. Run it through a factory and you've traded that certainty away. Something throws, you're holding a `Scene`, and the questions start: *which* concrete is this, and who asked for it? Now you're tracing back up the stack to find where the key came from, which is usually nowhere near the code that actually broke. A registry to keep loaded, and a `new` you can't jump to. All real — and all worth it the instant the alternative is my DigiPen switch, not a breath sooner.

## Last call

My first engine knew every scene in the game, and it was the worst thing about it. My current engine doesn't know a single one — and that took me from 2008 to right now to make look easy. The game loads the jukebox at startup, the loop punches a `SceneType`, a `Scene` drops onto the platter, and the engine has never heard of `MenuScene` and never will.

That's the move. Construction has to happen somewhere — you can't dodge naming a concrete type forever. But you can corner it. One spot, behind one interface, and everything else just punches a code.

[SOLID](/2026/05/06/house-rules-d/) said *put an interface on it.* [Builder](/2026/06/07/house-rules-built-not-shaken/) said *when the seam moves to the call site, shape it there.* Factory is the one where I finally admit my beloved interfaces have a blind spot — they can't build themselves — and hand the job to the one pattern I've never once gotten wrong since I figured it out.

I love interfaces. I'd still marry one. But the jukebox? The jukebox I'd take home to meet my mother.

Drop a coin. Punch B7. You don't need to know what's on the record.

---

*I didn't write 'em, but those are the rules.*
