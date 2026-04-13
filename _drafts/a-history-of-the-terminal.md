---
layout: post
title: "Standing on the Shoulders of xterm"
tags: [terminal, history, open-source, alacritty, kitty, wezterm, ghostty, iterm2, warp]
---

I wrote yesterday about how terminals take years. About the burnout cycles. About how the people who build them either pick a different problem or sell it to a startup.

Today I want to sit with that for a minute, and do some roll call for the people who actually did it.

Because every single crate I'm composing Flux out of — `wgpu`, `cosmic-text`, `alacritty_terminal`, `portable-pty`, `vte` — exists because somebody else spent years of their life on this exact problem before me. I didn't invent any of it. I didn't even come up with the *approach*. I'm just stitching together other people's work and adding blocks on top. That's worth being honest about.

This is a brief history of the open-source terminal emulators that got us here. It's not exhaustive. It's not academic. It's me looking at my own `Cargo.toml` and trying to trace the lineage back to the humans who made it possible.

## The Ground Floor: xterm (1984)

Before anything modern, there was **xterm**.

xterm was created by Mark Vandevoorde in 1984, as a student working under Jim Gettys, originally for the VAXStation 100. It wasn't meant to be the canonical X11 terminal emulator. It just became one, because nothing else existed, because it worked, and because it was bundled with every X Window System install for the next 40 years.

Jim Gettys once famously said:

> "Part of why xterm's internals are so horrifying is that it was originally intended that a single process be able to drive multiple VS100 displays."

In other words: it was architectural debt on day one. It still runs. It's still maintained (Thomas Dickey has been the steward since the mid-90s). The latest release was earlier this year. *Forty-two years*, still shipping.

Every terminal emulator we talk about below — including the ones that are "new" — inherits xterm's control sequence vocabulary. The escape codes you use to set cursor position, colors, bold, italic, alt-screen — all of it descends from xterm's extensions to the DEC VT100 / VT220 standards. When I [write a post about raw mode and vim](/2026/04/12/define-interesting/), the protocol I'm implementing was specified by a student in 1984 and patched continuously ever since.

We don't talk about xterm because it's old and ugly and nobody writes their Rust project about it. But every single modern terminal is standing on its shoulders.

## iTerm and iTerm2 (2002 → 2010)

The first "iTerm" started in 2002, written by Fabian, as an attempt to give macOS a decent tabbed terminal back when Terminal.app was barely usable. It was a starter project, and it sort of stalled.

In 2010, **George Nachman** forked it into **iTerm2**. iTerm2 rapidly became *the* macOS terminal for developers — split panes, full-screen mode, Growl notifications, profiles, instant replay, and eventually tmux integration and GPU rendering. It's written in Objective-C and is still maintained and shipping as of January 2026.

George is the reason macOS developers had a pleasant terminal for the entire decade between 2010 and 2020. He built iTerm2 mostly alone for most of its existence. When I finally got fed up with iTerm2's flickering with Claude Code a few weeks ago and started looking for alternatives, it wasn't because iTerm2 was bad. It was because it had been *fine* for me for ten years and was starting to struggle with an AI tool that does 60fps streaming.

iTerm2 is the quiet workhorse. Nobody writes Medium articles about it. It just keeps working for millions of people, for free, because George kept showing up.

## Alacritty (January 6, 2017)

**Joe Wilm** announced [Alacritty](https://github.com/alacritty/alacritty) on January 6, 2017. The pitch was blunt: *"The result of frustration with existing terminal emulators."*

Alacritty was the first modern open-source terminal to lean hard into GPU rendering for raw text throughput. Written in Rust. OpenGL. No tabs. No splits. No multiplexer. On purpose. Joe's philosophy was that Alacritty should do one thing — render text at the speed of the GPU — and let tmux handle everything else. The config was YAML (later TOML). Performance was unmatched. Benchmarks of scrolling and `cat`-ing a large file were its calling card.

Joe eventually stepped back from maintaining it. The project now runs under **Kirill Chibisov** and **Christian Dürr**, who've kept it going for years. That's a real thing too — the "I built this and then couldn't do it forever" pattern. Open source terminals burn out their maintainers.

Alacritty's lasting gift to the ecosystem is two crates:

- **[`vte`](https://crates.io/crates/vte)** — the ANSI escape sequence parser. 50 million downloads. If you're writing anything that touches terminal output in Rust, you are probably using `vte`, directly or transitively. Flux does.
- **[`alacritty_terminal`](https://crates.io/crates/alacritty_terminal)** — the terminal state machine itself, extracted as a reusable library. Grid, selection, scrollback, mode flags, the works.

I don't have a VT/ANSI parser in Flux. I have Joe Wilm's parser. It's the right call — the parser is one of those subtle, gnarly problems where 98% of correct isn't enough, because vim and tmux and fzf will find the 2% you missed within 30 seconds of launching them. [I learned that the hard way yesterday](/2026/04/12/define-interesting/).

I owe a lot to that crate. Flux literally doesn't run without it.

## Kitty (2017)

The same year, **Kovid Goyal** shipped [Kitty](https://github.com/kovidgoyal/kitty). You might know Kovid from another slightly-well-known project he also built mostly alone: **Calibre**, the ebook library manager, which somehow runs half of the world's personal e-reader libraries without most people knowing who made it. Kovid does not suffer from a lack of patience.

Kitty is written in a mix of C, Python, and Go. It's GPU-accelerated via OpenGL. But what makes Kitty distinctive isn't the rendering — it's that Kovid just kept *inventing things* for the terminal ecosystem. The list of things Kitty pioneered or standardized:

- **The Kitty image protocol** — inline images in the terminal. Before this, the only options were Sixel graphics (ancient) and iTerm2's proprietary protocol. Kitty's protocol is now supported by WezTerm, Ghostty, and a handful of others. Every time somebody renders an image inside `ranger` or `nvim` with a real bitmap, that's Kovid's protocol.
- **The Kitty keyboard protocol** — a cleaner way for terminals to report keyboard events, including modifier keys and key release events. A lot of modern TUIs depend on this.
- **Kittens** — Kitty's extension system. Python scripts that the terminal runs to add features. Image viewers, Unicode input, hints for opening URLs, all shipped as "kittens."
- **Tiling windows, tabs, and an entire graphical session concept** baked into the terminal, with no tmux needed.

Kitty is the terminal emulator that reminds me how far one person can push something if they don't stop. And Kovid has been doing this since *Calibre* in 2006. Two decades of shipping.

## WezTerm (2018)

**Wez Furlong** started [WezTerm](https://github.com/wezterm/wezterm) on February 7, 2018. Rust. GPU-accelerated. Cross-platform.

WezTerm's thing is that it's a terminal *and* a multiplexer in one app. If iTerm2 is "Terminal.app but good" and Alacritty is "just the rendering loop," WezTerm is "everything in one binary." Tabs, panes, splits, SSH client, serial port support, Lua config (which is *wonderfully* powerful once you get used to it), and a built-in multiplexer with optional persistent domains so you can detach from a GUI and re-attach later.

Wez also publishes two crates that the entire Rust terminal ecosystem depends on:

- **[`portable-pty`](https://crates.io/crates/portable-pty)** — cross-platform PTY management. Unix and Windows. 5+ million downloads. Flux uses this. I don't know how I'd handle Windows ConPTY without it.
- **[`termwiz`](https://crates.io/crates/termwiz)** — an alternative to `alacritty_terminal`, with richer protocol support including sixel graphics, iTerm image protocol, and hyperlinks.

WezTerm's README says, verbatim: *"This is a spare time project."* That note has been there for years while the project has accumulated 25k+ stars and 60+ releases. One person. Spare time. Millions of users.

I want to be the kind of person Wez is, but I am aware of how much of what Wez is comes from not stopping for eight years.

## Ghostty (2022, public 2024)

[Ghostty](https://ghostty.org/) is **Mitchell Hashimoto's** project. Mitchell is the co-founder of HashiCorp (Vagrant, Packer, Terraform, Consul, Nomad, Vault — if you've deployed cloud infrastructure in the last decade, you've used something Mitchell helped build). The Ghostty repo was created on March 29, 2022. It ran as a private beta for a long time while Mitchell tinkered. The public 1.0 shipped at the end of December 2024.

Ghostty is written in Zig, which is a choice. It uses Metal on macOS, OpenGL on Linux, and native platform UI toolkits — AppKit on macOS, GTK on Linux — for the window chrome. That last bit matters more than people realize: most modern GPU terminals look like they escaped from a game engine because they use `winit` or similar. Ghostty looks *native*, because it's actually using the OS widgets.

Mitchell has been very open about the fact that he waited until after HashiCorp before starting Ghostty. He had post-HashiCorp F.U. money, he had a problem he wanted solved, and he had the time. That's not a criticism — that's exactly the point I was making [yesterday](/2026/04/12/define-interesting/). Nobody builds a serious terminal emulator on weekends while working a full-time job. It's too much work for too little return, so the only people who finish them are the ones who can afford to spend the years on it.

Ghostty's 1.0 launch was a big deal. Within a couple of months it had more GitHub stars than Alacritty took in seven years. It's an excellent terminal and it instantly became a daily driver for a huge chunk of the macOS / Linux dev community. And it did what every good open-source terminal does: it pushed the baseline forward.

## The Closed-Source Outlier: Warp (2020)

**Zach Lloyd** founded [Warp](https://www.warp.dev/) in June 2020. Former Principal Engineer at Google, former interim CTO at TIME. Built in Rust. Closed source. VC-backed — Sequoia led a $50M Series B in June 2023, on top of earlier rounds totaling $23M. The pitch was *"the terminal for the 21st century"* — command blocks, a real input editor, AI command suggestions, shared team runbooks, the works.

Warp is what Flux is trying to be an open-source alternative to. I want to be clear about that: I'm not mad at Warp. I think Warp is a genuinely great product. Command blocks are the single best UX improvement to the terminal in decades, and they nailed the implementation. The input editor is clean. The block collapsing, the exit-code indicators, the one-click copy of just the output — all of it is *correct*.

The things that make Warp not work for me personally are:

1. It's closed source, which means I can't ship it at Meta, I can't modify it, and I can't see what it's doing.
2. It phones home by default, and while the telemetry can be turned off, the fact that it exists at all disqualifies it for a lot of companies' security policies.
3. It requires an account, and that account is tied to cloud features that I don't need.
4. It's VC-backed, which means its incentives are eventually going to diverge from mine.

None of that makes Warp *bad.* Warp is the obvious paid option if you want a modern terminal and don't care about the above. But my whole argument for Flux is that the feature set Warp ships should exist in open source, without any of the strings attached. A VC-backed company can't ship that — their investors don't want them to. That's not a moral failing, it's just math. Open source is the only place that thing can live.

Also: Zach Lloyd and the Warp team obviously know what they're doing. The fact that Warp is built in Rust is not a coincidence — they looked at Alacritty and said "we want that performance." The fact that they support tmux control mode for SSH is not a coincidence — they looked at iTerm2 and said "we want that integration." Warp is a commercial product standing on the same open source shoulders I'm standing on.

## The Other People I Owe

A few projects I haven't given their due yet but should:

- **[Rio](https://github.com/raphamorim/rio)** — written by Raphael Amorim. Rust + wgpu (the same GPU library Flux uses). If you want to see what a professional wgpu terminal looks like, read Rio's source. It's the closest thing to a "how would you do this in 2025" reference implementation. Its renderer crate, `sugarloaf`, is a hair too risky for me to depend on directly, but I've absolutely been reading it for patterns.
- **[Wave Terminal](https://www.waveterm.dev/)** — the one serious open-source Warp-alternative that already exists. Block architecture, cross-platform, Apache 2.0. But it's Electron, not GPU-native. Which means it's slower and heavier than the Rust/Zig options. I'm not shipping Electron, but Wave proved you *could* ship blocks in open source. The bar exists because they set it.
- **[cosmic-text](https://github.com/pop-os/cosmic-text)** — System76's text shaping engine, written in pure Rust. Handles what HarfBuzz does in C. Flux's [glyph atlas from 88 Miles Per Second](/2026/04/12/88-miles-per-second/) calls into cosmic-text for every glyph. System76 built it for their cosmic-epoch desktop environment, but the whole Rust ecosystem gets to use it because they open-sourced it.
- **[wgpu](https://github.com/gfx-rs/wgpu)** — the cross-platform GPU abstraction over Metal, Vulkan, and DX12. Maintained by the gfx-rs working group. Without wgpu I'd be writing three rendering backends instead of one.
- **[Oh My Zsh](https://ohmyz.sh/)** — yes, it's a zsh config framework, not a terminal emulator. But every developer I know who "has a nice terminal" has `oh-my-zsh` installed somewhere. It's been maintained for over a decade, originally by Robby Russell, and it's the reason a huge chunk of the developer population has a colorful, git-aware, useful prompt instead of `$`. Thank you, Robby.

## The Common Thread

Every terminal I listed up there — Alacritty, Kitty, WezTerm, Ghostty, iTerm2, even Rio — has roughly the same shape as a project. One or two people, years of work, a ton of subtle bugs, most of which are at the edges (rendering, font handling, protocol support, vim/tmux compatibility). None of it is core-loop glamour. All of it is polish.

And all of them had to get built before my stack was possible.

I'm writing Flux in 2026 with `wgpu` + `cosmic-text` + `alacritty_terminal` + `portable-pty` + `vte` and it feels *productive*. I'm composing libraries that each represent five to ten years of somebody else's work. Joe Wilm's parser. Wez Furlong's PTY layer. System76's text shaper. The gfx-rs working group's GPU abstraction. Alacritty's terminal state machine. When I ship a working demo after two days, that's not me being fast. That's me *finally standing on the shoulders of everyone who built the ground floor first.*

Before Alacritty, there was no Rust terminal state machine to reuse. Before WezTerm, there was no cross-platform PTY crate. Before cosmic-text, Rust text shaping was a pile of half-finished attempts. Before wgpu, you'd be writing Metal *and* Vulkan *and* DirectX separately. Five years ago, Flux wouldn't have been a weekend project for me. It would have been a five-year project. I'd be writing my own PTY layer, my own ANSI parser, my own text shaper, and my own GPU abstraction, *before* I got to even think about command blocks.

I keep writing on this blog that AI is the thing that changed my velocity. That's true. But it's only half the story. The other half is that the *open source ecosystem* changed my velocity. And the people who built that ecosystem — Joe Wilm, Kovid Goyal, Wez Furlong, Mitchell Hashimoto, George Nachman, Raphael Amorim, the gfx-rs team, System76 — did it the hard way, over years, without AI, mostly for free.

I'm just adding a layer on top of their work. And when Flux eventually ships and somebody builds their own thing on top of *my* crates, that'll be fine too. That's how this is supposed to work.

## The Honest Part

I started this post thinking I'd write a breezy "here's a history" piece. I'm ending it a little emotional, honestly. Not because I'm trying to be dramatic. Because I spent yesterday chasing four compounding Vim bugs and I thought I was doing something hard. Then I looked at how long Alacritty took, how long Kitty took, how long iTerm2 has been George Nachman's one-person project, and I'm reminded that "hard" has always been the job. It's the whole thing.

The thing I want Flux to be — open-source, GPU-accelerated, block-based, zero telemetry — is only possible because *five different terminals* and *dozens of Rust crates* already did the hard parts. Every one of those projects is a multi-year solo endeavor by somebody who just kept showing up. That's the real lesson, not anything about Big-O or sprite batching.

If you're using Flux someday, you're using Joe Wilm's parser, Wez Furlong's PTY, System76's text shaper, and gfx-rs's GPU layer, with a Flux-shaped wrapper around it. Including the bugs. Including the polish. Including the decade of work that went into making those crates good enough to trust.

I'm just the new name on the box.

Time for a drink, and one poured for all of you.
