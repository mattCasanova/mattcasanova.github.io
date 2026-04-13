---
layout: post
title: "Vim Can Wait"
date: 2026-04-13
tags: [flux, roadmap, planning, claude-code]
---

[Yesterday I wrote about getting stuck on Vim rendering bugs](/2026/04/12/define-interesting/). I sat with four compounding issues, shipped four one-line fixes, and ended the session feeling roughly half-accomplished and half-daunted. A classic "bit off more than I can chew" moment.

Today I did something about it. Not by fixing more bugs. By reorganizing the roadmap.

## Vim Wasn't the Wrong Thing to Fix

It was the wrong thing to fix *first*.

Here's what happened right after I finished the Vim rendering pass. I was trying to test the next feature. I opened Flux. I typed `vim `. And then I needed to paste the path to a file I had open in another window — something like `/Users/mattcasanova/src/flux/crates/flux-renderer/src/atlas.rs`.

I couldn't.

There was no clipboard support yet (I'd added it mid-session, but paste only worked inside the input editor, not the output area, and I'd copied the path from a different app where it existed as selected text, not clipboard). So I retyped the path. Then I made a typo. Then I retyped it again. Then I needed to re-run the same `vim` command thirty seconds later, and there was no command history, so I typed the whole thing again. Then I needed to paste a multi-line shell snippet into the input editor to test something, and the first newline fired Enter and ran half a command. Then I tried tab completion at `cd ~/workspace/fl<Tab>`, and nothing happened, because autocomplete didn't exist.

I'm building a terminal. And I couldn't use my own terminal to test my own terminal.

All those Vim bugs were real. All the fixes were right. The session wasn't wasted — Vim genuinely (mostly?) looks correct in Flux now. But none of it matters yet, because I can't *reach* Vim without a half-hour typing ritual. If the basics don't work, I can't even catch the next round of polish bugs.

**Daily driver UX comes before differentiators. Before polish. Before everything.**

This is a really obvious lesson in retrospect. It's the kind of thing people write blog posts about and you nod and think "of course." But it's incredibly easy to get wrong because the *technical* problem (Vim's bold is broken, here's why) is right in front of you, and the *meta* problem (you can't even reach Vim comfortably) is invisible until you sit in it.

## The Old Roadmap Was Organized Wrong

My original Flux plan had five phases, and the phase order was "what I thought about building, in order":

- **Phase 1** — Hello terminal. Window, GPU, PTY, basic input.
- **Phase 2** — Block architecture. The Warp differentiator.
- **Phase 3** — Tabs, themes, config.
- **Phase 4** — Splits, advanced editor, file tree.
- **Phase 5** — SSH and remote sessions.

What you can't see from that list is that Phase 4 was where I'd stashed the *actually important* stuff: command history, multi-line editing, undo/redo, tab completion. All the stuff I needed *today*, in order to use Flux as a daily driver, was in the *fourth* milestone. After blocks. After tabs. After themes. After configurable keybindings.

If I'd followed that order, I'd have spent months building "the Warp-killer differentiator" and "the theme system" and "the tab bar" with a terminal that was painful to use the entire time. I wouldn't have caught any of the polish bugs along the way, because I wouldn't have been living in it.

The phase structure wasn't dumb. It was a perfectly reasonable way to think about "what's the logical order to build these systems?" It was just the wrong question. The right question is: **at what point can I stop using iTerm2 and switch to Flux full-time?**

Answer: nowhere in the old roadmap. There was no milestone where I could imagine actually living in Flux. Every phase was a partial answer to "is this usable yet?"

## This Is My First Terminal

Here's the part that's actually embarrassing: I've been building software long enough to know better.

When I start a new mobile app, I build the dependency injection architecture before I build a single screen. The service layer. The view model boundaries. The way services get passed around. That's day one stuff — not a feature of the app, but the scaffolding that makes every feature possible. I would never skip it.

When I start a new website project, I don't write the Stripe integration first. I write the auto-deploy scripts. The one-click admin dashboard for database backups. The disaster recovery flow. The "click this button to redeploy to prod, including multi-server rollout" script. None of that is what the website is *for* — it's all the stuff that lets me *build* the website without losing sleep. I've shipped enough production apps to know that if you skip that layer, the first production incident will consume your whole weekend.

Even game engines, which are the closest analog to what Flux is. [I wrote a few days ago](/2026/04/12/the-right-algorithm-still-slower/) about finally adding broadphase collision to LiquidMetal2D after sitting on it for five years. That's not because broadphase is hard — I taught it to undergrads. It's because there was always more foundational work to do first. I still don't have a particle system in LiquidMetal2D. I'll get to it when the bones are ready.

So why did I get Flux backwards?

Because **this is my first terminal**. Probably my only terminal. And when you're new to a problem space, you reach for the cool stuff first — the stuff that makes the project feel real, the stuff that looks impressive in a screenshot. GPU rendering! Instanced cell quads! Glyph atlases! Tokyo Night Storm rendering in Vim! All of it is real work. All of it is progress. None of it is "can I paste a path into the input editor."

I don't have instincts for what's foundational in a terminal yet, because I've never built one. My brain wants to build the screen-printing pipeline because that's the part that *feels* like terminal work, when in reality the terminal work *is* the command history and the autocomplete and the multi-line editor that let you drive the damn thing in the first place. The rendering is the stage. The input editor is the actor. I spent a week polishing the stage lighting before I'd given the actor shoes.

To be fair, "let me see if I can print letters on screen" is pretty foundational. I had to do that first. But I stopped there mentally, thought "okay, rendering is solved, now let's polish the rendering," and missed that there's a whole other foundation layer I hadn't even started. Making sure Vim is 100% pixel-correct is *not* foundational. It's polish on top of a foundation that doesn't exist yet.

First projects in a new domain are humbling. You don't actually know which things are features and which things are table stakes until you try to use the thing.

## The New Shape

I reorganized around "versions that each ship a coherent usable product":

- **v0.2 — Daily Driver Foundation.** Clipboard, multi-line input, command history, undo/redo, autocomplete, scrollback, search, syntax highlighting in the editor, shell integration foundation (OSC 7 + OSC 133), configurable keybindings, theme system. Everything I need to stop needing iTerm2.
- **v0.3 — Blocks.** Now that OSC 133 is already flowing from v0.2, the Warp differentiator is a build-on-top instead of a build-from-scratch. Block model, navigation, per-block copy, exit codes, the floating header for long output.
- **v0.4 — Multi-pane.** Tabs and splits.
- **v0.5 — Persistence and Remote.** Mux daemon so the tabs survive a GUI restart. SSH with auto-tmux control mode.
- **v1.0 — Native SSH + Advanced Protocols.** Native russh, Kitty keyboard protocol, Kitty image protocol, OSC 8 hyperlinks.
- **v1.x — Polish Pass.** This is where the Vim rendering bugs went. This is where the "make it look *exactly* right" work lives. Not because it's unimportant, but because it's the polish pass you do *after* the daily driver works.
- **Future — AI, collaboration, CLI.** The Warp Drive / AI stuff. Local-first, BYOK, optional.

The point isn't the exact milestone structure. The point is that **every version is a coherent release** where I can stop and say "yes, I can live in this now." v0.2 is the one that matters most to me personally: it's the version where I uninstall iTerm2 and start eating my own dog food. After v0.2, every subsequent milestone is built by a user of the product (me), not just a developer of it.

And Vim rendering? Vim rendering is in v1.x. It's fine. It'll get there. But it can wait.

## The Catch I Almost Missed

Here's the part that's interesting about reorganizing a plan with an AI collaborator.

I had Claude help me work through the new structure. We dug through research on blocks, shell integration, autocomplete, multiplexing, SSH. Three parallel research passes turned into design docs that seeded the new milestone documents. Good work, fast work, the kind of synthesis that would have taken me a weekend on my own.

At the end, I had new milestone docs sitting next to the old phase docs, and I asked: *what should we do with the old ones?* The obvious answer was to archive them — they're historical context, not active plans. But then I asked: *are we sure we captured everything from those phase docs in the new structure?*

I don't know why I asked that. I just had a feeling.

I spun up a fresh agent to audit it — one that hadn't been in the reorganization conversation, so it had no memory of "surely that's already in the new docs, I'm sure." It read every old phase doc, cross-referenced against the new milestone docs, and came back with three real gaps:

1. The theme TOML schema and the built-in theme list from Phase 3 — the ones I want Flux to ship with.
2. The full TOML config schema with hot-reload details — also Phase 3.
3. The `EditorSnapshot` design for undo/redo from Phase 4.

Three non-trivial pieces of prior work that would have gotten silently dropped if I'd just archived the old docs.

That's the thing about reorganizing with an agent: **it's easy to lose context when you transform the structure, and the transformer is the worst one to audit itself.** Claude was confident everything had been migrated — the same way a human is confident about code they just refactored. The fresh pair of eyes caught what the active memory couldn't.

If I hadn't double-checked, those gaps would have shown up in two months as "wait, what did I decide about themes again?" and I'd have to rebuild them from scratch. Now they're slotted into v0.2 where they belong.

Small lesson with a big shape: when you reorganize anything — code, plans, documentation — audit with fresh eyes, not the same eyes that did the transformation. It's exactly the same reason you don't review your own PR.

## The Goal

The whole point of this reorganization is one thing: **get to a version I can use every day, and then keep going.**

v0.2 is the milestone I'm going to build next. When it ships, I'm going to stop using iTerm2. I'll use Flux for everything — code, builds, SSH, Claude Code sessions, the works. It'll be rough. It'll have bugs. Some of them will be the same Vim rendering bugs I chased yesterday, because Vim polish is in v1.x, not v0.2, and I'll just have to live with slightly-off bold text for a while.

But I'll be *in it*. And once I'm in it, I'll catch the real bugs — not the ones I can see in a debugging session, but the ones I only notice after the fifth day of using a tool full-time. Those are the bugs that matter. Those are the ones you can't find by polishing the thing you're not using.

After v0.2, I'll start showing it to friends. After v0.3, I'll put a proper release out. Somewhere around v0.5 or v1.0, Flux becomes something I'd recommend to a stranger instead of warning them away from. That's the staircase.

The roadmap isn't a prediction. It's a staircase I'm building so I can keep climbing.

Vim can wait.
