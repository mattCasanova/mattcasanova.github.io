---
layout: post
title: "Nobody Points at the Blade"
tags: [ai, architecture, testing, career, opinion]
summary: "Uncle Bob said AI buys you time to test harder than you've ever tested, and the replies lined up to tell a man with sixty years at the forge that he doesn't understand steel. Watch what none of them do: point at the code."
---

In 1993 I was ten, hooking two of my dad's spare computers together with modems and a regular phone line so a friend and I could shoot each other in Doom.

My dad was an IT guy, so there were always extra machines in the house. I'd been taking them apart and putting them back together since I was old enough to lose the screws. I was comfortable in DOS because in 1993 everybody was comfortable in DOS. None of that was special. The pattern was the special part: every year there was some new thing to hook up, and hooking it up was the fun.

By fourteen it was Diablo, and a buddy and I started a clan: UBM. Underground Battle.net Mafia. We were fourteen; the name was the least of it. Characters saved locally in those days, which made Battle.net an honor system, and we had no honor. Hacked scrolls, edited inventories, invite a stranger to your party and hit him with an invisible Nova the second the mobs closed in. We killed people and collected their ears.

Stupid. Glorious.

The clan needed a page, so I built one on GeoCities.[^neighborhood] HTML isn't programming, it's markup. I knew that even then. But it was the first time I wrote something down and other people used it, and I never really stopped. I built pages for ten years before I ever touched JavaScript.

That's the resume of a kid, not an engineer. I'm laying it out because of what it shows: the tools have been changing under me since before I could drive. They never stopped.

### The engine guy

DigiPen, 2006. Programming for real this time.

Freshman year I wrote a text game and could feel something wrong with it that I didn't have words for yet. Code repeated. Nothing bent without breaking. Sophomore year we built a game from scratch and I ended up writing the engine. Junior year, engine again. I kept landing in that seat, and I kept being fascinated by the difference between a codebase that absorbs change and one that shatters.

So I read everything. Every game architecture book I could order, then everything adjacent to it. *Agile Software Development.* *Clean Code.* *The Clean Coder.* Eventually the whole shelf, Uncle Bob and plenty of others. And I went back and reread them, because I knew the first pass hadn't gotten everything. I still assume that. The examples are in Java and I don't write Java, and it matters not at all, because the lesson was never the language.

### The ladder out of the language

Here's what I told students for seven years: pick one language and get good at it, because the fundamentals have to live somewhere. Then watch what happens as the years stack up.

Every language hands you roughly the same tools. Loops. Arrays. Some flavor of encapsulation. Interfaces. Once you're comfortable, you stop thinking about the tools and start thinking about the deck you're building. Then the toolbox grows: design patterns, which are just names for moves you'd eventually derive anyway ([one spoon](/2026/07/23/bad-advice-there-is-no-spoon/), you know the rest). Then principles above the patterns. SOLID is five of them, and [the answer is interfaces](/2026/05/06/house-rules-d/), but the point is the ladder: language, tools, patterns, principles. Every rung up is more abstract. Every rung up transfers further.

Which is why languages stopped mattering to me somewhere around the fifth one. C, C++, C#, Java, Objective-C before I ever taught a class. Swift, Kotlin, TypeScript, PHP since. Struggling in Kotlin because you've never written Kotlin? Work in it for a couple of days. That problem solves itself.

The rung that never transfers on its own, the one you have to carry on purpose, is the top one. The architecture. The shape that decides whether year three of the system costs a week per feature or a quarter per feature.

Hold that thought.

### The roof in Okinawa

There's a scene in *Kill Bill* where the Bride climbs to a roof in Okinawa to see Hattori Hanzō. The greatest swordsmith alive, retired, hiding above his own sushi bar. She needs what only he can make. The scene works because of what nobody in it says: when the man with a lifetime at the forge hands you his finest work, you do not lecture him about steel.

This week Uncle Bob [posted](https://x.com/unclebobmartin/status/2081332683582427641):

> AI agents can write code many times faster than a human. What this means is that you, the programmer, have a large amount of time to use those agents to write unit tests, to write acceptance tests, to write property tests, to torture test, to mutate test, to QA test, and to otherwise ensure that the code meets its functional and quality requirements. And even after spending all that time, you will still be many times more productive than a human programmer, and the result will be better.

Then read the replies.

People piling in like he's some confused old man who wandered onto the internet. Not the guy who wrote his first program in the sixties. Not the author of half the architecture shelf I just described. Sixty years at the forge, and the comment section is explaining steel to him.

And here's the tell. Nobody points at the blade.

Nobody links a repo, a file, a function, and says: here, this is bad, this test proves nothing, this abstraction leaks. Nobody sends the drink back. They argue with his principles. They argue with his tone. They argue with the *idea* of him. The work sits right there, public, and no one touches it.

He isn't even claiming the machine is safe. He says the opposite: it's fast and it's hard to trust, which is exactly why you spend the surplus torturing its output. That's not a man asleep at the wheel. That's a man describing the wheel.

I run a whole [series of confident bad advice](/series/bad-advice/) on this blog. None of it will ever be "ignore the guy with sixty years at the forge." He's not automatically right. Nobody is. But the benefit of the doubt is earned in decades, and arguing back should come with receipts.

### The same move, twenty years apart

Back in the C days we cranked the compiler all the way up. `-Wall -Wextra -ansi -pedantic -Werror`. A warning is the compiler telling you "I'm guessing what you meant here." A guess is a coin flip with a bug on the other side. So you promoted every guess to an error, and the build refused to exist until there was nothing left to guess about.

Today that same instinct is linters and static analysis and warnings-as-errors on every target I own, cranked to eleven, which is [exactly what I spent the last two weeks doing](/2026/07/22/the-rate-went-up/). New tools. Same move. Twenty years apart.

That's the whole shape of this business. The way we work changes constantly: phone lines to Battle.net, gcc flags to static analysis, me typing to me reading what the machine typed. The work underneath moves much slower, and the most important work hasn't moved at all. Build the thing so it survives being changed. Prove it still works after every change.

I've sat in interviews where a candidate's whole app lived in the view model, because that's what every tutorial shows. And it ships. Three screens, it genuinely ships. Then it needs its own API, and [error reporting](/2026/07/11/carry-that-weight/) to somewhere real because you can't read your users' logs, and a fourth screen, and a fifth, and now the tutorial architecture is the most expensive decision anyone made. The language was never the hard part. The architecture is the part you live with for years.

That isn't my line. That's the entire shelf of his books. It didn't stop being true when the typing got fast.

### Nine days back

Because the payload of that quote isn't the list of test flavors. It's the time.

The machine writes code ten times faster than me. Twenty on a good night. So the feature that used to cost seven days costs one, and the honest question is what happens to the other six.

I could have another beer. I was doing that anyway.

The wrong answer is ten times the features. That's the same silent tradeoff we've always made, the skipped test, the "we'll harden it later," except now it compounds at machine speed. The deadline pressure didn't go away; stakeholders still name a date without asking. Pushing back is still the job.

Bob's answer is the list: unit tests, acceptance tests, property tests, torture the thing. And the last item is the one I keep circling. QA. You are your own QA now.

We always should have been. Everybody's pulled a teammate's change that crashed on the happy path. First click, straight down. That crash means the author never once ran their own code, and I've watched it happen everywhere I've worked. There was always an excuse, and the excuse was always the calendar.

The calendar just gave you six days back.

The gaps while the agents run are [exactly QA-session sized](/2026/07/29/serve-it-neat/). The feature that took a day instead of seven can afford an hour of you actually using it. Poke the edges. Kill the network. Type garbage in the form. Be the user the spec never imagined. Call it an hour against six days and it's still the cheapest quality you will ever buy.

### Last call

Thirty-three years since Doom over a phone line. Every layer of the stack has been swapped out from under me at least twice. The work is still the same work: architecture that holds, proof that it holds, and somebody with taste actually using the thing before a customer does.

The old swordsmith says the new forge finally gives you time to test the steel properly. Maybe you think he's wrong. Fine. That argument has exactly one legitimate form.

Point at the blade.

See you, space cowboy.

[^neighborhood]: Times Square, I want to say. It's been thirty years, so if it was actually a different neighborhood, forgive me. The ears were real.
