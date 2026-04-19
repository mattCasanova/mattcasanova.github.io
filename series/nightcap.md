---
layout: page
title: "Nightcap"
permalink: /series/nightcap/
---

Short posts from the end of the week. One drink, one ledger, one sign-off.

The Nightcap is whatever's on my mind at 11pm on a Friday — what shipped, what's still sitting on the stool next to me, what's waiting for Monday. Less argument, more bar talk.

## Posts

{% assign nightcap = site.posts | where: "series", "nightcap" %}
{% for post in nightcap %}
- **[{{ post.title | remove: "Nightcap: " }}]({{ post.url | relative_url }})** — {{ post.date | date: "%B %-d, %Y" }}
{% endfor %}
