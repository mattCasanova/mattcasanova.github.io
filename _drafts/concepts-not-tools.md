---
layout: post
title: "Concepts, Not Tools"
tags: [teaching, career, opinion, digipen, learning]
---

> **PRE-DRAFT SEED — not ready to publish.** This is a placeholder started from content that originally lived in [A Confident Dose of Bad Advice: Just Roll Your Own DI](). Expand later with more examples from teaching and from my own career.

---

## The seed (pulled from the DI post)

*From the DigiPen philosophy section in the DI post:*

> I taught at DigiPen for seven years. One of the pedagogical choices DigiPen made that I always loved was that students learned to build their own game engines and their own renderers instead of starting on Unity or Unreal. People outside the school would sometimes ask why — "isn't that teaching an obsolete skill? Everyone uses Unity in industry." The answer is: **Unity is a tool, and tools change.** In ten years Unity might be dead and some new engine might dominate. If you taught students Unity, you'd have to re-teach them the new tool every time. But if you taught them the *concepts* — game loops, vertex buffers, instanced rendering, sprite batching, collision detection, component systems — then they could pick up any engine in a week because they already understood what was under the hood.

## Things to expand on

The thesis is: **learn the concepts, not the tools.** Tools change on a five-to-ten-year cycle. Concepts don't. If you understand *why* the tool exists and *what it's doing under the hood*, you can pick up any new tool in the category in a week. If you only know the tool, you have to re-learn every time the industry shifts.

Examples and receipts I can pull into this post later:

- **Game engines (DigiPen).** Students built their own engines before touching Unity. When Unity → Unreal → Godot churn happens, they adapt. When a new engine emerges in 2030, they adapt. If you'd taught them "how to use Unity" they'd be fine until Unity 7 breaks everything.
- **Dependency injection (from the DI post).** Rolling your own once teaches you the concept. Then Dagger, Hilt, Koin, Factory, Swinject are all just "fancier versions of the factory class you already wrote." See [the DI post](/drafts/bad-advice-roll-your-own-di/).
- **Networking libraries.** I rolled my own API layer in 2019 (pre-Retrofit for me). Then I used Retrofit for a while. Now I'm back to rolling my own for the current project. Why? Because I understand what Retrofit is doing under the hood, so the tradeoff becomes "do I want the library's convenience or the control of rolling it myself" — not "I don't know how to make HTTP calls without Retrofit."
- **UI frameworks.** SwiftUI / Jetpack Compose / React / Vue all share a lot of underlying concepts (unidirectional data flow, declarative rendering, diffing). Learn the concepts once (ideally from React, which is where they landed first) and the rest are variations.
- **Build systems.** CMake / Cargo / Gradle / npm / Bazel all share the concept of "describe your dependencies, let the tool resolve and compile in order." Learn dependency resolution once.
- **GPU rendering abstractions.** I've used OpenGL, DirectX, and Metal. When I started Flux, I picked wgpu — a new-to-me abstraction. Took a week to be productive because the concepts port. See [88 Miles Per Second](/2026/04/12/88-miles-per-second/).

## The meta-argument

There's an AI-era twist to this that I should address in the expanded version: **AI makes the cost of "learning a new tool" much lower than it used to be**, which could seem like it weakens the argument. "Why learn concepts when AI can write the framework code for you?"

The rebuttal: AI is great at translating between tools *if you already know the concepts*. If you ask Claude "write me a Hilt module for this service" and you already understand what dependency injection is, you'll get a great answer. If you ask the same question and you don't understand DI at all, Claude gives you an answer that compiles but you can't evaluate it, can't debug it, can't extend it. Tools without concepts is the failure mode. AI + concepts is the fast path.

This is basically a restatement of the thesis in [Syntax Is the Least of Our Skills](/2026/04/11/syntax-is-the-least-of-our-skills/) — AI handles the mechanical translation; you still need to know the shape of what you're asking for.

## Possible titles

- **"Concepts, Not Tools"** — clean, direct, matches the DigiPen pedagogy I'm citing
- **"The Tool Is Not the Skill"** — same argument, different frame
- **"The Half-Life of a Framework"** — time-scale framing
- **"What DigiPen Taught Me About Unity"** — personal framing, leads with the institution
- **"Why I Still Roll My Own"** — ties it to the recurring theme across my posts

## Possible angle

This one isn't bad advice — it's actual advice, closer to [Syntax Is the Least of Our Skills](/2026/04/11/syntax-is-the-least-of-our-skills/) in tone. Probably a Neon & Noise standalone essay, not a series entry.

## Notes

- The original DigiPen paragraph stays in the DI post. This pre-draft is the expanded version.
- Include more examples from my time as a lecturer — maybe the specific curriculum choices at DigiPen that reinforced this, the arguments I had with students who wanted to "just use Unity."
- Address the AI-era pushback up front, because a reader will immediately want to know if this argument still holds when frameworks are one prompt away.
