---
layout: page
title: "A Confident Dose of Bad Advice"
permalink: /series/bad-advice/
---

Opinionated takes from the end of the bar.

Every post in this series is a confident take on something I think I'm right about and am willing to be wrong about in public. If you were told once upon a time not to take advice from strangers, [consider this your friendly reminder](/2026/04/14/the-guy-whose-name-i-dont-remember/).

## Posts

{% assign bad_advice = site.posts | where: "series", "bad-advice" %}
{% for post in bad_advice %}
- **[{{ post.title | remove: "A Confident Dose of Bad Advice: " }}]({{ post.url | relative_url }})** — {{ post.date | date: "%B %-d, %Y" }}
{% endfor %}
