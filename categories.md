---
title: Categories
permalink: /categories/
layout: list
description: ""
---

{% comment %}カテゴリを最新投稿日順にソート{% endcomment %}
{% assign cat_entries = "" %}
{% for cat in site.categories %}
  {% assign latest_post = cat[1] | sort: "date" | last %}
  {% assign cat_date = latest_post.date | date: "%Y%m%d" %}
  {% assign cat_entries = cat_entries | append: cat_date | append: ":" | append: cat[0] | append: "," %}
{% endfor %}
{% assign sorted_entries = cat_entries | split: "," | sort | reverse %}

<div class="tag-index">
  <p class="tag-cloud">
    {% for entry in sorted_entries %}
      {% if entry == "" %}{% continue %}{% endif %}
      {% assign cat_name = entry | split: ":" | slice: 1, 99 | join: ":" %}
      <a href="#{{ cat_name }}" class="tag-link">{{ cat_name }} ({{ site.categories[cat_name].size }})</a>
    {% endfor %}
  </p>
</div>

{% for entry in sorted_entries %}
  {% if entry == "" %}{% continue %}{% endif %}
  {% assign cat_name = entry | split: ":" | slice: 1, 99 | join: ":" %}
  {% assign cat_posts = site.categories[cat_name] | sort: "date" | reverse %}
<h2 id="{{ cat_name }}">{{ cat_name }} <span class="tag-count">({{ cat_posts.size }})</span></h2>
<ul class="tag-post-list">
  {% for post in cat_posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <span class="dates">{{ post.date | date: "%b %d, %Y" }}</span>
    </li>
  {% endfor %}
</ul>
{% endfor %}
