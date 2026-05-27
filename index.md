---
layout: default
title: Home
---

# Rushikesh Deshpande

Deep-dive technical articles about **storage systems**, **distributed systems**, and **backend engineering** — filesystems, object stores, distributed architectures, and the engineering decisions behind them.

Currently tracking SeaweedFS internals, content-addressable storage, and POSIX filesystem design.

---

## Posts

{% for post in site.posts %}
- [**{{ post.title }}**]({{ post.url | relative_url }})  
  {{ post.date | date: "%B %-d, %Y" }} · {% for tag in post.tags %}`{{ tag }}` {% endfor %}
{% endfor %}

---

### Upcoming Topics

- SeaweedFS Filer Architecture
- Chunk Storage Internals
- Volume Server Design
- Distributed Locking in SeaweedFS FUSE
- Metadata Backends Explained
- Replication and Placement Strategies