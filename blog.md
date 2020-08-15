---
layout: page
title: Blog
order: 4
---

{% for post in site.posts %}
{% capture current_year %}{{ post.date | date: "%Y" }}{% endcapture %}
{% if current_year != previous_year %}
{% unless forloop.first %}
</ul>
{% endunless %}
<h2>{{ current_year }}</h2>
<ul>
{% assign previous_year = current_year %}
{% endif %}
<li><a href="{{ post.url }}">{{ post.title }}</a> - <em>{{ post.date | date_to_string }}</em></li>

{% if forloop.last %}
</ul>
{% endif %}
{% endfor %}
