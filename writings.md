---
layout: page
title: Writings
order: 2
---

{% assign groups = site.publications | group_by: "category" | sort: "date" %}
{% for group in groups %}

<h2> {{ group.name }}</h2>
{% for item in group.items %}
<dl>
<dt><a href="{{ item.url }}">{{ item.title }}</a> - <em>{{ item.date | date_to_string }}</em></dt>
<dd>{{ item.description }}</dd>
</dl>
{%endfor%}
{%endfor%}
