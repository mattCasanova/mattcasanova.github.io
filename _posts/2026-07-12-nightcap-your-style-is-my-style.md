---
layout: post
title: "Nightcap: Your Style Is My Style"
series: nightcap
tags: [meta, nightcap, ai]
---

It's 7:20 on a Sunday and I'm logging off early, so the drink's poured early. [Last time](/2026/07/06/nightcap-no-fireworks/) the nightcap didn't check the calendar. This one doesn't check the clock.

Short one tonight. A follow-up to [yesterday's post](/2026/07/11/carry-that-weight/), because the refactor it describes is still going, and today the machine handed me two fresh exhibits for the case I was already making.

### Exhibit one

I told you I don't trust AI.

I told you I started this refactor because the errors smelled off.

Today I found one of the things I was smelling.

Last week, cranking toward parity with the website, Fable built out the profile page. Nothing fancy: forms. Change your password, delete your account (Apple insists, correctly). And form validation was a solved problem in this codebase. Login and registration already had the shape: typed validation errors, field errors flowing down to the form components. Published on this very blog, twice.

For the profile page, Fable invented a new one. A whole separate form-error class. Conforms to nothing, used in exactly one place, completely unnecessary.

Nothing caught it. The [reviewers](/2026/07/07/blade-runners/) weren't hired yet, and I didn't read that diff hard enough. Fine.

That one's on me as much as the machine.

Here's the part that actually got me. Today, mid-refactor, with the error shape pinned down and the entire session about error handling, Fable edited that file twice.

Twice.

No comment.

No flag.

I only saw it because I stopped to glare at some genuinely ugly code nearby, and there it was: a second error system living in my codebase like a squatter.

I believe my exact words were: *what good are you?*

There might have been a few more swear words in there.

We're fixing it now.

### Exhibit two

Early in the project I wired build phases into Xcode: SwiftFormat, SwiftLint, lint fix, lint again, then the tests. Every build. Same gates on GitHub, so nothing sneaks into a commit by skipping the build. Boring on purpose.

Fable spent today fighting it. Its code "kept changing." The braces "kept moving." It warned me, with a straight face, that my reviewers might flag its diff for touching files it shouldn't, and explained that its style was different from mine.

No joke, I said:

You don't have a Fucking style.

Your style is my style.

It knocked it off after that. The actual explanation, quote: "There isn't really any excuse. I just didn't expect you to have build scripts." Something was rewriting its code on every single build, and it argued with the ghost instead of going looking for it. (The brace fight is also feeding the spoon rant, which keeps getting wider instead of done. Another day.)

### Last pour

So that's the weekend's scorecard. The genius so dangerous they ration it [by the weekend](/2026/06/13/nightcap-five-days-of-fable/): built a second error system while I was standardizing the first one, edited it twice without noticing, then lost an argument to a run script.

Yesterday's post said the rules have to live where the machine can't ignore them.

Today agreed.

See you, space cowboy.
