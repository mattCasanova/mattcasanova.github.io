---
layout: post
title: "The Night Shift Workflow: Superwhisper, Claude Code, and Parallel Agents"
tags: [workflow, claude-code, ai, productivity]
---

> **PRE-DRAFT SEED — not ready to publish.** Placeholder for a future standalone essay about how I actually work on side projects at night — Superwhisper for voice input, Claude Code for coding, parallel agents for research and planning, and juggling multiple projects at once.

---

## The seed

The shape of the post is "here's what the actual day-to-day looks like when I'm working on Flux, LiquidMetal2D, and the mobile apps all in the same evening." It's not a productivity-guru post. It's more honest than that — it's "I have a day job, I have limited night hours, and here's how the tooling has changed what I can actually accomplish in those hours."

This is the natural follow-up to [Signal Through the Noise](/2026/04/11/signal-through-the-noise/) and [Syntax Is the Least of Our Skills](/2026/04/11/syntax-is-the-least-of-our-skills/) — those posts were the *argument* for using AI. This one is the *implementation*. What it actually looks like when I sit down to work.

## Things to expand on

### The tools

- **[Superwhisper](https://superwhisper.com/)** — dictation tool for macOS. Hit a hotkey, talk, get text. I use this for almost all of my "ranting at Claude" inputs. It's why my prompts to Claude sound like stream-of-consciousness — because they *are*. I'm not typing them.
- **[Claude Code](https://docs.anthropic.com/en/docs/claude-code)** — terminal-based AI coding tool. Agentic, tool-using, reads my files, runs commands. My primary interface for actually writing code.
- **Parallel Claude Code sessions** — I run multiple instances at once, each in a different repo or working on a different task. One for Flux, one for LiquidMetal2D, one for the mobile app, maybe a fourth acting as a "learning partner" where I ask "what does this Rust line do" without polluting the coding context.
- **Git worktrees or isolated clones** for keeping the sessions' state clean.
- **A theme stack across editors:** Tokyo Night Storm everywhere — VS Code, iTerm2 (the [profile I built and shipped](https://github.com/mattcasanova/tokyo-night-storm-iterm-profile)), PHPStorm, Android Studio, Xcode. Visual consistency across contexts matters more than I expected.

### The workflow

Rough shape of a typical evening session:

1. **Brain dump into Superwhisper** to one of the Claude sessions. Usually a 2-3 minute monologue about what I want to do. "OK, tonight I want to build out the spatial grid for LiquidMetal2D, and I've been thinking about..."
2. **Claude turns the rant into a plan.** Not a full spec — just a rough ordering of what needs to happen and what the open questions are.
3. **I read the plan and push back.** This is the part that matters. Claude's plans are usually 80% right and 20% wrong in ways I can only catch because I've been doing this for 20 years. Pushing back is where the human contribution is.
4. **Code generation.** Claude writes the first draft. I review, I tweak, I flag the architecture-level stuff it got wrong.
5. **Run it. Look at it. Break things.** This is where the "bugs you only find by using the thing" surface. See the whole [Define 'Interesting'](/2026/04/13/define-interesting/) post.
6. **Rant about the results into Superwhisper** as a new message. Claude takes the rant, updates the plan, loops.
7. **Commit.** I commit manually because I like seeing exactly what's going in. Claude *could* do it, but I want the muscle memory and the oversight.
8. **Move to the next project** if the current one has a natural stopping point. Or dig deeper on the current one. Depends on energy.

### What actually changes with Superwhisper

This sounds minor but it's load-bearing: **I can talk faster than I can type, and ranting unlocks clarity that typing doesn't.** When I type, I self-censor in real time — I delete and rewrite sentences as I go. When I talk, I just keep going, and the ideas come out in the order they're arriving in my head. The resulting transcripts are messy but they capture the *thinking* better than typed input would.

Claude is good at handling messy input. I've noticed this with every session. I'll Superwhisper a 500-word monologue full of repetition and half-finished thoughts and Claude will extract the three points I was actually making. That's a capability that didn't exist in any meaningful form three years ago.

### The multi-project juggling

The other thing the tooling unlocks: **I can keep four projects going in parallel without losing the thread of any of them.** Each Claude session is basically a scoped memory — the context window holds the recent conversation, and the files on disk hold the deeper state. When I switch back to a project, I either re-read the recent conversation or ask Claude to summarize where we left off. It's faster than my own notes would be.

Before AI, I could maybe juggle two side projects at once, and the second one always slipped. Now I have Flux, LiquidMetal2D, the mobile apps, and a blog all moving forward in parallel, and I can pick any of them up on any given night and be productive.

### What it doesn't fix

- **The energy problem.** I still have to sit down and engage with the work. If I'm tired, I'm tired. No tool fixes that.
- **The "right direction" problem.** Claude can help me execute fast, but it can't tell me whether I'm building the right thing. That's still on me.
- **The review bottleneck.** I'm the only one reviewing the code. Claude can't catch the architectural mistakes Claude makes. That's why pushing back matters — see the [roll your own DI post](/2026/04/14/bad-advice-roll-your-own-di/) for what happens when I don't.

### The meta-observation

Most of this blog exists because of this workflow. [A ten-post series in five days](/) wouldn't have been possible five years ago — not because I didn't have the ideas, but because I couldn't have typed them all out between nights at work and weekends with the family. The writing is happening because I can rant into Superwhisper and have Claude shape it into something readable. I spend more of my time on *judgment* — is this the right framing, is this accurate, is this too long — and less of my time on *mechanics* — typing the paragraphs, looking up the Jekyll frontmatter syntax, remembering the Swift protocol syntax.

This is the "syntax is the least of our skills" thesis applied to writing, not just coding. The mechanical part is cheap now. The judgment is the work.

## Possible titles

- **"The Night Shift Workflow"** — fits the blog name, implies "here's how I actually work at night"
- **"Ranting into Superwhisper, Then Cleaning It Up"** — descriptive but maybe too long
- **"Four Projects at Once"** — the juggling angle
- **"The Tooling That Made This Blog Possible"** — meta angle
- **"How I Actually Work"** — direct, might be too generic

## Notes

- Not a "Confident Dose of Bad Advice" entry. This is a Neon & Noise standalone essay. Closer to *Signal Through the Noise* in tone — genuine, not opinionated-for-effect.
- Mention the actual workflow without being too "productivity-blogger" about it. The honesty is the point.
- Include a screenshot of a multi-session terminal setup if I can capture one that isn't proprietary.
- Link heavily to previous posts. The workflow post is a hub for the "how do I actually do this" argument that runs through the other posts.
