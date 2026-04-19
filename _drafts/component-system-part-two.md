---
title: "Component System, Part Two: Multiple Shaders for the Same Object"
draft: true
---

# Pre-seed notes (not prose — outline for Matt)

This is the fast follow-up to Part One. Part One introduced the component system.
Part Two shows it earning its keep: letting one GameObj render through multiple
shaders at once (e.g. alpha-blend sprite + debug wireframe overlay).

## The story arc

1. **Prior post (OCP):** we refactored LiquidMetal2D toward Open-Closed — renderer no
   longer owned the concrete render pass class, etc. Felt good, shipped it.
2. **But:** while sketching debug wireframe rendering, found the renderer was
   still too dependent on AlphaBlend. Uniform buffer stride hardcoded, `RenderPass`
   subclassed for AlphaBlend specifically, `GameObj.toUniform()` hardcoded to
   AlphaBlendUniform. OCP in spirit but not in fact.
3. **The real fix:** decouple shaders from the renderer, move per-object render
   state into components. Now one GameObj can carry an `AlphaBlendComponent` and a
   `WireframeComponent` and render through both shaders in one pass.
4. **Why this matters:** without components, we'd either need subclassing
   (`TexturedGameObj` / `WireframeGameObj`) which pushes users into a rigid
   hierarchy, or generics which sidestep Swift's array invariance but hide intent.
   Components sidestep both traps — cleaner, matches the existing architecture.

## Key talking points

### What was baked in before
- `DefaultRenderer` held one `MTLRenderPipelineState`, one `BufferProvider` sized
  to `AlphaBlendUniform.typeSize() * maxObjects`. One stride, one shader, ever.
- `AlphaBlendRenderPass` subclassed `RenderPass` and hardcoded the uniform stride
  math (`startIndex * AlphaBlendUniform.typeSize()`).
- `GameObj.toUniform()` returned `AlphaBlendUniform` — the base class presumed a
  single canonical uniform per object. Add a second shader and you'd pile up
  `toAlphaBlendUniform()`, `toWireframeUniform()` methods on the base class forever.

### The shader abstraction
- New `Shader` protocol. Each shader owns its pipeline state, its `BufferProvider`
  (sized to its own uniform stride), its batching strategy.
- `RenderPass` becomes a thin wrapper around the command encoder + drawable. No
  shader knowledge.
- Pass flow: `beginPass → useShader(A) → submit(...) → useShader(B) → submit(...) → endPass`.
  Flush-on-switch, no juggling multiple bound pipelines.

### The component angle (the Part Two thesis)
- Render state is **per-object-per-shader** data. That's exactly what components model.
- `AlphaBlendComponent` holds `textureID`, `tintColor`, `texTrans`. Knows how to
  build its uniform from (parent transform + own fields).
- `WireframeComponent` holds color/thickness. Knows how to build its uniform.
- Shader's submit: `for obj in objects { guard let comp = obj.get(MyComponent.self) else { continue } ... }`.
  O(N) walk, O(1) component lookup.
- One GameObj + two components = draws in both shaders. Debug toggle becomes
  `obj.add(WireframeComponent(...))` at spawn, no parallel debug-object tracking.

### Why not just put `toUniform()` on the base class?
- Forces one canonical uniform per object → one shader per object.
- Can't do additive effects on the same object (sprite + distortion overlay, etc.) —
  that's now a one-liner with components.
- Adding shaders grows the base class. Components keep render state co-located
  with the shader that uses it.

### Why not subclass (`TexturedGameObj`)?
- Subclassing forces one axis of specialization — pick shader or pick behavior,
  not both.
- Components already solve "pick a shader AND a behavior AND a collider" trivially.
- Making `GameObj` `final` commits to the position: compose, don't inherit.

### Why not generics?
- `func submit<T: GameObj>(objects: [T])` works around Swift's array invariance.
- But generics there are a workaround for variance, not a design intent — the
  shader doesn't actually use `T`'s concrete type.
- Components let `submit` take `[GameObj]` directly. No generics.

## Tying it back

- Part One (component system): "here's this compositional data model we added."
- Part Two (this one): "and here's the refactor where it actually pays off — a
  problem we literally could not solve cleanly without it."

## Concrete before/after snippet (rough — adjust voice)

Before:
```swift
let ship = GameObj()
ship.textureID = shipTex
ship.tintColor = Vec4(1, 0, 0, 1)
// render state baked into the base class
```

After:
```swift
let ship = GameObj()
ship.add(AlphaBlendComponent(parent: ship, textureID: shipTex,
                             tintColor: Vec4(1, 0, 0, 1)))
if debug {
    ship.add(WireframeComponent(parent: ship, color: .green))
}
// same ship, two shaders, zero duplication
```

## Things to maybe cut or expand

- The OCP callback is worth linking to the prior post but don't let it dominate.
- The generics tangent might be more detail than the post needs — could cut.
- Code snippet is rough — swap in the real API once the refactor lands.
- Maybe a screenshot/gif of the wireframe overlay once the `WireframeShader`
  lands (out of scope for the refactor PR, follow-up).
