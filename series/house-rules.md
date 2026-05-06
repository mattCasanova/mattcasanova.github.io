---
layout: page
title: "House Rules"
permalink: /series/house-rules/
---

The principles I actually build on, explained with the code I actually ship.

Not contrarian takes — just the stuff I've been teaching, building on, and arguing about for long enough that it stopped being opinion and started being load-bearing. Starting with SOLID, because most people know the S and fake the rest.

## The SOLID Arc

- **S** — [I Know the Other Four Letters](/2026/04/15/house-rules-the-other-four-letters/)
- **O** — [Whatever Happens Happens](/2026/04/17/house-rules-whatever-happens-happens/)
- **L** — [It Compiled. It Lied.](/2026/04/22/house-rules-it-compiled-it-lied/)
- **I** — [I Suppressed the Linter](/2026/05/05/house-rules-i-suppressed-the-linter/)
- **D** — [I Already Wrote This One](/2026/05/06/house-rules-d/)

## All Posts

{% assign house_rules = site.posts | where: "series", "house-rules" %}
{% for post in house_rules %}
- **[{{ post.title | remove: "House Rules: " }}]({{ post.url | relative_url }})** — {{ post.date | date: "%B %-d, %Y" }}
{% endfor %}
