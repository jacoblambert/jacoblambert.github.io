---
permalink: /
title: "About me"
excerpt: "Welcome to Jacob's switchboard"
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---

Welcome to my page!

Title
======
Text!

Recent Publications
------
{% include base_path %}
{% for post in site.publications limit:2 %}
  {% include archive-single.html %}
{% endfor %}

Recent Posts
------
{% include base_path %}
{% for post in site.posts limit:2 %}
  {% include archive-single.html %}
{% endfor %}