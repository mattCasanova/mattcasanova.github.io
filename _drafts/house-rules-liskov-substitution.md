---
layout: post
title: "House Rules: L (Liskov Substitution)"
series: house-rules
tags: [architecture, solid, opinion]
status: seed
---

## Pre-draft seed — not a post yet

### The hook

Before I got my job at Meta, I interviewed at another FAANG company. The interviewer was senior — team lead, running something big. At some point he asked me about SOLID. The way he said it, I could tell he didn't expect me to know. It had that gotcha energy — the tone of a question you ask because you know nobody gets it right.

I rattled them off. Again. He didn't seem thrilled about that.

Here's the thing — Liskov Substitution sounds scary and it isn't. It just has a person's name on it instead of a description. "Single Responsibility" tells you what it is. "Open-Closed" tells you what it is. "Liskov Substitution" tells you that someone named Liskov had an opinion about something. That's why interviewers weaponize it. It sounds academic. It sounds like textbook trivia. It's not.

### The principle

**If S is a subtype of T, then objects of type T should be replaceable with objects of type S without breaking the program.**

In plain terms: if your code expects an interface, every implementation of that interface should actually behave the way the interface promises. No surprises. No "well this one throws an exception the others don't." No "this implementation ignores that parameter."

### Connection to the arc

- S post: "one reason to change" — the foundation
- O post: (TBD)
- **L post: "every implementation honors the contract"** — the gotcha that isn't a gotcha
- I post: "no client should depend on methods it doesn't use" — the god menu / Detekt suppression story
- D post: already covered in the Bad Advice arc

### The angle that makes this not a textbook post

LSP is the one most people think is abstract and academic. The post should make it concrete and visceral. Real code examples where someone violated LSP and it broke something at runtime that the compiler couldn't catch.

### Potential title ideas

- House Rules: The L That Sounds Scarier Than It Is
- House Rules: Liskov Isn't a Gotcha
- House Rules: The One With the Person's Name
- House Rules: A FAANG Interviewer Tried to Stump Me With This One
