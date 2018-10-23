---
permalink: /
title:
excerpt: "Welcome to Jacob's switchboard"
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---

<img class="alignnone size-full" src="http://jacoblambert.github.io/images/tsukuba/highashell.jpg" alt="me, high as hell"/>

I'm a PhD candidate in Nagoya University currently focused on AI-based object detection and tracking for LiDAR sensors on autonomous cars. I'm also currently a C++/Python/ROS developer for [Tier IV](tier4.jp), improving the sensing capabilities of their open-source autonomous driving platform, [Autoware](https://github.com/CPFL/Autoware).

<h5>Recent Publications</h5>
{% include base_path %}
{% for post in site.publications limit:2 %}
  {{ post.citation }}
{% endfor %}

<h5>Recent Posts</h5>
{% include base_path %}
{% for post in site.posts limit:2 %}
  {% include archive-single.html %}
{% endfor %}

<h5>Recent Projects</h5>
{% for post in site.publications limit:2 %}
  {% include archive-single.html %}
{% endfor %}