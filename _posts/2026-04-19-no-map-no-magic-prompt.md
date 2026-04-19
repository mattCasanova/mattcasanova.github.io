---
layout: post
title: "No Map. No Magic Prompt."
tags: [workflow, ai, claude-code, opinion, linchpin]
---

Most of what I've shipped this month happened between 9pm and midnight.

Day job. Wife. The math doesn't leave many hours. But in that window — plus some weekends, plus the occasional Vegas trip — I've moved a blog, a GPU terminal emulator, a 2D game engine, a web app, and the matching iOS and Android apps all forward in parallel.

This post started as a write-up of the tool stack that made that possible. Somewhere in the middle of writing it, it turned into the other thing too — the reason the tool stack matters. Both halves are here.

## The Three Pieces

**[Claude Code](https://docs.anthropic.com/en/docs/claude-code).** Anthropic's terminal-based agentic coding tool. It's the primary interface. It reads my files, runs commands, edits code, executes tests, uses git. It doesn't just answer questions about code — it works *in* my codebase.

**[Superwhisper](https://superwhisper.com/).** A macOS dictation tool. Hit a hotkey, talk, get text. I use it for basically every prompt I send to Claude. That's why my prompts read like stream-of-consciousness monologues. Because they *are*.

**Parallel Claude sessions.** I usually have three or four running at once, each in a different repo. One for Flux. One for LiquidMetal2D. One for the iOS and Android apps. One for the website behind them. Sometimes a fifth as a *learning partner* — a session where I can ask *"what is this Rust line actually doing"* without polluting my coding context.

That's the whole stack. Everything interesting about how I work falls out of those three pieces interacting.

## How We Got Here

A year ago this didn't work.

I was using GitHub Copilot on my side projects and my company's internal equivalent at work. Both were good. Around the start of 2025, Zuckerberg said AI agents would be coding about as well as mid-level devs by the end of the year. He wasn't wrong. Just about two months off.

Copilot circa April 2025 was a good autocomplete. If you knew exactly what you wanted, and the task was small, it could do it. *"Here's an enum. Make me an interface with a getter and setter for each variant."* That kind of thing. Nothing architectural. Nothing cross-file. But for tactical work it saved real time.

What it couldn't do: understand my repo. Make changes across files. Run the tests. Look at the output and adjust. It was a tool that lived inside the editor and wrote snippets. Useful. Not transformative.

I spent an evening sometime in mid-2025 trying to get Copilot to refactor one of my provisioning scripts. I'm a deploy-script nerd — I write server provisioning for fun — so this wasn't me asking the tool to invent something I didn't understand. I knew exactly what I wanted. Copilot didn't. Two or three hours of one-file-at-a-time suggestions later, nothing was deployable. That was the ceiling.

February 2026. That's when I finally started using Claude Code.

It had been out for a while — this isn't a launch story. Lucky for me, my job gives us access to the best AI tools. I just wasn't using Claude. People around me kept mentioning it, and every time I asked *"how is it actually different from Copilot?"* I got an answer that didn't quite land. Eventually I got tired of wondering and just tried it.

That was the shift.

The underlying models were essentially the same as what I'd been using. What changed was **tool use.** Claude Code doesn't just suggest text — it reads files, runs commands, edits code, checks output, iterates. It can see your whole project. It can see *other projects on your machine.*

That last part is the one that made everything click. All my personal projects — the iOS app, the web app that backs it, the deploy scripts, the provisioning — all live on my laptop. Which means Claude can see all of it. I can sit in the mobile repo and say, *"there's an API endpoint that's returning the wrong shape. Go check the website repo and figure out what it's supposed to be."* Claude just goes and does it. No copying. No pasting. No context-switching in my head.

That capability didn't exist in any real form a year ago. At least not that I knew about.

## Why Superwhisper Is the Real Multiplier

Even with Claude Code, I still had to type. And the bottleneck was typing.

I can talk much faster than I can type. Probably two or three times faster. Over the course of a single minute of ranting into Superwhisper, I can get out several hundred words. My prompts end up long — half a page, a full page, sometimes more. I go deep into architectural detail because *it costs me nothing* to go deep. The ideas arrive in the order they arrive in my head, and I just let them out.

Typing changes what you say. You edit as you go. You delete half a sentence and rewrite it. You lose the thread while fixing punctuation. Ranting keeps the signal — even if the signal is messy.

Claude is shockingly good at handling messy input. I'll dump a 500-word monologue full of repetition, tangents, and half-finished thoughts. Claude extracts the three real points I was making and turns them into a plan. That's the part that sounds like a productivity hack but is actually load-bearing: *I produce structure by ranting, and it comes out cleaner than when I try to type it.*

Superwhisper is also what makes the multi-session juggling work. It's not just that typing detailed prompts into four sessions in one evening is exhausting — though it is. It's that **typing drops me down into the mechanics every time.** Picking the words. Fixing the typos. Losing my place in what I was actually thinking about. Talking keeps me at the altitude where the architecture lives.

Quick yes-or-no answer to one session's question? Two seconds, move on. Switch to the next session, read the plan, push back, switch again. Deep dive when I need one, which happens plenty. But the default state is up there, above the mechanics, where the thinking is.

## The Actual Loop

Rough shape of a typical task:

1. **Rant into plan mode.** Open Claude Code, flip plan mode on, brain-dump what I want to do — the feature, the architecture I have in mind, the constraints, the open questions. One to five minutes of nonstop Superwhisper.
2. **Claude produces a plan.** Not a spec. A rough sequence of steps, with the open questions flagged.
3. **I read the plan. I push back.** This is where the human contribution lives. Claude's plans are usually 80% right and 20% wrong in ways I can only catch because I've been doing this for a long time. Architecture decisions. Naming. Premature abstractions. The stuff a senior engineer would flag in a design review.
4. **Claude executes.** Code gets written. Tests get written. Commands get run.
5. **I review — carefully.** Here's the trick that took me a while to learn: **I flip plan mode back on when I'm reviewing.** Because if I ask *"why did you do X?"* without plan mode, Claude assumes I'm asking Claude to *change* X and starts changing it. A lot of the time I don't want it changed. I just want to talk about it. Plan mode keeps the conversation open.
6. **Claude commits, but only when I've said go.** No auto-commit. Claude proposes, I check the diff, I approve, Claude executes. The mechanics are Claude's. The decision is mine.
7. **Move on.** Next step on this project, or switch to a different session.

Plan. Execute. Review. Commit. The loop is simple. The discipline is remembering that every step has its mode.

## Don't Take Yes for an Answer

This one's worth saying plainly: **don't take yes for an answer.**

Claude is confident. Claude can be wrong. Those two facts are uncorrelated. A confident answer and a correct answer look identical in the terminal. So I push back on everything — not to be a jerk, because *confidence isn't evidence.*

If Claude says *"no, that's the wrong approach, here's why"* — I push back, make Claude defend it. Sometimes Claude's right. Sometimes I am. Either way I need the argument.

If Claude agrees with me — that's where I'm *most* suspicious. Agreement is the cheapest thing an LLM can produce. *"Are you sure? Walk me through the tradeoffs."* I treat it the way I'd treat a junior engineer nodding in a design review — maybe they genuinely agree, maybe they're being polite, maybe they didn't follow, and you can't tell the difference until you press.

The posture: be skeptical of both answers. Disagreement? One of us is missing something. Agreement? Maybe one of us is *still* missing something, and we just missed it together.

## Architect, Driver, Mechanic

Here's the framing I use to keep this all straight.

**I'm the architect.** I decide what the thing is. What it does, how it's structured, what the layers are, what's behind an interface and what isn't. That's the work I've been doing for a long time and I can do it while I'm walking to the kitchen.

**I'm the driver.** I decide where we're going next. Which project gets tonight's hours. Which feature. Which rabbit hole is worth going down. Which one isn't.

**Claude is the mechanic.** Fast, tireless, surprisingly knowledgeable about things I haven't looked up in years. Doesn't complain about tests. Doesn't get bored of boilerplate. Writes the Swift, the Rust, the shell scripts, the SCSS. Once I've told it where and why.

The mechanic is excellent. The mechanic doesn't decide where we're going. Nobody's going to hand you a destination.

## Fundamentals Matter More, Not Less

Everything above works only because I know what good architecture looks like. When I rant a plan into Superwhisper, I'm ranting architecture — layer boundaries, dependency direction, what lives behind an interface and what doesn't, where state belongs, how testing hooks in. That's stuff I can do in one breath because I've been doing it long enough. Claude is excellent at implementing that architecture. Claude is *not* excellent at inventing it.

Anyone can prompt Claude to build an iOS app. Very few people can prompt Claude to build an iOS app with *good* architecture — because that requires you to know the architecture and be able to describe it. The app I'm building right now is structured the way it is — core library, services behind protocols, view models that don't know about networking — because I've [written about](/2026/04/14/bad-advice-roll-your-own-di/) [those](/2026/04/15/house-rules-the-other-four-letters/) [patterns](/2026/04/17/house-rules-whatever-happens-happens/) enough times to have them in muscle memory. I'm *explaining* them to Claude, not discovering them.

That's the "[syntax is the least of our skills](/2026/04/11/syntax-is-the-least-of-our-skills/)" thesis in practice. The mechanical part — writing the Swift, writing the Rust, looking up the Jekyll frontmatter syntax — got cheap. Judgment about what to build did not. Judgment got *more* valuable.

## There Is No Map

Seth Godin has a book called *[Linchpin](https://www.amazon.com/dp/B003NX762Y)*. It came out in 2010. The audiobook has been in my rotation ever since — I'm somewhere past my tenth listen, possibly my fifteenth. I started it again recently and was struck by how much it reads like it was written yesterday.

The thesis, briefly: the jobs that paid you to show up are going away. Factory jobs first. Middle-management jobs next. The industrial-era promise — *do what you're told, don't stand out, we'll take care of you* — is over. Godin's answer isn't *hustle harder.* It's *do art.* He uses "art" broadly — not just painting or writing, but any act of bringing something into the world that didn't exist before, because *you* decided it should.

He has a line in the book roughly to the effect of: *someone knows how to code Python better than you. Someone knows how to write copy better than you. Someone can do the mechanics better.*

He was talking in 2010 about other people.

In 2026, that someone is Claude.

What Godin meant was: competing on mechanics is a losing game. If the only thing you bring is *"I can do this thing"*, and *the thing* can now be done by anyone — or worse, anything — you've been commoditized. The only defensible position is judgment. Taste. The craft of deciding what's worth doing, and then putting your fingerprints on how it gets done.

That's the book's thesis, and it maps exactly onto what working with AI has turned out to be. I can't out-mechanic Claude. Claude writes Swift faster than I do. It writes Rust faster than I do — a language I don't know well. It never gets bored of boilerplate.

But Claude doesn't know what I want to build. Claude doesn't know what's *worth* building. That's the art. That's still on me.

There is no map. Not for this. Not for any of it. Nobody has the playbook for working with AI agents right now. Not me. Not the people I work with. Not the people writing the most-retweeted threads about it. Everything I'm describing in this post is something I'm making up as I go. Two months ago my workflow looked different. Two months from now it will again.

Consider the giants. Uncle Bob — Robert Martin, the guy who gave us SOLID, *Clean Code*, and *Clean Architecture* — and Grady Booch — one of the Three Amigos who invented UML, author of *Object-Oriented Analysis and Design with Applications* — are both still doing this work. They've been at it far longer than I have. They're still testing new tools, still publishing, still on top of the game. And they keep landing in almost opposite directions on the hard questions.

If *those* two can't agree on the shape of this, nobody can. You're allowed to be making it up as you go. Everybody is.

A couple of things that follow from that:

- **These tools are non-deterministic.** Two agents, same prompt, can produce different outputs. Sometimes materially different. That's not a bug — it's the nature of the tool. There's no *correct* prompt. There are prompts that work often, prompts that work sometimes, and prompts that don't. You test. You adjust.
- **There's no magic prompt.** No incantation that produces perfect code. No sequence of words that collapses ten messy documents into a coherent summary on the first try. No *"build me an iOS app with perfect architecture."* Real work still requires the work.
- **Ten programmers will write a program ten ways.** AI doesn't change that. There's no canonical best output, because there's no canonical best program. The right answer is the one that works for your constraints, your architecture, your team.

No map. You make one.

## Barrier Down, Bar Up

Here's the paradox I don't think enough people are sitting with.

AI lowered the barrier to entry for basically every creative and technical discipline *at the same time.* That's why this blog exists. It is genuinely easier for me to sit down after a day job and produce a 2,000-word post than it has ever been in my life. Rant the draft, push back on Claude's shape of it, edit to taste, ship it. The barrier dropped for writing. The barrier dropped for coding. For art, for music, for product design. Everyone knows this.

Here's what the same move did at the top.

**The bar for excellence went up by exactly as much as the barrier for entry came down.**

If anyone can make a Mario clone in a weekend, Mario clones aren't interesting anymore. If anyone can make a Halo clone with slightly different skins, neither is Halo with vampires or Halo with dinosaurs or Halo with whatever. If anyone can generate a Cancun resort review video — and I've watched the output, thousands of YouTube videos with AI voiceover and stock photos from the hotel — generic travel content is done. Slop. Filter it out.

What survives is what genuinely takes judgment. Taste. Craft. Something distinguishable from the noise.

Paint isn't slop. Paint is a medium. A kid dipping all their brushes in one jar produces brown mud. A painter with taste produces a painting. Same paint. The skill is what the paint *becomes.*

AI is paint.

If you bring no judgment, you get slop. If you bring no taste, you get generic. If you're building the same shape of thing everyone else is building with the same tool everyone else is using, you produce nothing worth distinguishing from the background.

The barrier is lower than it's ever been. The bar is higher than it's ever been. The gap between *anyone can make something* and *that something is worth making* has never been wider. The people who thrive are the ones who reach for the bar instead of settling at the barrier.

## What It Doesn't Fix

A short list, in the spirit of not overselling:

- **The direction problem.** Claude can help me *execute* fast. Claude cannot tell me whether I'm *building the right thing.* That's judgment. It's still mine.
- **The review bottleneck.** I'm the only reviewer. Claude can't reliably catch the architectural mistakes Claude makes.

## What It Does Do

To balance the ledger: this workflow is *energizing.* I'm more pumped about side projects than I've been in years. Not because Claude makes me less tired — it doesn't. Because the scope of what I can move forward in a single evening is new. Four projects, one laptop, a few hours. That's the hook. That's why I keep showing up at midnight.

## Do More Art

Mechanics got cheap. Taste got valuable. Art is the job.

Every night shift — the side projects, the blog, the quiet hours between 9pm and midnight — is an argument for the same position Godin made in 2010. Don't compete on mechanics. Compete on art. Bring taste. Bring judgment. Build something somebody else wouldn't have built, and build it in a way that has your fingerprints on it.

This blog has Claude's fingerprints on it too. I'm not hiding that. [The Nightcap](/2026/04/17/nightcap-fifteen-posts-deep/) admits it. [Saturday's post](/2026/04/18/flying-motorcycles-movable-buildings/) admits it. So does this one. Ranting at Claude and pushing back on Claude is exactly how I ship work now. But the *shape* of the thing — the choice of what to write about, the voice it comes out in, the willingness to say *"no, that's the wrong frame, try this one"* over and over until something lands — that is still mine. That's the whole argument. That's the *art* part of Godin's thesis.

There is no map.

Go make one.
