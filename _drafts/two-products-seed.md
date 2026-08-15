---
title: "The Two Products (working title)"
tags: [architecture, product, tech-debt, career, opinion]
status: seed
---

> **PRE-DRAFT SEED — not ready to publish.** Captured 2026-08-14 from Matt thinking out loud, sparked ~30-40 min into the *Escaping the Build Trap* audiobook (Melissa Perri). These are raw ideas + a rough outline sketch. Matt will reread, rant more, then do a real narration before drafting. Voice/conceit decisions happen at narration time, not here.

> **ANONYMIZATION FLAG (hard rule):** several anchor anecdotes below come from the day shift. They go into the post as universal patterns only — no employer, team, product, or review-system names. Flagged inline as **[DAY SHIFT — pattern only]**. The pizza job and public-figure career examples are fair game.

---

## The thesis

**Every software company ships two products.** The **customer product** is the thing customers buy. The **code product** is the codebase itself — and its customer is the business. API design, seams, abstraction layers: that's product design for the second customer. (Matt's ordering, use it in the post: the code is arguably the *first* product — it exists before the customer product, and everything the customer ever gets ships through it.) Nobody disputes the customer product matters. The essay's claim: the code product is chronically unloved, and the neglect is *structural*, not a failure of intelligence.

Supporting beams:

- **Customers don't buy code.** Full concession, no strawman. The paying customer literally cannot see the architecture and shouldn't have to.
- **But customers feel bad code 2-3 steps removed.** Bugs (no tests — because the code was never written testable). Molasses delivery ("why is the second one taking months? I thought we already built one"). Slow product (the same API checked at three layers instead of passing the result through).
- **The code product's value is counterfactual, so it loses every prioritization fight.** You can't measure the bugs you don't have. Nobody leaves a Yelp review saying "I didn't get food poisoning again." A product whose benefits are invisible and whose costs are lagged and unattributable loses to a product with a dashboard, every time, regardless of who's smart.
- **The incentive systems encode the imbalance.** Impact-style performance reviews count shipped features; tests and refactors live in a separate, lesser section of the review. Senior engineers get rewarded for visibly hard technical problems, never for the run-of-the-mill 100-PR slog that made every variant of a thing actually work. **[DAY SHIFT — pattern only]**
- **"Engineering is treated as free."** Leadership pushes product outcomes with hard deadlines and prices the engineering quality work at zero. The tell: "spend 10% of your time writing tests" — okay, does the deadline move 10%? No. **[DAY SHIFT — pattern only]**
- **The balance claim, stated honestly:** the technical purists aren't 100% right, and neither are the outcome pushers. The problem is that at the higher levels the balance doesn't exist at all — it's all product one.
- **Standards have to be built in, not aspired to.** This is where the essay turns from complaint to argument. See the Godfather's Pizza story below: real quality standards are a *planned cost*, budgeted up front, not virtue squeezed from spare time.
- **What good product management would actually mean:** the PM question shouldn't stop at "when does the new feature ship?" It should include "what's wrong with our *first* product — the code — that we can't get the second one out?"
- **Product-focused ≠ outcome-focused (added 2026-08-14):** every manager Matt has had was product-focused — but *feature*-focused, which is output-focused. Feature-focus isn't a mild form of outcome-focus; it IS the build trap (Perri's own definition: measure success by shipping, never check what changed). John Cutler's "12 Signs You're Working in a Feature Factory" is the citable companion. So "my companies are product-obsessed AND in the build trap" is not a tension — it's the canonical case.

## The resolution beat — quality stated as an outcome (added 2026-08-14)

This is where the essay's apparent argument WITH the book dissolves, in both sides' favor:

- **"Bug-free code" is an outcome, not a feature.** The chain: write quality, testable code → write tests → fewer production defects → customers keep trusting the product → they stay. Push the chain one link past the code to customer behavior (fewer broken flows, fewer support tickets, less "it's buggy" churn) and it's a full-blooded outcome with measurable proxies: defect rate, change failure rate, time to restore.
- **"Build this feature with no bugs" is an output command wearing outcome clothes.** It names no mechanism and budgets nothing — which is why it always resolves to "ship the feature, skip the tests."
- **The outcome vocabulary is the strongest weapon the code product ever had.** Stop begging for refactor time on virtue grounds; name the outcome and make it compete on the same footing as every feature. The code product doesn't need charity — it needs its outcomes written on the same menu.
- This may be the essay's TRUE landing, stronger than (or paired with) "the balance isn't a vibe, it's a budget" — decide at narration.

---

## Anchor anecdotes inventory

1. **Godfather's Pizza, the 30-minute rule** (nameable — decades-old teen job, past employers in the abstract are fair game). $5 buffet. A pizza can only sit out 30 minutes. Some mornings you put out a full spread at opening, nobody shows for half an hour, and you throw away eight or nine whole pizzas. It sucks. But the standard was *built in* — which meant the morning prep list said make MORE pizzas, because we knew we'd throw some away. Quality standards cost real money and the operation planned for the cost. Nobody asked the buffet to hit the same pizza count while also throwing pizzas away.
2. **The product that needed to be a platform — the "not wrong ≠ right" story.** **[DAY SHIFT — pattern only]** A mature single product (~a year to build). Then someone wanted a second one: "we literally can't do that." The platform got rushed out under a live product — bugs were *inevitable*, not sloppiness, because everything was changing at once. It made money and it's still going, which makes the rush **not wrong**. Whether it was **right** is a different claim, and the honest answer is unknowable — because the question that would decide it never got asked. **The anatomy (re-refined 2026-08-14 — do NOT frame as "the rush was right"):**
   - The original engineers were right to build a product, not a platform — a speculative platform for a second product nobody had asked for yet would have been the ENGINEER'S build trap (outputs serving an imagined outcome).
   - The real decision was never "rush vs. another year." At the margin it was "rush vs. one or two more months to avoid locking in the fragile shape." Survival stakes → rush, no-brainer. A year's delay → rush, no-brainer. In between sits the question "what would six more weeks buy us?" — and it never gets asked, because the people deciding can see only the customer product. The unpriced trade isn't carelessness; it's the vision asymmetry — the SAME asymmetry the whole essay is about. And that's where the molasses starts: fragility locked in at the moment of rush, every later change costing more.
   - **Price the trade, schedule the repayment — and the hard truth: the repayment never gets scheduled**, for the same reason the trade never got priced. Bugs announced up front are interest payments; the same bugs arriving as a surprise get blamed on engineers, and the blame is the actual wrong.
   - The standing outcomes being traded against money-now, stated in outcome language: "do we have a product that CAN be bug-free? can it flex in ways we don't yet know we need?"
   - (Decision-study tie-in: don't "result" — the bug count doesn't judge the decision; the unasked question does.)
   **Role in the essay: the honesty valve, calibrated.** It refuses BOTH easy verdicts — not "the suits were wrong," not "the rush was right," but "it wasn't wrong, and nobody can tell you it was right, because nobody asked the question that would tell you." Beat 5.
3. **The six identical flags.** **[DAY SHIFT — pattern only]** Six flags that do essentially the same thing, because six people under outcome pressure each patched the same problem their own way. Three months later nobody knows why any of them exist.
4. **"Why is the second variant taking so long?"** **[DAY SHIFT — pattern only, scrub the product entirely]** A second instance of an existing thing takes a 100-PR, month-long rewrite because the first was built as a one-off. Nobody rewards that slog; nobody connects it to the architecture decision that caused it.
5. **The high-level test-coverage complaint.** **[DAY SHIFT — pattern only]** Directors asking "why is our coverage so bad?" while being the same layer that sets deadlines pricing tests at zero.
6. **The two-deadlines week.** **[DAY SHIFT — pattern only]** Two features, both due the same week. Half a week each, 12-14 hour days. The only way both ship is minimum-viable-ship, which means no tests. "Don't tell me tests are important" — the timeline IS the priority statement; everything else is decoration. The sharpest version of the priced-at-zero beat.
   - **Modern coda (upgraded 2026-08-14 — this is now PROOF of the thesis, not a caveat):** that crunch was pre-Claude. AI dropped the price of tests and quality work — and the budget for them STILL didn't appear, because post-Claude deadlines just moved up ("you can get more done now"). Capacity gains absorbed, quality line-item still zero. The zero-pricing was never about cost. Second beat: Claude gets more done, but you still have to coerce it into good code — which means you still have to KNOW what good code is. The typing got cheaper; the judgment didn't.
   - **Anecdote placement (confirmed):** Godfather's = cold open (beat 1). Front/back of house = the frame (beat 6, lightly woven throughout; pizza floor opens, kitchen generalizes — bookends, one restaurant-world conceit family). Crunch/MVP/no-tests = beat 4, the evidence exhibit. All three stay in THIS post; none bleed into the trajectory post.

---

## Conceit candidate: front of house / back of house

The bar/restaurant frame is almost pre-built for this one. Customers buy the room, the pour, the bartender's cocktail — that's what gets the Yelp reviews. Nobody orders the keg lines or the cellar temp, but every slow drink and every off pint traces back there. A clean kitchen's value is everything that *doesn't* happen: you can't count prevented food poisonings.

**Where the analogy breaks — and the break IS the argument:** restaurants don't treat the kitchen as optional. Health standards are built in, budgeted, enforced (the 30-minute pizza rule). Software is the industry that runs a restaurant where only the dining room appears on the books. The Godfather's story repairs the analogy exactly where it breaks, which might be the essay's pivot.

(Per house rules this is one candidate conceit, not the decision. Fresh-conceit call happens at narration.)

---

## Claude-contributed framings Matt reacted well to (keep/cut freely)

- **Architecture's job-to-be-done:** the business hires the architecture to *make the next change cheap*. That's the whole job. Kent Beck: "for each desired change, first make the change easy (warning: this may be hard), then make the easy change." Architecture isn't virtue; it's an option on future speed.
- **Under-thought product and under-valued foundations are the same absence** — nobody upstream did the thinking, so engineers guess, and the guesses calcify (the six flags ARE a product-management failure, viewed from below).
- **Short-termism wearing outcome-focus as a costume** — "it's making money" vs "it will never get fixed."
- **"Engineering management is a product job wearing an engineering costume, and nobody tells you until you've already chosen."**
- **DORA metrics as product two's attempted dashboard** (lead time for change, change failure rate, etc.) — the industry's one serious effort to make the second product measurable. Optional; may be too corporate for the room.

---

## The career thread — SPLIT OUT (decided 2026-08-14)

The engineer→product trajectory is its own post: see [unspoken-trajectory-seed.md](unspoken-trajectory-seed.md). **The split:** this post is about the software (the casualty is the codebase); that post is about the people (the casualty is the architecture-minded engineer's career). Shared hinge — "nobody buys your code" — gets ONE sentence here and one there; this post owns the deep version.

**Publish order: this one first**, so the trajectory post can lean on the two-products thesis in a line instead of re-arguing it. Keep the career material out of this essay entirely except possibly one closing breath.

---

## Tone note

**Not cocky.** This one is written mid-learning, not from a mountaintop: "I'm reading the product books, I agree with more of it than I expected, and here's the half of the picture I think they're missing." The stance is a guy arguing with an audiobook on a run, not a veteran issuing verdicts. That's a different opening energy than the declarative industry posts.

---

## Rough outline sketch

1. **Cold open:** throwing away eight pizzas at 10:30am (Godfather's, the 30-minute rule). Standards as a planned cost.
2. **The confession:** I'm listening to the product management books, and they're right about the thing engineers hate hearing — nobody buys your code.
3. **The two products:** the customer product, and the code product whose customer is the business. Architecture as product design for the second customer.
4. **Why the code product always loses:** counterfactual value (the bugs you don't have), lagged and unattributable costs, incentive systems that price engineering as free. The 10%-tests-no-deadline-move tell.
5. **The downstream bill:** the patterns — rushed platform, six flags, the 100-PR second variant. Customers feel all of it, 2-3 steps removed, and can name none of it.
6. **Back of house:** the conceit section. Where the restaurant analogy breaks is exactly the indictment: restaurants build the kitchen into the books; software doesn't.
7. **Landing (two candidates, decide at narration):** (a) the balance isn't a vibe, it's a budget; (b) the resolution beat — quality is an outcome you're allowed to name; the code product's outcomes belong on the same menu as every feature. Likely (b) as the argument's peak with (a) as the practical close. Optional final breath: the quiet career note (one line, pointing at the trajectory post).

---

## Possible titles

- *"Back of House"*
- *"The Two Products"* (current working title — declarative, maybe too dry)
- *"The Clean Kitchen"*
- *"Nobody Reviews the Kitchen"*
- *"Thirty-Minute Pizzas"*
- *"You Can't Measure the Bugs You Don't Have"* (thesis-as-title, long)

---

## Cut material (decided 2026-08-14 — don't re-litigate)

- **The redesign-treadmill story** (serial redesigns escalating into an org-wide rewrite, then another redesign) — CUT, Matt's own call. Doubly disqualified: (1) it's a *plot*, not a pattern — the persuasive power is the specific escalating timeline, and the timeline is the unremovable day-shift fingerprint; (2) the evidence section is already full (five exhibits) and the feature-factory beam covers the ground. **Salvage allowed:** at most one generalized line, e.g. "I've watched teams redesign the same surface three times in eighteen months without anyone asking what the previous redesign changed."
- **Possible separate future essay:** promotion-driven development — redesigns timed to review calendars (December/April/H2) are calendar-driven, not outcome- or customer-driven. Only buildable from industry-public examples; Matt's version is day-shift-locked. Park it.

## Open questions

- Overlap check before drafting: does `abstract-at-the-edges` (draft) or any published post already cover the architecture-advocacy ground? Differentiate or merge.
- Matt referenced an earlier essay of his about the pushback technical people get for wanting to do things right — find which post that is and link it.
- How much to cite *Escaping the Build Trap* directly? The essay partially agrees with it; naming it gives the post a hook ("I'm arguing with the book everyone told me to read").
- Keep the DORA beat or cut as too corporate for the bar?
- Does the career thread stay as a closing beat, or split into its own post?
