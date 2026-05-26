---
layout: default
title: Home
---

# Rushikesh Deshpande

Notes on storage, distributed systems, and backend engineering.

## Posts

{% for post in site.posts %}
- [{{ post.title }}]({{ post.url | relative_url }})  
  {{ post.date | date: "%B %-d, %Y" }}
{% endfor %}
