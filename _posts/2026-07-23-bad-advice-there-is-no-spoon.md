---
layout: post
title: "A Confident Dose of Bad Advice: There Is No Spoon"
series: bad-advice
tags: [architecture, design-patterns, learning, opinion]
summary: "I had a whole Matrix bit built for this one and it went cold on the shelf. Here's the rant that was underneath it: Strategy, State, and Command are three names for one move, and the move is interfaces."
---

I had a whole thing built for this post.

The kid in *The Matrix*. Bald, robes, bending a spoon with his fuckin mind.

> *Do not try and bend the spoon. That's impossible. Instead, only try to realize the truth: there is no spoon.*

And then the turn. Sitting at your desk asking *do I need Strategy here, or State, or Command* is trying to bend the spoon. Stop. Realize the truth. There is no spoon, there are no patterns, and the whole catalog dissolves in your hands.

I was pretty pleased with that.

Then I let it sit for a month, and when I came back the whole thing had gone cold. Read it again and it's a guy who found a movie quote that rhymes with his opinion, dressing up something ordinary in robes so it'll seem deep.

So I'm doing the annoying version instead. I'm going to tell you the bit, tell you the bit is bullshit, and then tell you the part underneath that I still believe. Because I do still believe it. That's why the quote fit in the first place.

## The advice

Here's tonight's bad advice: **stop memorizing design patterns.**

Not stop *studying* them. I want to be unusually clear about that for this series, because people skim and this one deserves the caveat up front: I am not telling you patterns are stupid. I am not telling you they're a waste of your time.

They aren't.

Same as when I ran the whole [SOLID series](/2026/05/06/house-rules-d/) and never once said don't bother learning SOLID.

I'm telling you the flashcard version is the wrong end of the thing.

And I'm allowed to say it. I taught this material for seven years, to freshmen and sophomores, at the university level. I [co-wrote a patterns book](https://www.amazon.com/Game-Development-Patterns-Best-Practices-ebook/dp/B01MRP7SPA), which meant reading every patterns book I could get my hands on, Gang of Four cover to cover included. This isn't a guy who skipped the reading and made peace with it.

It's a guy who did the reading, wrote some of it, and kept walking.

## Where it came from

I was writing the [House Rules post on Strategy](/2026/06/19/house-rules-two-drink-minimum/). The polite version. Careful, straight, here-is-precisely-how-Strategy-differs-from-State.

Every word of it is true and I stand behind it. But the whole time I was writing it I kept thinking the same thing:

This is goofy.

Memorizing this is goofy.

So let me take three of them apart and show you why.

## Strategy

A family of interchangeable algorithms behind one interface, selected from outside.

Fine. Now define *algorithm.*

Is it one function? A comparator you hand to a sort, sure, that's clean, that's the textbook picture and it makes perfect sense. But mine has two methods. Three. At what point does an algorithm stop being an algorithm and start being an object that happens to sit behind an interface?

It doesn't, is the answer. It was always an object behind an interface. Your widget takes in an interface, and how it does its job is abstracted away.

[Today's other post](/2026/07/23/kill-the-lights/) was a whole essay about passing something in. My connectivity monitor, behind a protocol, with a real one and a fake one, so a UI test could tell my app the network was down when it wasn't.

That's Strategy. Two interchangeable implementations behind a seam, chosen from outside. The "algorithm" is *ask Apple* or *lie to me.*

I never once thought the word Strategy while building it, and nobody reading that post would have called it that.

## State

Your widget can move between multiple states.

Okay. What's a state? What are the actions?

Abstracted. Same as everything else.

Structurally it's the identical diagram: an interface, some implementations, a thing holding one of them. The entire difference the textbook spends two chapters on is one bit of information.

Who pulls the swap. Outside is Strategy. Itself is State.

Two chapters. One bit.

## Command

Command is Strategy, except you don't call it yet.

That's the pattern. Pack the data and the algorithm together into one encapsulated object. The thing holding it doesn't know what data it's acting on or what it's about to do. Pass it around. Call it whenever you want.

Now keep them. In a list. In order.

Can you do the reverse action in the same command? Congratulations, you built undo.

And now you've got a genuinely great game editor: a stack of actions the user can walk backward and forward through, and none of the editor code knows what any of them do.

I want to stop here for a second, because this is exactly where "there are no patterns" gets somebody in trouble.

That's clever. Somebody had to notice that deferring a call buys you a history. That's a real invention and the day you need do-and-undo you should go read how Command handles it, because they thought about it harder than you're going to on a Tuesday night.

Learning that is not the waste. Filing it under a name you can recite is.

## Nobody asks you to list them

Twice on this blog I've mentioned, with some amount of pride, that I can rattle off all five SOLID principles.

Nobody has ever asked anybody to list all the design patterns. There are too many. There are whole *categories* of them.

And you don't need the list, because for most of them you can derive the behavior without ever knowing the name. Which tells you the name was never the asset.

The thing underneath has a name, though, and it isn't a pattern. It's the D.

[Dependency inversion](/2026/05/06/house-rules-d/). Don't code to concrete types. Don't code to concrete algorithms. Code to abstractions.

Design patterns are the dependency inversion principle wearing twenty-three hats. Some of those hats took a very smart person to invent. It's still one head.

Fine: Singleton isn't an interface. Singleton is also the one I told you to delete, in a book fifteen years ago and [again in April](/2026/04/14/bad-advice-delete-your-singletons/).

At least I'm consistent.

## The toolbox

Here's how I put it to students.

Learn arrays. Learn loops. Learn encapsulation. Then learn polymorphism, which in practice means learn interfaces.

That's the toolbox. It's a small toolbox and it's an enormous field, and both of those are true at the same time.

The mistake is treating interfaces like the specialty tool in the back of the drawer, the one you take out for a Design Pattern Occasion. It isn't that. It's the one in your hand all day, every day. Code to an interface and you're eighty percent of the way to whichever pattern somebody was going to name at you.

## Two questions

So if you're not reaching for a pattern, what do you actually do?

Two questions. I asked both of them in the other post today without noticing they were the same question.

*Is this thing going to get swapped out?* Then it needs to be an interface.

*Do I need to control what this thing says, so I can test around it?* Then it needs to be an interface.

One of those is about the future. One is about your test suite. They feel nothing alike.

Same answer, every single time.

## Last call

Study them. Learn the clever tricks, because they are clever and somebody smarter than both of us worked them out. Then stop counting them.

Build the seam where the change is coming, and let somebody walk past later and tell you which pattern you used.

There is no spoon.

Fine. There's one spoon. Interfaces.

BANG
