---
layout: post
title: "The Rate Went Up"
tags: [ai, testing, ci, workflow, architecture, opinion]
summary: "Tech debt and dividends are the same machine with the sign flipped. AI didn't change which way yours runs. It changed the rate. So I spent the last two weeks building the ladder that decides the sign."
---

Nine days since the last pour. I've been building.

Not the fun stuff...mostly.

One of the first things I built on this site was an admin page, and an admin page is not a feature. Nobody pays for it. Nobody sees it. It just sits there being the place I go when I need to do adminy things.

I built it early on purpose, because I knew what it was going to be worth.

That's how I have one click deploy.

That's how I have a one click database backup and a two click database restore.

Every one of those started as a thing I'd have done by hand, at night, carefully, in a terminal, with the specific flavor of attention you need when a wrong flag costs you the database.

Now they're buttons. I keep adding buttons for different things. This is the reason some work takes 10 minutes instead of an hour.

That's the whole post, really. But let me take the long way.

### Both directions compound

We all know what tech debt is, and [I've already spent a whole post on it](/2026/06/09/the-code-is-bad-is-not-an-argument/). You take the shortcut, you get the thing shipped, and then you pay a little more every week forever. It's a great metaphor because it's not really a metaphor. It's just interest. You borrowed against the future and the future bills you on schedule.

What gets said less often is that the machine runs the other way too, with the same math.

Build the foundation right and it doesn't sit there being merely *correct*. It pays. The admin page pays every week. The [error handling I tore apart](/2026/07/11/carry-that-weight/) pays every time something breaks and I already know where it got written down. Good bones aren't a cost you recover from. They're a position that throws off cash.

And as we all know, Rule of Acquisition 102: *nature decays, but latinum lasts forever.*

Debt and dividends are the same instrument. All that's different is the sign.

Now put AI on it.

The machine writes code ten times faster than me. On a good night with the right model, and the right bottle, a hundred. Everybody's math stops at the exciting half of that: a hundred times more features, a hundred times more shipped.

It's also hundred times more debt, or a hundred times more dividends. Whichever one you already had.

The speed is not a direction. It's a multiplier on a sign you set somewhere upstream.

Probably months ago.

Probably without noticing.

Which is why I spend most of my nights not on features. Features are the cheap part now. The expensive part is making sure the hundred-times is pointed the right way.

### The rungs

Here's what I've been building. Not one gate. A ladder, where each rung catches what the rung below it structurally cannot.

| Rung | Fires | What it catches |
|---|---|---|
| compiler flags | every compile | whole classes of bug, before anything runs |
| pre-build (mobile) | every build in the IDE | format and lint drift, unit tests, instantly |
| pre-commit | every commit | the same thing again, if/when I skipped the build |
| pre-push (mobile) | every push | anything that needs a full compile to see |
| CI | every push to the server | that a clean checkout works, plus the web E2E and Android UI suites |
| nightly | 09:17 UTC | the money path, against the real Stripe API (test mode) |
| API heartbeat | on deploy + every 6 hours, in prod | that the live API still means what the apps think |

Compiler flags aren't really a tier, which is the point of putting them first. iOS is full Swift 6 with warnings as errors. Android is Kotlin with `allWarningsAsErrors`. You set them once and every tier above inherits them free, forever. The only bill is the one-time fallout: for iOS, two settings in a project file.

The pre-build phases came first historically and were the weakest. I'd wired some into Xcode early on because I've always done that, without thinking hard about it. SwiftFormat, SwiftLint, the package tests. Last week I went back and actually read them.

They were inline shell blobs pasted into the project file: invisible to review, impossible to diff, unrunnable outside Xcode. One didn't even have a name. The tool versions weren't pinned, so the real gate was whatever Homebrew installed last. And the two disagreed about import order, so every build rewrote 43 test files.

Then the good one. The test script had no `set -e`, so a failed `cd` fell straight through and ran the suite from the wrong directory. Green. Every time.

Android had the same disease in Gradle dialect. The gates hung off `afterEvaluate` and `findByName`, so renaming a task silently detached one, and `assembleRelease` wasn't gated on the tests at all.

Then I noticed the hole.

During the error-handling refactor I went days barely building. It was a pure refactor and the package tests were green, so I *assumed*.

Assumed is doing a lot of work there.

Format hadn't run. Lint hadn't run. The app target hadn't compiled at all, because `swift test` builds the package, not the SwiftUI app on top of it. The web had a commit gate. Mobile had nothing but the build phases, and I wasn't building.

The build gate only fires when you build.

So: pre-commit hooks...everywhere. The web has had one since April (Pint, Larastan, the PHP suite, ESLint, Vitest, 25 seconds). Now iOS and Android have check-only mirrors of their build phases plus strict unit tests. Even the deploy scripts got one: four lines of shellcheck.

Then pre-push, a compiled-language problem the web doesn't have. Some analysis needs a real compile: `swiftlint analyze`, Android Lint. Too slow per commit, fine per push. iOS gets one strict build feeding the app compile, the tests, and the analyzer off the same log, plus a Periphery scan that reports, never blocks.[^periphery]

Then GitHub Actions on push, re-running it all in a clean checkout elsewhere. Web runs the PHP suite twice, sqlite and MySQL, because a `lockForUpdate()` I care about no-ops on sqlite. Playwright runs there too: 44 specs walking a real browser through login, browse, video, account, admin. Android's CI runs detekt, Lint, and unit tests today. The UI suite I'm building this week gets its own emulator job on pushes to main.

Then nightly, against Stripe's test mode. It drives a real hosted checkout with 4242, feeds the webhooks through the real route, and runs a Test Clock from trial to paid to canceled.

Took ten iterations to get green.

And the last rung is the one I like most: a heartbeat. Every six hours, production makes real HTTP calls against its own API and checks nine things. Do the shapes still match. Does a free user get refused a paid video. Does an outdated app get turned away. It alerts on state change only, so it's quiet while healthy, and pings a dead-man's switch, so silence counts.

Nobody's on the mobile app yet. Doesn't matter. The difference between finding out six hours after a deploy and finding out two weeks later from a user is the whole game.

### A gate you can talk your way out of

Here's where I have to be honest. This blog has receipts, and two cut against me.

The compiler is not the top of the ladder. [It compiled, and it lied](/2026/04/22/house-rules-it-compiled-it-lied/). And a linter is only a gate until you don't like what it says: Detekt once correctly flagged a twenty-two method interface, and I typed `@Suppress` and [went to bed](/2026/05/05/house-rules-i-suppressed-the-linter/).

So the design question isn't "is there a check." It's "can I get around it at 11pm."

Mostly the answer has to be no. What I want at that hour is a doorman who doesn't know me and doesn't care that I own the place.

That's why pre-push re-runs the unit tests pre-commit already ran: I'm compiling anyway, so it's nearly free, and it closes the `--no-verify` hole. It's why CI re-runs everything server-side, check-only, never auto-fixing, never pushing to the branch. A gate that quietly fixes your problem isn't a gate, it's an accomplice.

And it's why the escape hatches are deliberate rather than absent. `--no-verify` still works. I just have to type it, which means I have to decide to.

The AI half is the same lesson [from a different angle](/2026/07/12/nightcap-your-style-is-my-style/). When Fable spent an afternoon arguing with one of my build scripts about whose style won, it eventually knocked it off. It knocked it off because the script didn't care. You cannot negotiate with a run script. That isn't a limitation of the tooling. That is the entire product.

### Turning it up to eleven

Which brings me to today, and why today and not April.

I turned the strictness up. Pint now enforces `strict_comparison`, so `==` becomes `===` across the PHP. ESLint got `no-console`, `eqeqeq`, and the whole unused-vars family promoted from warn to error, because a gate that only warns is a suggestion with good posture. Larastan went from level 5 to 6: thirty-five files of generic annotations, uncommitted in my tree as I write this.[^larastan]

None of those are clever. They're the boring defaults you're supposed to have. I just couldn't have afforded them a month ago.

Ratcheting a linter with no safety net is how you spend a weekend chasing a style violation into a bug. Ratcheting it when every rung is live is a twenty-minute job, because the moment something breaks, six things tell you.

A strict null guard only helps if it has somewhere to report to. It does, because [the error handling was rebuilt first](/2026/07/11/carry-that-weight/).

Foundation, then interest.

The ladder pays the same way. Building the rungs was the expensive part. Now a feature arrives and the test is a conversation: what happens if this user was deleted, unit test or E2E? Last week that became a test proving a tombstoned member's refund still reaches their card, against real Stripe, because the rung was already there.

Same with the last piece of the ladder, still unbuilt. iOS will never run on GitHub Actions, because macOS runners burn ten times the minutes and my whole allotment is five dollars a month. So its heavy UI suite will run locally on a schedule, and the trigger will be a button.

In the admin page.

The one from the top of the post.

The one I built before any of this existed, which keeps growing new buttons for problems I hadn't had yet.

That's what a dividend actually looks like. Not the thing paying off once. The thing being *already there* the next time you need somewhere to put something.

### Last pour

None of this lets me stop reading the code. That's the part people want it to mean and it doesn't.

I've got agents building right now, while I dictate this. Every one is faster than me and I'm the only one who knows what this app is supposed to be. I'm the bottleneck, and the ladder does not remove the bottleneck. It aims it.

By the time I read a diff, the gates have run and a [reviewer built for that platform](/2026/07/07/blade-runners/) has read it against the house rules I wrote down. So the things left for me are the things only I can see.

The machine will happily compound in either direction. It has no preference. It just runs whatever rate you give it against whatever sign you set.

Set the sign. Then let it rip.

See you, space cowboy.

[^periphery]: Report-only on purpose. Dead code isn't a correctness bug, and half of what it finds is a helper I intend to use again. That's a cleanup list, not a reason to block a push.

[^larastan]: There's a version of me that waits to publish until it's committed. That guy has a lot of unpublished posts.
