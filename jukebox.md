---
layout: page
title: "The Jukebox"
permalink: /jukebox/
---

Every post, newest first. Punch a code.

{% assign posts_by_month = site.posts | group_by_exp: "post", "post.date | date: '%B %Y'" %}
{% for month in posts_by_month %}
## {{ month.name }}

{% for post in month.items %}
- **[{{ post.title }}]({{ post.url | relative_url }})** — {{ post.date | date: "%b %-d" }}
{% endfor %}
{% endfor %}
