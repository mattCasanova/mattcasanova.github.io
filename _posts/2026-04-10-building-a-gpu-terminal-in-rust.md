---
layout: post
title: "This Is Heavy"
date: 2026-04-10
tags: [flux, rust, terminal, gpu, warp, claude-code]
---

It started with flickering.

I've been using [Claude Code](https://docs.anthropic.com/en/docs/claude-code) a lot lately — at work, on side projects, basically everywhere. It's the first AI tool that felt like it actually *worked* with my workflow instead of alongside it. It runs in your terminal, reads your files, builds plans, writes code. I was hooked.

But Claude Code in iTerm2 flickers. Bad. As Claude generates output, the screen would wildly scroll — jumping to the top, snapping to the bottom — while you're trying to read the previous response. You basically couldn't read anything until Claude finished. Then they tried to fix it by compacting the screen buffer, which stopped the scrolling but also meant you couldn't scroll back to see what Claude had just printed. If you'd asked it to discuss something at length — which is how I use it — you just... couldn't see the output. The only workaround was having Claude dump everything to a text file, which defeated the point. They've since added an experimental fix (`export CLAUDE_CODE_NO_FLICKER=1` in your shell rc) that helps, but even today I was hitting scroll issues with the latest version at work. It's a hard problem, and iTerm2's streaming architecture makes it harder.

So I tried [Warp](https://www.warp.dev/).

And honestly? Warp is great. The block-based UI — where each command and its output is a discrete, collapsible, copyable unit — is a genuinely better way to use a terminal. Claude Code runs beautifully in it. No flickering. Clean output.

There's just one problem: I can't use Warp at work. I'm an engineer at Meta, and Warp's telemetry and closed-source nature make it a non-starter in that environment. And it's not just Meta — a lot of companies have the same policy. Plenty of individual developers feel the same way.

So last night, I started looking into it: **does an open-source, GPU-rendered, block-based terminal exist?**

Nothing that I could find.

- [Alacritty](https://github.com/alacritty/alacritty) — GPU-rendered, fast, no blocks.
- [Ghostty](https://ghostty.org/) — GPU-rendered, beautiful, no blocks.
- [Wave Terminal](https://www.waveterm.dev/) — has blocks, but it's Electron. Not GPU-native.
- [Warp](https://www.warp.dev/) — GPU-rendered, blocks, closed source.

Nobody occupies both quadrants: native GPU rendering AND block-based output AND open source. So I'm building it.

## A Game Engine Developer Walks Into a Terminal

I'm not a terminal emulator developer. I'm a game engine developer.

I taught C/C++ and game development as a professor for seven years. I [co-authored a book](https://www.amazon.com/Game-Development-Patterns-Best-Practices-ebook/dp/B01MRP7SPA/) on game development patterns. I've been building game engines as side projects for as long as I can remember — most recently [LiquidMetal2D](https://github.com/mattCasanova/LiquidMetal2D), a 2D engine in Swift + Metal for iOS.

So when I asked Claude Code to research what it would take to build a GPU-rendered terminal, the first thing I did was ask about the rendering. Should I use Metal? Something cross-platform? It pointed me to **Rust** — Warp is already built in Rust, and the ecosystem has mature crates for everything I'd need. Then it explained **wgpu** — a cross-platform GPU abstraction over Metal, Vulkan, and DX12. Claude described it as "like SDL for C++" — a cross-platform layer so you don't have to target each graphics API separately. I immediately got it.

Then Claude showed me some shader code for the terminal cell renderer. And that's when I had the moment.

The initial approach had the shader creating a vertex buffer for every cell, every frame. And I looked at it and went — *wait, why are we doing that? Why don't we just create the quad once?*

Because that's what you do in a 2D game engine. You create one quad at startup and reuse it for every sprite via instanced rendering. You pack your textures into an atlas. You build a per-instance buffer — position, UV coordinates, color — and fire a single draw call. The GPU does the rest.

I had *literally just done this* in LiquidMetal2D. I'd just finished adding instanced rendering the month before. So I pointed Claude at my game engine code on my local machine and said: *look at how this is organized. Look at how the graphics pipeline works. Can we do the same thing here?*

And it clicked. A terminal grid is the same problem. Each cell is a sprite. Each glyph is a texture in the atlas. One draw call renders the entire visible grid:

```rust
// 4 vertices (the quad), N instances (one per cell)
render_pass.draw(0..4, 0..cell_count);
```

That's the moment this went from "interesting idea" to "I'm actually going to build this." Not because I know terminals — I don't. But because the hardest part of a GPU terminal is the GPU part, and I've been solving that exact problem for years.

## This Is What AI-Assisted Development Actually Looks Like

I want to be transparent about something: Claude Code is not just the catalyst for this project — it's a core part of how I'm building it.

I used Claude Code to research every existing terminal emulator, analyze their architectures, and identify the gap in the market. It built detailed phase plans. It's writing Rust code alongside me. When I hit a Rust concept I don't fully understand, I learn it in context by building real features instead of reading a textbook.

But here's what makes it actually work: **it can see my code.** This isn't ChatGPT in a browser where you're copy-pasting snippets. Claude Code runs on my machine, in my terminal, with access to my files. When I said "look at how my game engine does instanced rendering," it could actually read `LiquidMetal2D/Sources/Renderer/` and understand the architecture. That's how it knew to restructure the terminal renderer to use the same pattern.

This is also why the terminal flickering bothered me enough to do something about it. Claude Code is a terminal-native tool. The terminal *is* the IDE. If the terminal can't keep up with the tool, everything breaks down.

In the last month alone, I used Claude Code to push through [25+ improvements](https://github.com/users/mattCasanova/projects/3) to LiquidMetal2D — spatial grid collision detection, a component system, renderer API cleanup, a collision stress test running 7,000 objects at 30fps on a phone. Work that would have taken months of weekend sessions. That velocity is what gave me the confidence to take on a terminal emulator as a side project.

This is what AI-assisted development looks like in 2026. Not "AI wrote my code." More like: I have 20 years of context about game engines and GPU rendering, and AI helps me apply that knowledge to a domain I'm still learning at a pace that would have been impossible a year ago.

## What I Don't Know

I'm going to be honest about the gaps.

**I'm not a Rust expert.** I taught C and C++ for seven years, so the mental model translates — ownership, borrowing, lifetimes all map to concepts I already understand. But I've never shipped a Rust project. I'm learning as I go.

**I've never written a terminal emulator.** VT/ANSI escape sequence parsing, PTY management, shell integration — all new territory. Fortunately, the Rust ecosystem has battle-tested crates for the hard parts: [alacritty_terminal](https://crates.io/crates/alacritty_terminal) for the terminal state machine, [portable-pty](https://crates.io/crates/portable-pty) for cross-platform PTY management, [vte](https://crates.io/crates/vte) for escape sequence parsing.

**I don't know if this will work.** The block detection problem — figuring out where one command ends and the next begins — requires shell integration via [OSC 133](https://gitlab.freedesktop.org/Per_Bothner/specifications/blob/master/proposals/semantic-prompts.md) escape sequences. It works in Warp. It works in Wave. But making it reliable across zsh, bash, and fish with arbitrary user configurations? Open question.

## The Plan

The project is called **Flux** — a Back to the Future reference, because why not.

The stack:
- **Rust** + **wgpu** — native GPU rendering via Metal (macOS) and Vulkan (Linux)
- **cosmic-text** — font rasterization and text shaping
- **alacritty_terminal** — terminal state machine (proven, embeddable)
- **portable-pty** — cross-platform pseudo-terminal management

The architecture is a Cargo workspace with focused crates — all GPU code isolated in `flux-renderer`, terminal state in `flux-terminal`, input handling in `flux-input`, shell integration in `flux-shell`. Same kind of separation I use in game engines: rendering pipeline, game objects, input, scene management. Different domain, same principles.

Phase 1 is a window that spawns a shell, renders its output via wgpu, and has a fixed input prompt at the bottom — the Warp-style layout. No blocks yet, just a working GPU-rendered terminal that proves the pipeline works.

The longer-term vision: blocks, tabs, split panes, themes, SSH with automatic tmux persistence, extreme customizability. Open source, MIT licensed, zero telemetry, no account required. Everything Warp does well, without the parts that keep people from using it.

## Follow Along

The code is at [github.com/mattcasanova/flux](https://github.com/mattcasanova/flux). I'll be blogging here about the specific challenges as I hit them — GPU text rendering, glyph atlas management, block detection, and whatever else turns out to be harder than I expected.

If you're interested in an open-source Warp alternative, star the repo. If you want to help build it, even better.

This is heavy. Let's go.
