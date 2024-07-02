---
layout: page
title:  "Blog"
description: "A small but slowly growing collection of articles and thoughts about Software Engineering"
---

A small but slowly growing collection of articles and thoughts about Software Engineering:

<ul class="posts">
	{% for post in site.posts %}
	  	<li><span class="post-published-date">{{ post.date | date: "%Y / %m" }}</span> <a href="{{ post.url }}">{{ post.title }}</a></li>
	{% endfor %}
</ul>