---
layout: post
title: "A Confident Dose of Bad Advice: Agile Is Right, You Are Wrong"
series: bad-advice
tags: [opinion, agile, scrum, process, project-management]
---

A guy at the bar orders a Manhattan. I'm three whiskies in. I hear *"I'm a manager."*

This is his problem now.

I turn. He's getting his drink. He's wearing the kind of quarter-zip that means he flew in for an offsite. There's a slack notification on his phone. I lean over.

**"Pull up a stool, friend. Let me tell you why your projects keep slipping."**

He says he didn't say that. I tell him it doesn't matter. The bartender refills my glass. We're doing this.

## The whole post in one line

**Agile is right. You are wrong.**

Or: you refuse to move the deadline when the shit changes, and you act surprised when the project is fucked.

That's it. That's the whole rant. Everything that follows is me, a stranger at a bar, explaining why you — yes, you, the manager-shaped person nursing a Manhattan — keep doing the same thing every quarter and getting the same result every quarter. I am about to be very confident about it. The advice will be bad. You should listen anyway.

## The guy who invented this told you in 1993

Scrum was invented by Jeff Sutherland in **1993.** The Agile Manifesto was written in **2001.** It is **2026.** We have had thirty-three years — *thirty-three years* — to read the books, internalize the principles, and stop doing the thing the fucking experts told us not to do.

You have not stopped doing the thing.

I am going to quote Sutherland's own book at you. Paragraph one. The man who *invented Scrum*, in the preface of his own book about Scrum, tells you exactly what doesn't work:

> The process was slow, unpredictable, and often never resulted in a product that people wanted or would pay to buy. Delays of months or even years were endemic to the process. The early step-by-step plans, laid out in comforting detail in Gantt charts, reassured management that we were in control of the development process — but almost without fail, we would fall quickly behind schedule and disastrously over budget.

That was the first paragraph!!!

Chapter one, in case you thought the first paragraph was a fluke:

> When you hear what happened, it sounds at first as if it makes sense: the people
at Lockheed sat down before they bid on the contract, looked at the requirements, and
started planning how to build a system that would do all that. They had lots of
intelligent people working for months, figuring out what needed to be done. Then they
spent more months planning how to do it. They produced beautiful charts with
everything that needed to be accomplished and the time it would take to complete each
and every task. Then, with careful color selection, they showed each piece of the
project cascading down to the next like a waterfall. These charts are called Gantt charts, after Henry Gantt, who developed them. With the advent of personal computers in the 1980s making it easy to create these intricate charts — and to make them really complex...

The inventor of Scrum spends the opening of his own book telling you that **Gantt charts don't work.**

That's not me making shit up.

That's him.

The Preface. Chapter one.

Block-quoted, right there, in the book on the shelf next to the desk of every project manager who's ever badged into a sprint planning meeting.

Now look at your screen. Is there a Gantt chart open on it?

There is, isn't there.

## "But we have deadlines"

You're going to say this. Every manager-shaped person at every bar I've ever ranted at has said this. Well — first they say *"maybe you've had one too many."* Then, after that, they say:

*"Dude, real businesses have deadlines. Sales has contracts. Investors want dates. Regulators don't care about your sprint."*

Fair. All true. I am not, despite the 12 shots, denying that calendars exist.

Let me tell you about a bridge.

Imagine you walk up to a bridge engineer and you say: *"I need a bridge across this river. It needs to hold trucks. I need it in six months."* And you don't ask them how long a bridge takes. You just tell them the date.

The engineer has two options.

**Option A:** they tell you the truth. *"Bridges take longer than six months. You can have the bridge, or you can have six months. You can't have both."* Now you negotiate. Now you have a real conversation. Now you maybe ship a smaller bridge in six months and the real bridge in eighteen, or you ship the real bridge in eighteen and stop pretending you ever had six. **This is what agile is.**

**Option B:** they nod. They build whatever fits in six months. The bridge goes up on time. People drive over it. Some of them die. **This is what your project is.**

"But dude, software's not bridges. Nobody dies." Right.

Just your project. Just every quarter.

Just forever.

## Software isn't bridges (and that's bad news for you)

Here's the thing the bridge guys actually have over you, friend. **A bridge gets built and stops needing decisions.** It has a foundation. The foundation has to hold a known load. That load was decided in advance — *trucks, cars, pedestrians, this much weight, this much wind* — and once the bridge is built, the load doesn't change. They poured the concrete. They went home.

Your software is not like that.

Your software is going to get a new requirement at 4pm on Friday for the rest of its life.

Look — I'm not a real architect. I'm no Art Vandelay. I'm no Ted Mosby. If a real bridge architect comes in and tells me a bridge gets a new lane every six months and a tollbooth retrofitted next year and a pedestrian deck bolted on after that, fine, I'm wrong, send me the textbook. But the bridges I drive over don't seem to wake up one Tuesday and decide they want to cross a different river. Your codebase does that. Your codebase gets told, in month nineteen, that point B isn't important anymore, point B is now point E, and by the way the customer wants to pay in a currency we haven't integrated. **The bridge has to hold a known load forever. Your foundation has to hold a load you haven't invented yet.**

That's a *harder* engineering problem. Not an easier one. The construction-industry word for *"we'll figure out the load later"* is **"the bridge fell down."**

So when somebody at your company says *"we don't have time for architecture right now, we just need to ship the feature,"* what they're actually saying is: *"we'd like the bridge built before we decide what it carries."* Sure. You can do that. The bill comes later.

## You don't have a deadline. You have a tradeoff you're not saying out loud.

Your deadlines are loud. They're on the slide. They're in the OKR. They're in the subject line of the email and the title of the calendar invite. Deadlines are never the silent part.

**The tradeoffs are.**

Every hack. Every quick fix. Every module that shipped without a test. Every concrete class that should have been an interface. Every `// TODO: refactor this later`. Those are silent — *probably* silent — tradeoffs somebody made on the project's behalf. And every one of them is going to affect a deadline. Maybe not *this* deadline. *Some* deadline. The bill is coming. You just don't know whose name is on it yet.

You said: *I need this by Friday.*

You did not say: *I need this by Friday, and to get there we are going to skip the integration tests, hardcode the config, copy the function instead of refactoring it, and absorb a bug-tail of unknown length over the next two quarters.*

But that's what *I need this by Friday* actually meant. Friday wasn't the decision. The decision was *"swap correctness and maintainability for speed and pretend it's free."* You just didn't say it out loud, so nobody got to disagree, and now the team owns a tradeoff they were never invited to make.

Here is the rule I will yell at you about until they cut me off:

**It almost doesn't matter what the tradeoff is, as long as somebody says it out loud and everybody agrees.**

You want to ship fast and skip tests because the prototype is going in the trash in three months? **Fine. Genuinely, completely fine.** That is a *valid* tradeoff. I will help you make it. I will write that prototype with you and I will not write a single test and I will not feel bad about it, because we agreed.

What's *not* fine is when *"we'll skip the tests this once"* gets said, implicitly, every sprint, forever, until the throwaway prototype is in production for nine years and nobody can remember why the auth flow has three feature flags pointing at each other.

Tradeoffs are honest. Silent tradeoffs are the bill you don't get to read until it's overdue.

## If you're a stakeholder, you must care about architecture

You think your job is features. You think the engineering team's job is *make the features go.* You think architecture is some private garden the engineers want to weed because they're perfectionists.

Wrong.

**If you are a stakeholder in a software product, you must care about architecture. Not because it's elegant. Because it's load-bearing for the features you haven't asked for yet.**

The features you can name today are the easy part. The hard part is the next year of features — the ones marketing hasn't dreamed up, the ones sales hasn't promised yet, the ones a competitor will force you to build in a panic in Q3. Every one of those features will land on top of the foundation you let your team build (or not build) this year. Good foundation, the next feature is a week. Bad foundation, the next feature is a quarter, and three other features break.

The features are the visible part. The architecture is the part that decides *whether next year's features are cheap or expensive.* You don't get to skip it just because it doesn't show up in a demo.

***It is literally the foundation of the demo.***

I worked a four-month sprint to a hard deadline once where we were cutting corners the entire way. The team knew it. I knew it. Every time I flagged it, my line was the same: *"fine, but after launch we have to spend time re-architecting this, because we are racking up debt right now and we will pay for it."* Launch came. The refactor didn't. Stakeholders wanted the next feature. Then the next. Then the next. For *two years.*

Two years of bugs landing in code we already knew was wrong, fixed by engineers who kept being told *next sprint we'll get to the cleanup.* We never got to the cleanup. Eventually the stakeholders ordered a full UI rewrite for unrelated reasons, and the worst of that code got thrown away as a side effect of a project that wasn't even about fixing it. The cleanup I'd been asking for, for two years, finally happened — *by accident.*

That's not vindication. That's an accident. The two years between *we should refactor this* and *we deleted it anyway* were paid in real engineering hours by real humans working around code that should've been fixed before launch.

A friend of mine worked at a video game company that had **three identical math libraries** in the codebase. Three. Same vector math, same matrix ops, same answer to the same question. Which one should you use? *Doesn't matter.* They were all the same. Three engineers got paid, on three different occasions, to write the same thing — because nobody on the receiving end of those features cared whether the foundation already had the brick. *That's* what *"we don't have time for architecture"* actually buys you. Not speed. Triplicate.

## "But Matt, this is the age of AI"

I can hear you already, three Manhattans in. *"Sure, Matt, but we're in the age of AI now. The AI writes the code. Who cares about the architecture?"*

Oh, friend.

It is *2026.* This is not the *"AI will write a function for me"* era. This is the *"AI just opened forty pull requests overnight"* era. Your team is shipping more code than they can read. Every single line of it lands on top of the foundation you let them build (or not build) last year.

If your foundation was good, the AI is a force multiplier and you're shipping like a god.

If your foundation was bad, the AI is an *AI-slop multiplier* and you're shipping like a god *into a swamp.* The bad architecture compounds faster, not slower. The duplicate functions arrive faster. The undocumented coupling arrives faster. The *we'll refactor it later* tickets arrive faster. The interest on your tech debt now compounds *daily,* because that's how often the AI commits.

AI didn't fix software engineering. It put the gas pedal on whatever you already had.

If what you already had was a mess, congratulations. You're now making the mess at mach speed.

## Fast, cheap, good — pick one

You know the old line. *"Fast, cheap, good — pick two."* This is the version that gets laser-printed onto the poster outside the conference room.

I am here to tell you the poster is a lie.

It's *pick one.*

And it's never the one you wanted.

You picked fast. You think you picked fast and good. You did not pick fast and good. You picked fast, the team cut corners, and now you have fast and *almost-good-looking-from-a-distance,* which in eighteen months will reveal itself to have been *neither fast nor good* once you count the rewrite. The cheap one was a lie too — the cheapness was a loan. The interest came due. You just hadn't opened the bill yet.

The companies who pretend to pick all three end up with one of them. And not the one they meant.

## What you call "agile"

You have a fixed deadline.

You have a fixed scope.

You have a fifteen-minute standup at 9:45 where everyone says what they did yesterday and what they're doing today and whether they're "blocked," a word you've trained them to never actually use.

**That is not agile. That is waterfall with standups.**

The whole point of agile — the entire reason the manifesto exists, the whole load-bearing idea behind every page of Sutherland's book — is that **when reality changes, the plan moves.** That's the deal. Requirements come in late? *Welcome them.* That's literally a principle, written down, by the people who invented this. *"Welcome changing requirements, even late in development."* It's right there.

But your plan doesn't move. The PM gets a new requirement on Tuesday of week two and slides it into the sprint. Nothing comes out. The sprint is now larger. The deadline is the same. The team is the same. The math has changed and you haven't.

**The slip happens in week three. You find out in month eighteen.** *Same time next quarter.*

So you're not doing agile.

You bought the t-shirt. You did not read the book.

## Devs don't write bad code for fun

I have never — *never* — met a developer who wanted to write hacky code. I've met developers who *did* write hacky code. Lots of them. I am one of them. I have written code so ugly I wanted to put a comment on top apologizing to the next person who'd open the file. *I'm sorry. I know this is bad. Please don't hate me. The deadline was Friday.*

Here's the thing. **The reason is never "I felt like it."** The reason is always *"the deadline I didn't set is on Friday and the requirements changed on Wednesday and nothing came out of the sprint when the new thing went in."* Devs don't choose hacky. Hacky is what's left when you take the time away.

Show me a codebase full of hacks and I'll show you a roadmap full of dates nobody on the team agreed to.

## The estimates are lying to you (and so are the devs)

You're going to ask the next reasonable question. *"Okay, so I'll ask the devs. I'll ask them how long it'll take."*

Don't.

**Devs lie. Estimates are always wrong.**

Not because devs are dishonest. Because *nobody is good at estimating software they haven't written yet.* Some things you think will take two weeks take an afternoon. Some things you think will take an afternoon take six weeks because you discovered, on Tuesday, that the auth library you're using has a bug that nobody's filed and the workaround requires rewriting your session layer. You could not have known. The dev could not have known. The estimate was a guess wearing a tie.

I'll go further. Last week I asked an AI how long a refactor would take. It said four to five days. *I had already written the code.* It took five minutes. Even the robot can't estimate. The robot has read every line of code on GitHub and it still can't estimate. This should tell you something.

The honest answer isn't *"trust the estimate."*

It's **stop estimating six months out.**

Ask: what can you ship in two weeks? Then watch what they ship. Then ask again. That's it. That's the loop. That's what Scrum *is.* Two weeks, ship something, measure, adjust. The thirty-three-year-old book on your shelf is full of this. You're welcome to read it. The bartender will hold your seat.

## Hey dev — this stool is for you too

The manager pays his check. He shakes his head. He leaves. He's going to land at noon tomorrow and add a thing to a sprint without taking anything out and find out in month eighteen.

I look down the bar.

There's a dev sitting four stools down, hood up, nursing a Modelo. *Hey. Yeah, you. Pull up.*

Because I have been pointing this rant at the guy with the Manhattan, and that's only half the story. **I am a developer. I have been one for twenty years.** The bad advice on this stool is for everyone, and the engineers in this room are not innocent.

Your job is *not* "take the ticket and ship the ticket." That's the junior version, and that's fine when you *are* a junior — you're allowed to be learning the game. But the moment you have any seniority on you, your job description grows a new line: **surface the tradeoff.** Out loud. In writing. In the planning meeting. In Slack. Not in your head.

Your PM asks: *"how long?"* You say *"two weeks."* You know — you *know* — that two weeks means no integration tests, an interface you should have abstracted, a config value you're going to hardcode, and a function that should have been three. You don't say any of that out loud. You give the date and you pocket the tradeoff. **The deadline is loud. The hack is silent. Yes — that's also you.**

Your job is to make the tradeoff loud.

*"Two weeks if we skip the integration tests. Three weeks if we don't."*

*"I can ship this Friday, but the architecture in module X is now committed to a shape that will cost us a quarter to back out of when the next feature hits the roadmap."*

*"You can have this fast. You can have it good. Pick one and I will deliver it. Don't pick, and I will guess, and you will not like my guess."*

Nobody around you will love hearing this. The PM will give you a look. The manager will say *"can you just figure it out?"* Tough. Push it up the chain anyway. **That's the gig.** You wrote the code, you know where the bodies are buried, and the people above you on the org chart genuinely do not know unless you tell them. And — to be clear — they're not the bad guy here. They want to ship a good product as much as you do. They don't have your information. **Give them the information.**

If the manager has read the book and the dev hasn't surfaced the tradeoff, the manager *still* ships a broken bridge.

Speaking of bridges.

Go back to the bridge for a second, and this time pretend *you're* the engineer. The boss tells you the bridge needs to hold a hundred thousand pounds. (Is that a lot? I don't know. Ask Mr. Mosby.) You start laying it out and you realize — given the timeline, the budget, the materials he's given you — what you're actually about to build will hold ten thousand. Maybe.

Should they tell their boss?

**I hope so.**
