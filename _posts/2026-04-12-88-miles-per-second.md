---
layout: post
title: "88 Miles Per Second"
date: 2026-04-12
tags: [flux, rust, terminal, gpu, performance, game-engine]
---

Flux renders. In about two days, I went from `cargo new` to a rough but working GPU terminal — a window, a shell, colored ANSI output, a cursor, a config file. The whole stack: wgpu for the GPU, cosmic-text for font rasterization, etagere for atlas packing, alacritty_terminal for the state machine, portable-pty for the shell process. One shader. One draw call for the entire grid using instanced quads — the same pattern [I wrote about](/2026/04/10/building-a-gpu-terminal-in-rust/) when I decided to build this in the first place.

Then I dragged the window edge.

It jittered. It felt sluggish. It looked nothing like Warp or iTerm2 or Ghostty. Something was wrong, and I had no idea what.

## The Wrong Rabbit Holes

My first instinct was that something in the renderer was slow. I described what I was seeing to Claude — the content looked stretchy during the drag — and it suggested this was the compositor. On macOS, when you resize a window, the OS stretches the last-rendered frame to fill the new bounds before your app can render a new one. If you're slow to present, you see that stretch as a visible artifact. Every GPU terminal has to deal with it. That explanation matched what I was seeing, so we went down that road.

I spawned a research agent to dig through Alacritty, Rio, and Ghostty's source code. It learned that:

- **Alacritty** reconfigures the surface in the main thread right before rendering, not in the resize event handler. Uses `set_resize_increments()` to snap the window to cell boundaries.
- **Rio** does immediate `surface.configure()` plus an immediate render on every resize. No debouncing.
- **Ghostty** sets `needsDisplayOnBoundsChange` on the Metal layer. Still has some reported resize issues on macOS.
- **Warp** uses Metal directly via `metal-rs`, which gives them tighter control over `CAMetalLayer` than wgpu exposes.

That last one was the scary one. If Warp needed raw Metal to get smooth resize, maybe wgpu was a dead end for me. Maybe the architectural choice was wrong and I'd need to rewrite the whole renderer against `metal-rs`. That's the kind of discovery that makes you want to close the laptop, pour the good whiskey, and pretend the last two days never happened.

Before I started that walk, I tried some smaller fixes:

1. **Debounce the resize.** Store the new dimensions when the event fires, apply them when the user stops dragging. Helped visually, but now the window showed stretched content during the drag and snapped at the end. Felt unresponsive.
2. **Snap to cell boundaries.** Alacritty's trick. Functional but weird — dragging the window in 12-pixel steps felt unnatural.
3. **Render immediately in the resize handler.** Instead of requesting a redraw and waiting for the next `RedrawRequested` event, call `renderer.render()` directly inside `WindowEvent::Resized`. Correct approach from wgpu's perspective, but didn't fix anything.

None of these were it. None of the framework-level fixes did anything, because none of them were the actual problem.

## Reading the Code

I should say upfront: I still haven't actually profiled this. I didn't open a profiler. I didn't measure function timings. None of the numbers in this post came from real measurement — they came from reading the code and doing math in my head. (That's on the to-do list. Eventually I'll measure and find out whether my instincts match reality.)

Here's the part I have to be honest about. Over those first two days, I had been tweaking and refactoring throughout the codebase — the window code, the renderer pipeline, the instance buffer layout, the shader. I touched basically everything. The one place I *hadn't* really looked was the glyph atlas. Claude had imported cosmic-text and etagere early on and wired them up into a working atlas, and it worked — so I moved on. I didn't know those libraries. I assumed Claude was using them correctly. No reason to second-guess code that was producing the right pixels on screen.

When resize started feeling bad and the rabbit holes didn't pan out, I ran out of places to look. So I finally opened the atlas code.

The top of the render path looked like this, called once per non-empty cell, every frame:

```rust
let shaped = self.atlas.shape_text(&character.to_string());
```

"What is `shape_text`?" I asked myself. I jumped to the definition. And that's where the story really starts.

## What `shape_text()` Actually Did

Here's the important part: **`shape_text()` wasn't a cosmic-text function.** It was a helper *I had written* — well, Claude had written, at my direction — that wrapped cosmic-text's lower-level API into something that felt convenient. The guts of it looked roughly like this:

```rust
pub fn shape_text(&mut self, text: &str) -> Vec<ShapedGlyph> {
    let metrics = Metrics::new(self.font_size, self.line_height);
    let mut buffer = Buffer::new(&mut self.font_system, metrics);   // ALLOC every call
    let mut attrs = Attrs::new().family(Family::Name(&self.font_family));
    if self.bold {
        attrs = attrs.weight(cosmic_text::fontdb::Weight::BOLD);
    }
    buffer.set_text(
        &mut self.font_system,
        text,
        &attrs,
        cosmic_text::Shaping::Advanced,                             // Full HarfBuzz shaping
        None,
    );
    buffer.shape_until_scroll(&mut self.font_system, false);        // Layout + line breaks

    let mut glyphs = Vec::new();                                    // ALLOC result vec
    for run in buffer.layout_runs() {
        for glyph in run.glyphs.iter() {
            let physical = glyph.physical((0.0, 0.0), 1.0);
            glyphs.push(ShapedGlyph {
                cache_key: physical.cache_key,
                x: glyph.x,
                y: run.line_y,
                w: glyph.w,
            });
        }
    }
    glyphs
}
```

It's a perfectly reasonable wrapper when you need to shape a paragraph of text once and get back positioned glyphs. But look at what it's actually doing under the hood:

1. **Allocate a `Buffer`** — cosmic-text's layout primitive. A heap allocation on every call.
2. **Resolve font attributes** — look up the font family, weight, and style against the system font database.
3. **Run the shaping engine** — this is [rustybuzz](https://github.com/RazrFalcon/rustybuzz), the pure-Rust port of [HarfBuzz](https://harfbuzz.github.io/). The same shaping engine that Chrome, Firefox, LibreOffice, and basically every modern text renderer uses. It handles ligatures (`fi` → `ﬁ`), kerning pairs (`AV`), combining characters, right-to-left scripts, complex emoji sequences like 👨‍👩‍👧‍👦. It's powerful. It's thorough. It's not cheap.
4. **Lay out the text** — compute line breaks (UAX #14 rules), word wraps, baseline positions, layout runs.
5. **Iterate the layout runs** — allocate a result `Vec` and walk the shaped output to extract positioned glyphs.

This is the absolutely correct tool when you're rendering a paragraph of rich text. It is absolutely the wrong tool for asking "what's the cache key for the letter 'a'?"

The wrapper made it look innocent. It was just `shape_text("a")` — a helper in my own code, right next to the other glyph functions. Nothing about the name or the signature hinted that calling it 2,000 times a frame would light the CPU on fire. It was only when I jumped into the implementation and saw `Buffer::new()`, `set_text()`, `shape_until_scroll()` happening on every call that it clicked.

That wrapper doesn't exist anymore. I deleted it entirely. The cosmic-text calls it used to make now live inside a private `rasterize()` function that only runs when a glyph is seen for the very first time. The shaping still happens — it has to, cosmic-text is still the only way to get from a character to pixel data — it just happens once per unique character, not once per cell per frame.

## The Numbers

My terminal at the time was roughly 100 columns × 50 rows. Call it 2,000 non-empty cells for a typical command output. macOS fires resize events at roughly display refresh rate while dragging — around 60 events per second.

Math:

**2,000 cells × 60 events/sec = 120,000 `shape_text()` calls per second**

Each call allocating a `Buffer`, running HarfBuzz, doing layout, destroying everything. For *single-character* input strings. For characters like `a`, `e`, `s`, `/` that I'd already rendered hundreds of times in the same session.

That's why resize felt laggy. Not wgpu. Not Metal vs Vulkan. I was doing 120,000 paragraph layouts per second to render a grid of single characters.

## The `loadTexture()` Moment

If you've ever worked on a 2D game engine, you already know what I'm about to say.

This is the exact shape of calling `loadTexture()` inside your render loop. Every 2D game programmer learns not to do this on day one:

```cpp
// Bad
void drawSprite(const char* filename, float x, float y) {
    Texture tex = loadTexture(filename);  // Opens file, decodes PNG, uploads to GPU
    renderQuad(tex, x, y);
}
```

You load textures once, at scene init. You cache the handles. Your render loop looks up handles, not files. Anything else is a crime.

I've known this for 20 years. I [wrote about it](/2026/04/10/building-a-gpu-terminal-in-rust/) in the first Flux post — I told the story of catching Claude building a new vertex buffer for every cell and saying "wait, why don't I just create the quad once?" Instanced rendering, single draw call. Classic game engine pattern applied to a terminal.

I caught that one. I missed this one.

The vertex buffer insight was obvious because we were talking about geometry, which is the thing my brain immediately thinks about when it hears "shader code." But the glyphs? I had just not thought about them as textures yet. In my head, text rendering was some separate mystical thing handled by cosmic-text. I didn't connect it to the sprite atlas pattern I've been using for two decades — until the profiler showed me `shape_text()` in a hot loop, and I mentally substituted it for `loadTexture()`, and the whole thing collapsed into a pattern I already knew by heart.

**A terminal is a sprite renderer.** Each cell is a sprite. Each glyph is a texture sub-rect in the atlas. Each frame is one instanced batch draw. If you've ever built a 2D game engine, you already know how to render a terminal — you just have to recognize the pattern.

I just hadn't recognized it yet for the glyph side of the pipeline. Claude certainly hadn't either — it was just using the function that worked. Neither one of us had stepped back and said "is this sprite batching?"

The moment I did, the whole architecture snapped into focus. Curse your sudden but inevitable betrayal, `shape_text`.

## The First Fix

The first fix took about 10 lines:

```rust
pub struct GlyphAtlas {
    // ... existing fields ...
    char_cache: HashMap<char, CacheKey>,
}

pub fn lookup_char(&mut self, queue: &wgpu::Queue, ch: char) -> Option<GlyphRegion> {
    if let Some(&cache_key) = self.char_cache.get(&ch) {
        return self.lookup(queue, cache_key);
    }

    // First time seeing this character — shape it once, cache the key
    let shaped = self.shape_text(&ch.to_string());
    if let Some(glyph) = shaped.first() {
        self.char_cache.insert(ch, glyph.cache_key);
        self.lookup(queue, glyph.cache_key)
    } else {
        None
    }
}
```

Each character gets shaped exactly once — the first time it appears. After that, it's a HashMap lookup. A typical terminal session uses maybe 80-100 unique characters (ASCII printable plus a handful of symbols). So the first frame rasterizes up to ~100 glyphs, and every subsequent frame — resize frames included — is just HashMap lookups.

Resize went from sluggish to buttery smooth. Typing went from "feels fine" to indistinguishable from iTerm2.

I could have stopped there. It was fast. It worked. The bug was fixed.

## Going Deeper: Actually Building It Like a Game Engine

But "fast enough" is a trap in a render loop. A 60fps frame is 16.67ms. Every millisecond you save in one place is a millisecond available for something else — the input editor at the bottom, command blocks, syntax highlighting, a file tree sidebar, a command palette, search highlights, maybe even AI features someday. The atlas isn't the whole terminal — it's one part of a budget that has to stretch across everything else.

And once I had the `loadTexture()` parallel in my head, I started looking at the fix and going: *this isn't actually how a 2D game engine would structure this.*

Two things bothered me.

### Problem 1: Two caches stacked is usually one cache

After the HashMap fix, the atlas had two caches on top of each other:

```rust
cache: HashMap<GlyphKey, GlyphRegion>,   // CacheKey → GlyphRegion (existing)
char_cache: HashMap<char, CacheKey>,     // char → CacheKey (new)
```

To go from a `char` in the grid to a `GlyphRegion` (the atlas UV rect), I had to do **two HashMap lookups**:

```
char → char_cache → CacheKey → cache → GlyphRegion
```

Two hash computations. Two bucket accesses. Two potential collision walks. In 2D game engine terms, that's like having a sprite atlas where to go from "sprite ID 42" to "UV coordinates" you had to:

1. Look up sprite 42 in `sprite_to_internal_handle` → get handle
2. Look up handle in `handle_to_uv` → get UV rect

No 2D game engine does that. They store `sprite_id → uv_rect` directly. The `CacheKey` middleman only existed because cosmic-text's rasterization API uses `CacheKey` internally — I was leaking an implementation detail into my hot path.

### Problem 2: HashMap is the wrong data structure for dense integer keys

99% of characters in a terminal are ASCII printable — codes 32 through 126. That's a dense, known-size keyspace. For dense integer keys, a **flat array** crushes a HashMap in every way:

- **HashMap:** hash the key, index into bucket table, walk the bucket for the matching entry, return the value. Cache-unfriendly — hash maps scatter data across memory.
- **Array:** take the index, add it to the base pointer, return the value. One memory read. The whole ASCII array fits comfortably in L1 cache.

HashMaps are for sparse, unknown-size keyspaces. Emoji, CJK, box-drawing characters — sure, use a HashMap for those. But `a` through `z`? That's an array.

### The Refactor

I merged the two caches and replaced the ASCII HashMap with a flat array:

```rust
pub struct GlyphAtlas {
    ascii_regions: [GlyphRegion; ASCII_COUNT],  // ASCII fast path
    unicode_regions: HashMap<char, GlyphRegion>, // Non-ASCII fallback
    // ... other fields ...
}
```

Lookup becomes trivial:

```rust
pub fn lookup_char(&mut self, queue: &wgpu::Queue, ch: char) -> GlyphRegion {
    let code = ch as u32;

    // Fast path: direct array access
    if (code as usize) < ASCII_COUNT {
        return self.ascii_regions[code as usize];
    }

    // Slow path: Unicode HashMap
    if let Some(&region) = self.unicode_regions.get(&ch) {
        return region;
    }
    let region = self.rasterize(queue, ch).unwrap_or(NULL_REGION);
    self.unicode_regions.insert(ch, region);
    region
}
```

One branch, one memory read for ASCII. The CacheKey middleman is gone — it's now a local variable inside the private `rasterize()` method, only existing long enough to feed cosmic-text's rasterizer before being discarded.

### Pre-load, don't lazy-load

Then I went one more step in the 2D game engine direction: **pre-rasterize all printable ASCII at startup.**

Before the refactor, the first time you typed a character, I'd rasterize it on the fly. Decode the font, shape it, generate the bitmap, upload to GPU, cache the result. That's a noticeable micro-stutter the first time you type each character.

2D game engines don't lazy-load textures during gameplay — the full set of textures for a level fits in memory and gets loaded at scene init, when the player expects a loading screen. (3D is a different story — big open worlds stream assets as you move. But 2D is small enough to preload.) I can do the same thing for the terminal — the user expects startup latency, not "first keystroke of `a` feels weird" latency.

```rust
fn preload_ascii(&mut self, queue: &wgpu::Queue) {
    for code in 32..127u32 {
        if let Some(ch) = char::from_u32(code) {
            if let Some(region) = self.rasterize(queue, ch) {
                self.ascii_regions[code as usize] = region;
            }
        }
    }
}
```

By the time the first frame renders, every printable ASCII glyph is already packed in the atlas. No first-keystroke latency. No cold-cache stuttering.

### The null object pattern

One last thing. The natural type for the array was `[Option<GlyphRegion>; ASCII_COUNT]` — some slots filled (printable ASCII), some empty (control characters). But `Option` in a hot path means a branch on every lookup:

```rust
if let Some(region) = self.ascii_regions[idx] {
    return Some(region);
}
```

Thousands of branches per frame where there doesn't need to be any. The **null object pattern** fixes this: instead of `Option<T>`, store a real `T` that represents "nothing to render."

```rust
pub(crate) const NULL_REGION: GlyphRegion = GlyphRegion {
    uv: [0.0, 0.0, 0.0, 0.0],
    placement_left: 0.0,
    placement_top: 0.0,
    pixel_width: 0.0,
    pixel_height: 0.0,
};
```

Initialize the array to all nulls. Control characters naturally resolve to `NULL_REGION` because I never overwrite those slots during preload. Missing font glyphs resolve to `NULL_REGION` because `rasterize()` returns `None` and I leave the default in place. Every slot is always "valid." The lookup has zero special cases.

The null check happens once at the top of the renderer:

```rust
let region = self.atlas.lookup_char(&self.gpu.queue, character);
if region.is_null() {
    return;
}
```

Same semantic behavior as before, but the null check is where it belongs: at the point of use, not baked into every lookup.

Cost of the pattern: 32 "wasted" slots in the array for control characters I'll never use. 32 × 40 bytes = 1,280 bytes. Completely irrelevant compared to the ~1MB atlas texture. You could save those bytes by shifting the index (`ascii_regions[(code - 32) as usize]`), but now you're adding a subtraction to the hot path — an arithmetic instruction per lookup. Honestly, the arithmetic isn't catastrophic either. It's one subtraction. Modern CPUs eat those for breakfast.

Is this over-optimization? Maybe. But it was also trivial to get rid of. You write it once, never think about it again, and you get the cleaner lookup as a bonus. That's the kind of call worth making.

**Memory is cheap. Hot loops are where you care about every instruction — even the ones you don't have to.**

## The Numbers, Updated

Three steps of optimization, each roughly 10x faster than the previous:

| Version | Per-cell cost | Resize feels like |
|---------|---------------|-------------------|
| Original | `shape_text()` — buffer allocation, HarfBuzz, layout | Unusable jitter |
| HashMap fix | Two HashMap lookups | Buttery smooth |
| Final refactor | One array access (ASCII) | Way under budget |

The HashMap fix was the dramatic improvement — "unusable" to "smooth" is the step that actually matters to users. The refactor from two HashMaps to a flat array is maybe 3ms per frame. That sounds small until you remember the 16.67ms frame budget: 3ms is 18% of a frame, freed up for something else. And something else is coming. A lot of something elses.

## What AI Actually Missed

Here's the part I've been thinking about most.

The initial code — the broken version — was correct. It passed tests. It worked for small inputs. If I'd been writing a text label widget or a README viewer, `shape_text()` per call would have been fine. Claude wasn't wrong in any absolute sense. It was wrong in context, and the context was "this is going to be called 120,000 times a second."

When I set Claude off to build the text rendering layer, it reached for the obvious function — the one that already worked on test strings. It's what I would have done too if I'd been treating cosmic-text as a black box. This isn't an AI failure mode. It's a *context* failure mode, and humans have it too. If you've never built a renderer, you might not automatically think about hot loops and caching when you wire up a text library.

When resize felt bad, Claude went down exactly the same rabbit holes I did. Debouncing. Resize increments. Framework quirks. Metal vs wgpu. It was working at the framework level because that's where "resize feels bad in a GPU app" questions usually live. The actual bug was one level up, in the code I had written, in a loop I had put there.

The fix required one specific thing: recognizing that glyphs are textures. That a terminal is a sprite renderer. That `shape_text()` in a hot loop is `loadTexture()` in a hot loop. That pattern recognition isn't something you get from reading framework docs — it's something you get from 20 years of watching renderers tank because someone put a file load in the render function.

This is exactly what I was [writing about in my last post](/2026/04/11/syntax-is-the-least-of-our-skills/). AI is phenomenal at syntax, at library discovery, at making things work. It composes libraries faster than any human can. What it's weaker at is the shape of the code in context — the judgment calls that come from experience with a specific domain. A game programmer reading the first version of my atlas would have caught it in 30 seconds: "why are you calling a shaping engine per cell?" That's not algorithmic genius. That's pattern matching built up over years of writing renderers.

The collaboration worked the way I want it to work. Claude built the prototype fast. I brought the pattern. Both steps were necessary. Neither would have been as productive alone.

## What's Next

The atlas is in good shape now, but there's more game-engine-ifying to do:

- **Color emoji atlas.** Currently I skip color glyphs. Need a second `Rgba8UnormSrgb` texture and a way to route glyphs between the two atlases — or bindless textures to keep it one draw call.
- **Per-style caching.** Bold, italic, and regular should each have their own atlas regions. Right now I assume one weight/style globally.
- **Dirty row tracking.** alacritty_terminal tells me which rows changed between frames. I should write only the dirty rows to the instance buffer via `queue.write_buffer` with an offset, instead of rebuilding the whole buffer every frame.
- **Actual profiling.** The numbers in this post are estimates. I haven't sat down with a profiler yet. Every optimization in this post was guided by pattern recognition, not measurement. That's fine for the dramatic wins (HashMap fix) but not fine for the small ones (array vs HashMap). The next step is making sure my instincts match the numbers.

And then, of course, the actual terminal features: the fixed input editor at the bottom, command blocks via OSC 133 shell integration, tabs, splits, themes. The stuff that makes Flux different from Alacritty. That's what's coming next.

## The Lesson

A terminal is a sprite renderer. I've been saying that for three posts now and it keeps being true in new ways.

The first post talked about [geometry — one quad, instanced, one draw call](/2026/04/10/building-a-gpu-terminal-in-rust/). This post was about textures — atlas, pre-load, direct lookup, null objects. Different parts of the same pattern. If you've ever built a 2D game, you already know how to build a terminal renderer. The hard part isn't figuring out something new — it's recognizing that what you're doing has been solved before, in a different context, by a pattern you already know.

That's the whole game, honestly. Not just for renderers. For most engineering problems. The hardest part isn't the code. It's recognizing which old pattern fits.

It was never the framework. It's almost never the framework.
