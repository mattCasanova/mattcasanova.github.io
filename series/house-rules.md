---
layout: page
title: "House Rules"
permalink: /series/house-rules/
---

The principles I actually build on, explained with the code I actually ship.

Not contrarian takes — just the stuff I've been teaching, building on, and arguing about for long enough that it stopped being opinion and started being load-bearing. Starting with SOLID, because most people know the S and fake the rest.

## Posts

{% assign house_rules = site.posts | where: "series", "house-rules" %}
{% for post in house_rules %}
- **[{{ post.title | remove: "House Rules: " }}]({{ post.url | relative_url }})** — {{ post.date | date: "%B %-d, %Y" }}
{% endfor %}
