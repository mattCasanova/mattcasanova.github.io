---
layout: post
title: "Don't Be Afraid to Try New Things"
tags: [career, opinion, language, learning]
---

> **PRE-DRAFT SEED — not ready to publish.** This is a placeholder started from content that originally lived in [A Confident Dose of Bad Advice: Just Roll Your Own DI](). Expand later with more examples from my own career.

---

## The seed (pulled from the DI post)

*From the V-Shred story in the DI post:*

> Quick aside, because it fits the theme of this post: I had never shipped Kotlin in production before. Google had recently made it their preferred Android language, but most of my previous Android experience was Java. I walked into the VP of Engineering's office and told him "I'm writing this in Kotlin, not Java, because Kotlin is better and if we start in Java we'll end up migrating later anyway." He shrugged, said "fine, you own it," and that was the whole conversation.

## Things to expand on

The pattern that's worth teasing out here: **when you're starting something fresh, don't default to the thing you already know.** Default to the thing you think is actually right, even if it means a little extra learning along the way.

Examples from my own career I can pull into this post later:

- **Kotlin at V-Shred (2019).** Greenfield Android project, picked Kotlin over Java despite never shipping Kotlin in production before. Google had marked it preferred; I bet on it. It paid off — we never had to migrate anything, and Java is basically done for Android now.
- **Swift + Metal for LiquidMetal2D (2020).** Picked Metal directly instead of using SpriteKit or Unity. Why? Because I wanted to actually understand the rendering pipeline, not have it handed to me pre-chewed. See also: [88 Miles Per Second](/2026/04/12/88-miles-per-second/) for how that paid off when I applied the same pattern to Flux.
- **Rust + wgpu for Flux (2026).** Never shipped Rust in production before starting Flux. Knew C and C++ deeply from teaching, figured Rust would translate. It mostly has. See [This Is Heavy](/2026/04/10/building-a-gpu-terminal-in-rust/).
- **Android (pre-Android Studio, ~2014).** Started building an Android game in Eclipse with C++ (wrong direction in retrospect, but the point is I tried it).
- **iOS (2010).** First iOS game, [Sync-Ball](https://apps.apple.com/ca/app/sync-ball/id356277312), when the App Store was basically brand new and there weren't many "how to ship an iOS game" tutorials.

The thread through all of this: the new thing is always intimidating, and the safe choice is always the thing you already know. But the safe choice ages. The new thing you spent a week learning ten years ago is now the thing you know.

## Possible titles

- **"Don't Be Afraid to Try New Things"** — clean, direct
- **"The Safe Choice Is the One That Ages"** — punchier, argument-first
- **"Kotlin, Metal, Rust, and Whatever Comes Next"** — list-of-bets framing
- **"A Week of Discomfort, Ten Years of Payoff"** — time-scale framing

## Possible angle

This could be its own "A Confident Dose of Bad Advice" post — "take the risky language choice for new projects." Or it could be a Neon & Noise standalone essay about career growth. Or both — the series version is the punchy take, the standalone is the longer-form personal story.

## Notes

- The original Kotlin paragraph stays in the DI post as the V-Shred aside. This pre-draft is the expanded version with more examples and a thesis.
- Make sure the closing argument is *not* "always pick the newest shiny thing." That's its own trap. The argument is more like "don't let unfamiliarity be the reason you default to the safer choice."

---

## Negative counterpoint to add — the engineer who refused to learn

(Carried over from the [three-post-seed](three-post-seed.md) Vein 2 rant. Reserved for *this* post specifically — do not double-spend in [Make It a Good One](make-it-a-good-one.md), which uses the friends-pattern abstractly instead.)

The flip side of *don't be afraid to try new things* is the engineer who never tried, ever. Four years on paper that's actually one year of experience repeated four times. The post needs at least one of these to make the positive case land.

### The Vegas iOS guy (~2017)

- First industry job after teaching. Took over an iOS project mid-development from someone who quit on a Friday — never-shipped product, in-flight code.
- The previous iOS dev had **4 years of iOS experience on paper.** Looked good on a resume.
- The codebase was using a **deprecated alert pop-up API** (specific name TBD — Matt to look up; UIAlertView vs UIAlertController is the candidate).
- Xcode (the IDE) was *literally telling him* — deprecation warnings, every build.
- He didn't care. Kept using what he knew. Wouldn't Google the new pattern.
- The deprecated calls were **littered through the codebase, not even wrapped in a single helper.** So when Matt fixed it, he had to fix it everywhere.
- This is the exact archetype of *not adaptive*: the IDE is screaming and you ignore it because Googling the new thing feels like more work than copy-pasting the old thing.

### Why this story matters for this post

- *Don't be afraid to try new things* sounds like motivational fluff without a counterpoint.
- The iOS guy is what *not trying* looks like. Frozen in mid-2010s iOS patterns despite the IDE warning him about it for years.
- The contrast makes the thesis sharp:
  - **Matt:** picked Kotlin / Metal / Rust *despite* never shipping in production. Got better. Got the next job.
  - **iOS guy:** kept using the deprecated API he learned in year one. Didn't get better. Quit before he got fired.
- Same period. Same industry. *Different posture.*

### Structure suggestion

Two ways to arrange the post:

1. **Negative-first:** Open with the iOS-guy story as the cautionary opener. Pivot to *"so when I started LiquidMetal2D / V-Shred Kotlin / Flux Rust, I made a different choice."* Walk through the positive bets. Close with *"the safe choice is the one that ages."*
2. **Positive-first:** Open with Matt's first scary bet (Kotlin at V-Shred). Use it as proof-of-concept. *Then* pivot — *"and here's what the alternative looks like"* — to the iOS guy. Either order works.

Recommendation: **negative-first** is punchier. The reader meets the warning before the lesson, and the thesis ("don't be the iOS guy") lands sharper.

---

## Self-cite map (don't double-spend across the AI/career arc)

- [Make It a Good One](make-it-a-good-one.md) — uses the *friends-not-adapting pattern* (anonymized). This post owns the *Vegas iOS-guy* specific anecdote.
- [Think, McFly, Think](think-mcfly-think.md) — agency / learning loop. This post is the *language-bet courage* version of the same thesis. Different angle, complementary.
