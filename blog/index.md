---
layout: default
title: Blog
---

# Blog

{% for post in site.posts %}
- [**{{ post.title }}**]({{ post.url | relative_url }})  
  {{ post.date | date: "%B %-d, %Y" }} · {% for tag in post.tags %}`{{ tag }}` {% endfor %}
{% endfor %}