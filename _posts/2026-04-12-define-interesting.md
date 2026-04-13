---
layout: post
title: "Define 'Interesting' (It's a Lot of Damn Work)"
date: 2026-04-12
tags: [flux, rust, terminal, vim, phase-1]
---

I opened Vim in Flux today.

It launched. The raw mode bypass I'd just finished building worked — the input editor got out of the way, the shell surrendered control, Vim took over the grid. First victory of the session.

Then I actually looked at what was on screen. And the rest of the day went sideways.

## What I Thought Phase 1 Had Left

Going into the session, I had the bones working. Window, wgpu renderer, the glyph atlas from [88 Miles Per Second](/2026/04/12/88-miles-per-second/), PTY, alacritty_terminal wrapper, basic keyboard input. The remaining list for Phase 1 read like a short afternoon:

- Fixed input prompt at the bottom, Warp-style
- Screen split into output area and input area
- Forward composed commands on Enter
- Basic scrollback
- Raw mode detection and bypass so Vim, nano, and less work
- Ctrl+C / Ctrl+D / Ctrl+L — mostly already working

I thought this was maybe a two-hour block. It ate the entire day, spawned about 17 new GitHub issues, and I *still* didn't touch scrollback.

## The Input Editor (the P0 version)

The Warp-style input editor is the whole point of Flux's UX. Unlike a traditional terminal that forwards every keystroke directly to the shell, Flux holds them in its own buffer. The shell sees nothing until you hit Enter.

This is not the final version. It's a rough draft — just enough to prove the pipeline works. Single-line only. No history. No multi-line. No syntax highlighting. No autocomplete. But the bones are in place, it works, and that's all I needed for Phase 1.

I wired it up as a dedicated crate (`flux-input::InputEditor`) with a single-line text buffer, byte-offset cursor tracking, and the usual edit operations — insert, backspace, delete forward, move, home, end. The app routes keyboard events through the editor instead of straight to the PTY; on Enter, it calls `take_line()` and writes `line + \r` to the shell. The renderer gained a `set_input_line(text, cursor_col)` method and now tracks output-grid instances and input-chrome instances separately, rebuilding the combined GPU buffer on either update. The layout math steals two rows from the PTY grid — one for the divider, one for the input — so the shell's prompt stays in the output area above the chrome.

That part worked clean. Cmd+C, Cmd+D, Cmd+L bypass the editor entirely because `text_with_all_modifiers()` folds control characters into single-byte outputs that go straight to the PTY. I just had to avoid breaking that handling from the previous session.

Then I started on raw mode detection, and everything got interesting.

## Raw Mode Bypass

For Vim, less, nano, htop, and tmux to work, the input editor has to *step aside* when they take over. That's "raw mode bypass" — shell in control, Flux intercepts keys; Vim in control, they pass through.

First attempt: use `tcgetattr()` on the PTY master to check if `ICANON` / `ECHO` are off, which is the obvious Unix way to detect "someone wants raw keystrokes." The input editor disappeared on startup before I could type a single character. Turns out every interactive shell puts the PTY into termios-raw mode as soon as it's ready for input — that's how readline, zle, and fish do per-keystroke autocomplete and history. Shells live in termios-raw mode 99% of the time. My check was firing constantly.

Fix: drop termios entirely and key off `TermMode::ALT_SCREEN` from alacritty_terminal. That catches Vim, less, man, htop, tmux, fzf, top. It misses edge cases like password prompts that only flip ECHO, or fzf with `--no-height` — tracked as follow-ups. Vim and friends work now, and that's all I needed for this session.

## Vim Doesn't Respect My Theme

Once raw mode bypass worked, I opened a Rust file in Vim inside Flux and compared it to the same file open in iTerm2.

They looked nothing alike.

In iTerm: keywords bold, comments italic, full background fill from edge to edge, everything the way Tokyo Night Storm is supposed to render.

In Flux: everything one weight. The color scheme seemed to only paint around the text characters, not in the whitespace. The background was the wrong color around the edges of the window. It just looked *broken*.

Claude's first guess was that Vim wasn't loading my vimrc. I pushed back — syntax highlighting was clearly happening, look at the colors, the problem was with the background. Claude was looking at the wrong thing. This is a thing I'm learning about AI-assisted debugging that didn't hit me until now: **Claude will commit hard to its first theory, and if you don't push back with what you actually see on screen, it'll keep iterating inside a wrong mental model.** My job wasn't to write the fix. It was to keep redirecting back to the evidence.

There turned out to be *four* compounding bugs — each one hiding inside the others.

**Bug 1: One atlas, one style.** The glyph atlas was rasterizing one weight/style and using it for every cell. Vim's bold/italic flags were being ignored. Fix: extend the flat-ASCII-array pattern from [88 Miles Per Second](/2026/04/12/88-miles-per-second/) into four arrays (Regular, Bold, Italic, BoldItalic), pre-rasterize all four at startup, pick the right slot per cell.

**Bug 2: Cell backgrounds only painted under glyphs.** `render_glyph` was pushing an instance sized to the glyph's bounding box, not the full cell — so Vim's theme background showed up as tiny rectangles behind each character, and spaces showed through to the window's clear color. Fix: emit a full-cell background rect per cell before the glyph. Ten lines. The real fix here is probably smarter — next session I want to explore reading the theme's actual background and updating the wgpu clear color directly, so the whole surface is tinted before we even start drawing cells.

**Bug 3: Leftover wall space at the edges.** Flux has padding around the grid — intentional, looks nice. But that meant when Vim took over, there was a frame of Flux's theme color around the outside. Fix: sync the wgpu clear color to the alt-screen program's background whenever we're in raw mode. Sample the top-left cell's background, use it as the clear color. Vim, nano, and less all reliably mark that cell with their theme's Normal background, so it works.

**Bug 4: The cursor was letter-sized.** Claude made the cursor `cell_h` instead of `glyph_h`. Didn't fix it. The real problem was instance ordering — the cursor block was pushed first, then the cell's background rect for that same cell was pushed on top, overdrawing everything but the glyph. Fix: skip the cell background rect when the cell is the cursor cell. Two lines.

None of these were foundational. Every one was a single misordered pass or a single line of code. But you only find them by *running the thing and looking at it next to a terminal that works.* The tests I have can't catch "Vim looks slightly off." Only comparing frames can.

## Paste Support (or: I Couldn't Test Anything Without It)

Somewhere around the third Vim bug, I realized I'd been typing every test path by hand.

`vim /Users/mattcasanova/src/games/LiquidMetal2D/Sources/Renderer/RenderPipeline.swift`

Perfectly.

Every time.

Because stupidly, I was trying to test if Vim worked before I had basic features like copy, paste, or autocomplete.

<br><br>

That's the thing about building a terminal from scratch. You forget how much of a shell's "feel" is actually tab completion and paste and history. Strip all of that away and you're back to typing commands into a DOS prompt from 1985. You don't even notice those features are there until they're not, and then it's all you can think about.

I stopped everything and added paste. `arboard` for cross-platform clipboard access. `WindowEvent::ModifiersChanged` tracking for modifier state. Cmd+V on macOS, Ctrl+Shift+V cross-platform fallback.

<br><br>

As it turns out, there are two paste paths — because of course there are freaking two. Why not:

- **Cooked mode** (Flux input editor active): read clipboard, collapse newlines to spaces so multi-line pastes don't fire submissions through Enter, call `input.insert_str`.
- **Raw mode** (Vim etc.): read clipboard, check if the alt-screen program has bracketed paste enabled (`TermMode::BRACKETED_PASTE`), wrap in `\x1b[200~ ... \x1b[201~` if so, forward to the PTY.

Bracketed paste is one of those things you don't notice until it's missing. Modern shells enable it by default so they can distinguish "user pasted a command" from "user typed a command" — it matters for history (pasted commands often shouldn't be saved) and for Vim (so `p` doesn't auto-indent pasted content into nonsense). Flux can't set bracketed paste itself; we just honor whatever the child has requested.

And still, copy is *not* done yet. That'll have to be later. Copy needs a selection concept — mouse click-drag in the output area — and the mouse event path is shared with a bunch of other future features (clickable paths, URL detection, link hover). So I'm deferring selection until I can build the whole mouse pipeline in one pass. Filed as issue #30.

## Bit Off More Than I Can Chew

Somewhere around the fourth Vim bug, still having trouble navigating because I hadn't written paste yet, I hit the wall.

It's kind of hard to use a terminal that doesn't support the basics. Even just the small tweaks to get Vim working were hard. And even after the tweaks, it still doesn't look perfect — there's definitely more to do. That's an "interesting bit off more than I can chew" moment. This was definitely not going to be a week-long project.

And the honest version:

I guess this is why nobody's hit this space before. It's a lot of damn work. Nobody's doing this for free.

Which, it turns out, is kind of true. Let me do the roll call:

- **Alacritty** — years of work, one main maintainer who burned out for a while.
- **Kitty** — Kovid Goyal's life's work. Decade-plus.
- **WezTerm** — Wez Furlong, another one-person show for years.
- **Ghostty** — Mitchell Hashimoto. Famously waited until he had post-HashiCorp fuck-you money before starting.

The Warp-style "smart terminal" niche is basically unoccupied in open source. Not because nobody noticed — maybe people are building it right now, I don't know — but because the people with the skills to build it usually picked a different problem or sold it to a startup.

Terminals take years. **That still hits me hard typing it now.** Terminals take patience. Terminals take the kind of staying power that most people's hobby projects don't have.

I'll be honest about where I am with this: I have *some* of the skills. I've been building 2D game engines for years. GPU rendering doesn't scare me. Instanced quads, texture atlases, dirty tracking, render-on-demand — that's the part I actually know how to do. What I didn't fully appreciate, until I spent a day chasing Vim rendering bugs, is how much of a terminal emulator is *not* rendering. It's the edges. It's the 40+ years of expectations users have built up.

If Vim doesn't work, you don't have a terminal.

If tmux doesn't work, you don't have a terminal.

If resizing makes it flicker, people will go back to the one that doesn't — which is literally [why I started building Flux in the first place](/2026/04/10/building-a-gpu-terminal-in-rust/). The whole project began because Claude Code was flickering in iTerm2. If I ship a terminal that flickers too, what was the point?

I went into this thinking "I can build the rendering layer and compose the rest from crates." That's still mostly true. But the "compose" part has a lot of damn sharp edges.

## Terminals Are Deceptively Simple at the Core and Brutally Complex at the Edges

The core loop of a terminal is simple. alacritty_terminal parses the bytes. You get back a grid of cells. You draw the cells. That's the entire hot path. The parser is solved, the state machine is solved, the rendering is "just GPU sprite batching," and I wrote three blog posts in a row about how that pattern isn't new.

The complexity is not in the core. It's at every edge. What happens when text overflows. When Vim sets its own background. When the alt screen engages. When the scale factor changes. When the cursor sits on a glyph you're about to overdraw. When a shell is in termios-raw mode for 99% of its lifetime because that's how readline works. When you render a cell's background behind the letter instead of behind the whole cell and the whole screen looks visibly broken. When your "clever" OR default inverts the actual style flags and nobody can escape bold italic mode. When the sliver on the right edge of the window is a different color than everything else because `window_width / cell_width` rounds down. When your cursor is there, painted first, and silently overdrawn by every cell background rect that came after.

None of that is the core loop. All of it is polish. And every one of those polish items is a thing Vim users will notice within 30 seconds of launching a session.

The upside — and I want to hold onto this — is that **what I built this session is real**. Every bug I chased was a real bug with a real fix. The architecture didn't need to be rethought. The `flux-renderer` / `flux-terminal` / `flux-input` / `flux-app` split still makes sense. `alacritty_terminal` is still the right state machine. The glyph atlas pattern from [88 Miles Per Second](/2026/04/12/88-miles-per-second/) still holds. Every bug landed in the same category: "the shape of the code is correct, we just need to handle one more thing." That's a much better position than "nothing works and I don't know why."

17 new GitHub issues from one session. Every one of them is a real thing a user will hit. Every one of them has a known fix. I just need a year of evenings to get through them.

## I'm Not Abandoning This. It's Just Going to Take a While.

I've been writing this blog for a few days now and every post has been about the velocity — "I can do things in an evening that used to take a weekend." That's still true. Today's session was a bunch of real, correct bug fixes shipped in about eight hours. Five years ago, finding and fixing four compounding Vim rendering bugs in one day would have been a huge win. Today I'm slightly disappointed that I didn't also get scrollback done.

That's the honest trap of using Claude Code. The velocity distorts your expectations. You start thinking "surely I can ship a terminal emulator in a couple of weeks." The answer is no. The answer is that a terminal emulator is still a multi-year project, even with AI help. AI makes the hard parts faster. It can't make the *number* of parts smaller.

I use terminals all day, every day. I love tweaking themes. I love writing config files. I love dotfiles. Flux is not a two-week sprint for me — it's a project I actually *want*, and the fact that it's going to take a long time is a feature, not a bug. Every session will yield another post. Every detour becomes content rather than failure. That's what this blog is for. That's why I'm writing the "bit off more than I can chew" moment down instead of pretending it didn't happen.

Alacritty took years. Kitty took years. WezTerm took years. Ghostty took years. Flux will take years.

But the bones are good.

Vim *almost* looks right.

That's enough for one night.

Time for a drink.
