---
title: "Posts"
permalink: /posts/
layout: archive
search: false
---

{% for post in site.posts %}
  {% include archive-single.html %}
{% endfor %}