---
layout: page
title: about
permalink: /about/
---

I'm a developer living in Dallas, TX.  I have a long time background in systems, and am currently focused on cloud automation and distributed systems. I like solving problems.

This site is a space for me to share my enthusiasm for computers, personal ideas, and occasionally some hand waving philosophy of sitting behind a drumset.

<h3>A few non-technical posts</h3>
<ul>
  {% for post in site.categories.personal %}
    <li>
      <span class="post-date">{{ post.date | date: "%b %-d, %Y" }}</span>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
