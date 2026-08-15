---
layout: post
title: "Tip the Kitchen"
tags: [architecture, product, tech-debt, opinion]
summary: "The product books are right: nobody buys your code. But the dichotomy is fake. There aren't product people and code people. There are two products, and one of them has been starving my whole career."
---

Think about the last restaurant you loved. What did you love about it? The burger. The room. The food came out fast and the server was funny.

Now tell me about the kitchen.

You can't. You've never seen it. You have never once, in your entire life, reviewed a kitchen.

Every restaurant is two operations wearing one name. Front of house sells you a night out.

Back of house is a factory with knives. Front of house gets the lighting, the playlist, the menu with the nice font. Back of house gets prep lists, health code, and standards that get followed at eight in the morning when no customer will ever know.

And here's the thing the restaurant industry figured out that mine still hasn't: **both of them are the product.**

That's tonight's rant. Every software company ships two products.

The features are one.

The code is the other.

And almost everybody is starving the second one, because nobody ever taught them it was a product at all.

### Concessions first

I've been reading the product shelf lately. Marketing, operations, product management. Right now it's *Escaping the Build Trap* by Melissa Perri, and before I start swinging, let me concede everything, so nobody mistakes this for a defense of code for code's sake.

The books are right. Talk to your customers. Solve real problems. Outcomes over outputs: don't ship features just to ship features, and, fair is fair, don't refactor just to refactor either.

Nobody buys your fucking code.

It doesn't matter if your game runs at 600 frames per second. If it's a shit game, nobody's gonna buy it.

The customer hires your product to do a job, and they cannot see your architecture and should not have to.

I concede all of it, and I'm not conceding from a distance. I've never been the product guy.

I've always been more Carmack than Romero, and given how Daikatana turned out, I've made my peace with that.

For years I quietly believed a programmer could just *do* the product job if they felt like it. Turns out it's a real set of skills, and I don't have them, which is why I'm reading the books and learning instead of arguing with them.

Well. Mostly instead.

### The Dichotomy is Fake

Because here's the line I've heard my whole career, and the books don't kill it, and it needs to die: *there are product people, and there are code people.*

No.

The line is real. It's just drawn between the wrong things. It's not two kinds of people. It's two products.

The first product is the one everybody means when they say product. Features, screens, the pricing page. Customers buy it. It makes the money. Nobody in any building has ever forgotten about it for a single afternoon.

The second product is the code. And the code is not a pile of output behind the product.

It *is* a product.

It has features: the things it can be made to do next. It has performance. It has a user experience, and if you don't believe me, ask whoever inherits yours. And it has a customer. The customer just isn't the person paying the invoice.

The customer of the code is the business.

I never considered myself a product person, but I was always a product person.

My product was the code.

API design, module boundaries, the seams where subsystems meet: that's product design, for the second customer.

My features are tests, and code that can actually be tested. Mockable repositories. Switching from `UserDefaults` to something more robust in a single afternoon. Swapping out the networking layer without rewriting the whole app. Adding a new device without rewriting the whole app. Adding a new screen without rewriting the whole app.

My customer signed the paychecks.

And my customer doesn't always realize what they bought. Sometimes they'd rather skip those features and pay for sloppy code, just to get the next feature out the door.

Maybe I've just been bad at selling it.

It was the best of products. It was the worst of products. Same company. Same repo.

### Nobody wrote down the outcomes

So why does the second product starve? Not villains. Start with what it's selling.

The second product's value is counterfactual.

That's a five-dollar word for a simple thing: its wins are the things that never happen.

You cannot measure the bugs you don't have. Nobody leaves a five-star review that says "didn't get food poisoning again."

A product whose benefits are invisible, and whose costs arrive late and two or three steps removed from the decision that caused them, will lose to a product with a realtime usage dashboard every single time, no matter how smart everyone in the room is.

I have personally been very smart in that room. The second product lost anyway.

The other half is framing. The build trap the book describes is measuring success by what you shipped instead of what changed. Outputs instead of outcomes. And the second product falls into the trap from the *other* side: because nobody ever states its outcomes, every hour spent on it reads as output for output's sake. The refactor looks like engineers polishing brass. ["The code is bad" is not an argument](/2026/06/09/the-code-is-bad-is-not-an-argument/), and I've already told you that. This is the other half: the argument nobody makes *for* it.

So let's state the outcomes. They're not subtle.

**Defect rate.** Your customer doesn't care about your code, directly. They care enormously, indirectly, the day you ship a bug because the code was never testable enough to have proper coverage.

**Time from bug to fix.** A bug found by a test at dev time costs minutes. A bug found at PR time, when the automation catches somebody else's change breaking your code, costs approximately zero: it never shipped, nobody paged, no customer ever knew. A bug found by a customer three weeks later costs a support thread, a war room, and a review you don't want. Same bug. The architecture decides which price you pay.

**Time to the next feature.** "Why is the second widget taking months? I thought we already built one." If you've never heard that sentence, you haven't been doing this long.

**Cost of a pivot.** Things change. That is the one guarantee this industry has ever honored. The whole point of architecture is that when the change arrives, it costs a week and not a quarter.

I've been saying that last one slightly wrong for months, even on this blog.

Architecture is about responding to change.

It is true, and it's fucking engineer-speak, and engineer-speak is Klingon to the people holding the budget.

Say it to sell: the next feature ships faster. The bug dies at PR time instead of in production. The pivot doesn't need a rewrite. Those are outcomes. That's a product with a feature list, and its customer is the business, and it deserves a roadmap like anything else you sell.

And if you want the tell for how companies actually price that product, here's the cleanest one I ever saw. It hits harder knowing where it came from: the top.

Director level. Leadership looked at the test coverage number and didn't like it. They asked, apparently baffled, why so many engineers weren't writing tests. And the answer was never that engineers didn't want to write them. There was just no room on any timeline to do it.

So a mandate came down: spend ten percent of your time on tests.

Okay. Did a single deadline move ten percent?

No. Not one date on one roadmap.

The mandate was real. The budget was zero. And when the ten percent has to come out of a schedule that was already too tight, the tests lose to the date every single time, which produces exactly the coverage number the directors were staring at when this whole thing started. The room asking why the kitchen is dirty is the room that scheduled no time to clean it.

### The Super Mega Light problem

You've met [my Govee app](/2026/04/14/bad-advice-delete-your-singletons/) before: the imaginary-but-barely app for the color-changing light strip. Build it the honest way first products get built: one device, one screen, ship it. Great. That's not a mistake; a speculative platform for devices nobody sells yet would be its own build trap.

Now the business does what businesses do when the first product works. They want the light bar. The neon panel. The Super Mega Light. A whole family of cool neon lights.

And engineering says the sentence I have heard more times than I can count, in rooms I'm not going to describe: *"we literally can't."* Because product one was never built to be flexible. And what you actually need now is a platform. And the deadline for the platform is not "when it fucking works."

So the team charges anyway.

As the Klingons say: Heghlu'meH QaQ jajvam.[^goodday] The platform gets built in a rush, underneath a live product, while money rolls in and everybody knowingly writes bugs, because everything is changing at once and there's no clean answer to questions as basic as who's eligible for what.

Nobody in that story is dumb. The people who built product one built the right thing. The people who demanded more lights were doing their jobs.[^lights] The bugs weren't sloppiness; they were inevitable, at that speed. The failure happened earlier and quieter: the second product's outcomes were never on anyone's menu, so nobody priced the flexibility until the day it was missing.

Six more weeks of foundation was never *rejected*. It was never asked.

### The rat

Which brings us Back to the kitchen.

What the customers buy: the burger, the room, the speed, the funny server. What they're also buying, without ever knowing it: *no rats.* Nobody has ever five-starred the rat they didn't see. The clean kitchen is invisible right up until the moment a rat crosses the dining room floor, and from that moment your reviews will mention nothing else, forever.

And the standards were never only about the rat. A clean, organized kitchen with real prep is *why the food is fast.* If every burger meant washing the lettuce, slicing the tomato, and grinding the beef to order, the burger would take two hours, and delicious-but-two-hours is a restaurant nobody visits twice. The prep is the speed. Testable architecture is prep.

Restaurants know all of this. That's why the kitchen has a health code, a prep budget, and somebody with a clipboard who doesn't care about your ambiance. Because if a restaurant measured nothing but customer satisfaction, the kitchen would be invisible too, right up until it was the only thing anybody could see.

Software is that restaurant. No inspector. No prep budget. And every number anybody tracks is a dining-room number.

### Tip the kitchen

Knuth called it the art of computer programming, and he wasn't wrong, and *that argument loses.* "Good code is good" sounds like craftsmen asking for craft time, and it polls exactly as well as you'd expect with people holding a roadmap.

The argument that wins is the one the product books handed me while I was busy yelling at them: outcomes. Fewer bugs. Faster features. Bugs that die at PR time for free. Pivots that don't need rewrites. Not engineering vanity. A second product, with its own features and its own customer, competing for budget in the same language as everything else on the menu.

I'm going to keep reading the books, because the first product deserves better than I've historically given it.

Customers, needs, strategy: I'm in. But I'm done letting anyone pretend there's only one product in the building.

Two products. One of them has had everyone's full attention forever. The other one is the reason the first one shows up hot, fast, and rat-free.

Put the second product on the menu.

And tip the kitchen.

Qapla'.[^qapla]

See you, space cowboy.

[^goodday]: Today is a good day to die.

[^lights]: I know. I bought eight of them.

[^qapla]: Success.
