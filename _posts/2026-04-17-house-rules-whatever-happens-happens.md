---
layout: post
title: "House Rules: Whatever Happens, Happens"
series: house-rules
tags: [architecture, solid, opinion, gamedev, swift, metal]
---

*SOLID series: [S](/2026/04/15/house-rules-the-other-four-letters/) → **O** → [L](/2026/04/22/house-rules-it-compiled-it-lied/) → I → D*

To be honest, I almost skipped this one.

I sat down to write about the Open-Closed Principle and my first thought was *"I don't think I have an example of this."* When I need a new API call, I add it to the existing code. I've got this huge class that holds all my API calls — the interfaces are separate now, we'll get to that in a later post — but the code? I just add another method. When I need another API call, or another string in my array, same thing. Open the file, add it, move on.

That's not extension. That's just coding.

Open-Closed Principle. Maybe I should skip it...

Oh shit.

And then I started looking at my game engine.

## Could You BE Any Less Clear?

Maybe everyone but me and Mr. Bing understands what this means, but here's the official definition:

> **Software entities should be open for extension, but closed for modification.**

This is the most academic-sounding sentence in all of SOLID. It sounds like something someone would put on a conference slide and then spend forty minutes not quite explaining. My immediate reaction every time I read it:

*What the F does that even mean?*

If I need to change something, I change it. I open the file and I type new code into it. That's the job. What exactly am I supposed to do instead?

Here's what the F it *even* means:

When I need a new shader, I shouldn't need to hop into my renderer code and add support for it.

When I need a new scene in my game, I shouldn't need to hop into the scene manager and hardcode that scene.

The stuff that's already working keeps working, no tests break, because those classes didn't know about the specific old thing and they don't know about the specific new thing. They're not dependent on concrete classes.

And here's what it does NOT mean: "never change a file." That's the version that makes everybody's eyes glaze over, because it's impossible and kind of stupid. Of course you change files. If I need to update game logic for a specific scene, I'm gonna do that. If I need to fix something, I fix it. If I want to add gravity to my physics engine, of course I'm changing that — I don't need to *extend* my physics engine to allow for gravity.

## The Obviously (I Hope) Bad Examples

Nobody would actually do this. Right? To be fair, I've seen this code, so it literally does happen.

Remember the Govee story from [the singletons post](/2026/04/14/bad-advice-delete-your-singletons/)? That gigantic `SuperDevice` class, straight to a proper device with an interface? Good ending. *Hypothetically.* But what if they'd done something intermediate? What if, instead of building the interface, they'd just started adding switch statements?

```kotlin
// SceneManager.kt
fun applyScene(device: Device) {
    when (device.type) {
        DeviceType.LAMP -> { /* lamp scene logic */ }
        DeviceType.LAMP_WITH_SPEAKER -> { /* lamp + speaker logic */ }
        DeviceType.ROPE_LIGHT -> { /* rope light logic */ }
        DeviceType.WALL_PANEL -> { /* wall panel logic */ }
    }
}

// ColorManager.kt
fun setColor(device: Device, color: Color) {
    when (device.type) {
        DeviceType.LAMP -> { /* lamp color logic */ }
        DeviceType.LAMP_WITH_SPEAKER -> { /* lamp + speaker color logic */ }
        DeviceType.ROPE_LIGHT -> { /* rope light color logic */ }
        DeviceType.WALL_PANEL -> { /* wall panel color logic */ }
    }
}

// FirmwareManager.kt
fun getFirmwareVersion(device: Device): String {
    when (device.type) {
        DeviceType.LAMP -> { /* ... */ }
        DeviceType.LAMP_WITH_SPEAKER -> { /* ... */ }
        DeviceType.ROPE_LIGHT -> { /* ... */ }
        DeviceType.WALL_PANEL -> { /* ... */ }
    }
}
```

Same switch. Three files. Now imagine it in eight.

Now imagine it's 80.

Now add a fifth device.

Miss one file? Runtime crash. Forget one? Silent bug. Closed for extension, open for modification — the exact inverse of what you want.

This isn't an exaggeration. I've seen this happen. Maybe it wasn't 80 files, but it was close.

Or even dumber:

```cpp
int levels[4] = { /* ... */ };

for (int i = 0; i < 4; i++) {
    loadLevel(levels[i]);
}
```

That `4` is hardcoded. It's in your level loader. It's in your save system. It's in your progress tracker. It's in eight different for loops across the codebase. Now add a fifth level. You change the array, and then you hunt through every file that hardcoded `4` and change it to `5`. Miss one? Off-by-one bug. Miss two? Crash.

We all know why we don't want magic numbers. This is why. That `4` means your code has to be *modified* — you have to go change that number — instead of *extended* — you can't just add a new thing to the array and have it work. Use `levels.size()` and suddenly you can. Add a fifth level, the loop picks it up. No files changed. That's OCP at its most obvious — and people still do it.

## The Renderer Doesn't Care

The 2D game engine I've been working on — [LiquidMetal2D](https://github.com/mattcasanova/LiquidMetal2D), Swift + Metal — deals with this exact problem. I'm going to talk about renderers and shaders here, but it could literally be anything. Quick explainer if you're not a graphics person:

In modern renderers, you can have multiple shaders. Shaders live on the GPU — your Swift or C++ or Kotlin code can't touch them directly. They expect a stream of bytes. Your code converts its data into that stream and sends it over. That stream is called a "buffer" — buffer literally just means array in this case. Nothing fancy.

Each element in that array — the chunk of data for one object — is called a "uniform." Different shaders need different uniforms. My alpha-blend shader needs a position, texture coordinates, and a color per object — 96 bytes. But when I add debug wireframes for collision-bound visualization, that's a completely different shader. No textures. No color — the shader itself can just hardcode everything as red. The uniform is just a position. 64 bytes. Different shader, different shape.

Shaders expect an array of uniforms. The renderer fills the array. That's it.

Now here's the problem. My engine currently runs a single alpha-blended shader. It expects a specific set of data per object, packed into a buffer and passed to the shader every frame.

I could have hardcoded that. Originally I did — one struct, one packing function, baked directly into the renderer. But then I realized: what if I want a different shader? Back in 2020 I wrote it the naive way — I knew I'd eventually want more shaders, but I didn't want to deal with that problem yet.

Eventually I did deal with it. I ripped out the hardcoded packing and wrote a protocol:

```swift
public protocol UniformData {
    var size: Int { get }
    func setBuffer(buffer: UnsafeMutableRawPointer, offsetIndex: Int)
    static func typeSize() -> Int
}
```

Three things. The uniform knows its size. It knows how to write itself into a raw GPU buffer. And it can report its type size for buffer allocation. That's the entire contract.

Here's the implementation I have today — `AlphaBlendUniform`, named to match the shader it belongs to:

```swift
public class AlphaBlendUniform: UniformData {
    public var transform: Mat4 = Mat4()
    public var texTrans: Vec4 = Vec4(1, 1, 0, 0)
    public var color: Vec4 = Vec4(1, 1, 1, 1)
    public var size: Int = AlphaBlendUniform.typeSize()

    public func setBuffer(buffer: UnsafeMutableRawPointer, offsetIndex: Int) {
        let mtxSize = MemoryLayout<Mat4>.size
        let vecSize = MemoryLayout<Vec4>.size
        let offset = offsetIndex * size
        memcpy(buffer + offset, &transform, mtxSize)
        memcpy(buffer + offset + mtxSize, &texTrans, vecSize)
        memcpy(buffer + offset + mtxSize + vecSize, &color, vecSize)
    }

    public static func typeSize() -> Int {
        return MemoryLayout<Mat4>.size + MemoryLayout<Vec4>.size * 2
    }
}
```

It knows exactly what the shader expects and exactly how to pack it. That's its one job.

Now here's the fun part. While writing this post, I looked at my renderer and found a problem. The renderer had a concrete uniform stored directly on it as a property — created once, reused for every object. This was legacy. Back in 2020, I had built in the idea that the uniform could be passed as a protocol — the `UniformData` protocol existed, the manual `draw(uniforms: UniformData)` path existed. But since I never had a second shader, I never actually made the renderer flexible enough to use it. I went halfway but not the full way. It technically worked — but only because there was only ever one shader. I knew what I was doing. I just never tested it.

So I fixed it. `GameObj.toUniform()` now returns a `UniformData` instead of taking a concrete `AlphaBlendUniform`:

```swift
// GameObj — each object produces its own uniform
open func toUniform() -> UniformData {
    let uniform = AlphaBlendUniform()
    uniform.transform.setToTransform2D(
        scale: scale, angle: rotation,
        translate: Vec3(position, zOrder))
    uniform.color = tintColor
    return uniform
}
```

And the renderer just uses whatever comes back:

```swift
open func submit(objects: [GameObj]) {
    // ... sorting, filtering, not important for this lesson ...

    for obj in sorted {
        let uniform = obj.toUniform()   // game object produces a uniform
        uniform.setBuffer(              // uniform packs itself into the GPU buffer
            buffer: worldBufferContents,
            offsetIndex: drawCount)
        // ... batching logic ...
        drawCount += 1
    }
}
```

Look at what's happening. The renderer takes `[GameObj]` — the base class. It has no idea what subclass it's getting. Maybe it's a ship. Maybe it's a collision object. Maybe it's something I haven't thought of yet. `GameObj` is intentionally bare — I have no idea what your game object is going to look like, so the base class only has the basics: position, scale, rotation, texture.

Each object produces its own uniform via `toUniform()`. The default implementation returns an `AlphaBlendUniform`, but a subclass can override it and return something completely different for a completely different shader. The game object knows what data it has and how to pack it. The renderer doesn't know and doesn't care. It just calls `setBuffer()` and moves on. Whatever happens, happens.

OCP on both sides — the renderer isn't coupled to a specific game object type, and it isn't coupled to a specific uniform type.

I don't have a second shader yet. But when I do, I just:

1. Load the new shader
2. Subclass `GameObj` and override `toUniform()` to return the right uniform type
3. Submit the objects

The renderer doesn't change. The existing `AlphaBlendUniform` doesn't change. The alpha-blend shader doesn't change. I write new code and plug it in.

## Whatever You Register, I'll Run

Same engine, different scale. Quick context if you've never built a game:

Game engines run in a loop. Every frame — 60 times a second — the engine does two things: **update** and **draw**. Update is where all the game logic lives: read input from the user, move characters, update behavior, check collisions, figure out who killed who. Draw is where you take the result of all that and render it to the screen. Update, draw. Update, draw. That's the game loop. That's every game engine ever.

Now, a game isn't one continuous thing. You've got a main menu. You've got the actual gameplay. You've got a pause screen. Maybe a game-over screen. Each one of these has completely different behavior — the menu listens for button taps, the gameplay handles physics and collision, the pause screen just sits there. These are what I call **scenes**.

A scene is not a level. Mario 1-1 and Mario 1-2 are the same scene — same gameplay logic, same physics, just different data loaded from a file. But the main menu and the gameplay? Those are different scenes. Different update logic. Different draw logic. Different everything.

The engine's job is to run the game loop and handle scene transitions. The programmer's job is to fill in what each scene actually does. The engine calls `update()` and `draw()` — you write what happens inside them.

Now here's the problem. My first game back at DigiPen literally had a giant switch statement for this. If you're in the menu, draw the menu. If you're in the gameplay, run the gameplay. If you're paused, draw the pause screen. A big switch on the current scene type. That works when you're making one specific game and you know all the scenes upfront.

A game engine can't do that. I don't know how many scenes your game is going to have. I don't know what they're called. I don't know if you have three or thirty. So I can't hardcode a switch. I need a system where you can register any scene you want, and the engine just runs it.

Here's what the game loop actually looks like. This is the engine's core — the thing that runs 60 times a second:

```swift
// DefaultEngine.gameLoop() — the entire game loop
sceneManager.currentScene.update(dt: dt)
sceneManager.currentScene.draw()
```

Two lines. Notice it says `currentScene.update` — not `collisionStressDemo.update`. The engine doesn't know what scene it's running. It doesn't care. It just calls `update()` and `draw()` on whatever the current scene happens to be.

Here's the protocol that makes that work:

```swift
@MainActor
public protocol Scene {
    static var sceneType: any SceneType { get }
    func initialize(sceneMgr: SceneManager, renderer: Renderer, input: InputReader)
    func update(dt: Float)
    func draw()
    func shutdown()
    static func build() -> Scene
}
```

And here's the factory — a registry that maps scene types to scene classes:

```swift
@MainActor
public class SceneFactory {
    private var sceneMap = [AnyHashable: Scene.Type]()

    public func addScene(_ sceneClass: Scene.Type) {
        sceneMap[AnyHashable(sceneClass.sceneType)] = sceneClass
    }

    public func addScenes(_ sceneClasses: [Scene.Type]) {
        for sceneClass in sceneClasses { addScene(sceneClass) }
    }

    func build(_ type: any SceneType) -> Scene {
        guard let sceneClass = sceneMap[AnyHashable(type)] else {
            fatalError("No scene registered for type: \(type)")
        }
        return sceneClass.build()
    }
}
```

No switch statements. No `if type == .menu, build MenuScene`. Just a dictionary. You register scene classes. The factory builds them. The factory doesn't know what any of them do.

Now look at the demo app. This is the *entire* setup to launch a game on LiquidMetal2D:

```swift
class ViewController: LiquidViewController {
    override func viewDidLoad() {
        super.viewDidLoad()

        let sceneFactory = SceneFactory()
        sceneFactory.addScenes([
            MassRenderDemo.self,
            TouchZoomDemo.self,
            InstanceDemo.self,
            SchedulerDemo.self,
            SpawnDemo.self,
            CollisionDemo.self,
            CollisionStressDemo.self,
            BezierDemo.self,
            CameraRotationDemo.self,
            AsyncLoadDemo.self,
            PauseDemo.self,
        ])

        let renderer = DefaultRenderer(
            parentView: self.view,
            maxObjects: GameConstants.MAX_OBJECTS,
            uniformSize: AlphaBlendUniform.typeSize())

        gameEngine = DefaultEngine(
            renderer: renderer,
            initialSceneType: SceneTypes.asyncLoadDemo,
            sceneFactory: sceneFactory)

        gameEngine.run()
    }
}
```

Eleven scenes. The engine doesn't know what a `CollisionStressDemo` is. The factory doesn't know. The renderer doesn't know. You want a collision stress test? Whatever happens, happens. You want a bezier curve demo? Whatever happens, happens. The engine just runs whatever you register.

Want a twelfth scene? Write the class, add a case to your `SceneTypes` enum, register it with the factory. Three steps, all new code. The engine doesn't change. The factory doesn't change. The scene manager doesn't change. No switch statements. No modifying existing files.

And the scene types enum means you never pass strings around — no typos, no runtime bugs from a mistyped scene name. When you want to transition, you call `sceneMgr.setScene(type: .collisionDemo)`, not `sceneMgr.setScene(name: "collision_demo")`. The compiler catches it.

I want to be clear about something: these aren't contrived examples for a blog post. [LiquidMetal2D](https://github.com/mattcasanova/LiquidMetal2D) is an open-source iOS game engine you can download right now and build a game with. It's not Unity or Unreal — it's old school, 100% code, no visual editor. It doesn't have networking yet, doesn't have particles yet, not 100% complete. But it's real. The uniform system and the scene system are real architecture in a real engine. And I literally just fixed the `toUniform()` coupling while writing this post because I was pretty sure it worked the way I intended — and then I looked and it was slightly off. That's the thing about OCP: you think you're following it until you actually check.

<iframe width="560" height="315" src="https://www.youtube.com/embed/7IZofpAHRzU" title="LiquidMetal2D Scene Demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

## We're All Building Engines

We're all building engines. We just don't call them that.

Now — someone might argue that a little. And that's fair. You're building an iOS or Android app, not a game engine. The platform itself is the engine. But somewhere in your app, there's a foundational core. You're selling products — you might need more products. You allow coupons at checkout — you might need to add or remove coupon types. You have a list of devices — you're shipping four new ones next quarter. That foundational stuff doesn't have to be everywhere, and you don't have to build an app engine on top of the platform. But somewhere in your logic, there's something that will very likely change — a new product, a new business rule, a new device — and you should write that code so it can handle those changes without surgery.

Nobody ships software and says *"that's it, I'm done forever, no more features."* The code is going to change. It always does. So the question isn't *"will I need to extend this?"* — it's *"when I do, will it cost me one new file or surgery on ten existing ones?"*

That's all OCP is. The renderer calls `setBuffer()` and doesn't ask what's inside. The factory calls `build()` and doesn't ask what the scene does. The next thing plugs in and the last thing keeps working.

## Whatever Happens, Happens

The S was the foundation — one reason to change. The O is the natural consequence — when the next change comes, does it cost you one new file or surgery on ten?

I've said it before, but we have to write code assuming that our app will change. Because it will. The code you've already written needs to be able to handle it. Not crash. Not break. Just do the correct thing.

Whatever happens, happens.

---

*This is the second entry in the **House Rules** series. Next up: the L. Liskov Substitution Principle, and why the scariest-sounding letter in SOLID is actually the simplest one once you stop being intimidated by the name.*

*So what have we learned so far? S: interfaces. O: interfaces. Want to guess what's coming next? Spoiler: interfaces.*

*I didn't write 'em, but those are the rules.*
