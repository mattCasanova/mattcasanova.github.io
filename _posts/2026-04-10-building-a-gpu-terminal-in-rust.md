---
layout: post
title: "Building a GPU Terminal in Rust: Why and How"
date: 2026-04-10
categories: [flux, rust, terminal]
---

I've been using Warp for a while and I love the block-based UI — each command and its output grouped into a discrete, collapsible, copyable block. But the closed-source nature and telemetry bother me. So I'm building an open-source alternative called [Flux](https://github.com/mattcasanova/flux).

## Why Another Terminal?

There are plenty of terminal emulators. Alacritty is fast. Kitty is feature-rich. iTerm2 is the macOS default for power users. But none of them rethink what a terminal *looks like*. Warp did, and it's great — but it's closed source, requires an account, and phones home.

Flux takes the best ideas from Warp — command blocks, a modern input editor, GPU rendering — and builds them in the open. Rust + wgpu. MIT licensed. No telemetry. No account.

## The Architecture

Flux is structured as a Rust workspace with focused crates:

- **flux-terminal** — PTY management and terminal state (parsing ANSI/VT sequences)
- **flux-renderer** — GPU rendering via wgpu (Metal on macOS, Vulkan on Linux)
- **flux-input** — the input editor, keybindings, and keymap
- **flux-shell** — shell integration (detecting command blocks via OSC 133)
- **flux-types** — shared types across crates
- **flux-app** — ties it all together with a winit event loop

The key insight from game engine development: render the entire terminal grid in a single draw call using instanced rendering. Each cell is a quad, each glyph is a texture atlas lookup. Same technique I used for sprites in my game engine, applied to text.

```rust
// One draw call for the entire terminal grid
render_pass.draw(0..4, 0..cell_count);
```

## What's Next

I'll be writing more about the specific challenges — GPU text rendering, font atlas generation, terminal state parsing, and the surprisingly hard problem of detecting where commands start and end.

Follow along on [GitHub](https://github.com/mattcasanova/flux) or right here on this blog.
