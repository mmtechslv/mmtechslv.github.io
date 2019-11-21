---
layout: page
title: Tags
order: 4
---

{% for document in site.documents %}
{% for tag in document.tags %}
  {% assign ttags = tag | append:'|' %}
  {% assign rawtags = rawtags | append:ttags %}
{% endfor %}
{% endfor %}
{% assign rawtags = rawtags | split:'|' | sort %}

{% assign tags = "" %}
{% for tag in rawtags %}
  {% if tag != "" %}
    {% if tags == "" %}
      {% assign tags = tag | split:'|' %}
    {% endif %}
    {% unless tags contains tag %}
      {% assign tags = tags | join:'|' | append:'|' | append:tag | split:'|' %}
    {% endunless %}
  {% endif %}
{% endfor %}

<ul>
{% for tag in tags %}
    <li><a href="/tag/{{ tag }}"><code class="highligher-rouge"><nobr>{{ tag }}</nobr></code></a></li>
{% endfor %}
</ul>
