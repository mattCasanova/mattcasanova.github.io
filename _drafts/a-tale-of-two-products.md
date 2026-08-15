---
layout: post
title: "A Tale of Two Products"
tags: [architecture, product, tech-debt, opinion]
summary: "Every software company ships two products: the one the customers buy, and the one the business lives in. The product books are right that nobody buys your code. Here's the half of the picture I think they're missing."
---

I used to throw away pizzas for a living.

Part of the living, anyway. Decades ago, as a teenager, I worked at a Godfather's Pizza with a five-dollar buffet. The rule: a pizza could sit on the buffet for thirty minutes. After that it came down, no matter what it looked like, no matter who hadn't shown up to eat it. And that wasn't our rule. It came down from somewhere above the franchise: health code, some Washington food-service regulation. I couldn't quote you the statute. Didn't matter. It got followed. Some mornings you'd put out a full spread at opening, nobody would walk in for half an hour, and you'd stand there sliding eight or nine whole pizzas into the trash.

It sucked. It also wasn't a mistake. The morning prep list said to make more pizzas than we expected to sell, because we knew some were going in the garbage. The standard cost money, the operation planned for the cost, and nobody asked the buffet to hit the same pizza count while also throwing pizzas away.

Hold that thought. It's the whole post.

### Arguing with an audiobook

I've been reading product books lately. Marketing, operations, product management, the whole shelf I spent twenty years not reading because I was busy on the technical side. Right now it's *Escaping the Build Trap* by Melissa Perri, and about thirty-five minutes into the audiobook I started arguing with it out loud, on a run, like a crazy person.

Here's the thing though: I was arguing because it's right. Right about the exact thing engineers hate hearing. Nobody buys your code. The customer hires your product to do a job: fix their back, get them in shape, split the bill. Nobody wants a drill; they want the hole. The paying customer cannot see your architecture and should not have to. I concede all of it. No strawman, no "well, actually."

I'm mid-book and mid-learning, so this is not a post written from a mountaintop. It's a guy yelling at his headphones. But the argument I was yelling deserves writing down, because I've been living it for twenty years.

### The second product

Every software company ships two products.

The first one is the customer product. Features, screens, the thing on the pricing page. Everybody in the building can see it, and everybody agrees it matters, because it's the thing that makes money.

The second one is the code. And the code is a product too. It has features (what it can be made to do next), it has performance, it has a user experience, and it has a customer. The customer just isn't the person paying the invoice. The customer of the code is the business itself.

(You could argue the code is the *first* product. It exists before the customer product does, and everything the customer will ever get ships through it.)

People love the line "there are product people and there are programmers." I believed a version of it for years, and it's wrong. I never considered myself a product person, but I was always a product person.

My product was the code.

API design, module boundaries, the seams where subsystems meet: that's product design for the second customer.

When Carmack built the Doom engine, and then the Quake engine, the engine *was* the product. Every feature the engine could do was something the level designers could use, which made the levels more interesting, which is what the customers actually reviewed. The chain ran straight from the second product to the first one. It always does. It's no accident that the product people we canonize came up next to the code: Gates wrote the BASIC, Zuckerberg wrote the first Facebook, Carmack wrote the engines.

It was the best of products. It was the worst of products. And they shipped from the same building.

### Why the second product always loses

So if both products matter, why does one get all the attention?

Because the second product's value is counterfactual.

That's a five-dollar word for a simple thing: its wins are the things that never happen.

And value made of things that never happen loses every prioritization fight ever held.

You cannot measure the bugs you don't have. Nobody leaves a five-star review that says "didn't get food poisoning again." A product whose benefits are invisible, and whose costs arrive late and two or three steps removed from the decision that caused them, will lose to a product with a dashboard every single time, no matter how smart everyone in the room is.

I have personally been very smart in that room. The second product lost anyway.

This isn't a villain story. It's structural.

And the incentives encode it. Here's the tell, and I've watched it at more than one company: leadership looks at the test coverage number, doesn't like it, and mandates that engineers spend ten percent of their time writing tests. Fine. Does the deadline move ten percent?

No. The deadline does not move.

That's the whole answer, told in one meeting. The feature has a date, the quality is priced at zero, and the same layer of the company asking why coverage is bad is the layer setting the deadlines that guarantee it stays bad.

I've lived the sharp end of it. Two tracks once landed on my desk with deadlines about a week apart: a big redesign already in flight, plus a feature somebody up high wanted and nobody else could take. I said I could do both, and I did, on twelve-to-fourteen-hour days. Tests were not in the requirements, so tests did not get written. That's not how I wanted to build it. That's what fit in the budget I was handed. When I told a manager we didn't have time to write tests, the answer was "you just write them at the same time," from someone who had been an engineer once and had apparently forgotten how code actually gets written: you make it work first, then you architect it better, and if the second pass isn't budgeted, it doesn't exist.

Here's the part that should end the argument. That crunch was before Claude. AI has since made tests and quality work dramatically cheaper. So did the budget for them finally appear? No. The deadlines moved up instead, because "you can get more done now." The capacity gains got absorbed and the quality line item is still zero, which tells you the zero was never about cost. And the machine doesn't hand you the second product for free anyway. The architecture in my apps looks the way it does because [I spent months forcing it to](/2026/07/11/carry-that-weight/). Left alone, the model ships the tutorial architecture. The typing got cheap. The judgment didn't.

### The bill, two steps removed

Two patterns I carry around, names filed off.

I watched a mature single product, about a year of work by the people before me, get a directive from up high: we want a second one, maybe a third. The honest engineering answer was "we literally can't." The first product hadn't been written to scale into a family. So it got rushed into becoming a platform underneath a live product, and for a stretch everyone knew we were writing bugs, because questions like "which users are even eligible for which product" had no clean answer while everything changed at once. Money rolled in the whole time.

Was it wrong? No. It shipped, it made money, it's still going. Was it *right*? Nobody can tell you, and that's the actual indictment. The real decision was never "rush versus another year." At the margin it was "rush versus six more weeks to avoid locking in the fragile shape," and that question never got asked, because the people deciding could see only one of the two products. Bugs you announce up front are interest payments everybody agreed to. [The same bugs arriving as silent tradeoffs](/2026/05/05/bad-advice-agile-is-right-you-are-wrong/) get blamed on the engineers. The blame is the wrong part.

Second pattern, smaller, same disease. I once inherited a system that had passed through many hands over a couple of years, with nobody left to ask about it, and found four flags in it, maybe six depending on how you counted, that all did essentially the same thing. Each one was a reasonable person under deadline pressure who needed to check something and couldn't afford the afternoon it would take to find out the check already existed. Nobody wrote that on purpose. The second product just had no owner, so everyone treated it like a hallway.

### The kitchen

Back to the restaurant, because "customers buy the pizza" goes further than it looks.

Customers buy the pizza, the atmosphere, the food showing up hot and fast. Nobody orders the prep. But look closer at what's on the menu invisibly: *no rats.* Nobody picks a restaurant because it has no rats. They just never come back after seeing one. You cannot run a dirty kitchen until a rat crosses the dining room and then start cleaning. The clean kitchen has to be the standard the whole time, precisely so that moment never happens.

And the prep isn't just safety. It's speed. If every burger meant washing the lettuce, slicing the tomato, and grinding the beef to order, the burger would take two hours, and delicious-but-two-hours is a restaurant nobody visits twice. Prep is why the food comes out fast. Testable architecture is prep.

Now here's where the analogy breaks, and the break is the point. Restaurants *budget* the kitchen. Health standards aren't aspirations there; they're regulations, built in, priced in, enforced by somebody with a clipboard. The Godfather's prep list said make more pizzas than we'll sell, because the thirty-minute rule was handed down, it cost money, and the business planned for the cost. Software is the industry that runs a restaurant where only the dining room appears on the books.

### Put it on the menu

So what do you do with this, besides be mad? The product books handed me the answer, which is the funny part.

Their whole argument is outcomes over outputs: stop measuring success by what you shipped, start measuring by what changed. Okay. "Zero defects" is an outcome. Walk the chain: fewer production bugs come from more tests, more tests come from testable code, testable code comes from architecture somebody was given the time to do right. "Build this feature with no bugs" is not that. It's an output command wearing outcome clothes, naming no mechanism and budgeting nothing, which is why it always resolves to "ship the feature, skip the tests."

Knuth called it the art of computer programming, and he's right, and the art is not the sell. The sell is the job the business hired the code for: make the next feature cheap, make the shipped feature actually done. A feature with bugs in it isn't a done feature. A feature the architecture can't deliver isn't a done feature. [I've already told you "the code is bad" is not an argument](/2026/06/09/the-code-is-bad-is-not-an-argument/). This is what the argument is. Not "give us refactor time because we're craftsmen." Name the outcome, price the work, and put the second product's outcomes on the same menu as every feature, where they can win or lose a fair fight.

Because the balance between the two products was never supposed to be a vibe. It's a budget. The pizza place knew that. It made extra pizzas on purpose and threw them away on schedule, and the buffet was better *because* the garbage can was part of the plan.

Dickens opened his tale with the best of times and the worst of times, one city gleaming and one starving, and the joke of this industry is that we run both at once: the best of times out front where the customers are, the worst of times in the kitchen. What that split does to the people who build the second product is its own story, and I'll tell it another night.

Two products. One menu.

See you, space cowboy.
